# Niveshaay - Automated Financial Results Pipeline (n8n)

An n8n workflow that takes a company name and a BSE/NSE result PDF link from an n8n form, extracts the consolidated P&L with Gemini using the assignment prompt, checks the numbers, builds a styled HTML table, renders it to an image with hcti.io, and sends it to WhatsApp. Everything runs inside n8n (Form Trigger, Code, HTTP Request and IF/Switch nodes). No external frontend or backend.

## Flow
Form -> Config -> Validate URL -> Download PDF -> PDF to Base64 -> Gemini Analyze PDF -> Check response -> Parse AI JSON -> Route (ok / no pnl / invalid json / gemini error) -> Number Check -> Build HTML Table -> HTML to Image (hcti.io) -> Split Recipients -> Send WhatsApp -> Aggregate Sends -> Check -> Success or WhatsApp Failed.

## Design choices
- Gemini is called with the PDF as inline data and temperature 0, so the JSON stays consistent.
- A Switch node routes four outcomes (ok, no_pnl, invalid_json, gemini_error) to separate form endings.
- "Number Check" recalculates EBITDA margin, PAT margin and profit before tax from the AI numbers and adds a warning if they differ by more than 0.05. It never changes the AI JSON.
- WhatsApp delivery is one setting, `delivery_provider` in the Config node: `meta` (Meta WhatsApp Cloud API), `evolution` (Evolution API, as in the brief) or `twilio`. The request is built in one Code node, so switching provider needs no node changes.
- Each recipient gets its own message, because most providers cannot send to groups.
- The Gemini key lives in an n8n credential. The other keys are plain fields in the Config node so they can be seen when testing. In this repo they are all `PASTE_...` placeholders.

## How to import and run
1. In n8n: Create workflow > three dots > Import from file > select the workflow JSON.
2. On the node "Gemini Analyze PDF", create a Header Auth credential (name `x-goog-api-key`, value = your Gemini key) and select it.
3. Open the "Config" node and replace the `PASTE_` values. Set `delivery_provider` and the matching provider fields, and `hcti_api_id` / `hcti_api_key` from hcti.io. Put the recipient as digits with country code, e.g. 919876543210.
4. Click Execute workflow, open the form Test URL, enter a company name and a BSE/NSE PDF link (use the red PDF icon link on the BSE page, not the XBRL link).

## What I tested (real BSE PDFs)
- Pranav Constructions Ltd and Manipal Payment and Identity Solutions Ltd ran through download, Gemini extraction, JSON parsing, the number check and the HTML table. See `samples/`.
- Error handling that I saw working: a PDF with no P&L went to the "No P&L Found" ending; Gemini errors went to "AI Processing Failed"; a rejected send went to "WhatsApp Failed" and showed the provider's error text.
- Both samples are Q1 FY27, which is the 4-column (standard) format. The 6-column Q2/Q4 format is built but I did not test it.
- Invalid-link and download-failed endings exist but I did not trigger them.

## What did not work (honest status)
- **WhatsApp delivery was never confirmed.** The Twilio trial rejected custom messages (error 21654 asked for a template; my template returned 21655). The Meta test number accepted my requests, but Meta's webhook log showed `status: failed`, error 131031 "Business Account locked"; a later send returned error 190. So I have no WhatsApp screenshots. Evolution API support is built but untested, because I had no Evolution instance.
- **The hcti.io image was not confirmed.** It returned 401 until real keys were entered, and Gemini rate limits (503, then 429) stopped my later runs before I could re-test it. When the image is missing, the workflow sends the P&L as text instead.
- I have 2 real samples, not 3, for the same reason.

## Security
All keys were removed from the exported workflow. Config values are placeholders. Keys that appeared in my screenshots were rotated after submission.

## Possible improvements
Retry with backoff on Gemini 429/503, a Wait node between steps, caching by PDF URL, group delivery through a provider that supports groups, and testing the Q2/Q4 format.
