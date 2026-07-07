# PS Calculator — Population Recipe

The PS Level-of-Effort calculator lives in Google Sheets and uses `COUNTUNIQUEIFS`, which only recomputes in native Sheets (it breaks in downloaded .xlsx). So we **copy the master template** and write into it via the Sheets API — never rebuild the file.

## Tools (load with ToolSearch first)
- `google_drive_copy_file`
- `google_sheets_make_api_mutating_request` (POST to `values:batchUpdate`)
- `google_sheets_make_api_get_request` (verify formulas computed)

## Master template to copy
`1bAfOWCgmlfKwWjeWCM7N62OC3fID4oEi7tz733H8E7Q`  → copy as "`<Client> — Scoping Tool`" into the SC's Drive.

## Use Case Scope tab — block layout

Each use case is a **block**: a header row, an entity sub-table, an API sub-table. Write ONLY data cells. Never write formula cells: **H** (Voice Tokens), **M** (Use Cases), and **J on sub-rows** (TYPE is auto-set).

**Header row columns:** A=Use Case · B=Department · C=Owner · D=Description · E=% Calls (fraction, e.g. 0.2) · F=Avg Seconds · G=Chat (boolean) · I=Status · J=Priority · K=Current FTE · L=Success Criteria
**Entity rows:** A=Entity Name · B=Description  (caller-collected only)
**API rows:** A=API Name · B=Contact · C=System · D=Description (incl. Input/Returns) · E=Status · F=Docs/Notes

**Block rows (verified):**
| Block | Header | Entity rows | API rows |
|---|---|---|---|
| 1 | 5 | 7–8 | 10 |
| 2 | 11 | 13–16 | 18–19 |
| 3 | 20 | 22–26 | 28–30 |
| 4 | 31 | 33–35 | 37–40 |
| 5 | 41 | 43–49 | 51–54 |
| 6 | 55 | 57–59 | 61–63 |
| 7 | 64 | 66–69 | 71–73 |

If the deal has more use cases than blocks, read the live sheet below row 73 to find further blocks, or note the overflow to the SC. Clear any stray template example data in unused API/entity rows (e.g. block 1 ships with a sample API in row 10) so it isn't counted.

## Summary cells (formulas — read to verify, never write)
- `B1` TOTAL CALLS (input — DO write this), `B3` TOTAL CHATS (input)
- `F1` API systems · `F2` API endpoints · `F3` use cases (In Scope+Scoping) · `H1` unique entities

## Level of Effort Calculator tab — SC inputs to write
`C4` languages · `C5` knowledge items · `C6` troubleshoot · `C7` FAQ sources · `C12` B2B outbound (Yes/No) · `C15` telephony (Existing (add-on) | Basic SIP integration | PSTN integration through Twilio | SIP integration with the extra features we already support (SMS etc.)) · `C16` chat · `C17` SMS (Yes/No) · `C18` extra · `C19` hypercare.
**Do NOT write** `C3`, `C8`, `C9`, `C10` — these auto-fill from the Use Case Scope tab.
**Leave the ARR cells alone** (`I21`, `I27`–`I29`) — the calculator is PS-fee-only; ARR lives in Salesforce.

## Write & verify
1. `values:batchUpdate` with `{"valueInputOption":"RAW","data":[ {range,values}, ... ]}`. Use `false` for Chat booleans; fractions for %; leave Avg Seconds blank for Scoping items with TBD timing (they contribute 0 tokens, correctly).
2. Verify with a GET on `Level of Effort Calculator!A1:J32` and `Use Case Scope!A1:H3` — confirm C3/C8/C9/C10 populated and `E23` (person-weeks) / `E24` (days) / `E29` (PS $) computed.
3. Report the live sheet link + the PS $ figure.
