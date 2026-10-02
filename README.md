# Niveshaay - Automated Financial Results Pipeline (n8n)

**Author:** Mansi Sonani  |  **Task issued by:** Arjun, Niveshaay (28 Sep 2026)  |  **Platform:** n8n 2.32.5 (self-hosted)

---

## 1. Introduction

Niveshaay receives corporate financial result PDFs from BSE/NSE. Someone then has to read each PDF, pull out the consolidated P&L, calculate margins, format it into a table and post it on WhatsApp.

This project automates that job **entirely inside n8n**. A person opens an n8n form, types a company name and pastes a BSE/NSE result PDF link. The workflow downloads the PDF, sends it to Gemini with the assignment prompt, receives a standardized P&L JSON, checks the numbers, builds a styled table, turns it into an image with hcti.io and sends it to WhatsApp.

No external frontend, no separate backend, no custom web page. The form, the processing, the AI call and the delivery all happen in n8n nodes.

> **Honest status in one line:** everything up to and including the P&L table was tested on real BSE PDFs. **Live WhatsApp delivery and the hcti.io image were not confirmed** (reasons in section 8).

---

## 2. What the user sees (example)

**Input (n8n form)**

| Field | Example |
|---|---|
| Company Name | Pranav Constructions |
| Result PDF URL | `https://www.bseindia.com/xml-data/corpfiling/AttachLive/4c6e9bba-b8a5-4031-81bf-ddaad6a30b4a.pdf` |

**Output from Gemini (short excerpt, standard 4-column format for Q1/Q3)**

```json
{
  "company_name": "Pranav Constructions Limited",
  "quarter_type": "standard",
  "row1": ["Particulars", "Q1 FY27", "Q4 FY26", "Q1 FY26"],
  "row2": ["Revenue", "164.54", "272.84", "142.16"]
}
```

The full outputs from real runs are in the `samples/` folder.

**Final screen on the form** shows either a success message or a clear error (invalid link, download failed, AI failed, no P&L found, invalid data, WhatsApp failed).

---

## 3. Architecture

```
 User
  |  Company Name + PDF URL
  v
[Form Trigger] -> [Config] -> [Validate URL] --false--> [Invalid Link]
                                   | true
                                   v
                            [Download PDF] -> [Check Download] --false--> [Download Failed]
                                                    | true
                                                    v
                                            [PDF to Base64]  (also builds the Gemini request with the prompt)
                                                    v
                                         [Gemini Analyze PDF] -> [Check Gemini Response] --false--> [AI Processing Failed]
                                                                          | true
                                                                          v
                                                                   [Parse AI JSON]
                                                                          v
                                                                [Route Parsed Result]
                                          ok |        no_pnl |       invalid_json |     gemini_error |
                                             v               v                    v                  v
                                      [Number Check]   [No P&L Found]       [Invalid Data]    [AI Processing Failed]
                                             v
                                     [Build HTML Table]
                                             v
                                   [HTML to Image (hcti.io)]
                                             v
                                     [Split Recipients]   <- builds one request per recipient for the chosen provider
                                             v
                                       [Send WhatsApp]
                                             v
                                      [Aggregate Sends] -> [Check WhatsApp] --true--> [Success]
                                                                    | false
                                                                    v
                                                             [WhatsApp Failed]
```

---

## 4. Node-by-node guide (24 nodes + 1 sticky note)

| # | Node | Type | What it does | Why it is there |
|---|---|---|---|---|
| 1 | **Financial Results Processor** | Form Trigger | Shows the form with two fields: Company Name and Result PDF URL. Waits for the final form ending. | Assignment requires an n8n form, not a custom page. |
| 2 | **Config** | Set | Holds all settings in one place: provider choice (`delivery_provider`), recipients, hcti keys, Meta/Evolution/Twilio fields. | One node to edit instead of changing many nodes. |
| 3 | **Validate URL** | IF | Passes only links that start with `http` and end with `.pdf`. | Stops bad links early. |
| 4 | **Download PDF** | HTTP Request | Downloads the PDF as a file. Continues on failure. | Gemini needs the file bytes. |
| 5 | **Check Download** | IF | Checks that a file was really received. | Turns a failed download into a clear message. |
| 6 | **PDF to Base64** | Code | Converts the PDF to base64 and builds the Gemini request body, including the prompt from Section 3 of the assignment. | Gemini accepts PDFs as base64 inline data. |
| 7 | **Gemini Analyze PDF** | HTTP Request | Sends the request to `gemini-3.5-flash`. API key comes from an n8n Header Auth credential. Continues on failure. | AI extraction of the consolidated P&L. |
| 8 | **Check Gemini Response** | IF | Checks that Gemini returned text and not an error (for example 429 or 503). | Clear message instead of a crash. |
| 9 | **Parse AI JSON** | Code | Removes markdown fences, parses the JSON and sets a status: `ok`, `no_pnl`, `invalid_json` or `gemini_error`. | The prompt asks for strict JSON, and the model sometimes adds fences. |
| 10 | **Route Parsed Result** | Switch | Sends each status to its own path. | Cleaner than nested IF nodes. |
| 11 | **Number Check** | Code | Recalculates EBITDA margin, PAT margin and Profit before Tax from the AI's own numbers. If a value differs by more than 0.05 it adds a warning. | Catches arithmetic slips by the AI without changing its JSON. |
| 12 | **Build HTML Table** | Code | Builds the styled P&L table (header bar, highlighted key rows, change columns). Also prepares a plain-text version of the table. | Image source, and fallback text. |
| 13 | **HTML to Image** | HTTP Request | Sends the HTML to hcti.io, which returns a public PNG link. Continues on failure. | Turns the table into an image. |
| 14 | **Split Recipients** | Code | Creates one request per recipient for the selected provider (`meta`, `evolution` or `twilio`). Uses the image when it exists, otherwise sends the P&L as text. | Providers cannot send to groups, and this keeps provider logic in one place. |
| 15 | **Send WhatsApp** | HTTP Request | Sends the prepared request. Continues on failure. | Generic sender. The URL, headers and body come from node 14. |
| 16 | **Aggregate Sends** | Code | Counts how many sends the provider accepted. | An IF node works per item, so results must be combined first. |
| 17 | **Check WhatsApp** | IF | True only if every send was accepted. | Chooses Success or WhatsApp Failed. |
| 18 | **Invalid Link** | Form Ending | Shows "Invalid link". | Error path 1. |
| 19 | **Download Failed** | Form Ending | Shows a download error. | Error path 2. |
| 20 | **AI Processing Failed** | Form Ending | Shows an AI error. | Error path 3. |
| 21 | **No P&L Found** | Form Ending | Shows that no P&L table exists in the PDF. | Matches the assignment rule for "no pnl found". |
| 22 | **Invalid Data** | Form Ending | Shows that the AI did not return valid JSON. | Error path 5. |
| 23 | **WhatsApp Failed** | Form Ending | Shows the provider's error text. | Error path 6. |
| 24 | **Success** | Form Ending | Shows "Done! P&L result sent to WhatsApp." | Happy path. |

