---
name: build-demo
description: "Build a complete PolyAI prospect demo: an Agent Studio voice/webchat agent with mock data, plus an optional Railway-hosted landing page and demo narrative page when a website walkthrough is wanted. Use when someone says 'build a demo for [company]', 'spin up a demo site', or 'create a prospect demo'. This is the full pipeline."
---

# Build Demo — Full Pipeline

This skill orchestrates a PolyAI prospect demo build — omnichannel by default. The Agent Studio project is the core deliverable and always gets built; a landing page/narrative website is an optional add-on for when the SC wants a browser-based walkthrough, and doesn't require Railway to be set up in advance (see Step 3).

This skill is shared across the SC team, not written for one account or one person. A couple of reference values below (a Studio account id, a shared secret name) reflect the `poly-scs-us` sandbox this was built against — if you're on a different account, confirm the equivalent names/ids there rather than assuming these literal strings exist.

---

## Step 0: Introduce Yourself, Then Scope the Build

**Do this first, before touching the critical rules or writing any code.** Don't silently launch into the full pipeline — most demos only need a slice of what this skill can build.

1. **Explain briefly what this skill builds:**
   > This builds a PolyAI prospect demo: an Agent Studio voice/webchat agent with mock data, and optionally a Railway-hosted landing + narrative website for a browser walkthrough. I can also wire in Salesforce (cases/handoff), Zendesk (tickets), SMS, outbound calling, or an Amazon Connect live-agent screen pop — whichever this demo actually needs.

2. **Offer the menu so the SC picks scope up front**, rather than assuming everything is in play:
   - **Channels** — voice, webchat, SMS, outbound calling (any combination)
   - **Integrations** — Salesforce, Zendesk, Amazon Connect handoff, or none (mock data only)
   - **Website** — landing + narrative walkthrough page, or skip it. Most demos don't need one (Agent Studio + the webchat extension + direct dial-in covers a live demo fine). This is also the one piece that needs *your own* Railway account, not shared team infra — if you want a website and don't already have Railway set up, say so and I'll walk you through the one-time setup (see Step 4) rather than assuming either way.
   - **Languages** — English + Spanish by default; English-only if they'd rather skip bilingual support

3. **Then run Discovery below** to fill in the specifics for whatever scope they chose. If they already gave you enough in their first message (a transcript, a use-case doc, "just voice + Salesforce"), skip straight to confirming what you inferred instead of re-asking.

---

## CRITICAL RULES — Read Before Writing ANY Code

These apply once you know the scope from Step 0 — skip the subsections for components that aren't in play (e.g. skip "Salesforce Secrets" entirely if this demo has no SF integration).

These are confirmed pitfalls from production builds. Violating any of these will break the demo.

### Function Structure
- `from _gen import *  # <AUTO GENERATED>` as first line of every function file
- `def func_name(conv: Conversation, ...)` — always
- `@func_parameter("name", "description")` — exactly 2 args. No type arg. 3 args breaks the validator. Stack as many as needed — one `@func_parameter` per field on a single multi-field collection function (e.g. address line 1/2, city, state, zip on one `confirm_address` call) is a proven pattern, not just single-field functions.
- `start_function` must be `def start_function(conv: Conversation):` — not a bare script
- `flow.goto_step(step_name, label=None)` — the optional `label` gives the transition a human-readable name in conversation review, useful once a flow has more than a few steps.

### Three Function Locations — don't conflate them
- **Top-level `functions/`** — global LLM tools, `@func_description`, called from anywhere via `{{fn:name}}`. The default for every demo.
- **Flow-scoped `functions/`** (`flows/<flow_name>/functions/`) — same `@func_description` requirement, but called via `{{ft:name}}` (not `{{fn:}}`) and only reachable from within that flow's own steps. Reach for this once a demo has enough flows that one flat top-level `functions/` directory gets unwieldy — confirmed at production scale (AdaptHealth: 85 `{{ft:}}` calls across 11 flows vs 12 `{{fn:}}`). For a normal-sized demo, top-level-only is still simpler and the right default — don't add flow-scoped dirs just because they exist.
- **`function_steps/`** — a different thing entirely. No `@func_description`, not LLM-callable, plain `def func_name(conv: Conversation, flow: Flow)` invoked via `flow.goto_step()` as a step's actual implementation. Since the LLM can't discover or call these directly, they're easy to end up as orphaned dead code if a refactor moves logic elsewhere — keep them only where a step's real behavior *is* Python, not an LLM decision point.

### Guaranteed Function Chaining
Use the ID&V "STOP" pattern (below) to block premature entry into a flow. Use the inverse — an explicit imperative in the return string — to guarantee a function actually gets called right after another one finishes (e.g. every booking must be followed by a confirmation SMS):
```python
return (
    "NEXT ACTION REQUIRED: you MUST now call send_confirmation_sms with the booking details. "
    "Do NOT skip this step, do NOT ask the customer first — just call it."
)
```
Without an explicit imperative like this, the LLM sometimes just *describes* the confirmation instead of actually calling the function.

### Outcome Tracking — write_metric
Call `conv.write_metric(name, value, write_once=True)` at any meaningful outcome — booking completed, escalation triggered, SMS sent, case created. `write_once=True` stops double-counting if the function path is somehow reached twice in one conversation. Not required for the demo to work, but every recent build uses it, and it's what gives you outcome stats to show after a demo session.

