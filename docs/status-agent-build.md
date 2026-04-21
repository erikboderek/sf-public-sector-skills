# Agent build status — Clerk Assist (what this repo contains)

This document describes **Clerk Assist / Agentforce-related files that actually exist** in the repository. It does **not** list Apex classes, flows, or GenAI metadata unless they appear under `force-app/main/default/`.

## Present on disk

| Path | Role |
|------|------|
| [force-app/main/default/aiAgents/README.md](../force-app/main/default/aiAgents/README.md) | How to map requested agent filenames to **Employee Agent** / Setup-authored metadata |
| [force-app/main/default/agentActions/README.md](../force-app/main/default/agentActions/README.md) | How agent actions relate to **GenAiFunction** when you add those functions |

**Not in this repo:** There are **no** `classes/*Clerk*`, `genAiFunctions/**`, `genAiPlugins/**`, `flows/SLG_Agent_*`, or Clerk-specific permission set XML files under `force-app/main/default/`. [manifest/package.xml](../manifest/package.xml) may still name GenAI and Apex members for a **future** deploy—only README stubs exist under `force-app/` until you add sources or trim the manifest.

## When you add Clerk Assist metadata (generic notes)

- Prefer **Setup-authored** Employee Agent and **GenAiPlugin** / **GenAiFunction** patterns described in Salesforce Help and upstream **`sf-ai-agentforce`** / **`sf-deploy`** skills; retrieve into `force-app/` when ready.
- Avoid Apex locals named `json` in the same scope as `JSON.serializePretty` (Apex treats `JSON` and `json` as the same identifier)—use `System.JSON.serializePretty` and a distinct variable name.
- **GenAiFunction** invocation target types and Metadata API shapes change by release—validate against your org’s API version.
- **Permission set licenses** for Agentforce and object access for PSS types are **manual** in full PSS orgs.

## Licensing

See **License Requirements** in [docs/adr-pss.md](adr-pss.md) for PSS, OmniStudio, Experience Cloud, CDS, Agentforce, and related add-ons (validate SKUs with your account team).
