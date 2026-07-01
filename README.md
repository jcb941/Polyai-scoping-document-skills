# PolyAI Scoping Document Skills

Claude Code skills for producing scoping and demo deliverables in PolyAI Solutions Consulting.

## Skills

| Skill | Command | What it produces |
|-------|---------|-----------------|
| **Call Flow Design** | `/call-flow-design` | Customer-facing HTML call flow document (landscape A3 PDF) with color-coded boxes, decision diamonds, API callouts, and scope columns |
| **Use Case Summary** | `/use-case-summary` | Annex-style use case summary with use case table, integrations, deployment approach, and expected outcomes |
| **Scoping Doc** | `/scoping-doc` | Formal Word/.docx solution scoping document — handles both new deployments and expansions. Covers use cases, integration architecture, phases, roles, open items, and future state |
| **Demo Narrative** | `/demo-narrative` | Dark-themed narrative page (narrative.html) for an existing demo site — customer profile, journey timeline, scenario cards with transcripts and key moment callouts. Add-on to `/prospect-demo` |

## Demo Sites

For prospect demo websites (Railway-hosted landing page + mock API + voice agent), use the **prospect-demo** skill from the [oisin-nikola-store](https://github.com/PolyAI-LDN/oisin-nikola-store) repo:

```bash
claude skills add PolyAI-LDN/oisin-nikola-store
```

Then invoke with `/prospect-demo`. That skill includes the full BUILD-KIT, PolyAI brand CSS, case studies, site architecture, and voice agent playbook.

## Setup

Add this repo as a skill repo in Claude Code:

```bash
claude skills add jcb941/Polyai-scoping-document-skills
```

Or add it manually in your Claude Code settings.

## Usage

Once installed, invoke any skill by name:

```
/call-flow-design
/use-case-summary
/scoping-doc
/demo-narrative
```

Each skill will ask for the required inputs (company name, use cases, integrations, etc.) before generating the deliverable.

## Repository Structure

```
Polyai-scoping-document-skills/
├── skills/
│   ├── call-flow-design/
│   │   └── SKILL.md
│   ├── use-case-summary/
│   │   └── SKILL.md
│   ├── scoping-doc/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── docx-template.md
│   └── demo-narrative/
│       └── SKILL.md
└── README.md
```

## Contributing

To update a skill, edit the `SKILL.md` file in the relevant skill directory and push. Users get the update on next sync.
