# Changelog

## 0.3.1
- Expanded `build-demo`'s outbound-calling setup (Step 6) from a vague "create a connector via Postman" into an actual runbook: the Postman workspace/environment names, the exact `Create Connector` request body, and a step to trigger a test call directly through Postman (against your own phone) before writing any function code — isolates connector/token problems from code problems.

## 0.3.0
- Rewrote `build-demo` end to end: dropped the stale dependency on the external `PolyAI-LDN/oisin-nikola-store` skill in favor of the current, self-contained pipeline (Agent Studio agent with in-project mock data; optional Railway landing/narrative website).
- Added a Step 0 self-intro + scope menu so `build-demo` explains what it can build and asks which channels/integrations/website are wanted, instead of assuming full scope — opens the skill up to the whole SC team, not just one person's workflow.
- De-personalized account-specific references (local file paths, one SC's secret name) with confirm/replace guidance for other accounts.
- Added a one-time Railway account/CLI setup section, since Railway is a personal-per-SC account, not shared team infra.

## 0.2.1
- Hardened the PS-calculator step: always copy the master first and edit the COPY; load the Sheets write tools by exact name via ToolSearch; added a graceful fallback when `make_api_mutating_request` isn't connected (copy + hand the SC the rows to paste, and name the connector to enable) instead of erroring.

## 0.2.0
- `scope-deal` can now pull deal context automatically: **Salesforce** (opportunity fields → volumes, use cases, integrations, telephony, languages, metrics) and **Gong** (recent call summaries / transcripts) by account name — instead of requiring pasted notes.
- Added `reference/data-sources.md` (SFDC field → MD mapping, Gong usage, grounding rules).

## 0.1.0
- Added `scope-deal` skill: one Markdown source of truth → call flow (HTML→PDF), scoping doc (.docx), and populated PS Level-of-Effort calculator (Google Sheet).
- Packaged the repo as a Claude Code plugin marketplace (`polyai-scoping` → `scoping-suite`).
