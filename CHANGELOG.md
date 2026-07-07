# Changelog

## 0.2.1
- Hardened the PS-calculator step: always copy the master first and edit the COPY; load the Sheets write tools by exact name via ToolSearch; added a graceful fallback when `make_api_mutating_request` isn't connected (copy + hand the SC the rows to paste, and name the connector to enable) instead of erroring.

## 0.2.0
- `scope-deal` can now pull deal context automatically: **Salesforce** (opportunity fields → volumes, use cases, integrations, telephony, languages, metrics) and **Gong** (recent call summaries / transcripts) by account name — instead of requiring pasted notes.
- Added `reference/data-sources.md` (SFDC field → MD mapping, Gong usage, grounding rules).

## 0.1.0
- Added `scope-deal` skill: one Markdown source of truth → call flow (HTML→PDF), scoping doc (.docx), and populated PS Level-of-Effort calculator (Google Sheet).
- Packaged the repo as a Claude Code plugin marketplace (`polyai-scoping` → `scoping-suite`).
