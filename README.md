# PolyAI Scoping Document Skills

Claude Code skills for producing scoping and demo deliverables in PolyAI Solutions Consulting.

## Skills

| Skill | Command | What it produces |
|-------|---------|-----------------|
| **Demo Site** | `/demo-site` | Railway-hosted Express app with brand-themed landing page, demo narrative, mock APIs, and Agent Studio project structure |
| **Call Flow Design** | `/call-flow-design` | Customer-facing HTML call flow document (landscape A3 PDF) with color-coded boxes, decision diamonds, API callouts, and scope columns |
| **Use Case Summary** | `/use-case-summary` | Annex-style use case summary with use case table, integrations, deployment approach, and expected outcomes |
| **SOW / Plan** | `/sow-plan` | Statement of Work / implementation plan — shared reference doc with scope, phases, deliverables, roles, and acceptance criteria |

## Setup

Add this repo as a skill repo in Claude Code:

```bash
claude skills add jcb941/Polyai-scoping-document-skills
```

Or add it manually in your Claude Code settings.

## Usage

Once installed, invoke any skill by name:

```
/demo-site
/call-flow-design
/use-case-summary
/sow-plan
```

Each skill will ask for the required inputs (company name, use cases, integrations, etc.) before generating the deliverable.

## Repository Structure

```
Polyai-scoping-document-skills/
├── skills/
│   ├── demo-site/
│   │   └── SKILL.md
│   ├── call-flow-design/
│   │   └── SKILL.md
│   ├── use-case-summary/
│   │   └── SKILL.md
│   └── sow-plan/
│       └── SKILL.md
└── README.md
```

## Contributing

To update a skill, edit the `SKILL.md` file in the relevant skill directory and push. Users get the update on next sync.
