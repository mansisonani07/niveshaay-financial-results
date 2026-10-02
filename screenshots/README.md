# Screenshots

n8n execution screenshots and one redacted log from 2 October 2026. No keys or tokens are shown, and personal identifiers in the Meta log are redacted.

| File | What it shows |
|---|---|
| `01_run1146_pranav_pipeline.png` | Run #1146, Pranav Constructions (Q1 FY27), first half: PDF to Base64, Gemini Analyze PDF, Check Gemini Response, Parse AI JSON, Route Parsed Result, Number Check. All nodes on the "ok" path ran (green). |
| `02_run1146_pranav_delivery_success_page.png` | Run #1146, second half: Build HTML Table, HTML to Image, Split Recipients, Send WhatsApp, Aggregate Sends, Check WhatsApp, then the "Success" ending. Success means Meta accepted the request. |
| `03_run1148_no_pnl_found.png` | Error handling. A real BSE PDF (Rajasthan Tube Manufacturing Company Ltd) had no P&L table, so the workflow took the "No P&L Found" path instead of crashing. |
| `04_run1149_manipal_pipeline.png` | Run #1149, Manipal Payment and Identity Solutions (Q1 FY27), first half: extraction, parsing and number check on the "ok" path. |
| `05_run1149_manipal_whatsapp_failed.png` | Run #1149, second half. The image step (hcti.io) returned 401 because its keys in Config were still placeholders, so the P&L was prepared as text. The send step then failed (Meta error 190) and the run ended on the "WhatsApp Failed" page. |
| `06_meta_log_131031_redacted.json` | Meta's webhook log for run #1146's message: `status: failed`, error 131031 "Business Account locked". This is why the message never reached the phone. Identifiers are redacted. It is text copied from the log, not a screenshot. |

Note: "Success" in this workflow means the provider accepted the request. It does not prove the message was delivered. File 06 shows this case.
