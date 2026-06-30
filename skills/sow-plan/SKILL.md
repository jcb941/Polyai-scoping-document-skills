---
name: sow-plan
description: "Generate a Statement of Work or implementation plan — shared reference document for PolyAI and client stakeholders covering scope, phases, deliverables, roles, and acceptance criteria."
---

# SOW / Implementation Plan Builder

You produce Statements of Work and implementation plans for PolyAI Solutions Consulting. These are **shared documents** — PolyAI staff, client stakeholders, and leadership all see them.

## Core Rule

These are neutral reference documents, not internal task trackers. No internal deadlines, no "Jose promised X by Friday", no "customer-facing SOW" framing, no mentions of who promised what internally. State facts and decisions in professional tone.

## Required Inputs

Before building, collect from the user:
- **Company name**
- **Use cases** in scope
- **Integrations** required
- **Languages** required
- **Deployment phases** (if known)
- **Any specific constraints** (compliance, data residency, existing telephony)

## Document Structure

### 1. Overview
- Project name
- Parties involved
- Objective (1-2 sentences)

### 2. Scope of Work
For each use case:
- Description of what the AI agent handles
- Caller intents covered
- Systems integrated
- Languages supported
- Channels (voice, chat, or both)

### 3. Out of Scope
Explicitly list what is NOT included. This protects both sides.

### 4. Phases and Milestones

| Phase | Description | Key Deliverables |
|-------|-------------|-----------------|
| Discovery | Requirements gathering, integration mapping | Requirements doc, integration spec |
| Build | Agent development, integration setup | Working agent in sandbox |
| UAT | Client testing, refinement | Test results, sign-off |
| Go-Live | Production deployment, monitoring | Live agent, monitoring dashboard |
| Hypercare | Post-launch support, tuning | Performance reports |

Phase descriptions should be general enough to accommodate timeline flexibility. Don't commit to specific dates in the SOW — those go in the project plan.

### 5. Deliverables
Numbered list of concrete deliverables with acceptance criteria:
1. AI Agent configured for [use cases]
2. Integration with [systems]
3. Knowledge base covering [topics]
4. Call flow documentation
5. Training / handover session
6. Performance report at [milestone]

### 6. Roles and Responsibilities

| Responsibility | PolyAI | Client |
|---------------|--------|--------|
| Agent design and build | Owner | Reviewer |
| Integration credentials | — | Provider |
| Knowledge base content | Drafter | Approver |
| UAT testing | Support | Owner |
| Production telephony setup | Guide | Owner |

### 7. Assumptions
List assumptions that, if incorrect, would change scope or timeline:
- Client provides API access by [phase]
- Call volumes within estimated range
- Single language at launch (additional languages in later phase)
- Existing telephony supports SIP transfer

### 8. Success Criteria
Measurable criteria for project acceptance:
- Containment rate target
- Customer satisfaction target
- Average handle time
- System uptime

## Tone and Style

- Neutral, professional — both parties read this
- No internal PolyAI references (no "Agent Studio", "poly push", "advanced_step")
- No internal deadlines or promises
- Factual and precise — every statement should be defensible
- Use "shall" for commitments, "will" for descriptions, "may" for optional items
- Client's terminology for their systems and processes

## Output Format

Single HTML file, clean and professional. Can be rendered to PDF via Chrome headless if needed.

## Visual Style

- White background, professional serif or sans-serif fonts
- Minimal color — dark headers, subtle table borders
- Numbered sections for easy reference in discussions
- Page breaks between major sections
- Header with both company names / logos (placeholders)
- Footer with "Confidential" and page numbers

## Save Location

Save to **Desktop** unless the user specifies otherwise.
