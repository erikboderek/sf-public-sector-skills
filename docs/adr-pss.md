# Architecture Decision Record (ADR): State and Local Government Public Sector Solutions (PSS)

## Context

**State and local government** agencies deliver regulated services—**benefits**, **professional and business licenses**, **permits**, **grants**, and other programs—by having residents and businesses **complete forms**, supply evidence, and move a filing through **review, verification, and decision** on a defined timeline. The same pattern applies whether the service is cash aid, a contractor license, a building permit, or a specialized regulatory program.

Agency programs differ in labels and statutes; this ADR describes **Salesforce object patterns** only. Any long-form “packet” your jurisdiction uses maps to the same native shapes—**applications**, **regulatory transactions**, **form fields**, and **documents**—without requiring a particular industry example.

**Problem we are solving:** Provide a **native Public Sector Solutions (PSS)** and **Salesforce Industries common layer** architecture for citizen and clerk channels—guided intake, document tracking, regulatory transaction lifecycle, and operational checklists—**without introducing custom Salesforce objects** for data and processes that PSS and the common layer already model (applications, regulatory transactions, party identity, document checklists, action plans).

Non-goals for this ADR: selecting a specific payment gateway, defining exact picklist values for every `StatusCode`, or specifying non-Salesforce **legacy line-of-business or billing system** integration contracts (those belong in separate integration ADRs).

---

## Domain Coverage

| Domain | PSS / Industries objects | Rationale |
|--------|--------------------------|-----------|
| **Citizen & business applications** | `IndividualApplication`, **`BusinessLicenseApplication`** (business licensing), `ApplicationType`, `ApplicationForm`, `ApplicationFormSection`, `ApplicationFormField` | Service requests are application-shaped workflows with typed sections and field-level capture. Use **`IndividualApplication`** when the **applicant is a person** (`ApplicantId` → `Individual`). Use **`BusinessLicenseApplication`** for **business license** apply or renew: the object uses **`AccountId`** → **`Account`** (B2B **Account** or **Person Account**) and **`ApplicantId`** / **`PrimaryOwnerId`** → **`Contact`** (including **PersonContact** for Person Accounts)—see [PSS `BusinessLicenseApplication`](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/sforce_api_objects_businesslicenseapplication.htm). Do not use **`IndividualApplication`** as the primary filing object for that scenario. PSS application objects replace bespoke custom form objects. |
| **Regulatory transactions** | `RegulatoryTxn`, `RegulatoryAuthorizationType`, `RegulatoryTxnParty` | Each distinct service path (new request, renewal, amendment, appeal, cross-program handoff) is a **regulatory authorization type** with parties in defined roles and a status lifecycle. |
| **Party & identity** | `Individual`, `Party`, `PartyProfile` (optional), `ContactPointAddress`, `ContactPointPhone`, `ContactPointEmail` | Applicants, dependents, sponsors, and partner organizations map to Individuals and/or Accounts with structured addresses and channels—no duplicate “person” custom objects. |
| **Document management** | `DocumentChecklistItem`, `ContentDocument` / `ContentVersion` / `ContentDocumentLink` | Required evidence (ID, income statements, inspection certificates, signed attestations, third-party letters) is tracked as checklist items with file evidence on the platform. |
| **Workflow & procedure** | `ActionPlanTemplate`, `ActionPlan`, `RecordAction` | Program deadlines, role handoffs, inspections, and **RecordAction** steps (including third-party or field verification when applicable) are modeled as templated plans and tasks—not a custom task framework. |
| **Service & channel logging** | `InteractionSummary`, `Case` (optional) | Counter visits, phone clarifications, and third-party verifier contacts are recorded for audit and service management where `Case` is already in use. |
| **Regulatory citations** | `RegulatoryCode` (Industries common) | Optional spine for statute / rule references used in training, reporting, and cross-program compliance. |
| **Field / special verification** | `Visit`, `ServiceAppointment` (optional); or clerk-completed `RecordAction` + `InteractionSummary` | On-site or mobile verification may use visit/scheduling patterns where the org licenses and configures them; otherwise attestation stays on the application/regulatory record. |

**Explicitly out of scope as first-class “registry” tables for every program-specific thing:** PSS does not provide one standard object per possible regulated asset, location, or case subtype for all agencies. **Asset** (standard) may be used only if the org explicitly standardizes on it for physical-asset tracking; this ADR assumes **transaction- and application-centric** modeling unless a follow-on ADR adopts `Asset`.

