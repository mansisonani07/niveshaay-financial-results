# Screenshots

These are n8n execution screenshots from 2 October 2026. No keys or tokens are shown.

| File | What it shows |
|---|---|
| `01_run1149_manipal.png` | Real run for Manipal Payment and Identity Solutions (Q1 FY27). The workflow downloaded the PDF, Gemini extracted the P&L, the JSON was parsed, the number check ran and the HTML table was built. The image step (hcti.io) returned 401 because the keys in Config were still placeholders, so the workflow used the text version. The send step then failed and the run ended on the "WhatsApp Failed" page. |
| `02_run1148_no_pnl_found.png` | Error handling. A real BSE PDF for Rajasthan Tube Manufacturing Company Ltd contained no P&L table. The workflow took the "No P&L Found" path instead of crashing. |
| `03_run1146_pranav.png` | Real run for Pranav Constructions (Q1 FY27). All nodes ran and the "Success" page appeared, which means the WhatsApp provider (Meta) accepted the request. Meta's delivery log later showed `failed` with error 131031 (Business Account locked), so the message did not reach the phone. |
| `04_meta_log_131031.png` | Only if present: Meta's webhook log for the failed delivery (status failed, error 131031). Phone number hidden. |

Note: "Success" in this workflow means the provider accepted the request. It does not prove the message was delivered.
