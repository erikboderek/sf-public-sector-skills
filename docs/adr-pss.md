# Architecture Decision Record (ADR): State and Local Government Public Sector Solutions (PSS)

## Context

**State and local government** agencies deliver regulated services—**benefits**, **professional and business licenses**, **permits**, **grants**, and other programs—by having residents and businesses **complete forms**, supply evidence, and move a filing through **review, verification, and decision** on a defined timeline. The same pattern applies whether the service is cash aid, a contractor license, a building permit, or a specialized regulatory program.

Agency programs differ in labels and statutes; this ADR describes **Salesforce object patterns** only. Any long-form “packet” your jurisdiction uses maps to the same native shapes—**applications**, **regulatory transactions**, **form fields**, and **documents**—without requiring a particular industry example.

**Problem we are solving:** Provide a **native Public Sector Solutions (PSS)** and **Salesforce Industries common layer** architecture for citizen and clerk channels—guided intake, document tracking, regulatory transaction lifecycle, and operational checklists—**without introducing custom Salesforce objects** for data and processes that PSS and the common layer already model (applications, regulatory transactions, party identity, document checklists, action plans).

Non-goals for this ADR: selecting a specific payment gateway, defining exact picklist values for every `Status` / `InternalStatus`, or specifying non-Salesforce **legacy line-of-business or billing system** integration contracts (those belong in separate integration ADRs).

---

## Domain Coverage

| Domain | PSS / Industries objects | Rationale |
|--------|--------------------------|-----------|
| **Citizen & business applications** | `IndividualApplication`, **`BusinessLicenseApplication`** (business licensing), `ApplicationType`, `ApplicationForm`, `ApplicationFormSection`, `ApplicationFormField` | Service requests are application-shaped workflows with typed sections and field-level capture. Use **`IndividualApplication`** when the **applicant is a person** (`ContactId` → `Contact`). Use **`BusinessLicenseApplication`** for **business license** apply or renew: the object uses **`AccountId`** → **`Account`** (B2B **Account** or **Person Account**) and **`ApplicantId`** / **`PrimaryOwnerId`** → **`Contact`** (including **PersonContact** for Person Accounts)—see [PSS `BusinessLicenseApplication`](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/sforce_api_objects_businesslicenseapplication.htm). Do not use **`IndividualApplication`** as the primary filing object for that scenario. PSS application objects replace bespoke custom form objects. |
| **Draft / save-and-resume intake** | **`PreliminaryApplicationRef`** | Native PSS anchor for a **saved-but-not-yet-submitted** application spanning **guest** and **authenticated** channels (`ApplicantId` is polymorphic to **`Contact`** / **`User`** / **`Account`**). Carries the resume URL, wizard type/category keys, and an **`IsSubmitted`** flag that flips on promotion to **`IndividualApplication`** or **`BusinessLicenseApplication`**—removes the need to create a filing parent in a `Status = Draft` state or a custom draft object. See [PSS `PreliminaryApplicationRef`](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/sforce_api_objects_preliminaryapplicationref.htm). |
| **Authorization types and issued licenses** | `RegulatoryAuthorizationType`, `BusinessLicense` | Each service path (new application, renewal, amendment, appeal) is defined via **`RegulatoryAuthorizationType`**; the issued authorization is a **`BusinessLicense`**. PSS v68.0 does not have a standalone `RegulatoryTxn` object; the application and authorization lifecycle lives on `BusinessLicenseApplication` and `BusinessLicense`. |
| **Citizen and party identity** | `Contact` (person), `Account` (org), `Individual` (GDPR/privacy companion only), `PartyProfile` (optional), `ContactPointAddress`, `ContactPointPhone`, `ContactPointEmail` | Person applicants, dependents, and sponsors map to **`Contact`**; organizations map to **`Account`**. `Individual` is the GDPR privacy management companion only and has no `FirstName`/`LastName`. There is no standalone `Party` object. Structured contact points track address, phone, and email channels. |
| **Document management** | `DocumentChecklistItem`, `ContentDocument` / `ContentVersion` / `ContentDocumentLink` | Required evidence (ID, income statements, inspection certificates, signed attestations, third-party letters) is tracked as checklist items with file evidence on the platform. |
| **Workflow & procedure** | `ActionPlanTemplate`, `ActionPlan`, `RecordAction` | Program deadlines, role handoffs, inspections, and **RecordAction** steps (including third-party or field verification when applicable) are modeled as templated plans and tasks—not a custom task framework. |
| **Service & channel logging** | `InteractionSummary`, `Case` (optional) | Counter visits, phone clarifications, and third-party verifier contacts are recorded for audit and service management where `Case` is already in use. |
| **Regulatory citations** | `RegulatoryCode` (Industries common) | Optional spine for statute / rule references used in training, reporting, and cross-program compliance. |
| **Field / special verification** | `Visit`, `ServiceAppointment` (optional); or clerk-completed `RecordAction` + `InteractionSummary` | On-site or mobile verification may use visit/scheduling patterns where the org licenses and configures them; otherwise attestation stays on the application/regulatory record. |

