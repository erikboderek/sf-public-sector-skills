---
name: sf-pss-project-conventions
description: >
  Public Sector Solutions (PSS) Salesforce Developer Experience (DX) template conventions: project layout, docs, and when to delegate
  to Jaganpro sf-skills (sf-apex, sf-flow, sf-lwc, sf-industry-commoncore-*).
  TRIGGER when: work happens in this repo’s force-app, config, or docs and team
  standards matter. DO NOT TRIGGER when: generic Apex/LWC/OmniStudio-only tasks with
  no PSS-specific context (use sf-apex, sf-lwc, sf-industry-commoncore-* directly).
license: MIT
metadata:
  version: "1.0.5"
---

# sf-pss-project-conventions

Use this skill when implementing or reviewing work **in the PSS program Developer Experience (DX) template** so agent behavior matches local structure and documentation.

## Principles

1. **Prefer upstream skills for platform work** — Use installed `sf-*` skills from [Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills) for Apex, Flow, **Lightning Web Components (LWC)**, **Salesforce Object Query Language (SOQL)**, OmniStudio Common Core, Agentforce, deploy, etc.
2. **Use this skill for repo-specific rules** — Follow `config/`, `docs/`, and `manifest/` conventions described in this project’s docs.
3. **Single source of truth** — Metadata under `force-app/main/default/`; high-level status and **Architecture Decision Records (ADRs)** under `docs/`.
4. **Applicant object choice (PSS)** — When a **business** applies for a **business license**, **`BusinessLicenseApplication`** is the native filing object; **`IndividualApplication`** is for **person** applicants (benefits, individual program intake, etc.). See **`sf-pss-data-dev`** and **`references/sf-pss-core-objects.md`**.

## Delegation map

| Topic | Delegate to |
|-------|-------------|
| Public Sector Solutions native objects (licensing, permitting, benefits, grants, etc.) | `sf-pss-data-dev` (this add-on pack) |
| OmniStudio authoring (OmniScript, FlexCard, **Integration Procedure (IP)**, Data Mapper) | `sf-industry-commoncore-omniscript`, `sf-industry-commoncore-flexcard`, `sf-industry-commoncore-integration-procedure`, `sf-industry-commoncore-datamapper` |
| Cross-Omni dependency / namespace analysis | `sf-industry-commoncore-omnistudio-analyze` |
| Agentforce metadata, prompts, **generative artificial intelligence (GenAI)** | `sf-ai-agentforce`, `sf-ai-agentscript` |
| Generic Salesforce code quality | `sf-apex`, `sf-testing`, `sf-debug` |

## Local references

Read project docs as needed (paths relative to repo root):

- `docs/adr-pss.md`, `docs/plan-pss-phased.md` — PSS ADR and phased plan
- `docs/status-agent-build.md` — Clerk Assist files on disk; compare to `manifest/package.xml` for declared-only metadata
- `docs/guide-agent-skills.md` — layered skill install
- `config/project-scratch-def.json` — scratch org shape
- `manifest/package.xml` — deployment manifest

## Output expectations

When answering, tie recommendations to this repository’s folders and docs when relevant; otherwise defer to the appropriate `sf-*` skill.
