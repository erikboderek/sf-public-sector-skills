# PSS Naming Drift Audit Report

**Reference:** Winter '27 (v68.0) PSS Developer Guide (objects.md, Sept 11 2026) + Online Data Model Gallery + kytc org live verification (2026-09-15)
**Audit date:** 2026-09-15
**Repo scanned:** sf-public-sector-skills
**Org verified against:** kytc (`erik@kytc1.demo`) — EntityDefinition + FieldDefinition SOQL + Apex anonymous picklist describe

---

## Section 1: Confirmed Mismatches

All items below were verified against objects.md section-by-section reads. "Not a field" means the field was searched within the object's section and not found.

| # | Location | What is in repo | Correct API name / value | Source | Severity |
|---|---|---|---|---|---|
| 1 | sf-pss-core-objects.md — BusinessLicense | `LicenseNumber` | `Identifier` (string) | objects.md §BusinessLicense | 🔴 |
| 2 | sf-pss-core-objects.md — BusinessLicense | `LicenseType` (picklist→LicenseType.Name) | `RegulatoryAuthorizationTypeId` (Lookup→`RegulatoryAuthorizationType`); no `LicenseType` field or object exists | objects.md §BusinessLicense | 🔴 |
| 3 | sf-pss-core-objects.md — BusinessLicense | `LicensedEntityId` (Lookup→Account) | Not a field; use `AccountId` (Lookup→Account) and `ContactId` (Lookup→Contact) | objects.md §BusinessLicense | 🔴 |
| 4 | sf-pss-core-objects.md — BusinessLicense | `IssuingAgencyId` (Lookup→Account) | Not a lookup field; use `Issuer` (string — issuing authority name) | objects.md §BusinessLicense | 🔴 |
| 5 | sf-pss-core-objects.md — BusinessLicense | `StatusCode` with values Active/Suspended/Revoked/Expired | Field is `Status` (picklist); values are **Draft, Inactive, Revoked, Verified** — not Active/Suspended/Expired | objects.md §BusinessLicense | 🔴 |
| 6 | sf-pss-core-objects.md — BusinessLicense | `EffectiveDate` | `PeriodStart` (dateTime) | objects.md §BusinessLicense | 🔴 |
| 7 | sf-pss-core-objects.md — BusinessLicense | `ExpirationDate` | `PeriodEnd` (dateTime) | objects.md §BusinessLicense | 🔴 |
| 8 | sf-pss-core-objects.md — IndividualApplication | `ApplicationTypeId` (Lookup→ApplicationType object) | `ApplicationType` is a **picklist** field, not a lookup; values are Change Of Circumstance, New, Recertification, Renewal. No `ApplicationType` PSS object exists. | objects.md §IndividualApplication | 🔴 |
| 9 | sf-pss-core-objects.md — IndividualApplication | `ApplicantId` (Lookup→Individual) | Not a field; use `ContactId` (Lookup→Contact) or `AccountId` (Lookup→Account) | objects.md §IndividualApplication | 🔴 |
| 10 | sf-pss-core-objects.md — IndividualApplication | `StatusCode` with values Draft/Submitted/Under Review/Approved/Rejected | Two status fields exist (org-verified): **`InternalStatus`** (PSS lifecycle — Invited/In Progress/Submitted/Application Accepted/Revision Requested/In Review/Approved/Denied) and **`Status`** (org-extended — Interest/Draft/In Progress/Submitted/Application Accepted/In Review/Approved/Complete/Denied/Distribution). Neither is `StatusCode` and none of the repo picklist values match. | kytc org + objects.md §IndividualApplication | 🔴 |
| 11 | sf-pss-core-objects.md — IndividualApplication | `SubmittedDate` | Not a field; closest is `AppliedDate` (dateTime — date application was received) | objects.md §IndividualApplication | 🟡 |
| 12 | sf-pss-core-objects.md — IndividualApplication | `ChannelCode` with values Online/In Person/Phone/Mail | Not a field on IndividualApplication | objects.md §IndividualApplication | 🟡 |
| 13 | sf-pss-core-objects.md — PreliminaryApplicationRef | `ApplicantId` (Poly→Contact/User/Account) | Not a field on PreliminaryApplicationRef; no applicant lookup exists on this object | objects.md §PreliminaryApplicationRef | 🔴 |
| 14 | sf-pss-core-objects.md — PreliminaryApplicationRef | `ApplicationCategory` (picklist) | `ApplicationCategory` is a field on **IndividualApplication** (values: Basic, Early Decision, Regular Decision, Special), not on PreliminaryApplicationRef. PreliminaryApplicationRef has `ApplicationType` (different picklist). | objects.md §IndividualApplication, §PreliminaryApplicationRef | 🔴 |
| 15 | sf-pss-core-objects.md — PreliminaryApplicationRef | `SubmissionDate` (Date) | Not a field on PreliminaryApplicationRef | objects.md §PreliminaryApplicationRef | 🟡 |
| 16 | sf-pss-core-objects.md — RegulatoryCode | `RegulatoryCodeNumber` | Not a field; the code identifier is the standard `Name` field (string) | objects.md §RegulatoryCode | 🔴 |
| 17 | sf-pss-core-objects.md — RegulatoryCode | `EffectiveDate` | Not a field; actual fields are `EffectiveFrom` (dateTime) and `EffectiveTo` (dateTime) | objects.md §RegulatoryCode | 🔴 |
| 18 | sf-pss-core-objects.md — RegulatoryCode | `StatusCode` with values Active/Superseded/Repealed | Not a picklist; field is `IsActive` (boolean) | objects.md §RegulatoryCode | 🔴 |
| 19 | sf-pss-core-objects.md — RegulatoryCode | `JurisdictionId` (Lookup→Account) | `RegulatoryAuthorityId` (reference to `RegulatoryAuthority`, not Account) | objects.md §RegulatoryCode | 🔴 |
| 20 | sf-pss-core-objects.md — Program | `ProgramTypeId` (Lookup→ProgramType) | Not a field on Program; no `ProgramType` object exists in PSS | objects.md §Program | 🔴 |
| 21 | sf-pss-core-objects.md — Program | `StatusCode` with values Active/Inactive/Draft | Field is `Status` (picklist); values are **Active, Cancelled, Completed, Planned** — Inactive and Draft do not exist | objects.md §Program | 🔴 |
| 22 | sf-pss-core-objects.md — Program | `OwningAgencyId` (Lookup→Account) | Not a field on Program | objects.md §Program | 🟡 |
| 23 | sf-pss-core-objects.md — Program | `FundingSourceCode` (Federal/State/Local/Private) | Not a field on Program | objects.md §Program | 🟡 |
| 24 | sf-pss-core-objects.md — ProgramEnrollment | `EnrollmentStatusCode` | Field is `Status` (picklist); values are **Applied, Completed, Denied, In Progress, Waitlisted, Withdrawn** — the field name is wrong and the values are wrong | objects.md §ProgramEnrollment | 🔴 |
| 25 | sf-pss-core-objects.md — ProgramEnrollment | `ProgramCohortId` (child link) | `ProgramCohortId` is a field on **ProgramCohortMember** (the junction object), not on ProgramEnrollment | objects.md §ProgramCohortMember | 🔴 |
| 26 | sf-pss-core-objects.md — BenefitType | `BenefitTypeCode` | Not a field on BenefitType | objects.md §BenefitType | 🟡 |
| 27 | sf-pss-core-objects.md — BenefitType | `DeliveryMethodCode` | Not a field (org-verified). Field is `Type` (picklist). `Category` also exists on `BenefitType` in kytc. | kytc org + objects.md §BenefitType | 🟡 |
| 28 | sf-pss-core-objects.md — Benefit | `RecipientId` (Lookup→Individual) | Not a field on Benefit | objects.md §Benefit | 🔴 |
| 29 | sf-pss-core-objects.md — Benefit | `ProgramEnrollmentId` (Lookup→ProgramEnrollment) | Not a field on Benefit; benefit-to-program link is `ProgramId` (Lookup→Program). Enrollment is tracked via `BenefitAssignment`. | objects.md §Benefit, §BenefitAssignment | 🔴 |
| 30 | sf-pss-core-objects.md — Benefit | `StatusCode` with values Pending/Active/Suspended/Closed | Field is `BenefitStatus` (picklist); values are **Active, Cancelled, Completed, Planned** — none of the four repo values exist | objects.md §Benefit | 🔴 |
| 31 | sf-pss-core-objects.md — Benefit | `StartDate` (Date) | `StartDateTime` (dateTime — not a date, a datetime) | objects.md §Benefit | 🟡 |
| 32 | sf-pss-core-objects.md — Benefit | `EndDate` (Date) | `EndDateTime` (dateTime — not a date, a datetime) | objects.md §Benefit | 🟡 |
| 33 | sf-pss-core-objects.md — Benefit | `PaymentMethodCode` | `PaymentMethodType` (picklist, available API v67.0+) | objects.md §Benefit | 🟡 |
| 34 | sf-pss-core-objects.md — BenefitDisbursement | `StatusCode` | Not confirmed; `BenefitDisbursement` section not fully reviewed — see Section 2 | — | 🟡 |
| 35 | sf-pss-core-objects.md — AssessmentQuestion | `IsRequired` | Not a field on AssessmentQuestion | objects.md §AssessmentQuestion | 🟡 |
| 36 | sf-pss-core-objects.md — AssessmentQuestion | `ResponseType` | Not a field; question format captured by `DataType` (picklist with 20+ values) | objects.md §AssessmentQuestion | 🟡 |
| 37 | sf-pss-core-objects.md — AssessmentQuestionVersion | `AssessmentQuestionSetId` | Not on AssessmentQuestionVersion; the question-to-set junction is `AssessmentQuestionAssignment`, which carries both `AssessmentQuestionId` and `AssessmentQuestionSetId` | objects.md §AssessmentQuestionVersion, §AssessmentQuestionAssignment | 🔴 |
| 38 | sf-pss-core-objects.md — AssessmentQuestionVersion | `VersionNumber` | Not a field; version state captured by `IsActive` (boolean) and `ActivationDateTime` (dateTime) | objects.md §AssessmentQuestionVersion | 🟡 |
| 39 | sf-pss-core-objects.md — AssessmentQuestionVersion | `ActiveFromDate` (Date) | `ActivationDateTime` (dateTime — not a date, a datetime) | objects.md §AssessmentQuestionVersion | 🟡 |
| 40 | docs/adr-pss.md — InteractionSummary | `SummaryNotes` | `MeetingNotes` (textarea) | objects.md §InteractionSummary | 🟡 |
| 41 | docs/adr-pss.md — InteractionSummary | `ChannelCode` | Not a field on InteractionSummary | objects.md §InteractionSummary | 🟡 |
| 42 | sf-pss-core-objects.md — PreliminaryApplicationRef | `ApplicationType` described as a free-form "wizard-type key" | `ApplicationType` is a **restricted picklist** with only 5 values: `ApplicationForm`, `BusinessLicenseApplication`, `BusinessPrescreening`, `IndividualApplication`, `PublicComplaint` — it cannot hold arbitrary wizard keys | objects.md §PreliminaryApplicationRef | 🟡 |