**Explicitly out of scope as first-class “registry” tables for every program-specific thing:** PSS does not provide one standard object per possible regulated asset, location, or case subtype for all agencies. **Asset** (standard) may be used only if the org explicitly standardizes on it for physical-asset tracking; this ADR assumes **transaction- and application-centric** modeling unless a follow-on ADR adopts `Asset`.

---

## Decisions

### Decision: Anchor regulated service work on the correct application parent and authorization model

- **Options considered:** (1) Custom parent object with child custom objects; (2) `Case`-only tracking; (3) Only `IndividualApplication` for every filing regardless of applicant type; (4) Standard `Opportunity` or `Order`; (5) **`IndividualApplication`** or **`BusinessLicenseApplication`** + **`ApplicationFormField`** as appropriate, with `RegulatoryAuthorizationType` defining the authorization class.
- **Decision:** Use **`IndividualApplication`** as the filing container when the **applicant is a person** (`ContactId` → `Contact`)—benefits, many personal permits, and other **`Contact`**-centric programs—extended via **`ApplicationForm`**, **`ApplicationFormSection`**, and **`ApplicationFormField`** where that model applies. Use **`BusinessLicenseApplication`** as the filing container for **business license** apply or renew: model **`AccountId`** (business **Account** or **Person Account**) and the human submitter / primary owner via **`ApplicantId`** / **`PrimaryOwnerId`** (**`Contact`**, including **PersonContact** where Person Accounts are used), plus **`BusinessLicense`** relationships per Object Manager (use `RegulatoryAuthorizationType` to define authorization class); **`IndividualApplication`** must not stand in for that primary business filing. Party roles on `BusinessLicenseApplication` are modeled as `AccountId`, `ApplicantId`, and `PrimaryOwnerId` fields — there is no `RegulatoryTxnParty` object in PSS v68.0. PSS v68.0 does not have a `RegulatoryTxn` object; the authorization lifecycle lives on `BusinessLicense` + `BusinessLicenseApplication`.
- **Rationale:** Aligns with PSS v68.0 application and authorization patterns; party roles are native fields on the filing objects rather than a separate party-transaction junction object; avoids parallel data models that would diverge from Industries upgrades and shared components.
- **Trade-offs:** More metadata design up front (authorization types, form definitions, action plan templates, and which application object each program uses). Less flexibility than unconstrained custom objects unless teams invest in configuration and integration discipline.

### Decision: Draft and save-and-resume anchored on PreliminaryApplicationRef, not on an early filing parent

