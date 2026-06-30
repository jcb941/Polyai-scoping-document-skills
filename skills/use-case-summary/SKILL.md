---
name: use-case-summary
description: "Generate a use case summary document (Annex-style) — structured overview of AI agent capabilities for a specific prospect, covering use cases, integrations, expected outcomes, and deployment approach."
---

# Use Case Summary Builder

You produce use case summary documents for PolyAI Solutions Consulting. These are shared with prospects and internal stakeholders to describe what the AI agent will do, how it integrates, and what outcomes to expect.

## Output Format

Single HTML file, professional and clean. Can be rendered to PDF if needed using Chrome headless (same as call flow design).

## Required Inputs

Before building, collect from the user:
- **Company name** and industry
- **Use cases** with brief descriptions
- **Current state** (how calls are handled today — IVR, human agents, outsourced)
- **Target integrations** (CRM, booking systems, telephony)
- **Expected call volumes** (if known)
- **Languages required**

## Document Structure

### 1. Executive Summary
2-3 sentences: what PolyAI will deliver, for whom, and the key business outcome.

### 2. Use Case Table
For each use case:

| Field | Content |
|-------|---------|
| Use Case Name | Clear, business-friendly name |
| Description | What the AI agent does in this flow |
| Caller Intent | Why someone calls about this |
| Key Actions | What the agent performs (verify identity, look up account, book appointment, etc.) |
| Integrations | Systems touched (API calls, CRM lookups, SMS) |
| Transfer Conditions | When and where to hand off to a human |
| Expected Containment | Estimated % handled without human transfer |

### 3. Integration Overview
Table of all systems the agent connects to:
- System name
- Integration type (API, webhook, SFTP, etc.)
- Purpose
- Data exchanged

### 4. Deployment Approach
- Phase 1 / Phase 2 / Phase 3 breakdown if applicable
- Languages supported
- Channels (voice, chat, both)
- Testing and go-live timeline indicators (without specific dates — those go in the SOW)

### 5. Expected Outcomes
- Call containment rate targets
- Average handle time reduction
- Customer satisfaction expectations
- Cost savings indicators

## Tone and Style

- Professional, neutral — this is a shared document
- No internal PolyAI jargon (no "advanced_step", "Agent Studio", "conv.state")
- No "demo" language — write as if this is going to production
- Specific enough to be useful, general enough that minor scope changes don't invalidate it
- Use the prospect's terminology for their services/products
- Numbers and targets should be presented as ranges or estimates, not commitments (unless confirmed)

## Visual Style

- White/light background, professional fonts
- PolyAI brand accent color (#6C63FF or current brand purple) for headers and borders
- Clean tables with alternating row shading
- Section dividers between major blocks
- Company logo placeholder at top

## Save Location

Save to **Desktop** unless the user specifies otherwise.
