# Salesforce Public Sector Solutions — Agent skills

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-0F766E)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Small add-on skill library for **Salesforce Public Sector Solutions (PSS)**: native data model guidance, Salesforce Developer Experience (DX) template conventions, and a technical architect agent. Use alongside **[forcedotcom/sf-skills](https://github.com/forcedotcom/sf-skills)** for Apex, Flow, Lightning Web Components (LWC), OmniStudio, deploy, and testing.

All skills use the **`sf-pss-*`** prefix so IDs stay distinct from upstream skills.

**Start here:** [Available skills](#available-skills) · [Agents](#agents) · [Documentation](#documentation) · [Installation](#installation)

---

## Available skills

| Skill | Purpose |
|-------|---------|
| [sf-pss-project-conventions](skills/sf-pss-project-conventions/) | PSS DX template layout (`force-app/`, `config/`, `manifest/`, `docs/`) and when to delegate to upstream sf-skills. |
| [sf-pss-data-dev](skills/sf-pss-data-dev/) | Native PSS objects, API names, Data Model Gallery links, native-vs-custom decisions, Discovery Framework overlap notes. Org-verified against Winter '27 (v68.0). |

Each folder contains a `SKILL.md` plus optional `references/` (see [sf-pss-data-dev/references/](skills/sf-pss-data-dev/references/)).

---

## Documentation

| Path | Purpose |
|------|---------|
| `docs/adr-pss.md` | Single PSS ADR (objects, DevOps, licenses, Action Plan XML note). |
| `docs/plan-pss-phased.md` | Phased delivery plan. |
| `docs/guide-agent-skills.md` | Layered install guide. |

---

## Agents

| Agent | Purpose |
|-------|---------|
| [sf-pss-architect](agents/sf-pss-architect.md) | PSS technical architect persona; loads `sf-pss-project-conventions` and `sf-pss-data-dev`; delegates OmniStudio mechanics to `omnistudio-omniscript-generate`, `omnistudio-datamapper-generate`, and related sf-skills. |

`npx skills add` does **not** register agents. For Claude Code, copy into your agents directory:

```bash
cp agents/sf-pss-architect.md ~/.claude/agents/
```

---

## Installation

Requires [Node.js 18+](https://nodejs.org/) for `npx`.

Install the base Salesforce skill library **first**, then this pack:

```bash
npx skills add forcedotcom/sf-skills
npx skills add erikboderek/sf-public-sector-skills
```

Install a single skill:

```bash
npx skills add erikboderek/sf-public-sector-skills --skill sf-pss-data-dev
```

Manual install — copy each folder under `skills/` into your agent's skill directory (e.g. `~/.claude/skills/`), preserving the folder name so it matches the `name` field in each `SKILL.md`.

---

## Repository layout

```
.
├── LICENSE
├── README.md
├── agents/
│   └── sf-pss-architect.md
├── docs/
│   ├── adr-pss.md
│   ├── guide-agent-skills.md
│   └── plan-pss-phased.md
└── skills/
    ├── sf-pss-project-conventions/
    │   └── SKILL.md
    └── sf-pss-data-dev/
        ├── SKILL.md
        └── references/
            └── sf-pss-core-objects.md
```

---

## Contributing

Open issues or PRs for skill text, references, or agent prompt updates. Keep guidance aligned with native PSS standard objects unless an explicit exception is documented.
