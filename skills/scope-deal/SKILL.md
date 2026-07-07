---
name: scope-deal
description: "Turn SC discovery notes into a complete PolyAI scope from one Markdown source of truth: a client-facing call flow (HTML → A3 PDF), a scoping document (.docx, doubles as deployment handoff), and a populated PS Level-of-Effort calculator (Google Sheet). Trigger with /scope-deal, 'scope this deal', or 'build the scope for [client]'."
---

# Scope a Deal

You turn raw SC discovery notes into three consistent deliverables generated from **one Markdown scope file** (the single source of truth). Capture once → generate the call flow, the scoping doc, and the PS calculator. They stay in sync because they come from the same source.

## The pipeline

```
Discovery notes  →  MD scope file (source of truth)  →  { Call Flow PDF, Scoping .docx, PS Calculator sheet }
```

## Step 0 — Gather inputs

Open with:
> "Paste your discovery notes (call transcript, notes, any customer docs). I'll build the scope file, then generate the call flow, scoping doc, and PS calculator."

Then confirm only what's missing from the notes:
1. **Client name** + **deployment type** (New Business / Expansion)
2. **Discovery notes** (pasted)
3. **Total annual call volume** (drives the PS calculator token math)
4. The **use-case list** you extracted, each with a **status** (In Scope / Scoping / Discussed / Excluded) — get the SC to confirm before generating
5. **Where to save outputs** (default: current folder)

Never invent integrations, entities, or metrics. If something wasn't discussed, mark it as an Open Item, not a fact.

## Step 1 — Build the MD scope file

Write the MD per `reference/md-schema.md`. Apply the **locked conventions** in that file (they are non-negotiable):
- API **return values live in the API entry, never as entities** (entities = collected from the caller only).
- **Capture depth scales with status** (In Scope/Scoping = full detail; Discussed = name + description only).
- **Scoping = sized** (has % + seconds, counts toward effort); **Discussed = parked** (counts toward neither).
- Use the **status taxonomies** exactly (use-case, API, priority) — see the schema file.
- **PS calculator = one-time PS fee only. ARR is owned by Salesforce** — never compute or restate ARR.
- **No invented success targets** — capture metric + current baseline only where discussed.
- **Escalation destination is out of scope pre-sales** — leave as "existing routing."

Save the MD to the output location, show it to the SC, and get confirmation before generating outputs. This is the one human checkpoint.

## Step 2 — Call flow (HTML → PDF)

Generate the client-facing call flow following the conventions in `~/.claude/skills/call-flow-design/SKILL.md` (color-coded box system, decision diamonds, API callouts with **explicit Input/Returns**, three-column scope section). Populate it from the MD. Then render to A3 PDF:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu \
  --print-to-pdf="<out>/<Client>_Call_Flow.pdf" --no-margins --print-to-pdf-no-header \
  --paper-width=16.54 --paper-height=11.69 "<out>/<Client>_Call_Flow.html"
```

## Step 3 — Scoping document (.docx)

Generate the scoping doc following the `docx` pattern in `~/.claude/skills/scoping-doc/references/docx-template.md` and the section structure in the scoping-doc skill (Background → How It Works Today / Current Deployment → Objectives → Use Cases → Integration Architecture with **Status** → Open Items → Success Measures). Populate from the MD. Requires Node + the `docx` package.

## Step 4 — PS Level-of-Effort calculator (Google Sheet)

Follow `reference/sheet-mapping.md` exactly: copy the master template, batch-write the Use Case Scope rows and LoE inputs from the MD, then verify the formulas computed. The sheet's counts (`COUNTUNIQUEIFS`) only work in native Google Sheets, which is why we copy the master rather than build a file. Load the required tools with ToolSearch first (drive copy, sheets mutating request, sheets get request).

## Step 5 — Report

Return: the MD path, the call-flow PDF path, the scoping .docx path, and the **live PS sheet link**. Summarize the computed LoE (person-weeks → days → PS $) and note any Open Items the SC still needs to close.

## Notes

- Outputs go wherever the SC chooses; do not assume a fixed folder.
- If a data source or tool is unavailable, generate what you can and tell the SC what's missing — never fabricate to fill a gap.
