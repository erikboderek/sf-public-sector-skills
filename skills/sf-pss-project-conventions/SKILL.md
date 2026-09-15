---
name: sf-pss-project-conventions
description: "PSS Salesforce DX template conventions: project layout, docs, and delegation to afv-library skills. TRIGGER: work in force-app/config/docs where team standards matter. SKIP: generic Apex/LWC/OmniStudio-only tasks without PSS context."
metadata:
  version: "1.0"
  domains:
    - "Government"
  minApiVersion: "66.0"
  accessCheck: "Requires Salesforce Public Sector Solutions add-on license."
  relatedSkills:
    - "platform-apex-generate"
    - "experience-lwc-generate"
    - "automation-flow-generate"
    - "omnistudio-omniscript-generate"
    - "omnistudio-flexcard-generate"
    - "omnistudio-integration-procedure-generate"
    - "omnistudio-dependencies-analyze"
    - "platform-apex-test-run"
    - "platform-apex-logs-debug"
---

# sf-pss-project-conventions

Use this skill when implementing or reviewing work **in the PSS program Developer Experience (DX) template** so agent behavior matches local structure and documentation.

## Principles

1. **Prefer upstream skills for platform work** — Use installed skills from [forcedotcom/afv-library](https://github.com/forcedotcom/afv-library) for Apex (`platform-apex-generate`), Flow (`automation-flow-generate`), **Lightning Web Components (LWC)** (`experience-lwc-generate`), OmniStudio, Agentforce, deploy, etc.
2. **Use this skill for repo-specific rules** — Follow `config/`, `docs/`, and `manifest/` conventions described in this project’s docs.
3. **Single source of truth** — Metadata under `force-app/main/default/`; high-level status and **Architecture Decision Records (ADRs)** under `docs/`.
4. **Applicant object choice (PSS)** — When a **business** applies for a **business license**, **`BusinessLicenseApplication`** is the native filing object; **`IndividualApplication`** is for **person** applicants (benefits, individual program intake, etc.). See **`sf-pss-data-dev`** and **`references/sf-pss-core-objects.md`**.

## Delegation map

| Topic | Delegate to |
|-------|-------------|
| Public Sector Solutions native objects (licensing, permitting, benefits, grants, etc.) | `sf-pss-data-dev` (this add-on pack) |
| OmniStudio authoring (OmniScript, FlexCard, **Integration Procedure (IP)**, Data Mapper) | `omnistudio-omniscript-generate`, `omnistudio-flexcard-generate`, `omnistudio-integration-procedure-generate`, `omnistudio-datamapper-generate` |
| Cross-Omni dependency / namespace analysis | `omnistudio-dependencies-analyze` |
| Agentforce metadata, prompts, **generative artificial intelligence (GenAI)** | `sf-ai-agentforce`, `sf-ai-agentscript` |
| Generic Salesforce code quality | `platform-apex-generate`, `platform-apex-test-run`, `platform-apex-logs-debug` |

## Local references

Read project docs as needed (paths relative to repo root):

- `docs/adr-pss.md`, `docs/plan-pss-phased.md` — PSS ADR and phased plan
- `docs/status-agent-build.md` — Clerk Assist files on disk; compare to `manifest/package.xml` for declared-only metadata
- `docs/guide-agent-skills.md` — layered skill install
- `config/project-scratch-def.json` — scratch org shape
- `manifest/package.xml` — deployment manifest

## Output expectations

When answering, tie recommendations to this repository’s folders and docs when relevant; otherwise defer to the appropriate afv-library skill.