Severity key: 🔴 breaks deploy or produces wrong SOQL / Apex / Flow references | 🟡 wrong but won't cause a deploy failure by itself | 🟢 cosmetic label only

---

## Section 2: Ambiguous / Needs Verification

| Item | Repo claim | Finding | Uncertainty |
|---|---|---|---|
| `ProgramCohort` | Listed as a key Program Management object in SKILL.md | `ProgramCohortMember.ProgramCohortId` declares `Refers To: ProgramCohort (the master object)`, confirming the object exists. But `ProgramCohort` is not listed in the Chapter 2 object index. | Likely accessible but not fully documented in v68.0 PDF. Use with caution; verify via `describeSObjects()`. |
| `BenefitDisbursement.StatusCode` | Listed as `StatusCode` with values pending/active etc. in sf-pss-core-objects.md | `BenefitDisbursement` section at line 6957 was not fully reviewed for all fields. | Read lines 6469–6890 of objects.md to confirm field name and picklist values. |
| `Benefit.BenefitDisbursement` child | Repo lists BenefitDisbursement as a child of Benefit with `DisbursementDate, Amount, StatusCode, PaymentMethodCode` | BenefitDisbursement exists as an object; parent relationship and field names not fully verified in this audit. | Verify field names in objects.md §BenefitDisbursement. |
| `Individual.GenderIdentity` (in refs) | Repo says `Individual` has `GenderIdentity` field | `Individual` exists in the org (org-verified) but is a GDPR privacy object — it has no `GenderIdentity`, no `FirstName`, no `LastName`. `GenderIdentity` is on `Contact`. The repo's description of `Individual` as a person-identity spine is wrong; `Contact` is the person record. | Move `FirstName`, `LastName`, `GenderIdentity` references to `Contact`; keep `Individual` only for privacy/consent use cases. |
| `RegulatoryAuthorizationType` description | Repo says it "defines actions a RegulatoryCode authorizes; used by RegulatoryTxn to validate transaction type" | The object EXISTS (line 37160) and represents "the authorization issued by the regulatory body" — essentially a license/permit type template. Used by `BusinessLicense.RegulatoryAuthorizationTypeId`. The repo description is imprecise but the object name is correct. | Clarify description in sf-pss-core-objects.md; no deploy risk from name alone. |
| `BusRegAuthorizationType` | SKILL.md and sf-pss-core-objects.md do not mention `BusRegAuthorizationType` | `BusRegAuthorizationType` IS a distinct, real PSS object (line 10546) representing "the association between authorization activity and license or permit type" — a junction/configuration object. Different from `RegulatoryAuthorizationType`. | Should be added to the references if the org uses licensing workflows. |
| `IndividualApplication.ApplicationCategory` picklist values | Repo uses "License / Registration / Complaint" as example grouping categories | Actual `ApplicationCategory` field in objects.md (API version 68.0) shows values Basic / Early Decision / Regular Decision / Special — these are Grantmaking/Education context values. PSS licensing/permit context may use the sibling `Category` field (License / Permit / Grant Application / Letter of Intent) instead. | Verify which field and which values align to your PSS program context before coding. |