---

## Decisions

### Decision: Anchor regulated service work on RegulatoryTxn and the correct application parent

- **Options considered:** (1) Custom parent object with child custom objects; (2) `Case`-only tracking; (3) `RegulatoryTxn` + only `IndividualApplication` for every filing; (4) Standard `Opportunity` or `Order`; (5) `RegulatoryTxn` + **`IndividualApplication`** or **`BusinessLicenseApplication`** + **`ApplicationFormField`** as appropriate.
- **Decision:** Use **`RegulatoryTxn`** as the regulatory spine (with **`RegulatoryAuthorizationType`** per scenario). Use **`IndividualApplication`** as the filing container when the **applicant is a person**—benefits, many personal permits, and other **`Individual`**-centric programs—extended via **`ApplicationForm`**, **`ApplicationFormSection`**, and **`ApplicationFormField`** where that model applies. Use **`BusinessLicenseApplication`** as the filing container for **business license** apply or renew: model **`AccountId`** (business **Account** or **Person Account**) and the human submitter / primary owner via **`ApplicantId`** / **`PrimaryOwnerId`** (**`Contact`**, including **PersonContact** where Person Accounts are used), plus **`LicensedEntity`** / **`BusinessLicense`** relationships per Object Manager; **`IndividualApplication`** must not stand in for that primary business filing (authorized people may still appear on **`RegulatoryTxnParty`** or related roles per Object Manager).
- **Rationale:** Aligns with PSS regulatory and application patterns, supports polymorphic party roles via **`RegulatoryTxnParty`**, and avoids parallel data models that would diverge from Industries upgrades and shared components.
- **Trade-offs:** More metadata design up front (authorization types, form definitions, action plan templates, and which application object each program uses). Less flexibility than unconstrained custom objects unless teams invest in configuration and integration discipline.

### Decision: Model all form-specific data as ApplicationFormField (or document evidence), not parallel custom “spreadsheet” objects

- **Options considered:** (1) Custom fields on a custom program-specific object per form; (2) Long-text “blob” on `RegulatoryTxn`; (3) Structured **`ApplicationFormField`** / sections per agency form packet.
- **Decision:** Map agency **field-level** capture to **`ApplicationFormField`** records (with appropriate **`ApplicationFormSection`** groupings). Use **`DocumentChecklistItem`** + files for signed PDFs and certificates.
- **Rationale:** Preserves native extensibility, supports OmniScript-driven intake, and keeps a clear separation between **transaction lifecycle** (`RegulatoryTxn.StatusCode`) and **asserted facts** (form fields).
- **Trade-offs:** Reporting across “same logical field” requires consistent **`DeveloperName`** on form field definitions; bulk analytics may need Data Cloud or careful field naming conventions.

### Decision: Citizen intake via OmniScript; clerk completion via console + Flow/Screen Flow

- **Options considered:** (1) Custom **Lightning Web Components (LWC)** portal; (2) Screen Flows only; (3) OmniScript + Experience Cloud for citizens, standard app + Flow for clerks; (4) Email/mail-only (no self-service).
- **Decision:** **OmniScript** on **Experience Cloud** for citizen and external partner submission; **Salesforce console** (or workspace) on **`IndividualApplication`**, **`BusinessLicenseApplication`**, or **`RegulatoryTxn`** (per program) for clerks, with **`RecordAction`** optionally launching **Screen Flow** or **internal OmniScript**.
- **Rationale:** Matches Industries guidance for citizen-facing PSS experiences and reuses the same form metadata where possible.
- **Trade-offs:** OmniStudio skill and deployment pipeline required; dual-channel testing (citizen vs clerk) for every change.

### Decision: Operational deadlines and branching via ActionPlanTemplate, not custom workflow engines

- **Options considered:** (1) Custom “deadline” object; (2) Process Builder-only; (3) **`ActionPlanTemplate`** + **`RecordAction`** per checklist step; (4) External BPM only.
- **Decision:** One **`ActionPlanTemplate`** per major service path your program defines (for example: initial application, renewal, amendment, appeal), with **`RecordAction`** rows for document collection, verification, fees, and decision/issuance.
- **Rationale:** Native Industries action framework is supported, reportable, and aligns with inspection- and permit-style patterns elsewhere in PSS.
- **Trade-offs:** Complex conditional logic (for example, branch-specific inspections) may require multiple templates or dynamic plan creation via automation—needs design standards.

