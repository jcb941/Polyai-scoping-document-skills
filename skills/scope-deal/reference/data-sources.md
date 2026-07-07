# Pulling Deal Context — Salesforce & Gong

When the SC gives an account / opportunity name, pull context automatically instead of asking them to paste. Load the tools first with **ToolSearch** (they are deferred): `sf_query`, `sf_get_opportunity`, `search_calls`, `get_call_summary`, `get_transcripts` (search by name — they live on the sales MCP).

## Salesforce — system of record for structured facts

Query the open opportunity for the account:

```sql
SELECT Id, Name, StageName, Type, Account.Name, Owner.Name,
  Overall_Call_Volume__c, Overall_Chat_Volume__c, AHT__c,
  Use_Case__c, Use_Case_Descriptions__c,
  Back_end_Integrations__c, Back_end_Systems__c, Telephony_Systems__c,
  List_of_Languages_Required__c, Chat_Integrations__c,
  Metrics__c, CoM_Metrics__c, Positive_Business_Outcomes__c,
  Competitors_Identified__c, NDA_Signed__c, Security_Questionnaire_Completed__c
FROM Opportunity
WHERE Account.Name LIKE '%<account>%' AND IsClosed = false
ORDER BY LastModifiedDate DESC LIMIT 5
```

Map fields into the MD schema:

| SFDC field | MD target |
|---|---|
| `Account.Name` / `Type` | Meta → client / deployment type |
| `Overall_Call_Volume__c` (or `Application_Call_Volume_pa__c`) | Volumes → total calls |
| `Overall_Chat_Volume__c` | Volumes → chats |
| `AHT__c` | per-use-case avg-seconds baseline |
| `Use_Case__c` / `Use_Case_Descriptions__c` | Use Cases |
| `Back_end_Integrations__c` / `Back_end_Systems__c` | Environment + API entries |
| `Telephony_Systems__c` | Environment → telephony |
| `List_of_Languages_Required__c` | LoE Inputs → languages |
| `Chat_Integrations__c` | LoE Inputs → chat |
| `Metrics__c` / `CoM_Metrics__c` / `Positive_Business_Outcomes__c` | Volumes & Success → success frame (no invented target) |
| `Competitors_Identified__c` | context only (not scope) |
| `NDA_Signed__c` / `Security_Questionnaire_Completed__c` | Open Items |

Empty fields are gaps → Open Items, never fabricate. If a field doesn't exist in this org's SFDC, drop it and proceed (don't abort).

## Gong — call narrative for flows and entities

1. `search_calls(query="<account>", from_date, to_date)` — max 45-day window; narrow dates to reach older calls.
2. `get_call_summary(call_id)` for up to the **3 most recent** calls — key points + action items.
3. `get_transcripts(call_id)` **only** for a specific call the SC wants analyzed in depth (20–50k tokens each — use sparingly).

Use call content to flesh out use-case flows, entities captured from callers, and which APIs were discussed (with the right status).

## Grounding rules

- **SFDC governs** numbers, names, systems, dates. Gong / pasted notes inform narrative.
- If Gong contradicts SFDC on a figure, **surface the discrepancy** — don't silently pick one.
- Anything not found in either source = an **Open Item with a question**, not a guessed fact.