---

## Section 3: Objects in Repo Not Found in Either Source

All objects below were searched in objects.md using `rg -n "^## <Name>"` and by scanning the chapter 2 object list (lines 288–600), and also checked against the six online data model gallery pages. None appear in either source.

### In `skills/sf-pss-data-dev/references/sf-pss-core-objects.md`

| Object name used in repo | Status | Likely correct PSS name |
|---|---|---|
| `Permit` | Not in PSS; no Permit object exists | `BusinessLicense` (PSS uses one "Authorization" object for both licenses and permits) |
| `PermitApplication` | Not in PSS | `BusinessLicenseApplication` |
| `PermitType` | Not in PSS | `RegulatoryAuthorizationType` (the permit/license type template) |
| `Inspection` (standalone) | Not in PSS | `InspectionAssessmentInd` is the inspection-assessment junction; no standalone Inspection object |
| `InspectionChecklistItem` | Not in PSS | No equivalent; checklist-style questions are modeled via `AssessmentQuestion` + `AssessmentQuestionVersion` |
| `InspectionFinding` | Not in PSS | `RegulatoryCodeViolation` is the regulatory violation junction |
| `LicenseType` (as object) | Not in PSS | `RegulatoryAuthorizationType` |
| `RegulatoryTxn` | Not in PSS; no such object exists | `RegulatoryTrxnFee` handles fee calculations associated with applications/inspections/violations. No wrapper "transaction" object exists. |
| `Individual` (as described in repo) | Object EXISTS in kytc org (org-verified). However it is the Salesforce GDPR/privacy object — fields are `BirthDate`, `HasOptedOutProcessing`, consent flags, etc. It has **no** `FirstName`, `LastName`, or `GenderIdentity`. The repo describes it as "Core constituent record; supersedes Contact" with those person-identity fields — that description is wrong. `FirstName`/`LastName`/`GenderIdentity` are `Contact` fields, not `Individual` fields. `Individual` is a privacy-management object, not a person-identity spine. |
| `Party` | Not in PSS chapter 2 object list | Not a standalone accessible object in v68.0 |
| `Grant` | Not in PSS | `FundingAward` |
| `GrantApplication` | Not in PSS | `IndividualApplication` (used for grant applications under Grantmaking license) |
| `GrantBudget` | Not in PSS | `Budget` |
| `GrantAllocation` | Not in PSS | `FundingDisbursement` |
| `GrantOpportunity` | Not in PSS | `FundingOpportunity` |
| `GrantType` | Not in PSS | No equivalent object found in v68.0 |
| `ApplicationType` (as lookup target object) | Not in PSS | `ApplicationType` is a picklist field on `IndividualApplication`, not a separate object |
| `ProgramType` (as lookup target) | Not in PSS | No equivalent object; Program does not have a type lookup |