*Known limitation of node 16/17:* "Success" means the provider **accepted** the request. It does not prove the message reached the phone. I saw this with Meta: the API accepted the message, but Meta's webhook log later reported `failed`.

---

## 5. Why the WhatsApp provider changed: Evolution -> Twilio -> Meta

| Step | Provider | What happened |
|---|---|---|
| 1 | **Evolution API** (original brief) | Niveshaay's current setup. I did not have an Evolution instance to test against, and Niveshaay approved Twilio's sandbox for my prototype instead. Evolution support is still in the workflow as a setting. |
| 2 | **Twilio WhatsApp sandbox/trial** | My login worked (after fixing the header). But every send was rejected: plain text gave error **21654** (ContentSid required), and my own template ID gave **21655** (invalid). Twilio's trial only allows fixed templates, so it cannot carry custom P&L numbers or an image. |
| 3 | **Meta WhatsApp Cloud API** (free test number) | Meta accepted my requests (message id returned). Meta's webhook log then showed `status: failed`, error **131031 "Business Account locked"**. A later request returned error **190** (authentication). |

**Design result:** instead of rewriting the workflow each time, the provider is now one setting (`delivery_provider` in Config: `meta`, `evolution` or `twilio`). Node 14 builds the right request for it. Niveshaay can switch to Evolution API without touching any other node.

---

## 6. How to import and run

1. In n8n: **Create workflow -> three dots -> Import from file** and pick `workflow/niveshaay_financial_results_workflow.json`.
2. Open **Gemini Analyze PDF**. Create a **Header Auth** credential (Name `x-goog-api-key`, Value = your Gemini key) and select it.
3. Open **Config**. Replace every `PASTE_...` value you need:
   - `delivery_provider`: `meta`, `evolution` or `twilio`
   - `recipient_numbers`: digits with country code, for example `919876543210` (no plus, no spaces)
   - `hcti_api_id` and `hcti_api_key`: from hcti.io
   - the fields of the provider you chose
4. Press **Ctrl+S**, then **Execute workflow**.
5. Open the first node, click the **Test URL**, enter a company name and a PDF link, and submit.

**Where to get the PDF link:** on `bseindia.com/corporates/ann`, set Category to *Result*, then right-click the **red PDF icon** and choose *Copy link address*. Do not use the blue XBRL button.

Notes: Gemini can answer with 503 (busy) or 429 (rate limit). Wait a few minutes and run again. The Gemini node is set to retry.

---

## 7. What I tested

| Item | Result |
|---|---|
| Form, Config, URL check, PDF download | Worked on real BSE PDFs |
| Gemini extraction | Worked for Pranav Constructions and Manipal Payment and Identity Solutions (both Q1 FY27, standard 4-column format) |
| Parse AI JSON, Number Check, Build HTML Table | Worked (see `samples/`) |
| `No P&L Found` path | Triggered by a real PDF (Rajasthan Tube) with no P&L table |
| `AI Processing Failed` path | Triggered by Gemini 503 and 429 responses |
| `WhatsApp Failed` path | Triggered by provider rejections; shows the provider's error text |
| Invalid Link / Download Failed / Invalid Data paths | Built, **not triggered** |
| Q2 / Q4 (6-column) format | Built, **not tested** |

---

## 8. Not working / not verified

- **WhatsApp delivery was never confirmed.** See section 5. No WhatsApp screenshots are included.
- **hcti.io image not confirmed.** It returned 401 until real keys were put in Config. Later Gemini rate limits (503, then 429) stopped my re-test. When no image is available, the workflow sends the P&L as text.
- **2 real samples instead of 3**, for the same Gemini limit reason.
- **Evolution API** support is implemented but untested.

---

## 9. Security

- All keys were removed from this exported workflow. Config contains `PASTE_...` placeholders only.
- The Gemini key is stored in an n8n credential. The other keys were kept as plain Config fields during testing so I could see what I pasted.
- Keys that appeared in my screenshots were rotated after submission.

---

## 10. Ideas for next steps

Retry with backoff for Gemini 429/503, check delivery status through the provider's webhook (so "Success" means delivered), cache results by PDF URL, group delivery through a provider that supports groups, and test the Q2/Q4 format.

---

## Repository contents

```
README.md                  this file
workflow/                  n8n workflow export (credentials removed)
samples/                   real JSON outputs from real BSE PDFs
screenshots/               n8n run screenshots 
```

