# PSS Object-Name Corrections Plan

**Source of truth:** [sf-pss-core-objects.md](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — org-verified v68.0 on kytc 2026-09-15, supplemented by field-level queries against dupage1 2026-09-16
**Files in scope:** `agents/sf-pss-architect.md`, `docs/adr-pss.md`, `docs/plan-pss-phased.md`
**Already fixed:** Grants domain (`Grant*` → `Funding*`) in `sf-pss-architect.md`

---

## Priority 1 — Deploy-breaking (wrong field or object name; will fail at compile/runtime)

### `RegulatoryTxn` and `RegulatoryTxnParty` do not exist

**Why:** [sf-pss-core-objects.md §Regulatory framework](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) explicitly states: "There is no `RegulatoryTxn` or `RegulatoryTransaction` object in PSS v68.0." The fee-side record is `RegulatoryTrxnFee`; the authorization lifecycle lives on `BusinessLicense` + `BusinessLicenseApplication`.

Any SOQL query, Flow variable, Apex field reference, or OmniStudio Data Mapper targeting `RegulatoryTxn` will fail at deploy or runtime.

**Fix per file:**
- `agents/sf-pss-architect.md` lines 38, 82: remove `RegulatoryTxn` from the native-only object list; remove from the complaints/appeals pattern. Replace complaint + enforcement references with `PublicComplaint`, `ComplaintCase`, `RegulatoryCodeViolation`, `ViolationEnforcementAction`.
- `docs/adr-pss.md` lines 21, 35–217 (many): replace every `RegulatoryTxn` spine reference with `BusinessLicense` / `BusinessLicenseApplication` for the authorization lifecycle. Remove `RegulatoryTxnParty`; party roles attach to the application or license record.
- `docs/plan-pss-phased.md` lines 7, 30, 87, 95, 120, 154, 181, 196: same replacement. The Mermaid diagram and OmniScript/Flow prescriptions need to be redrawn around `BusinessLicenseApplication` as the regulatory spine.

---

### `ApplicantId` on `IndividualApplication` is not a field

**Why:** [sf-pss-core-objects.md §IndividualApplication](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — person applicant field is `ContactId`. No `ApplicantId` on `IndividualApplication`. Note: `ApplicantId` DOES exist on `PreliminaryApplicationRef` and `BusinessLicenseApplication` — this correction applies to `IndividualApplication` only.

**Fix per file:**
- `docs/adr-pss.md` line 19, 150: replace `ApplicantId → Individual` with `ContactId → Contact` where the context is `IndividualApplication`.
- `docs/plan-pss-phased.md` lines 82, 88 (OmniScript/Flow for `IndividualApplication`): replace `ApplicantId` with `ContactId`; replace `Individual` with `Contact` as the person identity record.

---

### `StatusCode` on `IndividualApplication` is not a field

**Why:** [sf-pss-core-objects.md §IndividualApplication](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — lifecycle field is `InternalStatus`; org-extended values are on `Status`. "Neither field is `StatusCode`."

**Fix per file:**
- `docs/plan-pss-phased.md` lines 120, 196: replace `IndividualApplication.StatusCode` with `InternalStatus` (for PSS lifecycle transitions) or `Status` (for org-extended values). The Mermaid/Flow prescriptions and open items need to reflect this.
- `agents/sf-pss-architect.md` line 39: the example `StatusCode` in Principle 2 should be replaced with `InternalStatus` or `BenefitStatus` depending on the object being described.
- `agents/sf-pss-architect.md` line 69: `StatusCode` on `Benefit` → `BenefitStatus`.

---

### `SubmittedDate` on `IndividualApplication` is not a field

**Why:** [sf-pss-core-objects.md §IndividualApplication](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "No `SubmittedDate` field; nearest equivalent is `AppliedDate` (DateTime)."

**Fix per file:**
- `docs/adr-pss.md` line 154: replace `SubmittedDate` with `AppliedDate`.
- `docs/plan-pss-phased.md` line 120: same.

---

## Priority 2 — Wrong object names (objects exist under different API names)

### `Permit`, `PermitApplication`, `PermitApplicationReview` do not exist

**Why:** [sf-pss-core-objects.md §Licensing and permitting](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "PSS does not have separate Permit, PermitApplication, or PermitType objects. Both licenses and permits are modeled on `BusinessLicense` + `BusinessLicenseApplication`." See also: [Application and Authorization data model](https://developer.salesforce.com/docs/platform/data-models/guide/application-authorization.html).

**Fix:** `agents/sf-pss-architect.md` lines 38, 51–54: remove `Permit`, `PermitApplication`, `PermitApplicationReview` from the native-only list and the licensing/permitting pattern. Replace with `BusinessLicense` / `BusinessLicenseApplication`; permit vs. license distinction is made via `RegulatoryAuthorizationType.RegulatoryAuthCategory`.

---

### `LicenseType`, `PermitType` do not exist

**Why:** [sf-pss-core-objects.md §RegulatoryAuthorizationType](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "There is no separate `LicenseType` object — use `RegulatoryAuthorizationType`." See also: [Regulatory Area, Fees, and Enforcement data model](https://developer.salesforce.com/docs/platform/data-models/guide/regulatory-area-fees-enforcement.html).

**Fix:**
- `agents/sf-pss-architect.md` lines 53, 97: replace `LicenseType`/`PermitType` with `RegulatoryAuthorizationType`. Fee schedule lives on `RegulatoryAuthorizationType` fields; the lookup from `BusinessLicenseApplication` is `LicenseTypeId → RegulatoryAuthorizationType`.
- `docs/plan-pss-phased.md` line 73: same.
- `docs/adr-pss.md` (wherever used): same.

---

### `LicensedEntity` is not a PSS object

**Why:** [sf-pss-core-objects.md §BusinessLicense](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — the authorized party is tracked via `AccountId` (organization) or `ContactId` (individual) on `BusinessLicense`.

**Fix:**
- `agents/sf-pss-architect.md` line 50: remove `LicensedEntity` from the anchor pattern. Replace with "`BusinessLicense.AccountId` → `Account`".
- `docs/adr-pss.md` line 38, `docs/plan-pss-phased.md` line 73: same removal.

---

### `ExpirationDate` on `BusinessLicense` is not a field

**Why:** [sf-pss-core-objects.md §BusinessLicense](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — expiry field is `PeriodEnd` (DateTime).

**Fix:** `agents/sf-pss-architect.md` line 54: replace "Flow on `ExpirationDate` approaching" with "`PeriodEnd`".

---

### `LicensedEntityId` is not a field on any PSS object

**Why:** [sf-pss-core-objects.md §BusinessLicense](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — no `LicensedEntityId` field exists; authorized-party links are `AccountId` and `ContactId`.

**Fix:** `agents/sf-pss-architect.md` line 39 (Principle 2): replace the example field name `LicensedEntityId` with a real PSS field like `RegulatoryAuthorizationTypeId` or `ContactId`.

---

### `Violation` (standalone) → `RegulatoryCodeViolation`

**Why:** [sf-pss-core-objects.md §Enforcement](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "PSS v68.0 does not have a standalone `EnforcementAction` or `Violation` object." See also: [Regulatory Area, Fees, and Enforcement data model](https://developer.salesforce.com/docs/platform/data-models/guide/regulatory-area-fees-enforcement.html).

**Fix:** `agents/sf-pss-architect.md` line 58: replace "`Permit` or `Violation` → `Inspection`" with "`BusinessLicense` or `RegulatoryCodeViolation` → `Visit`/`InspectionType`".

---

### `InspectionFinding` → `RegulatoryCodeViolation`; `EnforcementAction` → `ViolationEnforcementAction`

**Why:** [sf-pss-core-objects.md §Enforcement](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md). See also: [Visits, Inspections and Dynamic Assessments data model](https://developer.salesforce.com/docs/platform/data-models/guide/visits-inspections-dynamic-assessments.html).

**Fix:** `agents/sf-pss-architect.md` lines 59, 62, 82: replace `InspectionFinding` with `RegulatoryCodeViolation`; replace `EnforcementAction` with `ViolationEnforcementAction`.

---

### `Inspection` (standalone), `InspectionChecklistItem` do not exist

**Why:** [sf-pss-core-objects.md §Inspections](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "PSS v68.0 does not have standalone `Inspection`, `InspectionChecklistItem`, or `InspectionFinding` objects." Checklist questions go through the Discovery Framework (`AssessmentQuestion*` via `InspectionAssessmentInd`). See also: [Visits, Inspections and Dynamic Assessments data model](https://developer.salesforce.com/docs/platform/data-models/guide/visits-inspections-dynamic-assessments.html).

**Fix:**
- `agents/sf-pss-architect.md` lines 58, 60–62, 98: replace `Inspection` (standalone) with `InspectionType` + `InspectionAssessmentInd`; replace `InspectionChecklistItem` with `AssessmentQuestion*` via `InspectionAssessmentInd`. The `Visit` object schedules and records the field visit.
- `docs/plan-pss-phased.md` line 174: replace the dependency note on `Inspection` with the correct objects.

---

### `Complaint`, `ComplaintFinding`, `ComplaintRemediation` → `PublicComplaint`, `ComplaintCase`, `ComplaintParticipant`

**Why:** [sf-pss-core-objects.md §Complaint management](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "PSS v68.0 does not have `Complaint`, `ComplaintFinding`, or `ComplaintRemediation` as standalone objects."

**Fix:** `agents/sf-pss-architect.md` lines 38, 80, 83: replace the entire complaint chain with `PublicComplaint` / `ComplaintCase` + `ComplaintParticipant`. Remove from the native-only object list or correct to `PublicComplaint`.

---

### `Appeal`, `AppealHearing`, `AppealDecision` have no PSS equivalents

**Why:** [sf-pss-core-objects.md §Appeals](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "PSS v68.0 does not have `Appeal`, `AppealAssociation`, `AppealHearing`, or `AppealDecision` objects. Appeals are not modeled in the base PSS package." If needed: custom objects or `CaseProceedingComplaint`/`CaseProceedingResult` from the Investigative Case Management add-on.

**Fix:** `agents/sf-pss-architect.md` lines 38, 81, 82: remove `Appeal` from the native-only list; remove the `Appeal` → `AppealHearing` → `AppealDecision` chain. Add a note that appeals require custom objects or the Investigative Case Management add-on.

---

## Priority 3 — Conceptual misuse (won't break deploys immediately, but will mislead architects)

### `Individual` described as person-identity spine

**Why:** [sf-pss-core-objects.md §Party and identity](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "This is the Salesforce GDPR/privacy management object, not a person-identity record. It does NOT have `FirstName`, `LastName`, or `GenderIdentity`. Use `Contact` as the person record."

**Fix:**
- `agents/sf-pss-architect.md` lines 41, 69: Principle 4 should read "person identity resolves through `Contact`; `Individual` is the GDPR privacy companion." The `ProgramEnrollment` note at line 69 should reference `ContactId`/`AccountId`, not `Individual`.
- `docs/adr-pss.md` line 22, `docs/plan-pss-phased.md` lines 7, 9: same conceptual fix in the domain coverage table and principles.

---

### `RegulatoryAuthorizationType.Category` is not a field — use `RegulatoryAuthCategory`

**Why:** Org-verified on dupage1 (v68.0) — `Category` does not exist on `RegulatoryAuthorizationType`. The correct picklist field is `RegulatoryAuthCategory`. Any reference to `RegulatoryAuthorizationType.Category = Permit` in code, Flow, or documentation is wrong.

**Fix:**
- `agents/sf-pss-architect.md` line 50 (licensing pattern): replace `RegulatoryAuthorizationType.Category = Permit` with `RegulatoryAuthorizationType.RegulatoryAuthCategory`.
- `docs/adr-pss.md`, `docs/plan-pss-phased.md`: update wherever `RegulatoryAuthorizationType.Category` is cited.

---

### `ApplicationCategory` example values in `adr-pss.md` are wrong

**Why:** `ApplicationCategory` exists on both `PreliminaryApplicationRef` and `IndividualApplication` (org-verified). But the example values in `docs/adr-pss.md` line 141 (License, Registration, Complaint) do not match the actual PSS picklist. For licensing/permit context use the sibling `Category` field (License, Permit, Grant Application, Letter of Intent).

**Fix:** `docs/adr-pss.md` lines 140–141: correct example picklist values — for grantmaking/education context: Basic, Early Decision, Regular Decision, Special; for licensing/permit context use `Category` (License, Permit, Grant Application, Letter of Intent).

---

### `ApplicationTypeId` described as a lookup

**Why:** [sf-pss-core-objects.md §IndividualApplication](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md) — "`ApplicationType` is a restricted picklist (Change Of Circumstance, New, Recertification, Renewal), not a lookup to an `ApplicationType` object."

**Fix:** `docs/adr-pss.md` line 149: rename `ApplicationTypeId` to `ApplicationType`; describe as a restricted picklist, not a lookup.

---

### `ChannelCode` on `IndividualApplication` is not a field

**Why:** Not listed in [sf-pss-core-objects.md §IndividualApplication](../skills/sf-pss-data-dev/references/sf-pss-core-objects.md).

**Fix:** `docs/adr-pss.md` line 153: remove `ChannelCode` or flag as a custom field to be added if submission channel tracking is required.

---

## What not to change

- `PreliminaryApplicationRef.ApplicantId`, `.ApplicationCategory`, `.SubmissionDate` — all exist (org-verified dupage1); prior plan wrongly flagged these; adr-pss.md and plan-pss-phased.md were correct.
- `ProgramEnrollment.ContactId` / `.AccountId` links are correct.
- `RegulatoryCode` as the citation spine across enforcement and inspections is correct.
- The Discovery Framework (`AssessmentQuestion*`) section in `sf-pss-architect.md` is correct.
- The `FundingOpportunity` / `FundingAward` grants chain is already fixed.

## Unverified — check on a benefits-focused org

- `BenefitAssignment.AccountId` / `.ContactId` — not present on dupage1 (field service org). The reference lists these as enrollee links but they were absent from the FieldDefinition query. Verify on an org with Social Program Management installed before using or changing.

---

## Suggested sequencing

1. Fix `agents/sf-pss-architect.md` first — it's the agent's instruction set and lowest risk.
2. Fix `docs/adr-pss.md` next — it's the design record. The `RegulatoryTxn` spine replacement is the biggest structural change; confirm the intended pattern (should be `BusinessLicenseApplication`) before editing.
3. Fix `docs/plan-pss-phased.md` last — it references ADR decisions, so ADR should be correct before the phased plan is revised. The Mermaid diagram will need to be redrawn.