- **Options considered:** (1) Create the **`IndividualApplication`** or **`BusinessLicenseApplication`** immediately with `Status = Draft` and let the citizen edit in place; (2) Custom `Draft__c` object; (3) Store draft JSON in an OmniScript-only save cache; (4) Native **`PreliminaryApplicationRef`** as the draft anchor, promoted to the correct filing parent on submit.
- **Decision:** Use **`PreliminaryApplicationRef`** as the **draft / save-and-resume** record for both **guest** and **authenticated** intake. `ApplicantId` is polymorphic to **`Contact`** / **`User`** / **`Account`**, so the same object supports guest-to-authenticated handoff without a rewrite. Carry the wizard key (**`ApplicationType`**), grouping (**`ApplicationCategory`**), auto-generated name (**`ApplicationName`**), and the resume URL (**`SavedApplicationUrl`**) on the preliminary record. On submit, create or populate the correct filing parent — **`IndividualApplication`** for person filings or **`BusinessLicenseApplication`** for **business** license apply/renew (setting **`BusinessAccountNameId`** in parallel where the draft was a business flow) — and stamp **`IsSubmitted = true`** + **`SubmissionDate`** on the preliminary record.
- **Rationale:** Native PSS object; removes the need for a `Status = Draft` state on the filing parent, keeps unfinished intakes out of clerk work-queues and reporting, and cleanly separates **portal wizard state** from the **submitted regulatory record**. The polymorphic `ApplicantId` lets the same OmniScript path serve guest and community users without branching data models.
- **Trade-offs:** A second object in the intake path (extra Data Mapper / IP wiring); requires promotion logic (Flow or Invocable Apex) to hydrate the correct filing parent from the preliminary record on submit; PII on the preliminary record must be minimized and secured with the same **field-level security (FLS)** rigor as the filing parent.

### Decision: Model all form-specific data as ApplicationFormField (or document evidence), not parallel custom “spreadsheet” objects

- **Options considered:** (1) Custom fields on a custom program-specific object per form; (2) Long-text “blob” on `BusinessLicenseApplication`; (3) Structured **`ApplicationFormField`** / sections per agency form packet.
- **Decision:** Map agency **field-level** capture to **`ApplicationFormField`** records (with appropriate **`ApplicationFormSection`** groupings). Use **`DocumentChecklistItem`** + files for signed PDFs and certificates.
- **Rationale:** Preserves native extensibility, supports OmniScript-driven intake, and keeps a clear separation between **application lifecycle** (`BusinessLicenseApplication.Status` / `InternalStatus`) and **asserted facts** (form fields).
- **Trade-offs:** Reporting across “same logical field” requires consistent **`DeveloperName`** on form field definitions; bulk analytics may need Data Cloud or careful field naming conventions.

### Decision: Citizen intake via OmniScript; clerk completion via console + Flow/Screen Flow

- **Options considered:** (1) Custom **Lightning Web Components (LWC)** portal; (2) Screen Flows only; (3) OmniScript + Experience Cloud for citizens, standard app + Flow for clerks; (4) Email/mail-only (no self-service).
- **Decision:** **OmniScript** on **Experience Cloud** for citizen and external partner submission; **Salesforce console** (or workspace) on **`IndividualApplication`**, **`BusinessLicenseApplication`**, or **`BusinessLicense`** (per program) for clerks, with **`RecordAction`** optionally launching **Screen Flow** or **internal OmniScript**.
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

- **Data Cloud** (**sf-datacloud** family) is an **optional** cross-filing and segmentation layer for **reporting and activation**—not the system of record for authoritative program facts. Transactional truth remains the **application** record in use (**`IndividualApplication`** and/or **`BusinessLicenseApplication`**, per program), **`BusinessLicense`** (for the issued authorization), and form capture.
- **Skill coverage:** `sf-datacloud`, `sf-datacloud-*` as licensed.

### Decision: Callable Apex only for gaps IPs/DataRaptors cannot cover

