---
name: sf-pss-data-dev
description: "PSS v68.0 native data model: Licensing/Permitting, Inspections, Regulatory Framework, Applications, Benefits, Programs, Grantmaking, Discovery Framework, Complaints, Enforcement, Visits, Party/Identity. TRIGGER: PSS object design, API name mapping, native-vs-custom decisions. SKIP: non-PSS work or pure OmniStudio."
metadata:
  version: "1.0"
  domains:
    - "Government"
  minApiVersion: "66.0"
  accessCheck: "Requires Salesforce Public Sector Solutions add-on license. RegulatoryCode, ActionPlanTemplate, and OmniStudio require Industries Cloud foundation."
  relatedSkills:
    - "omnistudio-omniscript-generate"
    - "omnistudio-datamapper-generate"
    - "omnistudio-integration-procedure-generate"
    - "omnistudio-flexcard-generate"
    - "omnistudio-dependencies-analyze"
    - "platform-apex-generate"
    - "automation-flow-generate"
    - "platform-metadata-deploy"
    - "platform-metadata-retrieve"
    - "platform-apex-test-run"
    - "platform-permission-set-generate"
---

# sf-pss-data-dev: Public Sector Solutions data architecture

Use native [Public Sector Solutions](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm) data architecture only. Do **not** introduce custom objects for standard PSS capabilities (Applications, Benefits, Regulatory Transactions, etc.) when a native object already covers the use case.

## Sources

