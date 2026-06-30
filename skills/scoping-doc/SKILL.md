---
name: scoping-doc
description: "Build a formal PolyAI solution scoping document (Word/.docx) for a new client deployment or an expansion of an existing one. Covers use cases, integration architecture, phases, roles, open items, and future state. Trigger on phrases like 'scoping doc,' 'scope this out,' 'write up the use cases for [client],' 'expansion scope,' or 'build a doc like we did for [other client].'"
---

# PolyAI Solution Scoping Document

This skill produces a formal Word document scoping a PolyAI voice (or chat) AI deployment. It handles both **net-new deployments** and **expansions** of existing ones. Structure, tone, and revision patterns are drawn from real client scoping work (Brittain Resorts & Hotels, Fogo de Chão, and others).

## Before you start

1. **Determine the mode.** Ask the user (if not obvious): is this a **new deployment** or an **expansion** of an existing one? This changes several sections.
2. **Gather inputs.** Meeting notes, call transcripts, prior scope docs, stakeholder names, the client's existing tech stack (telephony platform, PMS/CRM, booking/ordering engine, payment processor). If the user references "notes" or "the transcript" without pasting it, ask them to paste or upload it — don't invent details.
3. If the user says "use the same format as [other client]," the structure below covers it. If they want a deviation, confirm explicitly before building.

## Document structure

### New Deployment (all sections)

1. **Background** — One paragraph. Who the client is, their existing tech stack (telephony platform, PMS/CRM/booking engine, any third-party API), and a one-line framing of what this document covers and when it was scoped (e.g. "scoped from discovery and working sessions held in [month/year]").

2. **How It Works Today** — One paragraph describing the current manual/live-agent process this deployment will replace or augment. Be concrete: what agents do manually today, what compliance or operational pain points exist, what's NOT currently self-service.

3. **Objectives** — 3-5 bullets. Each bullet is a single outcome-oriented sentence written from the business's perspective ("Enable guests to...", "Allow callers to...", "Provide self-service options for..."). The last bullet is often a forward-looking "foundation for future phase" statement if applicable.

4. **Use Cases** — Numbered subsections (4.1, 4.2, ...), one per distinct use case. See "Use case writing rules" below. Each use case subsection should cover, in prose:
   - What triggers the use case
   - What the agent does, in present tense, agent-centric voice
   - What the resolution/handoff looks like
   - Any integration dependency, presented as a short table if there are multiple distinct API calls/fields, otherwise inline

5. **Integration Architecture** — Prose or short subsections per system being integrated (e.g. the client's booking API, SMS/messaging provider, telephony platform, payment processor). State what each integration does and what data flows through it. Note any future-phase integration paths separately and label them as future (not committed scope).

6. **Phases & Deliverables** — Table or subsections breaking the work into phases:
   - Phase 1: Discovery & requirements finalization
   - Phase 2: Agent build & integration setup
   - Phase 3: UAT & client testing
   - Phase 4: Go-live & monitoring
   - Phase 5: Hypercare & optimization
   Each phase lists its key deliverables and acceptance criteria. Don't commit to specific dates — those go in the project plan.

7. **Roles & Responsibilities** — Table with columns: Responsibility | PolyAI | Client. Cover: agent design/build, integration credentials, knowledge base content, UAT testing, telephony setup, ongoing tuning.

8. **Open Items** — Table with columns: Item | Description / Open Question | Owner / Needs Confirmation From. See "Open Items table conventions" below.

9. **Future State** (when applicable) — Short paragraph describing the next-phase vision (e.g. full AI-handled transaction once a current blocker is resolved, additional use cases, additional languages).

### Expansion (modified sections)

When scoping an expansion of an existing deployment, adjust as follows:

- **Section 1 (Background):** Add a sentence summarizing the existing deployment: what went live, when, and key metrics (containment rate, call volume).
- **Add Section 2: Current Deployment** — What the AI agent handles today, which integrations are live, current performance metrics (containment, AHT reduction, CSAT). This replaces "How It Works Today" for expansions.
- **Section 3 (Objectives):** Frame as incremental: "Extend self-service coverage to...", "Add [language/channel] support for...", "Reduce transfer rate for [use case] by..."
- **Section 4 (Use Cases):** Mark each use case as:
  - **New** — not currently handled by the agent
  - **Enhancement** — modification of an existing use case (state what changes)
  - **Migration** — moving from one system/integration to another
- **Section 5 (Integration Architecture):** Split into "Existing (no changes)" and "New / Modified" subsections.
- **Sections 6-9:** Same structure as new deployment.

## Use case writing rules (critical — most common revision request)

These conventions came directly from client feedback and should be applied by default:

- **Present tense, agent-centric voice.** "The agent confirms the reservation details and sends a checkout link via SMS." Not "The agent will confirm..." and not narrating the AI's internal reasoning.
- **No editorializing or internal commentary inside use case prose.** Don't write "this assumes Fuel's API supports X, which needs confirmation" inline. That caveat belongs in the Open Items table. The use case section should read like a clean summary — what the agent does, period.
- **Frame unconfirmed/desired behavior carefully.** If a flow depends on something not yet validated, describe it as the **desired flow**, not as settled fact, and add the open question to Open Items. Don't silently assert capabilities that haven't been confirmed.
- **Be specific about choice points.** If a caller has a choice (e.g. self-service cancellation vs. transfer to a live agent), state both paths plainly.
- **Static vs. dynamic details matter.** If something must be static (not dynamically generated per call), say so explicitly and flag the actual value as needing client confirmation in Open Items.
- **Match the Annex tone if the client has an existing SOW/Annex.** Concise, declarative, no hedging words like "should" or "we believe," no first-person PolyAI voice.
- **No internal PolyAI jargon.** No "advanced_step", "Agent Studio", "conv.state", "poly push". Write as the client would understand it.

## Open Items table conventions

- Every caveat, unconfirmed endpoint, assumption about data shape, or pending decision goes here — nothing stays buried in use case prose.
- Phrase each row as a clear question or pending confirmation. Bad: "Refund inbox unclear." Good: "Refund notification inbox must be a static address (not dynamic); client to confirm the address to use."
- Include an owner or "needs confirmation from" column whenever you know which party (client, partner/API vendor, PolyAI) needs to resolve it.

## Shared document rules

This is a shared document — PolyAI staff, client stakeholders, and leadership all see it.

- No internal deadlines or promises ("Jose promised X by Friday")
- No "customer-facing" framing — it IS customer-facing
- No first-person PolyAI voice ("we will build...") — use neutral third-person ("PolyAI configures...")
- State facts and decisions in professional tone
- Use "shall" for commitments, "will" for descriptions, "may" for optional items

## Output format

Generate a Word/.docx file. Follow the code pattern in `references/docx-template.md` for the exact styling (Arial font, heading hierarchy, bullet formatting, table styling).

### Workflow

1. Confirm inputs (stakeholders, tech stack, use cases, any transcript/notes). Ask only for what's missing.
2. Draft section by section in markdown first — get the structure and prose approved before generating the .docx, since revisions to wording are common and cheaper before formatting.
3. Once content is approved, generate the .docx using the template pattern in `references/docx-template.md`.
4. Save to `~/Desktop/[Client]_Scoping.docx`.
5. Expect at least one revision round focused on Use Cases and Open Items — apply the writing rules above proactively so revisions are minimal.