- Use **minimal Invocable/callable Apex** (**sf-industry-commoncore-callable-apex**, **sf-apex**) when **polymorphic links** (e.g. `BusinessLicense` → **`BusinessLicenseApplication`** or cross-object lookups between **`IndividualApplication`** and related records) or **dynamic field detection** across org versions cannot be expressed safely in Flow alone.
- Phase 1 Flow logic to create the filing parent (`IndividualApplication` or `BusinessLicenseApplication`) and link to `BusinessLicense` may delegate to Invocable Apex; replace with pure Flow if your org’s field model is fixed and documented. PSS v68.0 has no `RegulatoryTxn` object.

### Decision: Action Plan Template source format

- **ActionPlanTemplate** metadata XML is **fragile** across **application programming interface (API)** versions (item/detail **XML Schema Definition (XSD)**). Prefer **authoring in Setup** and **retrieve** into git, or ship **shell-only** templates (name, description, target) and add items in-org—per **sf-metadata** and deploy troubleshooting practice.
- **Skill coverage:** `sf-metadata`, `sf-deploy`.

---

## Object Mapping

> **Pattern:** Most line items on a long agency packet do **not** have a dedicated standard **application programming interface (API)** field. The native approach is **`ApplicationFormField`** (grouped in **`ApplicationFormSection`**) on the correct **application parent**—**`IndividualApplication`** for person-centered filings or **`BusinessLicenseApplication`** for **business** license apply/renew—plus **`BusinessLicense`** (issued authorization), parties, documents, and clerk steps below. Confirm **`ApplicationForm`** parent type and field API names in **Object Manager** for your org and API version.

PSS stores asserted facts in **`ApplicationFormField.FieldValue`** (or equivalent typed storage per your PSS release). Parties and addresses use **`Contact`** (person) / **`Account`** (org) and **`ContactPoint*`** as noted.