### Decision: Compliant Data Sharing only where program scope requires it

- **Options considered:** (1) **Compliant Data Sharing (CDS)** for all program records; (2) CDS only when Individual participates in CDS-governed programs or highly restricted attributes are centralized; (3) No CDS; sharing rules only.
- **Decision:** Apply **Compliant Data Sharing (CDS)** (`DataUsePurpose`, `AuthorizationFormConsent`, `IndividualShare`, `PartyConsent`) when the same **`Individual`** is in scope for CDS-governed benefits/health-style programs **or** when the org mandates purpose-based access for sensitive program **personally identifiable information (PII)** beyond baseline **organization-wide defaults (OWD)**; otherwise use strict **profile** and **field-level security (FLS)**, **Experience Cloud** authentication, and audit via **`InteractionSummary`**.
- **Rationale:** Program data is often sensitive (statutory privacy, sector-specific disclosure rules, and analogous controls in each domain); CDS is an additional license and operational burden and should not be assumed org-wide without legal/product alignment.
- **Trade-offs:** Mixed model (some users CDS, some not) increases architecture complexity if not documented clearly.

---

## Delivery, DevOps, and platform practice

Cross-cutting practices after review of **sf-pss-data-dev**, **sf-industry-commoncore-*** (OmniScript, FlexCard, Integration Procedure, Data Mapper, Callable Apex), **sf-flow**, **sf-apex**, **sf-permissions**, **sf-deploy**, and **sf-testing**.

### Decision: OmniStudio build order (orchestration)

- Implementation must follow Industries practice: **analyze dependencies → Data Mapper (DataRaptor) → Integration Procedure → OmniScript → FlexCard** (see **sf-industry-commoncore-omniscript**, **sf-industry-commoncore-integration-procedure**, **sf-industry-commoncore-datamapper**).
- OmniScripts consume **Integration Procedure (IP)** and **DataRaptor (DR)** bundles; FlexCards may launch OmniScripts or show summary state—shipping scripts before server-side bundles causes rework and broken previews.
- **Skill coverage:** `sf-industry-commoncore-omniscript`, `sf-industry-commoncore-integration-procedure`, `sf-industry-commoncore-datamapper`, `sf-industry-commoncore-flexcard`, `sf-industry-commoncore-omnistudio-analyze`.
- **Phased plan impact:** Phase 1 must land **stub or real** DataRaptors + IPs before the citizen OmniScript is marked complete; Phase 2 FlexCards depend on stable JSON/context from Phase 1 save path.

### Decision: Deployment and activation gates

- **DevOps** ordering: metadata foundation → **permission sets** → **Apex** (with tests) → **Flows** (often deploy as Draft, then activate after validation)—per **sf-deploy**.
- Reduces failures related to **field-level security (FLS)**, compile, and “inactive flow” deployment failures in PSS/OmniStudio orgs.
- **Skill coverage:** `sf-deploy`, `sf-metadata` (where relevant).
- **CI/CD** “green” definition should include `sf project deploy start --dry-run` (or equivalent) before production paths; OmniStudio artifacts included in the same pipeline where licensed.

### Decision: Automated testing scope

- **Apex unit tests** (**sf-testing**, **sf-apex**) are required for **Invocable/callable Apex** and any **Apex** used by Integration Procedures. **Record-triggered / autolaunched Flows** are validated primarily in **sandbox / user acceptance testing (UAT)** (manual or dedicated UI test automation)—not assumed to be covered by Apex alone.
- Salesforce does not expose Flow logic to standard Apex unit tests without brittle patterns; Apex used for regulatory linking and **Integration Procedure (IP)** stubs must still meet coverage gates.
- **Skill coverage:** `sf-testing`, `sf-flow` (for Flow design), `sf-apex`.

### Decision: Permission sets and personally identifiable information (PII) field-level security (FLS)

- Clerk vs citizen (integration) access is modeled with **dedicated permission sets** and strict **FLS** on government-issued ID, program identifiers, and related sensitive fields—not ad hoc profile edits only—per **sf-permissions**.
- Aligns with **minimum-necessary** access, statutory privacy expectations, and auditability; supports Experience Cloud integration users separately from internal clerks.
- **Skill coverage:** `sf-permissions`.

### Decision: Agentforce vs citizen OmniScript

