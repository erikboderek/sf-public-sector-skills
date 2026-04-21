# Agent skills setup (layered install)

This **Salesforce Developer Experience (Salesforce DX)** repository does **not** vendor [Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills). Install the upstream bundle on your machine, then add this program’s small add-on skills pack.

**Two folder layouts:** In the **full PSS DX template**, skills live under **`.cursor/skills/`** and agents under **`.cursor/agents/`**. If you cloned only **[erikboderek/sf-public-sector-skills](https://github.com/erikboderek/sf-public-sector-skills)**, use **`skills/`** and **`agents/`** at the repository root instead (same `sf-pss-*` folder names). Optional **`docs/`** on that repo holds ADRs and guides when published from the template.

## Step 1 — Upstream sf-skills (always first)

- **Skills only** (Codex, Gemini CLI, OpenCode, Cursor agents, etc.):

  ```bash
  npx skills add Jaganpro/sf-skills
  ```

- **Full Claude Code experience** (skills + hooks + **Language Server Protocol (LSP)** + agents):

  ```bash
  curl -sSL https://raw.githubusercontent.com/Jaganpro/sf-skills/main/tools/install.sh | bash
  ```

- **Windows / continuous integration (CI) / no bash**:

  ```bash
  curl -sSL https://raw.githubusercontent.com/Jaganpro/sf-skills/main/tools/install.py | python3
  ```

Updates: `npx skills update` or `python3 ~/.claude/sf-skills-install.py --update` (after full install).

OmniStudio **Industries Common Core** skills (for example `sf-industry-commoncore-omniscript`) are included in that upstream repo—no separate install for those unless you maintain a fork.

## Step 2 — Public Sector Solutions (PSS) add-on (this template)

In the **DX template**, project-specific skills and agents follow the **Cursor layout** under **`.cursor/skills/`** and **`.cursor/agents/`** (one folder per skill with `SKILL.md`, plus agent markdown next to other industry packs).

An optional **standalone Agent Skills** install is published as **[erikboderek/sf-public-sector-skills](https://github.com/erikboderek/sf-public-sector-skills)** (`https://github.com/erikboderek/sf-public-sector-skills.git`). That repo uses the `npx skills` layout (`skills/` at the repository root). The template’s **`.cursor/README.md`** and **`.cursor/LICENSE`** are published as that repo’s root **README** and **LICENSE**. **`docs/`** is included when you publish with **`scripts/publish-skills-orphan.sh`** (not with `git subtree split --prefix=.cursor` alone). **Keep the remote in sync** with `.cursor/` and `docs/` here if you maintain both.

**Publishing:** In this template, a Git remote such as **`skills-origin`** points at that URL; see the repository root **[`README.md`](../README.md)** (“Publishing the skills pack to GitHub”) for **`git subtree split`** vs **`scripts/publish-skills-orphan.sh`** vs rewriting **this** repo with **`git filter-repo`**.

Install from GitHub **after** step 1:

```bash
npx skills add erikboderek/sf-public-sector-skills
```

**Using this DX repo only (no `npx skills add`):** from the repository root, copy into Claude’s skill directory or symlink:

```bash
cp -R .cursor/skills/sf-pss-project-conventions ~/.claude/skills/
cp -R .cursor/skills/sf-pss-data-dev ~/.claude/skills/
cp .cursor/agents/sf-pss-architect.md ~/.claude/agents/
```

## What gets installed where

- Upstream installs to the standard Agent Skills / Claude layout (for example `~/.claude/skills/sf-*` on Claude Code).
- **In the DX template**, the add-on content lives under `.cursor/skills/` (for example `sf-pss-project-conventions`, `sf-pss-data-dev`); see the [repository root `README.md`](../README.md) for the skill/agent index. **In the standalone skills repo**, the same folders are under `skills/`.
- **Agents** live under `.cursor/agents/sf-pss-architect.md` in the template (`agents/sf-pss-architect.md` in the standalone pack). They are not registered by `npx skills add` — copy them to `~/.claude/agents/` when using Claude Code, or follow your editor’s agent configuration.

## Optional: hooks registry

Upstream’s hook dispatcher and `skills-registry.json` apply to **upstream** skills. Custom `sf-pss-*` skills are discovered from `SKILL.md`; full hook parity with sf-skills would require a fork of upstream—usually unnecessary if `SKILL.md` documents triggers and delegation clearly.
