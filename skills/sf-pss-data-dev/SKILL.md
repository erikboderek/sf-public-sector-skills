---
name: sf-pss-data-dev
description: >
  Public Sector Solutions (PSS) native data model: Licensing, Permitting, Inspections,
  Regulatory Transactions, Individual Applications, Benefit and Program Management, Grants,
  Discovery Framework, Complaints, Appeals, Enforcement, Visits, Party and Identity.
  TRIGGER when: designing or implementing PSS data on standard objects, mapping domains to
  Salesforce application programming interface (API) names, or reviewing whether to use native vs custom objects.
  DO NOT TRIGGER when: non-PSS Salesforce work, or generic OmniStudio without PSS data context
  (use sf-industry-commoncore-*); prefer sf-docs for unrelated API reference.
license: MIT
metadata:
  version: "1.0.7"
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

**Primary references:** [PSS Developer Guide — overview](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm) · [PSS Data Model Gallery](https://developer.salesforce.com/docs/platform/data-models/guide/public-sector-solutions-category.html) — see Sources for topic-level gallery diagrams and Employee Experience Help.

---

## Domain map

| Domain | Key objects | Entry point | Notes |
|--------|-------------|-------------|-------|
| **Licensing** | `BusinessLicense`, `BusinessLicenseApplication`, `LicenseType`, `LicensedEntity` | `BusinessLicense` | License issuance, renewals, revocations. **Business** apply/renew: anchor on **`BusinessLicenseApplication`**—not **`IndividualApplication`**. |
| **Permitting** | `Permit`, `PermitApplication`, `PermitType`, `PermitApplicationReview` | `Permit` | Construction, land-use, event permits from application to issuance |
| **Inspections** | `Inspection`, `InspectionType`, `InspectionQuestion`, `InspectionFinding`, `InspectionChecklistItem` | `Inspection` | Scheduling, checklists, findings, pass/fail |
| **Action plans (Industries)** | `ActionPlanTemplate`, `ActionPlan`, `RecordAction` | `ActionPlanTemplate` | Templated **clerk / procedural** task sequences (deadlines, roles, document steps) on applications, regulatory records, or cases. Complements **field compliance** from `Inspection` / `InspectionVisit` and **dynamic assessments** (`Assessment*`, indicators) in the [Visits, Inspections & Dynamic Assessments](https://developer.salesforce.com/docs/platform/data-models/guide/visits-inspections-dynamic-assessments.html) gallery—does **not** replace `InspectionType` checklist lines cloned to `InspectionChecklistItem`. |
| **Regulatory transactions** | `RegulatoryCode`, `RegulatoryAuthorizationType`, `RegulatoryTxn`, `RegulatoryTxnParty` | `RegulatoryCode` | Code citations, authorization types, transaction records |
| **Applications (person)** | `IndividualApplication`, `ApplicationForm`, `ApplicationFormField`, `ApplicationFormSection` | `IndividualApplication` | Person-centric filings (benefits, programs where the applicant is an **`Individual`**). Do **not** substitute this for a **business** license filing—use **`BusinessLicenseApplication`** (see **Licensing** row). |
| **Benefit management** | `BenefitType`, `Benefit`, `BenefitDisbursement`, `BenefitAssignment`, `ProgramEnrollment` | `BenefitType` | Benefits, eligibility, disbursements, assignments |
| **Program management** | `ProgramType`, `Program`, `ProgramCohort`, `ProgramEnrollment`, `ProgramEnrollmentStatusHistory` | `Program` | Programs, cohorts, enrollee lifecycle |
| **Grants management** | `Grant`, `GrantApplication`, `GrantType`, `GrantBudget`, `GrantAllocation`, `GrantOpportunity` | `Grant` | Grant lifecycle from opportunity through allocation and reporting |
| **Discovery Framework** | `AssessmentQuestion`, `AssessmentQuestionSet`, `AssessmentQuestionVersion`, `AssessmentIndicator` | `AssessmentQuestionSet` | Intake assessments; eligibility screening and triage |
| **Complaint management** | `Complaint`, `ComplaintAssociation`, `ComplaintFinding`, `ComplaintRemediation` | `Complaint` | Intake, investigation, findings, resolution |
| **Appeals** | `Appeal`, `AppealAssociation`, `AppealHearing`, `AppealDecision` | `Appeal` | Appeals linked to regulatory decisions and enforcement |
| **Enforcement** | `EnforcementAction`, `EnforcementActionType`, `Violation`, `ViolationEnforcementAction` | `Violation` | Violations, orders, fines, compliance |
| **Visits and cases** | `Visit`, `VisitQueue`, `Case`, `ServiceAppointment` | `Visit` | Field visits; links to inspections and enforcement |
| **Party and identity** | `Individual`, `Party`, `PartyProfile`, `ContactPointAddress`, `ContactPointEmail` | `Individual` | Constituent identity across PSS domains |

---

## Architecture decisions

- All domain objects resolve constituent identity through `Individual` / `Party` — never create standalone Contact-only models for PSS citizen flows.
- `RegulatoryCode` is the master reference for citations across Inspections, Enforcement, and Complaints.
- **`BusinessLicenseApplication`** is the native filing spine when a **business** (Account / licensed entity) applies for or renews a **business license**—do **not** use **`IndividualApplication`** as the primary application record for that scenario.
- **`IndividualApplication`** is not replaced by a custom object for **person** applicants: use `ApplicationForm` + `ApplicationFormField` for dynamic intake where that model applies; extend via Discovery Framework for eligibility logic.
- `BenefitDisbursement` carries financial line detail; `BenefitAssignment` maps the benefit to a `ProgramEnrollment`.
- Inspections use `InspectionChecklistItem` for per-checklist-line results; `InspectionFinding` for violations found.
- **`ActionPlanTemplate` / `ActionPlan` / `RecordAction`** (Industries action framework): use for **repeatable human task orchestration**—issuance, verification, fees, branching—instead of a custom task or BPM engine; aligns with native PSS program patterns (see project **Architecture Decision Records (ADRs)**). Prefer **`Visit`** / **`InspectionVisit`** and **`Assessment*`** flows where the product model is **field activity or scored assessment**; use **`ActionPlan`** where you need **structured RecordAction steps** on the same case, regulatory, or visit-adjacent context (parent targets vary by org—confirm supported types in Object Manager and the [ActionPlanTemplate](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_actionplantemplate.htm) object reference).
- Author **`ActionPlanTemplate`** in **Setup**, then **retrieve** into source control; **`ActionPlanTemplate`** metadata XML is **fragile** across **application programming interface (API)** versions—when using this skill **inside the PSS DX template**, see **`docs/adr-pss.md`** (Action Plan Template source format) for shell-only vs full-template practice; use upstream **`sf-metadata`** / **`sf-deploy`** ([Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills)) when troubleshooting deploy order and **XML Schema Definition (XSD)**.
- Grant budget tracking uses `GrantBudget` + `GrantAllocation`; integrate with Accounting Subledger for financials when required.
- Discovery Framework assessments share objects with **Education Cloud Integrated Case Companion (ICC)** (`AssessmentQuestion*`); versioning is managed via `AssessmentQuestionVersion`.

---

## Reference files

- [references/sf-pss-core-objects.md](references/sf-pss-core-objects.md) — Core PSS object API names, key fields, and relationships

## Cross-skill integration

- **OmniStudio / DataRaptor / Integration Procedure (IP)** for UI and orchestration: use [Jaganpro/sf-skills](https://github.com/Jaganpro/sf-skills) skills such as `sf-industry-commoncore-omniscript`, `sf-industry-commoncore-datamapper`, `sf-industry-commoncore-integration-procedure` for implementation mechanics; use **this skill** for which **PSS objects** to persist.
- **Discovery Framework** assessment authoring overlaps Education Cloud ICC; confirm org licensing and use official Discovery Framework docs alongside this skill.