- **Agentforce / staff agents** are **out of scope** for **primary citizen or business self-service intake UI**; they do not replace OmniScript on Experience Cloud for regulated filings in this program. Agent automation may augment **internal** clerk or contact-center tasks under separate ADRs.
- **Skill coverage:** `sf-ai-agentforce` (when internal agents are in scope), contrasted with `sf-industry-commoncore-omniscript`.

### Decision: Optional analytics (Data Cloud)

- **Data Cloud** (**sf-datacloud** family) is an **optional** cross-filing and segmentation layer for **reporting and activation**—not the system of record for authoritative program facts. Transactional truth remains the **application** record in use (**`IndividualApplication`** and/or **`BusinessLicenseApplication`**, per program) **`RegulatoryTxn`**, and form capture.
- **Skill coverage:** `sf-datacloud`, `sf-datacloud-*` as licensed.

### Decision: Callable Apex only for gaps IPs/DataRaptors cannot cover

- Use **minimal Invocable/callable Apex** (**sf-industry-commoncore-callable-apex**, **sf-apex**) when **polymorphic links** (e.g. `RegulatoryTxn` → **`IndividualApplication`** or **`BusinessLicenseApplication`**) or **dynamic field detection** across org versions cannot be expressed safely in Flow alone.
- Phase 1 Flow “create RegulatoryTxn” may delegate to Invocable Apex; replace with pure Flow if your org’s field model is fixed and documented.

### Decision: Action Plan Template source format

- **ActionPlanTemplate** metadata XML is **fragile** across **application programming interface (API)** versions (item/detail **XML Schema Definition (XSD)**). Prefer **authoring in Setup** and **retrieve** into git, or ship **shell-only** templates (name, description, target) and add items in-org—per **sf-metadata** and deploy troubleshooting practice.
- **Skill coverage:** `sf-metadata`, `sf-deploy`.

---

## Object Mapping

> **Pattern:** Most line items on a long agency packet do **not** have a dedicated standard **application programming interface (API)** field. The native approach is **`ApplicationFormField`** (grouped in **`ApplicationFormSection`**) on the correct **application parent**—**`IndividualApplication`** for person-centered filings or **`BusinessLicenseApplication`** for **business** license apply/renew—plus **`RegulatoryTxn`**, parties, documents, and clerk steps below. Confirm **`ApplicationForm`** parent type and field API names in **Object Manager** for your org and API version.

PSS stores asserted facts in **`ApplicationFormField.FieldValue`** (or equivalent typed storage per your PSS release). Parties and addresses use **`Individual`** / **`Account`** and **`ContactPoint*`** as noted.