### Filler Utterances During Slow Calls — @func_latency_control
```python
@func_latency_control(
    delay_before_responses_start=1500,
    silence_after_each_response=800,
    delay_responses=[("One moment while I check that...", 2000), ("Thanks for your patience...", 4000)],
)
```
Plays a filler line if the function is still running past `delay_before_responses_start` ms, with further fillers as it keeps running. Use on any function with a real or simulated slow lookup/API call so the caller isn't sitting in dead air.

### Imports & Libraries
- `secret_vault` must be explicitly imported: `from agent_v3.deployment_service.agents.functions.builtin import secret_vault` — NOT available from `from _gen import *`
- `pytz` is NOT available. Use `datetime.now()` or `zoneinfo.ZoneInfo`
- **Use `requests` for ALL external HTTP calls, NOT `urllib.request`** — `urllib` with SSL fails silently in the Agent Studio runtime. `requests` works reliably.
- Alternative secret pattern: `conv.utils.get_secret("Name")` returns a dict, use `.get("value")`

### Outbound Calling — call the API directly, Railway is not part of this
Railway simulates the client's backend for the demo **website** (reservations, order lookups). It has nothing to do with triggering outbound calls — don't route outbound through it.

Store the connector token as an Agent Studio secret (`PolyOutboundToken`) and call the outbound API straight from the function with `requests` (confirmed working in production builds):

```python
from agent_v3.deployment_service.agents.functions.builtin import secret_vault
import requests

token = secret_vault("PolyOutboundToken")  # or conv.utils.get_secret("PolyOutboundToken").get("value")
resp = requests.post(
    "https://api.us-1.platform.polyai.app/v1/outbound-calling",
    json={"to_number": to_number, "metadata": {...}},
    headers={"X-PolyAi-Auth-Token": token, "Content-Type": "application/json"},
    timeout=10,
)
```

This covers both outbound cases:
- **Proactive outbound** (appointment reminders, follow-ups, not initiated by the bot) — trigger it straight from Postman against the same endpoint with the connector token. No code, no Railway.
- **SMS/webchat→voice callback** ("call me") — a `trigger_outbound_call.py` function calls the endpoint directly as above. The LLM invokes it via `{{fn:trigger_outbound_call}}`, same rule as any transfer function — don't call it programmatically from inside another function.

Metadata arrives in the outbound call's `conv.sip_headers` — read it in `start_function.py` to drive a personalized greeting instead of a generic one.

### Webchat — NO ANI, NO SMS
- Webchat has **NO phone number and NO ANI**. Never say "I recognize the number you're calling from" on webchat.
- **Never offer SMS confirmation on webchat.** There is no phone to send to.
- Webchat ID&V: ask for **name + email** → SOQL Contact lookup → confirm address from SF. NOT address-first.
- Webchat greeting is controlled by the **webchat channel config** in Agent Studio, NOT `voice/configuration.yaml`. The `$first_utterance` variable only works for voice. Set the webchat greeting in rules.txt or update it in Agent Studio UI.

### Webchat — quick-reply suggestion bubbles
Use `conv.set_response_suggestions(["Option 1", "Option 2"])` at key decision points to show tappable quick-reply chips under the agent's message, instead of making the user type. Not documented in `poly docs` and not used by default in any template — found via the `_gen/conversation.py` SDK stub (`set_response_suggestions(self, suggestions: list[str]) -> None`). Gate it with `if conv.channel_type == "webchat.polyai":` since it's a webchat-only UI feature (harmless but pointless on voice). Good spots: initial greeting/menu, any yes/no offer (insurance changed?, want a text link?, start this order?), anywhere the agent asks a question with a small fixed set of likely answers. Can't be verified via `poly chat` CLI (the field doesn't surface in its debug/JSON output) — confirm in the actual widget.

### rules.txt vs Flows — NEVER DUPLICATE
If rules.txt has detailed step-by-step instructions for scheduling/reschedule/cancel AND there are flows, the LLM follows the rules and **never calls route_intent**. The flow prompts never execute.

**Rules should say:** "When the customer wants to schedule → call route_intent with intent 'service_scheduling'. Do NOT handle details yourself."

**Flows handle everything:** triage, data collection, case creation, handoff, SMS.

### route_intent — ID&V Gate
`route_intent` MUST check `is_known_contact` before entering ANY flow. If the customer isn't identified:

```python
if not is_known:
    conv.state["pending_intent"] = intent
    return (
        "STOP — do NOT enter the flow yet. The customer has not been identified. "
        "Ask for their name and email on file so you can look up their account. "
        "Once found, call route_intent again."
    )
```

Without this gate, the LLM enters flows with no contact/case data and everything silently fails.

### SMS Text Formatting vs TTS
The LLM applies `<channel:voice>` TTS expansion rules to SMS text bodies unless explicitly told not to. It will write "six oh six one one" instead of "60611" in a text message.

**Every flow step that composes SMS** must say: "Use normal WRITTEN format — NOT voice/TTS expansion."

**rules.txt must have a standalone section:**
```
## SMS TEXT FORMATTING
When composing SMS message bodies:
- Write numbers normally: "60611" not "six oh six one one"
- Write addresses normally: "Suite 1200" not "Suite twelve hundred"
- Write confirmation numbers normally: "TK-CHI-37578" not "T K, C H I..."
```

