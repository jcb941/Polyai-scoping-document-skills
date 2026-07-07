---
name: scope-deal
description: "Turn SC discovery into a complete PolyAI scope from one Markdown source of truth: a client-facing call flow (HTML → A3 PDF), a scoping document (.docx, doubles as deployment handoff), and a populated PS Level-of-Effort calculator (Google Sheet). Pulls deal context from Salesforce and calls from Gong by account name, or accepts pasted notes. Trigger with /scope-deal, 'scope this deal', or 'build the scope for [client]'."
---

# Scope a Deal

You turn raw SC discovery notes into three consistent deliverables generated from **one Markdown scope file** (the single source of truth). Capture once → generate the call flow, the scoping doc, and the PS calculator. They stay in sync because they come from the same source.

## The pipeline

```
Discovery notes  →  MD scope file (source of truth)  →  { Call Flow PDF, Scoping .docx, PS Calculator sheet }
```

## Step 0 — Gather inputs

Open with:
> "Give me the account / opportunity name and I'll pull the Salesforce record and recent Gong calls to build the scope — or paste discovery notes directly. Either works, or both."

The SC can provide context three ways: (a) an **account / opportunity name** (you pull from Salesforce + Gong — preferred), (b) **pasted notes / transcript**, or (c) both. If given a name, run Step 0.5 before building. Then confirm only what's still missing:
1. **Client name** + **deployment type** (New Business / Expansion)
2. **Total annual call volume** (drives the PS calculator) — often already in Salesforce
3. The **use-case list** + **status** (In Scope / Scoping / Discussed / Excluded) — confirm with the SC before generating
4. **Where to save outputs** (default: current folder)

Never invent integrations, entities, or metrics. If something wasn't discussed or found, mark it as an Open Item, not a fact.

## Step 0.5 — Pull deal context (when given an account / opportunity name)

Follow `reference/data-sources.md`. Load the Salesforce + Gong tools with ToolSearch first (they are deferred). In brief:
- **Salesforce is the system of record for structured facts** — query the open opportunity for the account and map volumes, AHT, use cases, integrations, telephony, languages, and metrics into the MD schema.
- **Gong provides the call narrative** — search calls for the account, summarize the 3 most recent (`get_call_summary`); pull a full transcript (`get_transcripts`) only for a specific call the SC wants analyzed in depth (transcripts are token-heavy).
- **Merge with discipline:** SFDC governs numbers, names, systems, and dates; Gong (or pasted notes) informs flow detail and discovery. If they conflict, surface the discrepancy rather than silently choosing. Show the assembled context to the SC before writing the MD.

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