| Logical capture (examples; map to your program) | PSS / Industries object | API name (pattern) | Notes |
|--------------------------------------------------|-------------------------|-------------------|--------|
| **Person** application / filing reference | `IndividualApplication` | `ApplicationNumber`, `Id` | Use when applicant is an **`Individual`**. |
| **Business** license application reference | `BusinessLicenseApplication` | `Id`, `Status` / status fields (org picklists) | Primary filing for business license apply or renew; not interchangeable with **`IndividualApplication`**. |
| Business license — applicant’s account | `BusinessLicenseApplication` | `AccountId` → `Account` | B2B **Account** or **Person Account** (sole proprietor); per PSS field reference. |
| Business license — submitter / primary owner | `BusinessLicenseApplication` | `ApplicantId` → `Contact`, `PrimaryOwnerId` → `Contact` | Individual submitting and primary owner; **PersonContact** satisfies **Contact** for Person Accounts. |
| Application type | `IndividualApplication` or `BusinessLicenseApplication` | `ApplicationTypeId` where applicable | Lookup to configured `ApplicationType` per object. |
| Applicant (person) | `IndividualApplication` | `ApplicantId` → `Individual` | Person-centric programs. |
| Licensed entity / `BusinessLicense` link | `BusinessLicenseApplication` | `LicensePermitNameId` → `BusinessLicense`, `LicenseTypeId`, `LicensedEntity` per org | Align with licensing data model in Object Manager. |
| Filing status | `IndividualApplication` / `BusinessLicenseApplication` | `StatusCode` or `Status` | Draft → Submitted → review → decision (org values). |
| Submission channel | `IndividualApplication` (and org pattern for business path) | `ChannelCode` where used | Online / In Person / Phone / Mail. |
| Submitted timestamp | Application record used | `SubmittedDate` where used | When citizen or clerk files. |
| Regulatory transaction | `RegulatoryTxn` | `Id`, `StatusCode` | Parallel lifecycle to the application. |
| Transaction kind | `RegulatoryTxn` | `RegulatoryAuthorizationTypeId` | New, renewal, amendment—your configured authorization types. |
| Party in role (applicant, beneficiary, sponsor, secured party, provider, etc.) | `RegulatoryTxnParty` | `PartyId`, role fields per metadata | Links `Party` / `Individual` / `Account` to transaction. |
| Person name | `Individual` | `FirstName`, `LastName`, … | As applicable per version. |
| Organization (employer, partner agency, vendor, applicant business) | `Account` | `Name`, address fields | B2B account; relate via `Party` / `RegulatoryTxnParty` / licensing lookups per org pattern. |
| Physical / mailing address | `ContactPointAddress` | `Street`, `City`, `State`, … | Multiple addresses as needed. |
| Phone | `ContactPointPhone` | `TelephoneNumber`, `IsPrimary` | |
| Email | `ContactPointEmail` | `EmailAddress`, `IsPrimary` | |
| Program-specific or case-specific identifiers (any domain) | `ApplicationFormField` | `FieldValue` + definition `DeveloperName` | No universal field per line item—configure per packet. |
| Eligibility or qualification facts (income, category, household size, etc.) | `ApplicationFormField` | `FieldValue` | Policy-driven; strict **FLS**. |
| Government-issued ID or sensitive identifier | `ApplicationFormField` | `FieldValue` | **Highly sensitive**—minimize exposure via FLS and channel. |
| Jurisdiction for fees, tax, or routing | `ApplicationFormField` and/or address / integration | `FieldValue` / derived | Often from address or rules engine; document in integration ADR. |
| Third-party or external system reference | `ApplicationFormField` and/or `RecordAction` | `FieldValue` / completion | External system may be source of truth; clerk verification. |
| Multi-section attestations or declarations | `ApplicationFormSection` + `ApplicationFormField` | Sections + `FieldValue` | Structure per agency packet. |
| Inspection or compliance certificate reference | `ApplicationFormField` and/or `DocumentChecklistItem` | `FieldValue` + file | Evidence on `ContentDocument` when required. |
| Proof of coverage or bond (if collected) | `DocumentChecklistItem` | `Name`, `Status`, … | File on `ContentDocument`. |
| Signed PDF / uploaded evidence | `DocumentChecklistItem` + `ContentDocument` | Checklist + file link | Polymorphic parent per org design. |
| Fee estimate or billing snapshot | `ApplicationFormField` or integration-only | Per integration ADR | Native fee objects vary; confirm system of record. |
| Notary / witness / certification document | `DocumentChecklistItem` + `ContentDocument` | Notarized PDF | Clerk **`RecordAction`** as needed. |
| Privacy / consent acknowledgment | `ApplicationFormField` (boolean/date) | `FieldValue` | “Read and acknowledged” for online channel. |
| Clerk notes / verifier identity | `InteractionSummary` | `SummaryNotes`, `ChannelCode`, … | Optional `Case` link. |
| Demo / parent regulatory citation | `RegulatoryCode` | `Name`, `Description`, `StatusCode` | Do not assume `RegulatoryCodeNumber` exists in every org—field names vary by release/package. Use **Object Manager** before Flow/IP writes. |
| Regulatory txn link to application | `RegulatoryTxn` | *dynamic* — lookup or polymorphic reference to **`IndividualApplication`** and/or **`BusinessLicenseApplication`** | API name differs by org. Use **Invocable Apex** with `Schema` describe or document the org-specific field in a follow-on **integration ADR**. |
| Citizen save / prefill | OmniStudio | `OmniIntegrationProcedure` + `OmniDataTransform` bundle names | Logical surface is **procedure key** + **DataRaptor (DR)** developer names—not a PSS sObject column. |
| Portal / clerk summary | OmniStudio | `OmniUiCard` (FlexCard) definition | Displays application, checklist, and txn status; reads via **IP** / **DR**—see FlexCard skill. |

---

## Gaps