### In `skills/sf-pss-data-dev/SKILL.md` domain map

| Object name used in SKILL.md | Status | Likely correct PSS name |
|---|---|---|
| `LicensedEntity` | Not in PSS | Not a standalone object; the licensed party is the `AccountId` or `ContactId` on `BusinessLicense` |
| `InspectionQuestion` | Not in PSS | `AssessmentQuestion` (questions are defined in the Discovery Framework, not inspection-specific) |
| `VisitQueue` | Not in PSS | Not a PSS object; visits are assigned via `Visit` and `Visitor` records |
| `Complaint` | Not confirmed in PSS | `PublicComplaint` (line 34672 in objects.md) or `ComplaintCase` |
| `ComplaintAssociation` | Not in PSS | `ComplaintParticipant` (chapter 2 object list) |
| `Appeal` | Not in PSS | No Appeal object family exists in PSS v68.0 |
| `AppealAssociation` | Not in PSS | No equivalent |
| `AppealHearing` | Not in PSS | No equivalent |
| `AppealDecision` | Not in PSS | No equivalent |
| `EnforcementAction` | Not in PSS | `ViolationEnforcementAction` is the correct PSS object |
| `EnforcementActionType` | Not in PSS | Not a standalone object; enforcement actions are typed via `ViolationType` |
| `Violation` (standalone) | Not in PSS | `RegulatoryCodeViolation` (regulatory code violation junction); `ViolationEnforcementAction` (enforcement record) |
| `ViolationEnforcementAction` | Confirmed in PSS (line 39041) | Correct name |
| `ProgramEnrollmentStatusHistory` | Not in PSS object list | The platform auto-generates `ProgramEnrollmentHistory` for tracked field history; no separate status-history object exists |

