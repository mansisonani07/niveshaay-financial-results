# Samples

Real outputs produced by this workflow from real BSE result PDFs. They are the "Parse AI JSON" node output of each run.

| File | Company | Quarter format | Source PDF | Source run |
|---|---|---|---|---|
| `sample1_pranav.json` | Pranav Constructions Limited | Q1 FY27, standard (4 columns) | https://www.bseindia.com/xml-data/corpfiling/AttachLive/4c6e9bba-b8a5-4031-81bf-ddaad6a30b4a.pdf | n8n execution #1146 (2 Oct 2026, 14:56) |
| `sample2_manipal.json` | Manipal Payment and Identity Solutions Ltd | Q1 FY27, standard (4 columns) | https://www.bseindia.com/xml-data/corpfiling/AttachLive/4fc4ab15-a4a8-4ef0-9a87-5583cc3b7600.pdf | n8n execution #1149 (2 Oct 2026, 15:36) |

## Error-handling test

| Company | Source PDF | Result |
|---|---|---|
| Rajasthan Tube Manufacturing Company Ltd | https://www.bseindia.com/xml-data/corpfiling/AttachLive/00c071fc-0454-482e-9208-af312133ad7a.pdf | The workflow correctly took the "No P&L Found" path (execution #1148). |

## Notes
- The numbers come from Gemini using the assignment prompt. The workflow's Number Check node recalculates margins from them and adds a warning if they differ by more than 0.05.
- No Q2 or Q4 (6-column) sample is included: Gemini rate limits (503, then 429) stopped my later runs.
- There are no WhatsApp screenshots because delivery was blocked (Twilio trial template rule, then Meta error 131031). See the main README, sections 5 and 8.