| # | URL | Status |
|---|-----|--------|
| 1 | [PSS API overview](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm) | Referenced |
| 2 | [PSS licensing](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_licensing.htm) | Referenced |
| 3 | [PSS permitting](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_permitting.htm) | Referenced |
| 4 | [PSS inspections](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_inspections.htm) | Referenced |
| 5 | [PSS benefit management](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_benefit_management.htm) | Referenced |
| 6 | [PSS regulatory transactions](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_regulatory_transactions.htm) | Referenced |
| 7 | [PSS grants management](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_grants_management.htm) | Referenced |
| 8 | [PSS program management](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_program_management.htm) | Referenced |
| 9 | [PSS Data Model Gallery](https://developer.salesforce.com/docs/platform/data-models/guide/public-sector-solutions-category.html) | Referenced |
| 10 | [PSS discovery framework](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_discovery_framework.htm) | Referenced |
| 11 | [PSS complaint management](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_complaint_management.htm) | Referenced |
| 12 | [PSS appeals](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_appeals.htm) | Referenced |
| 13 | [PSS setup overview (Help)](https://help.salesforce.com/s/articleView?id=sf.psc_setup_overview.htm) | Referenced |
| 14 | [Application & Authorization data model](https://developer.salesforce.com/docs/platform/data-models/guide/application-authorization.html) | Referenced |
| 15 | [Asset Management data model](https://developer.salesforce.com/docs/platform/data-models/guide/pss-asset-management.html) | Referenced |
| 16 | [Interaction Summary data model](https://developer.salesforce.com/docs/platform/data-models/guide/interaction-summary.html) | Referenced |
| 17 | [Investigative Case Management data model](https://developer.salesforce.com/docs/platform/data-models/guide/justice-investigative-case-management.html) | Referenced |
| 18 | [Provider Management data model](https://developer.salesforce.com/docs/platform/data-models/guide/provider-management.html) | Referenced |
| 19 | [Regulatory Area, Fees, & Enforcement data model](https://developer.salesforce.com/docs/platform/data-models/guide/regulatory-area-fees-enforcement.html) | Referenced |
| 20 | [Social Program Management data model](https://developer.salesforce.com/docs/platform/data-models/guide/social-program-management.html) | Referenced |
| 21 | [Talent Recruitment Management data model](https://developer.salesforce.com/docs/platform/data-models/guide/talent-recruitment-management.html) | Referenced |
| 22 | [Visits, Inspections & Dynamic Assessments data model](https://developer.salesforce.com/docs/platform/data-models/guide/visits-inspections-dynamic-assessments.html) | Referenced |
| 23 | [Employee Experience setup (Help)](https://help.salesforce.com/s/articleView?id=ind.psc_employee_experience_setup.htm&type=5) | Referenced |
| 24 | [ActionPlanTemplate (Object Reference)](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_actionplantemplate.htm) | Referenced |
| 25 | [PSS `PreliminaryApplicationRef` (Object Reference)](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/sforce_api_objects_preliminaryapplicationref.htm) | Referenced |

**Primary references:** [PSS Developer Guide — overview](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm) · [PSS Data Model Gallery](https://developer.salesforce.com/docs/platform/data-models/guide/public-sector-solutions-category.html) — see Sources for topic-level gallery diagrams and Employee Experience Help.

---

## Domain map

| Domain | Key objects | Entry point | Notes |
|--------|-------------|-------------|-------|
| **Licensing and permitting** | `BusinessLicense`, `BusinessLicenseApplication`, `RegulatoryAuthorizationType`, `BusRegAuthorizationType` | `BusinessLicense` | License and permit issuance, renewals, revocations. **Business** apply/renew: anchor on **`BusinessLicenseApplication`**. Permits are `BusinessLicense` records with `RegulatoryAuthorizationType.Category = Permit` — **there is no `Permit`, `PermitApplication`, `PermitType`, or `LicenseType` object** in PSS v68.0. |
| **Inspections** | `InspectionType`, `InspectionAssessmentInd` | `InspectionType` | Inspection type definitions and linkage to Discovery Framework assessments. No standalone `Inspection`, `InspectionChecklistItem`, or `InspectionFinding` objects — violations use `RegulatoryCodeViolation`; dynamic questions use `AssessmentQuestion*`. |
| **Action plans (Industries)** | `ActionPlanTemplate`, `ActionPlan`, `RecordAction` | `ActionPlanTemplate` | Templated **clerk / procedural** task sequences (deadlines, roles, document steps) on applications, regulatory records, or cases. Complements **field compliance** from `Inspection` / `InspectionVisit` and **dynamic assessments** (`Assessment*`, indicators) in the [Visits, Inspections & Dynamic Assessments](https://developer.salesforce.com/docs/platform/data-models/guide/visits-inspections-dynamic-assessments.html) gallery—does **not** replace `InspectionType` checklist lines cloned to `InspectionChecklistItem`. |
| **Regulatory framework** | `RegulatoryCode`, `RegulatoryAuthority`, `RegulatoryAuthorizationType`, `RegulatoryTrxnFee` | `RegulatoryCode` | Code citations, regulatory authorities, authorization types, fee records. **No `RegulatoryTxn` or `RegulatoryTxnParty` object** in PSS v68.0 — fees use `RegulatoryTrxnFee`; authorization lifecycle is on `BusinessLicense` / `BusinessLicenseApplication`. |
| **Applications (person)** | `IndividualApplication`, `ApplicationForm`, `ApplicationFormField`, `ApplicationFormSection` | `IndividualApplication` | Person-centric filings (benefits, programs where the applicant is a **`Contact`**). Do **not** substitute this for a **business** license filing—use **`BusinessLicenseApplication`** (see **Licensing** row). |
| **Applications (draft / resume)** | `PreliminaryApplicationRef` | `PreliminaryApplicationRef` | Portal-side save-and-resume anchor for guest or authenticated intake; **`IsSubmitted`** flips on promotion; stamp **`AppliedDate`** on the filing parent. **`ApplicationType`** is a restricted picklist (5 values: ApplicationForm, BusinessLicenseApplication, BusinessPrescreening, IndividualApplication, PublicComplaint) — not a free-form key. Do not hold authoritative program facts here beyond what the resume flow re-hydrates. |
| **Benefit management** | `BenefitType`, `Benefit`, `BenefitDisbursement`, `BenefitAssignment`, `ProgramEnrollment` | `BenefitType` | Benefits, eligibility, disbursements, assignments |
| **Program management** | `Program`, `ProgramCohort`, `ProgramCohortMember`, `ProgramEnrollment` | `Program` | Programs, cohorts, enrollee lifecycle. **No `ProgramType` or `ProgramEnrollmentStatusHistory` object** — enrollment status is a picklist on `ProgramEnrollment.Status`. |
| **Grantmaking** | `FundingOpportunity`, `FundingAward`, `FundingDisbursement`, `Budget`, `BudgetAllocation` | `FundingOpportunity` | Grant lifecycle from opportunity through disbursement. **No `Grant`, `GrantApplication`, `GrantType`, `GrantBudget`, `GrantAllocation`, or `GrantOpportunity` object** in PSS v68.0. |
| **Discovery Framework** | `AssessmentQuestion`, `AssessmentQuestionSet`, `AssessmentQuestionVersion`, `AssessmentIndicator` | `AssessmentQuestionSet` | Intake assessments; eligibility screening and triage |
| **Complaint management** | `PublicComplaint`, `ComplaintCase`, `ComplaintParticipant` | `PublicComplaint` | Intake, investigation, resolution. **No standalone `Complaint`, `ComplaintFinding`, `ComplaintRemediation`, or `ComplaintAssociation` object** in PSS v68.0. |
| **Appeals** | — | — | **PSS v68.0 base package has no Appeal objects.** No `Appeal`, `AppealAssociation`, `AppealHearing`, or `AppealDecision`. Implement via custom objects or Investigative Case Management add-on. |
| **Enforcement** | `RegulatoryCodeViolation`, `ViolationEnforcementAction`, `ViolationType` | `RegulatoryCodeViolation` | Violations and enforcement orders. **No standalone `EnforcementAction` or `Violation` object** — use `RegulatoryCodeViolation` → `ViolationEnforcementAction`. |
| **Visits and field activity** | `Visit`, `Visitor`, `VisitedParty`, `Case`, `ServiceAppointment` | `Visit` | Field visits; links to inspections and enforcement. **No `VisitQueue` object** — use `Visit` record queues. |
| **Party and identity** | `Contact`, `Account`, `Individual`, `Party`, `PartyProfile`, `ContactPointAddress`, `ContactPointEmail` | `Contact` | `Contact` is the person identity record (name, gender, demographics). `Individual` is the GDPR/privacy companion (consent flags, opt-outs) — it is **not** a person record and has no `FirstName`/`LastName`. |

---

## Architecture decisions

- `Contact` is the person identity record in PSS citizen flows; `Individual` is the GDPR/privacy companion. Do not anchor `FirstName`/`LastName`/`GenderIdentity` on `Individual`. Resolve constituent identity through `Contact` / `Account` / `Party`.
- `RegulatoryCode` is the master reference for citations across Inspections, Enforcement, and Complaints.
- **`BusinessLicenseApplication`** is the native filing spine when a **business** (Account / licensed entity) applies for or renews a **business license**—do **not** use **`IndividualApplication`** as the primary application record for that scenario.
- **`IndividualApplication`** is not replaced by a custom object for **person** applicants: use `ApplicationForm` + `ApplicationFormField` for dynamic intake where that model applies; extend via Discovery Framework for eligibility logic.
- **Draft / save-and-resume** is anchored on **`PreliminaryApplicationRef`**, not a custom draft object or an early-created **`IndividualApplication`** / **`BusinessLicenseApplication`** in a `StatusCode = Draft` state. Track draft state via **`IsSubmitted`** and the resume URL on the preliminary record; on submit, create or populate the correct filing parent and stamp **`SubmissionDate`**. Applies equally to guest and authenticated **Experience Cloud** flows (`ApplicantId` is polymorphic to `Contact` / `User` / `Account`).
- `BenefitDisbursement` carries financial line detail; `BenefitAssignment` maps the benefit to a `ProgramEnrollment`.
- Inspections: no `InspectionChecklistItem` or `InspectionFinding` objects in PSS v68.0. Violations use `RegulatoryCodeViolation` → `ViolationEnforcementAction`; dynamic assessment questions use `AssessmentQuestion*` via `InspectionAssessmentInd`.
- **`ActionPlanTemplate` / `ActionPlan` / `RecordAction`** (Industries action framework): use for **repeatable human task orchestration**—issuance, verification, fees, branching—instead of a custom task or BPM engine; aligns with native PSS program patterns (see project **Architecture Decision Records (ADRs)**). Prefer **`Visit`** / **`InspectionVisit`** and **`Assessment*`** flows where the product model is **field activity or scored assessment**; use **`ActionPlan`** where you need **structured RecordAction steps** on the same case, regulatory, or visit-adjacent context (parent targets vary by org—confirm supported types in Object Manager and the [ActionPlanTemplate](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_actionplantemplate.htm) object reference).
- Author **`ActionPlanTemplate`** in **Setup**, then **retrieve** into source control; **`ActionPlanTemplate`** metadata XML is **fragile** across **application programming interface (API)** versions—when using this skill **inside the PSS DX template**, see **`docs/adr-pss.md`** (Action Plan Template source format) for shell-only vs full-template practice; use upstream **`platform-metadata-retrieve`** / **`platform-metadata-deploy`** ([forcedotcom/afv-library](https://github.com/forcedotcom/afv-library)) when troubleshooting deploy order and **XML Schema Definition (XSD)**.
- Grantmaking uses `FundingAward` + `Budget` / `BudgetAllocation` + `FundingDisbursement`; integrate with Accounting Subledger for financials when required. There are no `Grant`, `GrantBudget`, or `GrantAllocation` objects.
- Discovery Framework assessments share objects with **Education Cloud Integrated Case Companion (ICC)** (`AssessmentQuestion*`); versioning is managed via `AssessmentQuestionVersion`.

---

## Reference files

- [references/sf-pss-core-objects.md](references/sf-pss-core-objects.md) — Core PSS object API names, key fields, and relationships

## Cross-skill integration

- **OmniStudio / DataRaptor / Integration Procedure (IP)** for UI and orchestration: use [forcedotcom/afv-library](https://github.com/forcedotcom/afv-library) skills such as `omnistudio-omniscript-generate`, `omnistudio-datamapper-generate`, `omnistudio-integration-procedure-generate` for implementation mechanics; use **this skill** for which **PSS objects** to persist.
- **Discovery Framework** assessment authoring overlaps Education Cloud ICC; confirm org licensing and use official Discovery Framework docs alongside this skill.
