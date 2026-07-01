---
name: demo-narrative
description: "Generate a dark-themed demo narrative page (narrative.html) for an existing PolyAI prospect demo site. Adds a visual walkthrough with customer profile, journey timeline, and scenario cards with transcripts. Use after /prospect-demo has built the landing page, or to add a narrative to any Railway-hosted demo site."
---

# Demo Narrative Builder

This skill generates a `narrative.html` page for an existing PolyAI prospect demo site. The narrative is a dark-themed, cinematic walkthrough that follows one customer through multiple scenarios — showing the agent's capabilities through realistic transcripts and key moment callouts.

**This is an add-on skill.** It assumes a landing page already exists (built via `/prospect-demo` from the oisin-nikola-store BUILD-KIT or manually). It generates the narrative page and adds a nav link to the existing landing page.

## Required Inputs

1. **The existing demo site** — either a Railway URL or a local project path. Read the landing page to extract the prospect name, use cases, brand colors, and customer data from the mock API.
2. **Customer persona** (or derive from the mock API) — name, account details, the data that makes the narrative feel real and specific.

If the user provides just a Railway URL, fetch the landing page and the mock API (`/api/customer/+15551234567` or similar) to get everything needed.

## Visual Spec (Non-Negotiable)

The narrative page uses a **dark theme** — this is what differentiates it from the landing page and gives it its cinematic feel. Reference: the Lenovo narrative at `illustrious-rebirth-production.up.railway.app/narrative.html`.

### CSS Variables (dark theme)

```css
:root {
  --bg: #060609;
  --surface: #0e0e14;
  --surface-2: #16161f;
  --border: rgba(255,255,255,0.06);
  --accent: /* derive from the landing page's primary accent */;
  --accent-dim: /* accent at 12% opacity */;
  --accent-glow: /* accent at 25% opacity */;
  --text: #f0f0f5;
  --text-dim: rgba(255,255,255,0.55);
  --text-muted: rgba(255,255,255,0.35);
  --radius: 16px;
  --radius-sm: 10px;
}
```

Use the same font family as the landing page. Body background is `var(--bg)`, all text is light-on-dark.

### Page Structure (exact order)

#### 1. Top Navigation (fixed)
- Logo (prospect or PolyAI) left, nav links right
- Links: Overview, Story, Profile, and one link per scenario (e.g., "Repair", "FAQ", "Warranty", "Upsell")
- Frosted glass effect: `background: rgba(6,6,9,0.85); backdrop-filter: blur(20px);`
- Include a "← Back to Landing Page" link to `/`

#### 2. Hero Section
- Full viewport height, centered text
- Radial glow behind the headline using the accent color
- Badge pill: `"DEMO NARRATIVE"` with pulsing dot animation
- Large headline with gradient text on the key phrase
- Subtitle: "Follow [Customer Name] through [N] real scenarios — from [first use case] to [last use case]"
- Stats bar: 4 stats summarizing the demo (e.g., "4 Capabilities", "10 FAQ Topics", "58 Days to Warranty Exp.", "EN/ES Bilingual")

#### 3. Capabilities Overview
- Section label with accent dash: `"── OVERVIEW"`
- Title: "What [Agent Name] handles"
- Arc visual: horizontal node-and-connector chain showing the flow (e.g., 🔍 Lookup → 📋 Status → 📱 SMS → 🛡 Verify)
- 4 pillar cards below: one per use case, with icon, title, and one-line description
- Cards: `background: var(--surface); border: 1px solid var(--border); border-radius: 16px;`
- Hover: `border-color: rgba(accent, 0.2); transform: translateY(-2px);`

#### 4. The Story Section
- Title: "The Story" or "One call, four outcomes"
- 4 chapter blocks, each with a number, title, and 2-3 sentence summary
- Bold text for key outcomes
- This is the narrative arc — set up what the reader is about to see

#### 5. Customer Profile Card
- Large card with dark surface background
- Customer name prominently displayed
- Structured data in labeled groups:
  - **Contact**: phone, email
  - **Account/Device/Policy details**: the fields from the mock API, specific and real-looking
  - **Status**: current situation (repair status, claim status, reservation, etc.)
  - **Key detail**: the thing that creates narrative tension (warranty expiring, payment overdue, etc.)
- Make it scannable — labels in small muted text, values in regular text

#### 6. Scenario Cards (one per use case — this is the core of the narrative)

Each scenario follows this structure:

```
┌─────────────────────────────────────────────────────┐
│  SCENARIO [N]                                        │
│  ┌─────────────────────┬───────────────────────────┐ │
│  │   Visual/Icon       │  Title                    │ │
│  │   (large emoji      │  Context paragraph        │ │
│  │    or illustration) │  Tags: [Recognition]      │ │
│  │                     │        [Capability]        │ │
│  │                     │        [Revenue]           │ │
│  └─────────────────────┴───────────────────────────┘ │
│  ┌──────────┬──────────┬──────────┐                  │
│  │ Detail 1 │ Detail 2 │ Detail 3 │  (key moments)   │
│  │ label    │ label    │ label    │                  │
│  │ value    │ value    │ value    │                  │
│  └──────────┴──────────┴──────────┘                  │
│                                                      │
│  TRANSCRIPT                                          │
│  Agent: "Welcome back, [Name]..."                    │
│  Caller: "Hi, I'm calling about..."                  │
│  Agent: "Let me look that up..."                     │
│  [System: look_up_order("ORD-12345") → found]        │
│  Agent: "I can see your order..."                    │
│                                                      │
│  ⚡ KEY MOMENT                                       │
│  Description of what made this interaction special    │
└─────────────────────────────────────────────────────┘
```

**Scenario card styling:**
- Top section: 2-column grid (visual left, info right)
- Tags: colored pill badges — green for recognition, blue for capability, yellow for revenue
- Bottom section: 3-column detail strip with border-top separator
- Transcript: monospace-ish styling, Agent turns in accent color, Caller turns in regular text
- System calls shown as inline code: `` `look_up_claim("CLM-1921")` ``
- Key moment callout: accent-bordered box with ⚡ icon and bold label

**Transcript writing rules:**
- Present tense, natural conversation — not scripted
- Agent uses the customer's first name after lookup
- One question per turn
- Show the system call inline when the agent does a lookup or action
- Include the API response summary (e.g., "→ found: repair in progress, ETA June 15")
- 6-10 turns per scenario — enough to show capability, not so long it drags
- End each scenario with a clear resolution

#### 7. Footer
- Simple: "[Prospect] × PolyAI — Voice AI Demo Narrative"
- Copyright line

### After Generating

1. Save `narrative.html` to the project's `public/` directory
2. Add a "Demo Narrative" link to the landing page's navigation bar
3. If the project is already deployed to Railway, remind the user to `railway up` to push the update

## Quality Bar

- The narrative must feel **cinematic, not clinical** — dark theme, smooth animations, gradient accents
- Customer data must be **specific and believable** — real-looking order numbers, dates, phone numbers, addresses
- Transcripts must read like **real conversations** — warm, natural, one question at a time
- Key moments should highlight **why this matters** — not just what happened, but the business value (saved a transfer, captured revenue, resolved in 90 seconds)
- Every scenario should demonstrate a **different capability** — don't repeat the same pattern
- The page should work as a **standalone artifact** someone can scroll through without the landing page
