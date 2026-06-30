---
name: demo-site
description: "Build a prospect demo website — Railway-hosted landing page + narrative HTML + Express mock API + Agent Studio project structure. Produces the full deliverable stack for an SC demo."
---

# Demo Site Builder

You build complete prospect demo websites for PolyAI Solutions Consulting. Each demo is a self-contained Express app deployed to Railway, paired with an Agent Studio project.

## What You Produce

Two deliverables deployed together on Railway:

1. **Landing page** (`public/index.html`) — Brand-themed visual prop. Dark/premium aesthetic customized to the prospect's brand colors and industry.
2. **Demo narrative** (`public/narrative.html`) — Dark-themed scenario walkthrough with customer profile, journey timeline, and scenario cards with sample transcripts. Linked from the landing page nav.

Plus the Express server with mock APIs and the Agent Studio project structure.

## Required Inputs

Before building, collect from the user:
- **Company name** and industry
- **Use cases** (e.g., lead qualification, scheduling, account management)
- **Brand colors** (or let you research them)
- **Key services/products** the bot needs to know about
- **Any specific customer data** (ZIP codes, service areas, pricing)

If the user doesn't provide these, ask before building. Never guess at the business domain.

## Landing Page Pattern

Single-file HTML with inline CSS. Structure:

```
- Fixed nav bar (brand name + section links + "Demo Narrative" link)
- Hero section (full viewport, brand gradient background, animated accents)
- Use case cards (grid layout, one card per use case)
- Features/services section
- Footer (brand + "Powered by PolyAI")
```

Style rules:
- Google Fonts appropriate to the brand
- Brand colors as primary accent (gradient on hero, card borders, headings)
- Dark background (#0a0a0a or similar), light text
- Cards with subtle border glow on hover, translateY(-4px) lift
- Subtle CSS animations (pulse, glow, parallax) — no JS libraries
- Mobile responsive grid (minmax 300px)
- No stock images — use CSS gradients and icons/emojis for visual interest

## Narrative Page Pattern

Dark-themed page with:
- Customer profile card (name, phone, address, account details)
- Journey timeline (visual flow of what happens on the call)
- Scenario cards for each use case with:
  - Scenario title and context
  - Full sample transcript (Agent / Caller turns)
  - Key moments highlighted (ANI match, verification, booking confirmation)
- "Back to Landing Page" nav link

## Express Server (server.js)

```javascript
const express = require('express');
const app = express();
app.use(express.static('public'));
app.use(express.json());

// Mock API endpoints
app.get('/api/customer/:phone', (req, res) => { /* ANI lookup */ });
app.get('/api/service-area/:zip', (req, res) => { /* ZIP coverage */ });
app.post('/api/service-request', (req, res) => { /* Action endpoint */ });
app.get('/healthz', (req, res) => res.json({ status: 'ok' }));

app.listen(process.env.PORT || 3000);
```

API rules:
- `GET /api/customer/:phone` — normalize phone (strip non-digits, drop leading "1" → 10-digit key). Return full record for KNOWN demo numbers only, `{"match": false}` for unknown. NEVER return a record for all numbers.
- Load ZIP/service data from `data/` JSON files
- Keep responses fast — the agent calls these during live calls

## Agent Studio Project Structure

```
functions/
  start_function.py      # ANI lookup + date/time context
  route_intent.py        # conv.goto_flow(intent)
  check_availability.py  # 2-3 time slots, filter by preference
  validate_zip.py        # Railway API call
  book_[action].py       # Mock locally (generate confirmation, pick specialist, send SMS)
  escalate_call.py       # "Let me connect you with a team member"
  goodbye_and_hang_up.py
```

Function rules:
- `start_function.py` sets `conv.state["today"]`, `conv.state["next_7_days"]`, does ANI lookup. Phone from `conv.caller_number` first, fallback `conv.sip_headers.get("From")`.
- Booking functions MOCK LOCALLY — generate confirmation numbers, pick from specialist pool, send SMS. No Railway API calls for bookings (timeouts kill the demo).
- All functions return context strings telling the LLM what happened.

## Conversation Design Rules (bake into step prompts)

- ANI match → greet by name, skip identity questions
- Always verify by ZIP before reading account data
- Always confirm address after ZIP
- One question per turn, never stack
- Context awareness — if caller mentioned details, don't re-ask
- Availability check before any booking (2-3 slots)
- Full verbal confirmation before executing
- SMS confirmation after every action
- Never reference being a demo — the bot IS the company's assistant
- Warm for new leads, empathetic for complaints/warranty

## Flow Architecture

- 1 advanced_step per flow. NEVER use default_step.
- Each step prompt contains the FULL conversation instructions.
- Functions are top-level in `functions/`, called via `{{fn:name}}`.
- Include TTS expansion rules in `<channel:voice>` tags.
- Include bilingual English + Spanish support.

## File Structure Output

```
{company}-demo/
  server.js
  package.json
  data/serviceable_zips.json
  public/
    index.html
    narrative.html
  .gitignore
  README.md
```

## Quality Bar

- Must replicate real call center interactions
- Deep domain knowledge — research the actual company
- Handle off-script gracefully
- Problem-to-service mapping (callers describe problems, not service names)
- Conversational, not transactional — one question at a time
