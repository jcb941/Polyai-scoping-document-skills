# PolyAI Scoping Document Skills

Claude Code skills for producing scoping and demo deliverables in PolyAI Solutions Consulting.

## Skills

| Skill | Command | What it produces |
|-------|---------|-----------------|
| **Build Demo** | `/build-demo` | Full prospect demo pipeline — landing page + mock API + voice agent + dark-themed demo narrative page. Orchestrates `/prospect-demo` (from PolyAI-LDN) then adds the narrative. **Start here for demos.** |
| **Demo Narrative** | `/demo-narrative` | Dark-themed narrative page (narrative.html) as a standalone add-on to any existing demo site |
| **Call Flow Design** | `/call-flow-design` | Customer-facing HTML call flow document (landscape A3 PDF) with color-coded boxes, decision diamonds, API callouts, and scope columns |
| **Use Case Summary** | `/use-case-summary` | Annex-style use case summary with use case table, integrations, deployment approach, and expected outcomes |
| **Scoping Doc** | `/scoping-doc` | Formal Word/.docx solution scoping document — handles both new deployments and expansions. Covers use cases, integration architecture, phases, roles, open items, and future state |

## Setup

**Both repos are needed for the full demo pipeline:**

```bash
# This repo — scoping docs, call flows, demo narrative, build-demo orchestrator
claude skills add jcb941/Polyai-scoping-document-skills

# PolyAI-LDN repo — the prospect-demo landing page + mock API + voice agent BUILD-KIT
claude skills add PolyAI-LDN/oisin-nikola-store
```

## Usage

```
/build-demo              # Full demo pipeline (landing page + narrative) — start here
/demo-narrative           # Add narrative page to an existing demo site
/call-flow-design         # Customer-facing call flow document
/use-case-summary         # Annex-style use case overview
/scoping-doc              # Formal scoping document (Word/.docx)
```

Each skill will ask for the required inputs (company name, use cases, integrations, etc.) before generating the deliverable.

## Repository Structure

```
Polyai-scoping-document-skills/
├── skills/
│   ├── build-demo/
│   │   └── SKILL.md
│   ├── demo-narrative/
│   │   └── SKILL.md
│   ├── call-flow-design/
│   │   └── SKILL.md
│   ├── use-case-summary/
│   │   └── SKILL.md
│   └── scoping-doc/
│       ├── SKILL.md
│       └── references/
│           └── docx-template.md
└── README.md
```

## Contributing

To update a skill, edit the `SKILL.md` file in the relevant skill directory and push. Users get the update on next sync.