---

## Section 4: Remediation Plan

### 🔴 Severity — Fix before deploy

These are field and object names that will break SOQL queries, Apex field references, Flow variable assignments, and REST API calls.

**File: `skills/sf-pss-data-dev/references/sf-pss-core-objects.md`**

```
# BusinessLicense field renames
sd 'LicenseNumber' 'Identifier'
sd 'LicenseType\(Picklist→LicenseType\.Name\)' 'RegulatoryAuthorizationTypeId(Lookup→RegulatoryAuthorizationType)'
sd 'LicensedEntityId\(Lookup\(Account\)\)' 'AccountId(Lookup(Account)), ContactId(Lookup(Contact))'
sd 'IssuingAgencyId\(Lookup\(Account\)\)' 'Issuer(string — issuing authority name)'
sd 'StatusCode\(Active/Suspended/Revoked/Expired\)' 'Status(picklist — Draft/Inactive/Revoked/Verified)'
sd 'EffectiveDate' 'PeriodStart'   # in BusinessLicense block only
sd 'ExpirationDate' 'PeriodEnd'    # in BusinessLicense block only

# IndividualApplication field renames
sd 'ApplicationTypeId\(Lookup\(ApplicationType\)\)' 'ApplicationType(picklist — Change Of Circumstance/New/Recertification/Renewal)'
sd 'ApplicantId\(Lookup\(Individual\)\)' 'ContactId(Lookup(Contact)) or AccountId(Lookup(Account))'
sd 'StatusCode\(Draft/Submitted/Under Review/Approved/Rejected\)' 'InternalStatus(picklist — Invited/In Progress/Submitted/Application Accepted/Revision Requested/In Review/Approved/Denied)'  # IndividualApplication block

# PreliminaryApplicationRef — remove nonexistent fields
# Remove line: ApplicantId(Poly→Contact/User/Account)
# Remove line: ApplicationCategory(Picklist)  [move note to IndividualApplication block]
# Remove line: SubmissionDate(Date)

# RegulatoryCode field renames
sd 'RegulatoryCodeNumber' 'Name'   # in RegulatoryCode block only
sd 'EffectiveDate' 'EffectiveFrom / EffectiveTo'  # in RegulatoryCode block only
sd 'StatusCode\(Active/Superseded/Repealed\)' 'IsActive(boolean)'  # in RegulatoryCode block only
sd 'JurisdictionId\(Lookup\(Account\)\)' 'RegulatoryAuthorityId(Lookup(RegulatoryAuthority))'

# Program field renames
sd 'ProgramTypeId\(Lookup\(ProgramType\)\)' '# No ProgramType lookup; removed'
sd 'StatusCode\(Active/Inactive/Draft\)' 'Status(picklist — Active/Cancelled/Completed/Planned)'  # Program block

# ProgramEnrollment field renames
sd 'EnrollmentStatusCode' 'Status'
# Add note: Status picklist values are Applied/Completed/Denied/In Progress/Waitlisted/Withdrawn
# Remove: ProgramCohortId from ProgramEnrollment; add note that cohort membership is via ProgramCohortMember

# Benefit field renames
sd 'RecipientId\(Lookup\(Individual\)\)' '# No RecipientId; benefit recipient determined via BenefitAssignment.AccountId / ContactId'
sd 'ProgramEnrollmentId\(Lookup\(ProgramEnrollment\)\)' 'ProgramId(Lookup(Program))'
sd 'StatusCode\(Pending/Active/Suspended/Closed\)' 'BenefitStatus(picklist — Active/Cancelled/Completed/Planned)'  # Benefit block

# AssessmentQuestionVersion fixes
# Remove: AssessmentQuestionSetId from AssessmentQuestionVersion block
# Add note: question-to-set assignment is via AssessmentQuestionAssignment object
# Remove: VersionNumber
# sd 'ActiveFromDate' 'ActivationDateTime'  # in AssessmentQuestionVersion block
```

