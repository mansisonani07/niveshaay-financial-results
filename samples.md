# Samples

Real outputs produced by this workflow from real BSE result PDFs. They are the "Parse AI JSON" node output of each run.

| File | Company | Quarter format | Source run |
|---|---|---|---|
| `sample1_pranav.json` | Pranav Constructions Limited | Q1 FY27, standard (4 columns) | n8n execution #1146 (2 Oct 2026, 14:56) |
| `sample2_manipal.json` | Manipal Payment and Identity Solutions Ltd | Q1 FY27, standard (4 columns) | n8n execution #1149 (2 Oct 2026, 15:36) |

## Notes
- The numbers come from Gemini using the assignment prompt. The workflow's Number Check node recalculates margins from them and adds a warning if they differ by more than 0.05.
- No Q2 or Q4 (6-column) sample is included: Gemini rate limits (503, then 429) stopped my later runs.
- A third PDF (Rajasthan Tube Manufacturing Company Ltd) correctly returned **No P&L Found**. See `screenshots/error_handling_no_pnl.png`.
- There are no WhatsApp screenshots because delivery was blocked (Twilio trial template rule, then Meta error 131031). See the main README, sections 5 and 8.