| Gap | Recommended approach (no custom object for native-equivalent capability) |
|-----|-----------------------------------------------------------------------------|
| No native **program-specific “registry”** object for every asset or case | Keep **program-specific attributes** on **`ApplicationFormField`** for the filing; use **`RegulatoryTxn`** as the long-lived “transaction” record; if the org later adopts **`Asset`** for physical assets, document a migration ADR. |
| Fee calculation and payment posting | Integrate with agency fee engine or Salesforce payments stack; store **display snapshots** in form fields or read-only fields only if required for audit—**system of record** for money in separate ADR. |
| External partners (inspection vendors, payment processors, legacy systems) | Use **`InteractionSummary`** and external **`Account`** records for partners; use **`RecordAction`** for human verification steps—not custom “partner transaction” objects unless truly outside PSS scope. |
| Statutory privacy, disclosure, or sector-specific handling rules | **Policy and integration** layer; Salesforce stores data with **FLS, event logging, and CDS** where mandated—legal interpretation is out of band. |
| Field or identity verification polymorphic target | If **`Inspection`** cannot target `RegulatoryTxn` in your org, use **`Visit`** + **`InteractionSummary`** or clerk **`RecordAction`** with structured **`ApplicationFormField`** for verifier type (internal clerk vs. external field agent). |

---

## License Requirements

List every Salesforce add-on or **Permission Set License (PSL)** the implementation is expected to need. Exact SKU names change by contract—validate with your account team.

| License / product | Why |
|-------------------|-----|
| **Salesforce Public Sector Solutions** | Core PSS objects (including `IndividualApplication`, `BusinessLicenseApplication`, `RegulatoryTxn`, and related application/licensing types as licensed). |
| **Salesforce Industries / Industry Cloud foundation** | Underlying Industries objects (`Party`, `ActionPlan`, `RegulatoryCode`, OmniStudio entitlement bundles as licensed). |
| **OmniStudio** (OmniScript, FlexCard, DataRaptor, Integration Procedure as licensed) | Citizen (and optional clerk) guided intake. |
| **Experience Cloud** | External citizen and partner portal. |
| **Customer Community Plus** (or equivalent external identity tier) | Required when **Compliant Data Sharing** applies to community users accessing CDS-protected data. |
| **Compliant Data Sharing** PSL | If CDS objects (`DataUsePurpose`, `AuthorizationFormConsent`, `IndividualShare`, …) are in scope per **Decisions** section. |
| **Agentforce** / **Einstein** features | When internal agents or generative features are in scope; not required for OmniScript-first citizen intake. |
| **Service Cloud** (if `Case` used for service workflows) | Optional; depends on whether follow-up inquiries are modeled on `Case`. |
| **Field Service / Scheduling** (if `ServiceAppointment` / mobile scheduling is adopted) | Optional; only if field verification or site visits are productized on those objects. |
| **Data Cloud** or analytics add-on (optional) | Cross-program analytics if form data volume requires separate modeling— not required for MVP. |

---

## Open Questions

1. **System of record** for authoritative program data: legacy host only vs. Salesforce holding a read-only cache—**integration ADR** and data retention rules.
2. **Exact `StatusCode` picklists** for `IndividualApplication`, `BusinessLicenseApplication` (if used), and `RegulatoryTxn` aligned to **State and Local Government** program business states (e.g., “Pending inspection,” “Pending payment”).
3. Whether **`Asset`** (standard) will represent physical assets for future services—impacts whether some “fields” move from form-only to Asset fields.
4. **Payment capture** channel: integrated pay in Experience Cloud vs. clerk-only vs. external—drives OmniScript steps and **Payment Card Industry (PCI)** scope.
5. **External financial or partner systems** rules and which fields must be **read-only** from integration vs. user-entered.
6. **CDS** final call: org-wide for sensitive PII vs. program-scoped vs. none—legal/stakeholder sign-off.
7. **RegulatoryCode** catalog scope: which statute or rule sections are loaded for training/reporting vs. integration-only citations.
8. **Multi-language** (official form parity in other languages): OmniScript localization and `ApplicationFormField` label strategy.
9. **RecordAction** completion automation vs. manual only for high-risk steps (notary, field verification, issuance).

---

*Status: Proposed — pending agency architecture review and legal/privacy sign-off.*

*Related artifacts: PSS / Industries reference in `.cursor/skills/sf-pss-data-dev/`. Phased delivery: [plan-pss-phased.md](plan-pss-phased.md). This DX template keeps a **minimal** `force-app/main/default/` tree; [manifest/package.xml](../manifest/package.xml) may list additional metadata types for deploys you have not yet retrieved into source.*