**Object name replacements (whole-doc scope):**

```
sd 'Permit\b' 'BusinessLicense'          # as a PSS object reference (not field/picklist value)
sd 'PermitApplication' 'BusinessLicenseApplication'
sd 'PermitType' 'RegulatoryAuthorizationType'
sd 'RegulatoryTxn\b' 'RegulatoryTrxnFee'
sd 'GrantOpportunity' 'FundingOpportunity'
sd 'GrantAllocation' 'FundingDisbursement'
sd 'Grant\b' 'FundingAward'              # standalone Grant references
sd 'GrantApplication' 'IndividualApplication (grantmaking context)'
sd 'GrantBudget' 'Budget'
sd 'ProgramEnrollmentStatusHistory' 'ProgramEnrollmentHistory (auto-generated)'
```

**File: `skills/sf-pss-data-dev/SKILL.md`**

```
sd 'Complaint\b' 'PublicComplaint'
sd 'ComplaintAssociation' 'ComplaintParticipant'
sd 'EnforcementAction\b' 'ViolationEnforcementAction'
sd 'Violation\b' 'RegulatoryCodeViolation'  # standalone Violation object references
```

### 🟡 Severity — Can batch with 🔴 pass

**File: `skills/sf-pss-data-dev/references/sf-pss-core-objects.md`**

- `IndividualApplication.SubmittedDate` → rename to `AppliedDate` (dateTime)
- `IndividualApplication.ChannelCode` → remove; no equivalent field exists; add note if needed
- `Program.OwningAgencyId` → remove; no equivalent field
- `Program.FundingSourceCode` → remove; no equivalent field
- `BenefitType.BenefitTypeCode` → remove; no equivalent field
- `BenefitType.DeliveryMethodCode` → rename to `Type` (picklist: Goods/Monetary/Service)
- `Benefit.StartDate` → `StartDateTime` (dateTime)
- `Benefit.EndDate` → `EndDateTime` (dateTime)
- `Benefit.PaymentMethodCode` → `PaymentMethodType`
- `AssessmentQuestion.IsRequired` → remove; field does not exist
- `AssessmentQuestion.ResponseType` → remove; question format is captured by `DataType`
- `AssessmentQuestionVersion.VersionNumber` → remove; use `IsActive` and `ActivationDateTime`
- `PreliminaryApplicationRef.SubmissionDate` → remove; no equivalent field

