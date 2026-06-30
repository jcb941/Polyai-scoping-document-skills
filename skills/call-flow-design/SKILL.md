---
name: call-flow-design
description: "Generate a customer-facing call flow design document — HTML rendered to landscape A3 PDF with color-coded boxes, decision diamonds, API callouts, and scope columns. Presented to clients for sign-off."
---

# Call Flow Design Builder

You produce customer-facing call flow design documents for PolyAI Solutions Consulting. These documents are presented to clients for approval and sign-off — they must look professional and contain zero internal implementation references.

## Output Format

Single HTML file → rendered to landscape A3 PDF via Chrome headless.

**PDF rendering command:**
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu \
  --print-to-pdf="output.pdf" \
  --no-margins --print-to-pdf-no-header \
  --paper-width=16.54 --paper-height=11.69 \
  "input.html"
```

After generating the HTML, render the PDF automatically and confirm it looks correct.

## Required Inputs

Before building, collect from the user:
- **Company name** and use case(s)
- **Call flows** (e.g., reservations, scheduling, FAQ, transfers)
- **Integrations** (CRM, ticketing, POS, telephony)
- **Transfer destinations** (where calls hand off to humans)
- **In-scope vs out-of-scope** items

If the user doesn't provide these, ask. Never invent integration details.

## Visual Style (Color-Coded Box System)

| Element | Color | Use |
|---------|-------|-----|
| Start/entry points | Green (#e8f5e9 border, #2e7d32 text) | Call start, flow entry |
| Bot speech | Blue (#e3f2fd border, #1565c0 text) | What the agent says |
| Transfers/handoffs | Yellow (#fff8e1 border, #f57f17 text) | Human handoff with screen pop |
| API/integration callouts | Orange dashed (#fff3e0, dashed border) | System integration points |
| Internal processing | Gray (#f5f5f5 border, #616161 text) | Behind-the-scenes logic |
| Decision diamonds | Rotated square | Routing decisions |

## Page Break Rules

- `page-break-inside: avoid` on individual boxes, API callouts, columns, scope boxes
- Do NOT use `page-break-inside: avoid` on flow-section containers (too tall → empty pages)
- Do NOT use forced `page-break-before` divs — let content flow naturally
- `page-break-after: avoid` on h2 so headers stay with content

## Document Structure

1. **Header** — Company name, "Call Flow Design", date, version
2. **Call Start** → variant resolution callout → start function → dynamic greeting
3. **Intent Routing** → decision diamond with all paths
4. **Transfer Section** — cross-cutting, available at any point, with integration callout (e.g., Zendesk/Salesforce) and SIP/transfer details
5. **Main Flows** — steps with speech examples, API callouts at integration points, decision diamonds for branching
6. **FAQ/Knowledge Base** — list all topics covered
7. **Scope Section** — three columns:
   - In-Scope (what the bot handles)
   - Integrations (systems + endpoints)
   - Out-of-Scope (what transfers to humans)

## Customer-Facing Rules (Non-Negotiable)

- No "demo", "simulated", or "mock" language anywhere
- No code references (conv.state, {{ft:...}}, function names) — except in API callout boxes where technical detail is expected
- Keep references general ("all locations") not specific ("Hollywood, Chicago")
- Packages/items described generically ("available packages based on location menu")
- TBD items are fine (e.g., "Twilio number TBD")
- Professional tone throughout — this goes to the client for sign-off

## Speech Examples in Flow Boxes

Write natural, conversational speech examples:
```
"Welcome to [Company]! I can help you with [use cases]. 
What can I do for you today?"
```

Not robotic:
```
"Please state your intent from the following options: 
reservations, hours, or general inquiry."
```

## Save Location

Save the HTML and PDF to **Desktop**, not inside any Agent Studio project directory. These are standalone deliverables.

## CSS Template Base

Use a clean print-friendly stylesheet:
- White background, professional fonts (Inter, Roboto, or similar)
- Max-width container (1400px) for readability
- Box shadows for depth, rounded corners
- Clear visual hierarchy with consistent spacing
- Diamond shapes via CSS transforms for decision points
- Dashed borders for API callout boxes
- Three-column grid for the scope section
