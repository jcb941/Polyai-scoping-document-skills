# MD Scope File — Schema & Locked Conventions

The MD file is the single source of truth. All three outputs are generated from it. Keep it human-readable and consistently structured.

## Status taxonomies (use these exact values)

- **Use-case status:** `In Scope` · `Scoping` · `Discussed` · `Excluded`
  - Only In Scope + Scoping count toward effort and token volume.
- **API status:** `PolyAI Proposed` (RED FLAG — no known API, we invented it) · `Discussed` (client says it exists, no docs seen) · `Documentation` (we've seen docs + confirmed access) · `Validated` (we have sandbox/postman) · `Known` (system we've integrated before)
- **Priority:** `Initial Phase` · `Second Phase` · `Third Phase` · `Not Scheduled`

## Locked conventions (non-negotiable)

1. **Entities = collected from the caller only.** API return values go in the API entry (Input/Returns), never in the entity list — otherwise the entity count and PS effort inflate.
2. **Capture depth scales with status.** In Scope / Scoping = full detail (entities, APIs, %, seconds). Discussed = name + description only (parked).
3. **Scoping vs. Discussed.** Scoping = sized (has % of calls + avg seconds). Discussed = parked (no numbers).
4. **PS calculator = one-time PS fee only.** ARR is owned by Salesforce; never compute or restate it.
5. **No invented success targets.** Capture the metric + current baseline where discussed; targets come at commitment.
6. **Escalation destination is out of scope pre-sales** — "existing routing."

## Structure

```markdown
# Scope Source of Truth — <Client>

## Meta
- Client / SC / AE / Date · Version / Deployment type (New Business | Expansion) / Primary channel

## Environment
| Layer | System | Status |
(telephony, CRM, backend, middleware, chat — each with an API-status value)

## Volumes & Success
- Total calls/yr: <n>   (feeds the PS calculator TOTAL CALLS)
- Success frame: <metric · direction · current baseline if known>   (no invented target)

## Use Cases
### UC<n> — <Name>
`Status: <> | Priority: <> | % Calls: <> | Avg Sec: <> | Chat: <yes/no>`
- Dept / Owner
- Description
- Flow: <one-line happy path → fallback → escalation>
- Entities (from caller): <name — description/format>   (omit for use cases that collect nothing)
- APIs:
  | API | System | Contact | Description (incl. Input/Returns) | Status |

## LoE Inputs   (so the whole PS calculator is fed from one source)
- Languages: <list>            (1 = no add; each extra adds testing time)
- Knowledge items (FAQs): <n>  (~60 per week)
- Troubleshoot / walk-alongs: <n>
- FAQ import sources: <n>       (per KB system)
- Telephony: <Existing add-on | Basic SIP | PSTN via Twilio | SIP + extra features>
- Chat: <none | Supported PAI Widget | Supported Other Widget | New Chat System>
- SMS: <Yes/No>
- Hypercare weeks: <n>

## Scope Summary
- In scope (Phase 1): …
- Scoping / Phase 2: …
- Out / deferred: …

## Open Items
| # | Item | Question | Owner |
```

Every use case's Entities and APIs feed the PS calculator's counts; the LoE Inputs feed the remaining calculator cells. Keep entity and API names **consistent across use cases** (the calculator counts unique names).