**File: `docs/adr-pss.md`**

- `InteractionSummary.SummaryNotes` → rename to `MeetingNotes`
- `InteractionSummary.ChannelCode` → remove; field does not exist on InteractionSummary

**File: `skills/sf-pss-data-dev/SKILL.md`**

- Remove `LicensedEntity` from key objects list (not a PSS object)
- `InspectionQuestion` → `AssessmentQuestion`
- `VisitQueue` → remove (no PSS equivalent)
- Remove Appeal domain row entirely (`Appeal`, `AppealAssociation`, `AppealHearing`, `AppealDecision` do not exist in PSS v68.0)
- `ProgramCohort` → OK to keep as a concept but note that the accessible object is `ProgramCohortMember`
- `ProgramEnrollmentStatusHistory` → `ProgramEnrollmentHistory`
- `EnforcementActionType` → remove (not a standalone object; enforcement type is `ViolationType`)
- `Individual` references → `Contact` for CRM person data; note `Individual` is only a Data Cloud DMO in Tax/Revenue context

### 🟢 Severity — Bulk label cleanup

- All descriptions that say "StatusCode" on objects where the actual field is `Status` or `BenefitStatus` or `IsActive` — update the label in comments and prose only (no deploy impact if the field isn't referenced in code)
- References to `LicenseType object` as a named lookup target → clarify as `RegulatoryAuthorizationType` in prose descriptions
- `RegulatoryAuthorizationType` description in sf-pss-core-objects.md should be updated: it is the license/permit type template (not specifically a "regulatory code action authorizer"); it links to `BusinessLicense` via `RegulatoryAuthorizationTypeId`
- Add `BusRegAuthorizationType` to the references file as a distinct object: it is the junction between `BusinessType` and `RegulatoryAuthorizationType` used in licensing workflows

---

## Audit Notes

1. **`Permit` / `PermitApplication` / `PermitType`** — PSS v68.0 does not have separate Permit objects. The object `BusinessLicense` represents both licenses and permits (it is described as "an authorization issued by a regulatory agency"). `BusinessLicenseApplication` handles both license and permit applications. `RegulatoryAuthorizationType` defines the type of authorization (license vs. permit type, duration, issuing department). This was the most pervasive name family error in the repo.

2. **`RegulatoryTxn`** — No `RegulatoryTxn` or `RegulatoryTransaction` object exists. The fee-side of a regulatory transaction is `RegulatoryTrxnFee`. The authorization itself is `BusinessLicense`. There is no generic transaction wrapper object.

3. **Grant domain** — All grant-specific object names (`Grant`, `GrantApplication`, `GrantOpportunity`, `GrantBudget`, `GrantAllocation`, `GrantType`) are absent from PSS v68.0. The Grantmaking model uses `FundingAward`, `FundingOpportunity`, `FundingDisbursement`, `IndividualApplication`, and `Budget`.

4. **Appeal domain** — No Appeal objects exist anywhere in PSS v68.0. If this use case is needed, it would require custom objects or a future managed package.

5. **`Individual` object** — `Individual` is a Data Cloud DMO (Data Model Object) used only in the Tax and Revenue Management feature. It is not a standard PSS CRM object. All references in the skills files to `Individual` as a person-data record should be `Contact`. `GenderIdentity` is a field on `Contact`.

6. **`AssessmentQuestion` vs `InspectionQuestion`** — PSS uses the Discovery Framework `AssessmentQuestion` object for all checklist/questionnaire scenarios including inspections. There is no separate `InspectionQuestion` or `InspectionChecklistItem` type.

7. **Online docs vs objects.md** — Where they conflicted, objects.md (Sept 11 2026, v68.0) was treated as authoritative. The online gallery pages only provide display names, not API names; display names were mapped to confirmed API names using objects.md.