### Emergency Escalation
The word "stuck" triggers false emergency escalations. An elevator that stopped or won't move is a SERVICE issue, not an emergency.

**Add explicit rules:**
```
An emergency is ONLY when someone explicitly says:
- A person is TRAPPED inside the elevator
- Equipment is sparking, smoking, or on fire
- Someone is injured
- The caller explicitly says "emergency"

ALWAYS ask "Is anyone trapped inside?" before escalating.
```

### SF Case Linking on Webchat
When creating a Case on webchat (no ANI), do a Contact lookup by name in the scheduling function so the Case gets a ContactId. Without this, the Case is orphaned and can't be found when the customer calls/texts later.

### Idempotent Case/Order Creation — ensure_X_exists
If a case/order/ticket can be created from more than one entry point in the same conversation (e.g. a support flow AND a separate handoff function), wrap creation in an `ensure_X_exists(conv)` helper that checks `conv.state` for an existing id first. Generate the id deterministically (e.g. `hashlib.md5(conv.id.encode()).hexdigest()[:8]`) rather than with `random` — so no matter which entry point runs, or if one runs twice, the same call never creates a duplicate.

### Phone Number Read-Back
When confirming a phone number back to the caller (SMS opt-in, callback number, etc.): validate the digits → format → convert to spoken form for TTS (grouped, not run together) → ask the caller to confirm → on rejection, ask once more, then transfer to a human rather than looping indefinitely. If a demo has more than one place that reads a number back (e.g. SMS opt-in and a callback flow), implement it once as a shared helper.

### Salesforce Secrets (poly-scs-us)
- `PolySalesforceClientID`, `PolySalesforceClientSecret`, `Poly_SalesforceUsername`, `PolySalesforcePassword` — for OAuth password flow
- `Salesforce API Credentials` — for webchat handoff (org_id, developer_name, url)
- SF instance: `polyai2-dev-ed.develop.my.salesforce.com`, API version `v58.0`
- Webchat handoff defaults: org_id `00DHr0000011gte`, developer_name `PolyAI_Webchat`, url `https://polyai2-dev-ed.develop.my.salesforce-scrt.com`

### Webchat Extension
- Lives in the team's `solutions-consulting` repo at `tools/webchat-extension` — clone it (or ask your team lead for access) if you don't already have it locally
- **MUST be v2.1.0+** — old version lacks CSP stripping and widgets won't load on prospect sites
- Always `git pull` your local clone before loading
- `chrome://extensions` → Load unpacked → select the `webchat-extension` folder
- Widget script URL field takes the **bare URL** — `https://messaging.us-1.poly.ai/widget/XXXXX.js` — NOT `<script src="...">`
- Enable "Hide incumbent" for the prospect's chat vendor (Salesforce, Intercom, etc.)

---

## Discovery — Ask Before You Build

Continuation of Step 0 above: make sure you have enough to build something useful before writing any code. If the user gave you just a company name, ask the right questions. If they've already provided context (a call transcript, meeting notes, a use case doc), extract what you can and only ask about gaps.

### Required (must have before starting)
1. **Company name** — who is this for?
2. **Use cases / flows** — what should the bot handle? (billing, booking, cancellations, FAQ, intake, etc.)
3. **Channels** — voice only? Voice + webchat? Full omnichannel?

### Important (ask if not obvious from context)
4. **Integrations** — Zendesk tickets? Salesforce cases/handoff? Real CRM reads/writes?
5. **Agent personality** — formal/professional or warm/casual? Any brand voice guidelines?
6. **Existing project** — building in a new project or an existing one? If existing, get the project ID and Studio URL.
7. **Website?** — does this need a landing page/narrative for the prospect to click through, or just the Agent Studio demo itself (voice/webchat)? Most demos don't strictly need one — plenty of real builds have shipped as Agent Studio + the webchat extension + direct dial-in, with no website at all. Don't build one unless it's actually wanted; if the SC seems unsure, say so explicitly rather than defaulting to "yes."
8. **Railway, only if a website is wanted** — Railway accounts are personal, not a shared team org, so check: does the SC already have a Railway account set up and the CLI installed/logged in on their machine? If yes, proceed to Step 4. If no, offer the choice rather than assuming: walk them through the one-time account/CLI setup (see "Railway Account Setup" under Step 4), or skip the website for this demo and stick with Agent Studio only. Either is fine — it's their call.

