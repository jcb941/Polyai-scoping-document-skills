# Changelog

## 0.3.3
- **Website/browser walkthrough (Step 4) is no longer prescriptive.** The skill used to default to a fixed landing+narrative Railway template and size polish against named past demos (Hotel, Lenovo). It now has the SC describe how the prospect should experience the demo — on the prospect's own real site via the Chrome extension, a hosted site of whatever shape fits the story (pitch page, fuller self-service app, separate presenter narrative), a local walkthrough, or none — and builds to that instead of assuming a shape.
- **Widget token troubleshooting** added to Step 4: the "invalid connector token" fix for orphaned webchat/polyphone widget tokens (env-dropdown round-trip to force reissue; delete+recreate for polyphone/WebRTC when the token doesn't actually change). Publishing/redeploying does not fix this.
- **SMS setup (Step 5) rewritten as an actual runbook** from the Notion source of truth ("Setup Number for SMS"): the exact DataDog `service:kamailio` query, the `X-Twilio-AccountSid` header to read, the real Twilio-API Postman workspace (a different workspace from outbound's) and its two requests, and the confirmed `conv.channel_type == "sms.twilio"` value — added alongside the other confirmed channel values in Channel Detection.
- **Outbound calling (Step 6):** documented `variantId` on the call-trigger body — one connector per project covers every Studio variant; no need for a separate connector/token per variant.
- **Testing (Step 7):** added exercising every channel's actual code path via `create-a-new-debug-chat-session`'s `channel` field (`sip.polyai` for voice, `chat.polyai` for webchat) before calling a voice-enabled demo verified — a passing chat-only test can hide a voice-only crash.

## 0.3.2
- Added patterns to `build-demo` learned from studying Poly Hospital's build (Agent Studio project `PROJECT-DELF3ZLS`):
  - **Reuse one status token across every gated function** (verification, missing-prior-step, etc.) instead of each function hand-writing its own guard string — generalized from the existing `route_intent` STOP pattern.
  - **Emergency Escalation** reframed as explicit, first, and overriding — not just one more rules.txt bullet — with guidance to adapt the trigger list per vertical (healthcare/banking fraud/etc.), keeping the existing elevator "stuck" false-positive lesson.
  - **Infer, Don't Interrogate** — check whether a yes/no disambiguation question is already answerable from phrasing the caller used, before adding it to rules.txt/a flow step.
  - **Mock Data Option C**: Studio Variants + Attributes as a structured alternative to per-entity mock dicts/KB docs when a demo has many similar sub-entities (doctors, branches, properties) needing distinct personalization.
  - **KB Topics**: added the fact/pointer/workflow topic-type distinction — pick one per topic, never blend, and never store the same fact as both static KB text and behind a function.

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