| Logical capture (examples; map to your program) | PSS / Industries object | API name (pattern) | Notes |
|--------------------------------------------------|-------------------------|-------------------|--------|
| Draft / save-and-resume anchor | `PreliminaryApplicationRef` | `Id`, `ApplicationName` | Portal-side draft record; auto-generated **`ApplicationName`** (String 255) shown to citizen. Do **not** carry authoritative program facts here beyond what the resume flow re-hydrates. |
| Draft applicant (guest or authenticated) | `PreliminaryApplicationRef` | `ApplicantId` → polymorphic (`Contact` / `User` / `Account`) | Supports **guest** intake and **authenticated Experience Cloud** users on the same object; resolve to **`Contact`** on promotion for person-centric filings. |
| Draft wizard key / grouping | `PreliminaryApplicationRef` | `ApplicationType` (picklist), `ApplicationCategory` (picklist) | **`ApplicationType`** = wizard-type key (align with configured **`ApplicationType`** record and OmniScript key). **`ApplicationCategory`** = draft-level grouping (verify picklist values in Object Manager per deployment; values vary by object — for `IndividualApplication` the values are Basic, Early Decision, Regular Decision, Special in grantmaking/education context; use `Category` for licensing/permit context). |
| Draft resume URL | `PreliminaryApplicationRef` | `SavedApplicationUrl` (URL 255) | Deep-link into the OmniScript / Experience Cloud page; surfaced to the citizen. |
| Draft submission flag | `PreliminaryApplicationRef` | `IsSubmitted` (boolean), `SubmissionDate` (date) | Flip **`IsSubmitted = true`** and stamp **`SubmissionDate`** when the draft is promoted to a filing parent; replaces a separate `Status = Draft/Submitted` state on the filing parent. |
| Draft business account link | `PreliminaryApplicationRef` | `BusinessAccountNameId` → `Account` | For **business** flows; parallels **`BusinessLicenseApplication.AccountId`** and is copied on promotion. |
| **Person** application / filing reference | `IndividualApplication` | `ApplicationReferenceNumber`, `Id` | Use when the applicant is a **person** (`ContactId` → **`Contact`**). Promoted from **`PreliminaryApplicationRef`** on submit for person-centric filings. |
| **Business** license application reference | `BusinessLicenseApplication` | `Id`, `Status` / status fields (org picklists) | Primary filing for business license apply or renew; not interchangeable with **`IndividualApplication`**. Promoted from **`PreliminaryApplicationRef`** on submit (with **`BusinessAccountNameId`** → **`AccountId`**). |
| Business license — applicant’s account | `BusinessLicenseApplication` | `AccountId` → `Account` | B2B **Account** or **Person Account** (sole proprietor); per PSS field reference. |
| Business license — submitter / primary owner | `BusinessLicenseApplication` | `ApplicantId` → `Contact`, `PrimaryOwnerId` → `Contact` | Individual submitting and primary owner; **PersonContact** satisfies **Contact** for Person Accounts. |
| Application type | `IndividualApplication` | `ApplicationType` (restricted picklist) | Picklist values: Change Of Circumstance, New, Recertification, Renewal. There is no `ApplicationTypeId` lookup field and no `ApplicationType` object — this is a picklist. |
| Applicant (person) | `IndividualApplication` | `ContactId` → `Contact` | Person-centric programs. There is no `ApplicantId` field on `IndividualApplication`; the applicant field is `ContactId`. |
| `BusinessLicense` link and authorization type | `BusinessLicenseApplication` | `LicensePermitNameId` → `BusinessLicense`, `LicenseTypeId` → `RegulatoryAuthorizationType` | `LicensedEntity` is not a PSS object; the authorized party is `AccountId` / `ContactId` on `BusinessLicense`. Align with licensing data model in Object Manager. |
| Filing status | `IndividualApplication` | `InternalStatus` (PSS lifecycle: Invited, In Progress, Submitted, Approved, Denied…) or `Status` (org-extended values) | There is no `StatusCode` field. Use `InternalStatus` for PSS lifecycle states, `Status` for org-extended values. |
| Filing status | `BusinessLicenseApplication` | `Status` (org-configured picklist) | Draft → Submitted → Under Review → Approved/Rejected. There is no `StatusCode` field. |
| Submission channel | `IndividualApplication` (and org pattern for business path) | Custom field — `ChannelCode` does not exist on `IndividualApplication` in PSS v68.0; implement as a custom picklist field if required | Online / In Person / Phone / Mail. |
| Submitted timestamp | `IndividualApplication` | `AppliedDate` (DateTime) | Date the application was received. There is no `SubmittedDate` field on application records. |
| Issued authorization | `BusinessLicense` | `Id`, `Status`, `RegulatoryAuthorizationTypeId`, `PeriodStart`, `PeriodEnd`, `IsActive` | The issued license or permit record. PSS v68.0 has no `RegulatoryTxn` object; the authorization lifecycle lives on `BusinessLicense` + `BusinessLicenseApplication`. |
| Authorization class | `RegulatoryAuthorizationType` | `Id`, `RegulatoryAuthCategory`, `RegulatoryAuthCode` | New, renewal, amendment scenarios are differentiated via `BusinessLicenseApplication.LicenseTypeId` → `RegulatoryAuthorizationType`. |
| Party roles on application | `BusinessLicenseApplication` | `AccountId` (org applicant), `ApplicantId` → `Contact` (human submitter), `PrimaryOwnerId` → `Contact` (primary owner) | Party roles are native fields on the application record. There is no `RegulatoryTxnParty` object in PSS v68.0. |
| Party roles on person application | `IndividualApplication` | `ContactId` → `Contact` (person applicant), `AccountId` → `Account` (org applicant) | Use `ContactId` for person applicants; there is no `ApplicantId` field on `IndividualApplication`. |
| Person name | `Contact` | `FirstName`, `LastName`, `GenderIdentity`, … | Person identity fields are on `Contact`, not `Individual`. `Individual` is the GDPR privacy companion only and has no `FirstName`/`LastName`. |
| Organization (employer, partner agency, vendor, applicant business) | `Account` | `Name`, address fields | B2B account; relate via `AccountId` on application objects or licensing lookups per org pattern. There is no standalone `Party` object in PSS v68.0. |
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
| Clerk notes / verifier identity | `InteractionSummary` | `MeetingNotes`, … | Optional `Case` link. No `SummaryNotes` or `ChannelCode` field on `InteractionSummary`. |
| Demo / parent regulatory citation | `RegulatoryCode` | `Name`, `Description`, `IsActive` (boolean formula), `EffectiveFrom`, `EffectiveTo` | No `StatusCode` or `RegulatoryCodeNumber` field. Parent is `RegulatoryAuthority` (not `Account`). Use **Object Manager** before Flow/IP writes. |
| Authorization-to-application link | `BusinessLicense` | `LicensePermitNameId` on `BusinessLicenseApplication` → `BusinessLicense` | PSS v68.0 has no `RegulatoryTxn` object. The `BusinessLicense` record links to the filing via `BusinessLicenseApplication.LicensePermitNameId`. Verify the exact relationship field in Object Manager and document org-specific extensions in a follow-on **integration ADR**. |
| Citizen save / prefill | OmniStudio | `OmniIntegrationProcedure` + `OmniDataTransform` bundle names | Logical surface is **procedure key** + **DataRaptor (DR)** developer names—not a PSS sObject column. |
| Portal / clerk summary | OmniStudio | `OmniUiCard` (FlexCard) definition | Displays application, checklist, and txn status; reads via **IP** / **DR**—see FlexCard skill. |