### Nice to have (ask if the user seems engaged in planning)
9. **Demo audience** — who will see this? (prospect's engineering team, business stakeholders, live event)
10. **Mock data** — any specific customer names, accounts, or scenarios they want to walk through?
11. **"Wow" moment** — what should impress them? (ANI recognition, upsell, real-time ticket creation, cross-channel context)
12. **Languages** — English only or English + Spanish?

### How to ask
Don't dump all 10 questions at once. Group them naturally:

> "Before I start building, a few quick things:
> 1. What flows should the bot handle? (e.g., billing inquiries, cancellations, bookings)
> 2. Voice only or voice + webchat?
> 3. Do we need any integrations — Zendesk for tickets, Salesforce for cases, anything like that?
> 4. And is there an existing Agent Studio project or should I create a new one?"

If the user gave you a call transcript or meeting notes, extract the answers from those first and confirm.

## Pipeline

### Step 1: Plan the Demo Story

Before building anything, discuss the customer journey with the user:
- What channels make sense? (voice, chat, SMS, outbound, proactive outbound)
- What flows? (booking, intake, status check, reschedule, cancel, upsell, FAQ)
- What's the "wow" moment? (ANI recognition, cross-channel context, SF case lifecycle)
- Map the demo acts (typically 3-5 acts across channels)
- One customer across all channels — that's the story

### Step 2: Build the Agent Studio Project

1. **Create/init project** — `POLY_ADK_KEY=$(cat ~/.poly/credentials.json | python3 -c "import sys,json; print(json.load(sys.stdin)['us-1'])") ~/agent-studio-skills/.venv/bin/poly init --region us-1 --account_id poly-scs-us --project_id PROJECT-XXX`
2. **Model config** — Speech-native mode + Dialog-RSN-1 in Studio's Voice → Model tab (see below)
3. **Knowledge base** via MCP — create on a branch (can't write to main directly), then merge
4. **Start function** — channel detection, SF OAuth + ANI SOQL lookup (voice/SMS), outbound context reading, dynamic greetings per channel
5. **Functions** — route_intent (with ID&V gate), lookup_contact, switch_language, goodbye, escalate, domain functions, send_confirmation_sms, trigger_outbound_call, handoff_to_agent
6. **Flows** — one advanced_step per flow. Flows handle ALL details. Rules just route.
7. **SMS templates** — `${state_sms_body}` dynamic templates
8. **Voice config** — `welcome_message: $first_utterance`, keyphrase boosting, phrase filtering
9. **Rules.txt** — channel detection, greeting, ID&V (webchat: name+email, voice: ANI+address), intent routing (JUST route, no details), SMS formatting section, TTS expansion in `<channel:voice>`, chat style, emergency rules, hard guardrails
10. **Push** — `poly push` creates a branch, merge via MCP

### Step 3: Mock Data — pick the approach that fits

Every demo needs realistic fake data (customer lookups, orders, accounts) for the agent to look up. There are two equally valid ways to serve it — default to Option A, only reach for Option B when there's a real reason to.

**Option A — inside Agent Studio (default, no external setup)**

Functions serve mock data directly from a small in-project dataset (dict/JSON keyed by phone/account/order id). No external service, nothing to deploy, works the moment the project exists. Use this for every demo unless a website (Step 4) is also being built.

```python
# lookup_customer.py — mock data lives in the function itself
CUSTOMERS = {
    "+15551234567": {"name": "James Carter", "account_id": "ACC-1042", ...},
}

def lookup_customer(conv: Conversation, phone_number: str):
    record = CUSTOMERS.get(phone_number)
    if not record:
        return "No matching account found."
    conv.state.customer_name = record["name"]
    ...
```

Keep confirmation numbers generated locally too (e.g. `f"GX-{random...}"`) rather than round-tripping anywhere — nothing to time out mid-conversation.

**Option B — Railway mock API (only when a website is also being built, Step 4)**

If the demo includes a landing/narrative website, that website and the agent should read from the *same* mock data, so a shared Railway backend makes sense there:
```
GET /api/customer/:phone
GET /api/order/:reference
GET /healthz
```
Functions call these with `requests`. Don't stand up Railway just to serve mock data to the agent alone — that's what Option A is for. Railway is never required to start a build; add it only when Step 4 is in scope.

### Step 4: Landing Page + Narrative (optional — only for website/browser demos)

Skip this step entirely for a voice/webchat-only demo — the Agent Studio project from Step 2 is the whole deliverable, and the SC can demo it live via the Chrome webchat extension or by dialing the number directly. Build this only when the SC specifically wants a browser-based "try it live" experience for the prospect.

#### Railway Account Setup (one-time per SC — skip if already set up)

Railway accounts are **personal**, not a shared team org — each SC needs their own. If this is your first website build, do this once:

1. Sign up at [railway.app](https://railway.app) (GitHub login is the fastest path) — the free Trial plan is enough for a demo site, no card required to start.
2. Install the CLI: `npm i -g @railway/cli` (or `brew install railway` on macOS).
3. `railway login` — opens a browser to authenticate the CLI against your account.
4. `railway init` in the project directory to create a new Railway project linked to your account (or `railway link` to attach to one you already made in the dashboard).

After this, `railway up --service <name>` (used later in this step) deploys to *your* account. If a demo needs to be handed off or co-owned (e.g. someone else needs to redeploy it later), share access to the specific Railway project from the dashboard — don't share account credentials.

1. Stand up a Railway Express app serving the mock API (Step 3, Option B) plus a landing page and, optionally, a narrative walkthrough.
2. Landing page = pitch only (hero, what it handles, proof, next steps) — no phone number or call widget here.
3. Narrative page = the single call-in/test surface (call widget or phone number, demo account details, suggested prompts), invoke `/demo-narrative` to generate `narrative.html`.
4. Personalize for the prospect — the persona is the real person who'll call in, not a fictional character.
5. Deploy: `railway up --service <name>`.

### Step 5: Set Up SMS (manual steps with user)

1. Add Twilio number to project in Agent Studio
2. Call the number → DataDog lookup → get AccountSid (should match the team's shared Twilio account — check with your team lead if it looks unfamiliar)
3. Postman GET "Get Number SID" → query with number → get PNxxxxxxxx
4. Postman POST "Add Campaign" → set PhoneNumberSid → send
5. Wait a few minutes, test

### Step 6: Set Up Outbound Calling (manual steps with user)

**Enable outbound for the project — create the connector:**
1. Connect to **Wireguard us-1 VPN**. Connector creation requires mTLS, which only works through Postman's configured client certs — it can't be done with plain `curl`.
2. Open the team Postman workspace: `https://polyai-team-y.postman.co/workspace/03eccd5f-bda1-41a0-b352-25a0d757f8ee`
3. Select environment **"Setup us-1 prod"** — or **"Setup us-1 staging"** if the project lives on `studio.staging.poly.ai`. Staging and prod are fully separate infra (different connector-service host, different mTLS root CA, different token) — a prod-issued cert will not validate against staging, and vice versa.
4. Open the **"Create Connector"** request (`POST {{BASE_URL}}/api/v1/connector`), set the body:
```json
{
    "name": "<PROJECT_NAME>_OUTBOUND",
    "account_id": "poly-scs-us",
    "project_id": "<PROJECT_ID>",
    "client_env": "sandbox",
    "variant_id": "",
    "extra_info": {"telephony": {"asr_lang_code": "en-US", "tts_lang_code": "en-US", "ari": true}},
    "outbound_integration_name": "outbound_sales"
}
```
5. Send → copy `connection_token` from the response. This connector + token is what "enables" outbound for the project — there's no separate toggle in Studio itself.
6. Store the token as an Agent Studio secret `PolyOutboundToken`.

**Test the connector before writing any function code:**
7. Switch Postman to environment **"Call us-1 prod"** — no VPN needed, base URL `https://api.us-1.platform.polyai.app`.
8. Build/open a request: `POST {{BASE_URL}}/v1/outbound-calling`, header `X-PolyAi-Auth-Token: <connection_token>`, body `{"to_number": "+1...", "metadata": {...}}`.
9. Send it against your own phone number first. If it doesn't ring, the problem is the connector/token, not any code you haven't written yet — much faster to isolate here than mid-conversation later.

**Wire up however the demo needs to trigger it:**
10. Proactive/demo-operator calls (reminders, follow-ups): just repeat the Postman request from steps 7-8 whenever you want to place one — no code required.
11. Bot-triggered callbacks ("call me"): build `trigger_outbound_call.py` per the direct-call pattern in Critical Rules, then test it from an actual SMS/webchat conversation, not just Postman.

**One token per project** — using project A's token from project B silently routes the call into project A's conversation review instead of erroring, which is a confusing thing to debug live. Keep a note of which token belongs to which project.

### Step 7: Deploy & Test

- If a website was built (Step 4): deploy Railway (`railway up --service <name>`)
- Publish Agent Studio to sandbox via MCP merge
- Test each channel in scope (chat → SMS → outbound → inbound voice)
- Verify: ANI recognition, cross-channel context, SMS delivery, SF case lifecycle, handoff

## Function Checklist (every demo needs these)

| Function | Purpose |
|----------|---------|
| `start_function.py` | Channel detection, SF OAuth, ANI SOQL lookup (voice/SMS), outbound context + SF lookup by name, dynamic greeting per channel, date/time context |
| `route_intent.py` | **ID&V gate** — blocks flow entry until customer identified. Routes to flows. |
| `lookup_contact.py` | SOQL Contact lookup by name + email. Used on webchat and SMS fallback when ANI fails. |
| `switch_language.py` | EN/ES language switching |
| `goodbye_and_hang_up.py` | Channel-aware bilingual goodbye + hangup |
| `escalate_call.py` | Voice/chat transfer to human |
| `handoff_to_agent.py` | Webchat Salesforce Live Agent handoff with pre_chat_fields |
| `trigger_outbound_call.py` | SMS/webchat→voice callback — calls the PolyAI outbound API directly via `requests` + `PolyOutboundToken` secret. No Railway. |
| `sms_consent_recorded.py` | Record SMS consent (yes/no) |
| `send_confirmation_sms.py` | Send SMS via template — sets `conv.state["sms_body"]` then `conv.send_sms_template()` |
| Domain-specific functions | schedule_X, reschedule_X, cancel_X, lookup_X, etc. |
| `amazon_connect.py` | **Optional** — only if the demo needs a voice handoff with a live-agent screen pop (see Integration Reference below) |

## Model Config (standard for all demos)

Set in the Studio UI, not via a manual `experimental_config.json` block — the platform moved to speech-native models, which fold ASR + language understanding into one model instead of the old separate ASR/VAD/EOT/barge-in stack.

**Voice → Model tab:**
1. Mode: **Speech-native mode** (not Modular mode)
2. Model: **Dialog-RSN-1** (PolyAI)
3. Studio shows a "Recommended settings — LLM: PolyAI Dialog-RSN-1 — Applied" banner once selected — the rest of the tuning is handled for you, nothing else to configure manually.

Don't hand-roll the old ASR/EOT/VAD/barge-in/audio-enhancement JSON block for new demos — it was the pre-RSN-1 way of approximating what Speech-native mode now does natively.

## Step-Level ASR Biasing (per flow step)

Beyond the project-wide model config above, each flow step YAML has its own `asr_biasing` block — booleans for what's expected at that specific point in the conversation (alphanumeric, numeric, relative_date, address, etc.), plus `custom_keywords` and a `dtmf_config` block. Set true only for what that step actually collects (e.g. `numeric` + `address` on an address-collection step, not on a greeting step) — biasing everything on every step dilutes the effect. See a real example: `agent_studio/poly-scs-us/AGENT-2R5UT25F/flows/installation_scheduling/steps/handle_installation.yaml`.

## rules.txt Template

Every demo's rules.txt should include these sections in this order:
1. **ROLE** — who the bot is, what it does
2. **GREETING** — webchat greeting text (voice uses $first_utterance)
3. **CHANNEL DETECTION** — $channel routing, outbound_type handling
4. **EMERGENCY CHECK** — voice only, clarify "stuck" ≠ emergency
5. **ANI RECOGNITION** — if $is_known_contact, greet by name
6. **VERIFICATION** — webchat: name+email→lookup_contact. Voice/SMS ANI: confirm address. No ANI: name+email fallback.
7. **INTENT ROUTING** — route to flow IMMEDIATELY. No detailed steps. "Do NOT handle details yourself."
8. **SMS CONFIRMATION** — voice/SMS only, NEVER webchat. Written format, not TTS.
9. **SMS TEXT FORMATTING** — explicit "use written format" instructions
10. **SMS → VOICE ESCALATION** — trigger_outbound_call pattern
11. **CALLBACK VS TRANSFER** — "call me back" (trigger_outbound_call) is not the same as "transfer me to a person" (escalate_call); don't let the LLM conflate them
12. **RETURNING CALLER OVERRIDE** — if this call is itself an outbound callback (metadata/sip_headers say so), resume the prior topic instead of restarting small talk — that context beats generic greeting/closing logic
13. **SALESFORCE CASE RULES** — don't read IDs to caller, confirmation number format
14. **MULTI-LANGUAGE** — switch_language function
15. **LOOP / JAILBREAK DETECTION** — 3-strikes: if the caller repeats the same unresolvable request 3 times, stop retrying the same response and transfer or end the call
16. **CLOSING** — ask if anything else, then goodbye
17. **HARD GUARDRAILS** — verification before actions, no demo references, emergency rules
18. **STYLE AND PERSONALITY** — warm, conversational, no repetition, contractions, use the caller's name at most once per call (not every turn)
19. **EMERGENCY RULES** — explicit criteria, "stuck" is service not emergency
20. **`<channel:voice>` TTS expansion** — numbers, addresses, abbreviations, state names
21. **CHAT STYLE** — line breaks, formatting, emoji inventory

## poly CLI Reference

- CLI: `~/agent-studio-skills/.venv/bin/poly`
- Auth: `POLY_ADK_KEY=$(cat ~/.poly/credentials.json | python3 -c "import sys,json; print(json.load(sys.stdin)['us-1'])")`
- Init: `poly init --region us-1 --account_id poly-scs-us --project_id PROJECT-XXX`
- Local projects: `~/poly-scs-us/PROJECT-XXX/` (init creates here, not in ~/agent_studio/)
- Validate: `poly validate`
- Push: `poly push` (creates a branch, merge via MCP)
- Step YAML filenames must match step names (snake_cased): step "Schedule Service" → `schedule_service.yaml`

## KB Topics via MCP

Can't write to main branch directly. Create a branch first:
```
mcp__polyai__create-branch → get branchId
mcp__polyai__create-knowledge-base-topic (on the branch) — repeat for each topic
mcp__polyai__merge-branch → deploys to sandbox
```

Always include Spanish example queries in every topic.

---

## Integration Reference (use only what this demo needs)

The patterns below are deep-dives for specific integrations. Read the one(s) relevant to this build — Salesforce, Zendesk, Amazon Connect handoff — skip the rest.

### Salesforce Integration

#### OAuth Pattern (sf_helpers.py)
```python
def _secret_str(conv, name):
    raw = conv.utils.get_secret(name)
    if isinstance(raw, dict):
        return str(raw.get("value", raw.get("password", raw)))
    return str(raw)

def bootstrap_sf_token(conv):
    payload = urllib.parse.urlencode({
        "grant_type": "password",
        "client_id": _secret_str(conv, "PolySalesforceClientID"),
        "client_secret": _secret_str(conv, "PolySalesforceClientSecret"),
        "username": _secret_str(conv, "Poly_SalesforceUsername"),
        "password": _secret_str(conv, "PolySalesforcePassword"),
    })
    resp = requests.post("https://login.salesforce.com/services/oauth2/token", ...)
    conv.state.sf_access_token = data.get("access_token")
    conv.state.sf_instance_url = data.get("instance_url")
```

#### ANI Contact Lookup
Generate phone variants (10-digit, 11-digit, +1, formatted) → SOQL `WHERE Phone = '...' OR MobilePhone = '...'` across all variants.

#### Case CRUD
- `sf_create(conv, "Case", body)` → returns (success, record_id, error)
- `sf_update(conv, "Case", record_id, body)` → returns (success, error)
- `sf_query(conv, soql)` → returns list of records
- Always set `ContactId` when creating Cases so they're linked to the customer

#### Webchat Handoff
```python
return {
    "utterance": "Connecting you now...",
    "handoff": {
        "reason": reason,
        "destination": "salesforce",
        "text_chat": {
            "salesforce_integration": {
                "organization_id": "00DHr0000011gte",
                "developer_name": "PolyAI_Webchat",
                "url": "https://polyai2-dev-ed.develop.my.salesforce-scrt.com",
                "pre_chat_fields": {...},
            },
        },
    },
}
```

### Zendesk Integration

**Sandbox URL:** `https://d3v-polyai.zendesk.com/api/v2`

**Secret:** check the project's secret vault (Studio → Secrets) for the Zendesk auth secret's actual name before hardcoding one — on the shared `poly-scs-us` sandbox it's currently named `Zendesk Basic Auth ( Jose)`, but that's account-specific and may not exist on yours. Whatever the name, it returns the full Authorization header value. Use directly. Do NOT add `"Basic "` prefix. Do NOT base64 encode.

```python
def create_zd_ticket(conv, ticket_title, ticket_details):
    import requests
    from agent_v3.deployment_service.agents.functions.builtin import secret_vault

    ZENDESK_AUTH_SECRET = "Zendesk Basic Auth ( Jose)"  # confirm/replace with your account's secret name

    headers = {
        "Authorization": secret_vault(ZENDESK_AUTH_SECRET),
        "Content-Type": "application/json",
    }
    response = requests.post(
        "https://d3v-polyai.zendesk.com/api/v2/tickets.json",
        headers=headers, data=payload, timeout=10,
    )
```

### Amazon Connect Voice Handoff (optional — screen pop to a live queue)

Use this instead of (or alongside) the Salesforce webchat handoff when a demo needs a **voice** escalation that pops guest/context onto a real Amazon Connect agent's screen. Confirmed working on the Hotel template, 2026-08-13.

**Two things both have to match a proven reference — the Python file below AND the Connect flow shape.** A Hotel build burned real time trying a per-project number-matching variant of the Python (looked more correct, wasn't) and a Connect flow cloned from a project with an extra `UpdateContactAttributes`/`Compare` branch that a confirmed-working reference doesn't have. Copy both exactly as given here.

#### The infra is shared — do not provision new AWS resources per demo
One Lambda, one DynamoDB table, one IAM role serve **every** demo (Rocket, Invisalign, IKEA, Hotel). Never create a new one:
- Secret: `PolyAIPoc-AC-Key` (`access_key` / `secret_key`)
- DynamoDB table: `PolyAIConnectContactAttributes-britishgasdemo`
- IAM role: `arn:aws:iam::886338079529:role/PolyAIRoleToReadAndUpdateContactAttributes-britishgasdemo` — **read/delete only, denies PutItem**. Only Connect's own Lambda ever writes the initial row.
- Region: `eu-west-2`

#### `functions/amazon_connect.py` — copy VERBATIM, do not "improve" it

**Do not deviate from this file.** A fancier-looking version with per-project number matching and normalization logic was tried on the Hotel build and burned two weeks across multiple wrong theories before a colleague confirmed, on a second live project, that this simpler original is the one that actually works. The key insight: the parking Lambda keys every project's row under **one shared, hardcoded PoC number**, never the project's own number.

```python
from _gen import *  # <AUTO GENERATED>

# Amazon Connect integration — do NOT "improve" this file. dynamo_key_override is a single
# shared, hardcoded PoC number, confirmed working across multiple independent demos — the
# parking Lambda keys EVERY project's row under this same value, never the project's own
# number, the caller's ANI, or the Connect claimed DID. Trying to be "smarter" here (per-project
# number matching, phone normalization, candidate lists) has been tried and burned real time —
# it looks more correct and is not. Only edit the two ALL-CAPS constants below.

import random

import boto3
import plog
from agent_v3.deployment_service.agents.functions.builtin import secret_vault


CONNECT_CONFIG = {
    "secret_name": "PolyAIPoc-AC-Key",
    "dynamo_table": "PolyAIConnectContactAttributes-britishgasdemo",
    "iam_role_arn": "arn:aws:iam::886338079529:role/PolyAIRoleToReadAndUpdateContactAttributes-britishgasdemo",
    "aws_region": "eu-west-2",
    "dynamo_key_override": "+447700171252",  # ALWAYS this value — do not change per project
}

# EDIT PER DEMO: this project's Studio conversation URL.
STUDIO_CONVERSATION_URL = "https://studio.us.poly.ai/poly-scs-us/PROJECT-XXXX/conversations/{conv_id}"


@func_description('Amazon Connect utilities — not callable directly')
def amazon_connect(conv: Conversation):
    return "Amazon Connect utility module."


def _is_chat(conv):
    return conv.id.startswith("CHAT_") or conv.id.startswith("AS_CHAT")


def _get_aws_session():
    secret = secret_vault(CONNECT_CONFIG["secret_name"])
    region = CONNECT_CONFIG["aws_region"]
    sts_client = boto3.client("sts", aws_access_key_id=secret["access_key"],
                               aws_secret_access_key=secret["secret_key"], region_name=region)
    assumed = sts_client.assume_role(RoleArn=CONNECT_CONFIG["iam_role_arn"],
                                      RoleSessionName=f"session-{random.randint(0, 9999)}")
    creds = assumed["Credentials"]
    return boto3.Session(aws_access_key_id=creds["AccessKeyId"],
                          aws_secret_access_key=creds["SecretAccessKey"],
                          aws_session_token=creds["SessionToken"], region_name=region)


def read_contact_attributes(conv):
    """Call unconditionally from start_function on every voice call."""
    if _is_chat(conv):
        plog.info("Chat conversation — skipping Amazon Connect read")
        return
    try:
        session = _get_aws_session()
        ddb = session.client("dynamodb")
        dynamo_key = CONNECT_CONFIG.get("dynamo_key_override") or conv.callee_number
        item = ddb.delete_item(TableName=CONNECT_CONFIG["dynamo_table"],
                                Key={"ExternalPolyNumber": {"S": dynamo_key}}, ReturnValues="ALL_OLD")
        if "Attributes" not in item:
            plog.warn("No contact attributes found in DynamoDB")
            return
        attrs = item["Attributes"]
        conv.state.initial_contact_id = attrs["ContactId"]["S"]
        conv.state.connect_instance_arn = attrs["InstanceARN"]["S"]
        session.client("connect").update_contact_attributes(
            InitialContactId=conv.state.initial_contact_id,
            InstanceId=conv.state.connect_instance_arn,
            Attributes={"PolyAI_CallLink": STUDIO_CONVERSATION_URL.format(conv_id=conv.id)})
        plog.info(f"Amazon Connect contact loaded: {conv.state.initial_contact_id}")
    except Exception as e:
        plog.warn("Failed to read Amazon Connect contact attributes", exception=e)


def call_link(conv) -> str:
    return STUDIO_CONVERSATION_URL.format(conv_id=conv.id)


def write_contact_attributes(conv, attributes) -> bool:
    """Call from the transfer function at handoff time."""
    if _is_chat(conv):
        return False
    if not getattr(conv.state, "initial_contact_id", None):
        plog.warn("No initial_contact_id — cannot write to Connect")
        return False
    try:
        session = _get_aws_session()
        session.client("connect").update_contact_attributes(
            InitialContactId=conv.state.initial_contact_id,
            InstanceId=conv.state.connect_instance_arn, Attributes=attributes)
        plog.info(f"Amazon Connect attributes updated: {list(attributes.keys())}")
        return True
    except Exception as e:
        plog.warn("Failed to write Amazon Connect attributes", exception=e)
        return False
```

#### Wiring it in
1. `start_function.py`: call `read_contact_attributes(conv)` unconditionally on the voice path — the function's own internal chat-id check handles skipping non-voice channels.
2. Transfer function (e.g. `escalate_call.py`), **must be reachable from an `advanced_step`** — `default_step`/`function_step` cannot transfer:
```python
from functions.amazon_connect import call_link, write_contact_attributes

write_contact_attributes(conv, {
    "polyAICallId": conv.id,
    "PolyAI_CallLink": call_link(conv),
    "PolyAI_Intent": "...",
    "PolyAI_Queue": "...",
    "PolyAI_Summary": "...",
})
return {"utterance": "Let me connect you with our team.", "hangup": True}
```
No `PolyAI_Action` needed — the flow shape below routes to queue unconditionally.

#### CRITICAL GOTCHA — the DynamoDB key is NOT project-specific, at all
`ExternalPolyNumber` is a single shared, hardcoded value (`+447700171252`) used by every project on this Lambda — not the caller's ANI, not the bot's own number, not the Connect claimed DID. Do not build per-project number-matching logic here.

#### Amazon Connect console setup — do ONCE, reuse across every future demo
The claimed number, queue, and routing profile are **not per-demo**:
1. Claim one phone number, create one queue + routing profile
2. Build one contact flow, this exact shape — no more, no fewer steps: `Recording → Invoke shared Lambda → Transfer to third party (this project's bot number) → Set target queue → Set screen-pop event hook (DefaultAgentUI → shared screenpop module) → Transfer to queue`, any error path → Disconnect. Do not add an attribute-defaults step or a Compare/branching step — a confirmed-working reference (IKEA) doesn't have either.
3. The screen-pop module (an Amazon Connect **View**, e.g. "PolyAI Screenpop") is shared/generic — reads `PolyAI_CallLink`/`PolyAI_Intent`/`PolyAI_Queue`/`PolyAI_Summary`, no per-demo changes needed

**New demo, same setup:** edit the one `ThirdPartyPhoneNumber` field in the existing flow's third-party-transfer block to the new demo's bot number, republish, update `connect_fronted_numbers` to match in `amazon_connect.py`. No new flow, no new AWS resources.

Note: a Connect **instance admin** login (queues, routing profiles, claiming numbers) and full **AWS IAM console** access (Lambda, DynamoDB) can be two completely separate logins even on the same AWS account — don't assume one implies the other when troubleshooting.

---

## Output

A deployed demo, scoped to what was actually asked for:
- **Agent Studio project** (KB, functions, flows, SMS templates, realtime config) — the core deliverable, always built
- **Mock data** — in-project (default) or Railway-backed, per Step 3
- **Landing page + narrative** — only if a website walkthrough was requested (Step 4); most demos don't need one
- Working channels: webchat, inbound voice, 2-way SMS, outbound voice (whichever were in scope)
- Salesforce integration (cases, contacts, webchat handoff), if scoped
- ANI recognition with name+email fallback
- Cross-channel context (one customer, one case, all channels)
- Demo script for the user
