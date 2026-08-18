# PolyAI Scoping Document Skills

Claude Code plugin for producing scoping and demo deliverables in PolyAI Solutions Consulting. Distributed as the **`scoping-suite`** plugin via the **`polyai-scoping`** marketplace.

## Install

```bash
claude plugin marketplace add jcb941/Polyai-scoping-document-skills
```
```bash
/plugin install scoping-suite@polyai-scoping
```

Update later (after new versions are pushed):
```bash
claude plugin marketplace update polyai-scoping
```

Installed via the plugin, skills are **namespaced** — e.g. `/scoping-suite:scope-deal`.

## Skills

| Skill | Command | What it produces |
|-------|---------|-----------------|
| **Scope a Deal** | `/scoping-suite:scope-deal` | **Start here for scoping.** One Markdown source of truth → a client call flow (HTML→PDF), a scoping doc (.docx, doubles as deployment handoff), and a populated PS Level-of-Effort calculator (Google Sheet). Pulls deal context from **Salesforce + Gong** by account name, or accepts pasted notes. |
| **Build Demo** | `/scoping-suite:build-demo` | Self-contained prospect demo pipeline — Agent Studio voice/webchat agent with mock data, optionally a Railway-hosted landing + narrative website. Opens by explaining what it can build and asking which channels/integrations you want before starting. **Start here for demos.** |
| **Demo Narrative** | `/scoping-suite:demo-narrative` | Dark-themed narrative page as an add-on to any existing demo site |
| **Call Flow Design** | `/scoping-suite:call-flow-design` | Customer-facing HTML call flow (landscape A3 PDF) with color-coded boxes, decision diamonds, API callouts, and scope columns |
| **Use Case Summary** | `/scoping-suite:use-case-summary` | Annex-style use case summary with table, integrations, deployment approach |
| **Scoping Doc** | `/scoping-suite:scoping-doc` | Formal Word/.docx scoping document — new deployments and expansions |

## Requirements

- **Claude Code** with this plugin installed.
- **A write-enabled Google Sheets / Drive MCP connected** — required for `scope-deal`'s PS-calculator step (it copies the master calculator and writes the Use Case Scope rows).
  - Check / connect it with **`/mcp`** in Claude Code: it lists connected MCP servers and their auth status — authenticate the Google server if prompted. If it isn't listed at all, it needs adding to your MCP config (ask whoever manages the team's Claude Code setup).
  - If this MCP isn't connected, `scope-deal` still copies the master calculator and hands you the rows to paste manually — it won't fail.
- **Salesforce + Gong MCP connected** — for `scope-deal`'s auto-pull of deal context. Without them, paste notes instead.
- **Local Node** + the `docx` npm package (scoping .docx) and **Google Chrome** (call-flow PDF rendering).
- **A personal Railway account** — only if you want `build-demo` to also stand up a landing/narrative website. Railway isn't shared team infra; the skill will walk you through one-time signup + CLI setup if you don't have it. Skip it entirely for voice/webchat-only demos.

## Repository structure

```
Polyai-scoping-document-skills/
├── .claude-plugin/
│   ├── marketplace.json      # marketplace: polyai-scoping
│   └── plugin.json           # plugin: scoping-suite
├── skills/
│   ├── scope-deal/
│   │   ├── SKILL.md
│   │   └── reference/        # md-schema, sheet-mapping, data-sources
│   ├── build-demo/SKILL.md
│   ├── demo-narrative/SKILL.md
│   ├── call-flow-design/SKILL.md
│   ├── use-case-summary/SKILL.md
│   └── scoping-doc/SKILL.md + references/
├── CHANGELOG.md
└── README.md
```

## Contributing / updating

To ship an update: edit the `SKILL.md` (or reference files), **bump `version` in both `.claude-plugin/plugin.json` and the marketplace entry**, add a `CHANGELOG.md` line, and push to `main`. Teammates pull it with `claude plugin marketplace update polyai-scoping`.
