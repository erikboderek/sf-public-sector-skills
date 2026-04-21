# Salesforce Public Sector Solutions — Agent skills

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-0F766E)](https://agentskills.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Small add-on skill library for **Salesforce Public Sector Solutions (PSS)**: native data model guidance, Salesforce Developer Experience (DX) template conventions, and a technical architect agent. Use alongside **[Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills)** for Apex, Flow, Lightning Web Components (LWC), OmniStudio Common Core, deploy, and testing.

All skills use the **`sf-pss-*`** prefix so IDs stay distinct from upstream `sf-*` skills.

**Start here:** [Available skills](#available-skills) · [Agents](#agents) · [Documentation](#documentation) · [Installation](#installation)

---

## Available skills

| Skill | Purpose |
|-------|---------|
| [sf-pss-project-conventions](skills/sf-pss-project-conventions/) | PSS DX template layout (`force-app/`, `config/`, `manifest/`, `docs/`) and when to delegate to upstream `sf-*` skills. |
| [sf-pss-data-dev](skills/sf-pss-data-dev/) | Native PSS objects, API names, Data Model Gallery links, native-vs-custom decisions, Discovery Framework overlap notes. |

Each folder contains a `SKILL.md` plus optional `references/` (see [sf-pss-data-dev/references/](skills/sf-pss-data-dev/references/)).

---

## Documentation

When **`docs/`** is published alongside this pack, open these paths at the **repository root** (same names as in the parent DX template):

| Path | Purpose |
|------|---------|
| `docs/adr-pss.md` | Single PSS ADR (objects, DevOps, licenses, Action Plan XML note). |
| `docs/plan-pss-phased.md` | Phased delivery plan. |
| `docs/status-agent-build.md` | Clerk Assist: README stubs only in this repo. |
| `docs/guide-agent-skills.md` | Layered install guide. |

If you are browsing this file **inside** the parent template as **`.cursor/README.md`**, the same files live one level up: **`../docs/…`**.

---

## Agents

| Agent | Purpose |
|-------|---------|
| [sf-pss-architect](agents/sf-pss-architect.md) | PSS technical architect persona; loads `sf-pss-project-conventions` and `sf-pss-data-dev`; delegates OmniStudio mechanics to `sf-industry-commoncore-*`. |

`npx skills add` does **not** register agents. For Claude Code, copy into your agents directory (see your tooling’s docs), for example:

```bash
cp agents/sf-pss-architect.md ~/.claude/agents/
```

---

## Installation

Requires [Node.js 18+](https://nodejs.org/) for `npx`.

### Recommended (layered)

Install the base Salesforce skill library **first**, then this pack:

```bash
npx skills add Jaganpro/sf-skills
npx skills add erikboderek/sf-public-sector-skills
```

Install a single skill from this repo:

```bash
npx skills add erikboderek/sf-public-sector-skills --skill sf-pss-data-dev
```

List skills before installing:

```bash
npx skills add erikboderek/sf-public-sector-skills --list
```

### Without `npx` (manual)

Copy each folder under `skills/` into your agent’s skill directory (for example `~/.claude/skills/`), preserving the folder name so it matches the `name` field in each `SKILL.md`.

---

## Cursor, VS Code, and Agentforce Vibes

### Cursor

Clone or open a project that contains this **`skills/`** tree (for example the parent PSS DX template, which keeps the same folders under **`.cursor/skills/`**). In [Cursor](https://cursor.com), open that workspace so **`.cursor/skills/*/SKILL.md`** and **`.cursor/agents/`** are on disk; Cursor uses them as **project-local** agent guidance—tune AI / rules in Cursor **Settings** as needed.

### Visual Studio Code

In [Visual Studio Code](https://code.visualstudio.com/), open the repo and browse **`skills/*/SKILL.md`** as **documentation** while you work. For **skill-based AI** inside VS Code, install and use **Agentforce Vibes** (next section) or use **Cursor** with the same files under **`.cursor/`** in the DX template.

### Agentforce Vibes

The Salesforce guide [Skills in Agentforce Vibes](https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/skills.html) describes the **`SKILL.md`** format, progressive loading (`references/`, `assets/`, `scripts/`), and workspace layout:

| What | Where |
|------|--------|
| Project skills (recommended) | **`.a4drules/skills/`** at the root of your Salesforce / DX workspace |
| Enable the feature | VS Code **Settings** → **Agentforce Vibes** → **Enable Skills** (on by default) |

**To use this pack with Agentforce Vibes:** copy each folder under **`skills/`** (for example `sf-pss-data-dev`, `sf-pss-project-conventions`) into **`<your-project>/.a4drules/skills/`**. Folder names must match the **`name`** in each `SKILL.md` frontmatter. Then open the **Skills** panel in Agentforce Vibes to verify they load.

`agents/sf-pss-architect.md` is for **Cursor / Claude-style** agents, not the Vibes skill loader—reuse the prose manually in Vibes if you want the same persona.

---

## Repository layout

```
skills/
├── sf-pss-project-conventions/
│   └── SKILL.md
└── sf-pss-data-dev/
    ├── SKILL.md
    └── references/
        └── sf-pss-core-objects.md
agents/
└── sf-pss-architect.md
docs/
├── adr-pss.md
├── plan-pss-phased.md
├── status-agent-build.md
└── guide-agent-skills.md
```

`docs/` appears when the parent template publishes with **`scripts/publish-skills-orphan.sh`**; it is **not** part of **`git subtree split --prefix=.cursor`**.

---

## Full DX template

This GitHub repository is the **installable skills + agents bundle**, and may include **`docs/`** when published from the parent repo. If you also want the full **Salesforce Developer Experience (Salesforce DX)** project (metadata, scratch def, manifests), use the **parent Public Sector Solutions (PSS) template repository** that publishes this content (same `sf-pss-*` skills under its `.cursor/` folder).

---

## Contributing

Open issues or PRs against this repository for skill text, references, or agent prompt updates. Keep guidance aligned with **native PSS** standard objects unless an explicit exception is documented.