---

## Gaps

| Gap | Recommended approach (no custom object for native-equivalent capability) |
|-----|-----------------------------------------------------------------------------|
| No native **program-specific “registry”** object for every asset or case | Keep **program-specific attributes** on **`ApplicationFormField`** for the filing; use **`BusinessLicense`** as the long-lived issued authorization record and **`BusinessLicenseApplication`** for the filing lifecycle; if the org later adopts **`Asset`** for physical assets, document a migration ADR. PSS v68.0 has no `RegulatoryTxn` object. |
| Fee calculation and payment posting | Integrate with agency fee engine or Salesforce payments stack; store **display snapshots** in form fields or read-only fields only if required for audit—**system of record** for money in separate ADR. |
| External partners (inspection vendors, payment processors, legacy systems) | Use **`InteractionSummary`** and external **`Account`** records for partners; use **`RecordAction`** for human verification steps—not custom “partner transaction” objects unless truly outside PSS scope. |
| Statutory privacy, disclosure, or sector-specific handling rules | **Policy and integration** layer; Salesforce stores data with **FLS, event logging, and CDS** where mandated—legal interpretation is out of band. |
| Field or identity verification polymorphic target | If **`Inspection`** cannot target `BusinessLicenseApplication` in your org, use **`Visit`** + **`InteractionSummary`** or clerk **`RecordAction`** with structured **`ApplicationFormField`** for verifier type (internal clerk vs. external field agent). PSS v68.0 has no `RegulatoryTxn` object. |

---

## License Requirements

List every Salesforce add-on or **Permission Set License (PSL)** the implementation is expected to need. Exact SKU names change by contract—validate with your account team.

| License / product | Why |
|-------------------|-----|
| **Salesforce Public Sector Solutions** | Core PSS objects (including `IndividualApplication`, `BusinessLicenseApplication`, `BusinessLicense`, `RegulatoryAuthorizationType`, and related application/licensing types as licensed). PSS v68.0 has no `RegulatoryTxn` object. |
| **Salesforce Industries / Industry Cloud foundation** | Underlying Industries objects (`ActionPlan`, `RegulatoryCode`, OmniStudio entitlement bundles as licensed). There is no standalone `Party` object. |
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
2. **Exact `InternalStatus` / `Status` picklist values** for `IndividualApplication` and `BusinessLicenseApplication` aligned to **State and Local Government** program business states (e.g., “Pending inspection,” “Pending payment”). There is no `StatusCode` field on these objects and no `RegulatoryTxn` object in PSS v68.0.
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
