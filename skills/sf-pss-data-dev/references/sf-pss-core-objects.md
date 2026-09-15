<!-- Parent: sf-pss-data-dev/SKILL.md -->

# Public Sector Solutions (PSS) core objects reference

Source: [PSS application programming interface (API) overview](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm)
Org-verified: kytc (`erik@kytc1.demo`), Winter '27 (v68.0), 2026-09-15

---

## Licensing and permitting

> PSS does **not** have separate `Permit`, `PermitApplication`, or `PermitType` objects. Both licenses and permits are modeled on `BusinessLicense` + `BusinessLicenseApplication`, with `RegulatoryAuthorizationType` defining the authorization type. A `BusinessLicense` record whose `RegulatoryAuthorizationType.Category = Permit` is a permit. Do not create custom Permit objects.

### `BusinessLicense`

Represents an authorization issued by a regulatory agency (license, permit, or service authorization).

| Field | Type | Notes |
|-------|------|-------|
| `Identifier` | String(255) | Unique identifier for the authorization (replaces the human-readable "license number") |
| `RegulatoryAuthorizationTypeId` | Lookup(RegulatoryAuthorizationType) | Defines the authorization category (License / Permit / Service Request) and its rules |
| `AccountId` | Lookup(Account) | Authorized organization |
| `ContactId` | Lookup(Contact) | Authorized contact (individual) |
| `UserId` | Lookup(User) | Authorized user |
| `Issuer` | String(255) | Name of the issuing regulatory authority (plain string, not a lookup) |
| `Status` | Picklist | Draft, Inactive, Revoked, Verified |
| `PeriodStart` | DateTime | Authorization effective date/time |
| `PeriodEnd` | DateTime | Authorization expiry date/time |
| `IsActive` | Boolean | Whether the authorization is still valid |

### `BusinessLicenseApplication`

- Primary filing record for both license applications and permit applications
- `AccountId` → `Account` (B2B Account or Person Account for sole proprietors)
- `ApplicantId` / `PrimaryOwnerId` → `Contact` (human submitter and primary owner)
- `LicensePermitNameId` → `BusinessLicense` (the issued authorization being applied for or renewed)
- `LicenseTypeId` → `RegulatoryAuthorizationType` (the authorization type — license or permit class)
- Status lifecycle: Draft → Submitted → Under Review → Approved/Rejected (org-configured values on `Status` picklist)

### `RegulatoryAuthorizationType`

- Defines the authorization class (license type, permit class, service authorization type)
- `Category` picklist: License, Permit, Service Request
- Linked from `BusinessLicense.RegulatoryAuthorizationTypeId` and `BusinessLicenseApplication.LicenseTypeId`
- **There is no separate `LicenseType` object** — use `RegulatoryAuthorizationType`

### `BusRegAuthorizationType`

- Junction object linking `BusinessType` to `RegulatoryAuthorizationType`
- Used in licensing workflows where a business type requires specific authorization types
- Distinct from `RegulatoryAuthorizationType` itself

---

## Inspections

> PSS v68.0 does **not** have standalone `Inspection`, `InspectionChecklistItem`, or `InspectionFinding` objects. Field compliance findings use `RegulatoryCodeViolation` and `ViolationEnforcementAction`. Dynamic assessment questions use the Discovery Framework (`AssessmentQuestion*`). The objects that do exist for inspections are `InspectionType` and `InspectionAssessmentInd` (the junction between an inspection type and an assessment indicator).

### `InspectionType`

- Reference object defining the inspection category and its rules
- Drives what assessment questions are presented via the Discovery Framework

### `InspectionAssessmentInd`

- Junction between an inspection type and an `AssessmentIndicator`
- Links the inspection to Discovery Framework scored questions

---

## Regulatory framework

### `RegulatoryCode`

| Field | Type | Notes |
|-------|------|-------|
| `Name` | String(255) | Code citation (e.g., "IBC 1001.1") — there is no separate `RegulatoryCodeNumber` field |
| `Description` | String(255) | |
| `EffectiveFrom` | DateTime | Start of this code's effective period |
| `EffectiveTo` | DateTime | End of this code's effective period |
| `IsActive` | Boolean (Formula) | Whether the code is currently in effect — there is no `StatusCode` picklist |
| `RegulatoryAuthorityId` | Master-Detail(RegulatoryAuthority) | Owning regulatory authority — the lookup target is `RegulatoryAuthority`, not `Account` |

### `RegulatoryAuthority`

- Represents the regulatory body (agency, department, jurisdiction)
- Parent of `RegulatoryCode` via `RegulatoryAuthorityId`

### `RegulatoryTrxnFee`

- Fee calculation record associated with a regulatory transaction
- **There is no `RegulatoryTxn` or `RegulatoryTransaction` object** in PSS v68.0
- `RegulatoryTrxnFee` handles fee-side data; authorization lifecycle lives on `BusinessLicense` + `BusinessLicenseApplication`

---

## Applications

**Applicant type:** `IndividualApplication` = person/program intake where the applicant is a `Contact` (or `Account`). `BusinessLicenseApplication` = business license apply or renew. Do not use `IndividualApplication` as a stand-in for a business license application. `PreliminaryApplicationRef` = the portal-side save-and-resume draft that precedes either filing parent.

### `PreliminaryApplicationRef`

Native PSS object that tracks a saved-but-not-yet-submitted application (draft / resume state).

| Field | Type | Notes |
|-------|------|-------|
| `ApplicationName` | String(255) | Auto-generated draft name shown to the citizen |
| `ApplicationType` | Restricted Picklist | **Five values only:** `ApplicationForm`, `BusinessLicenseApplication`, `BusinessPrescreening`, `IndividualApplication`, `PublicComplaint` — this is not a free-form wizard key |
| `BusinessAccountNameId` | Lookup(Account) | Business-flow account link; copied to `BusinessLicenseApplication.AccountId` on promotion |
| `IsSubmitted` | Boolean | Draft-vs-submitted flag; flip to `true` on promotion |
| `SavedApplicationUrl` | URL(255) | Resume deep-link back into the OmniScript / Experience Cloud page |

- **No `ApplicantId` field** — applicant identity is not tracked on the preliminary record
- **No `ApplicationCategory` field** — that field lives on `IndividualApplication`
- **No `SubmissionDate` field** — stamp `AppliedDate` on the filing parent on promotion

### `IndividualApplication`

| Field | Type | Notes |
|-------|------|-------|
| `ApplicationReferenceNumber` | String | Auto-generated reference shown to the citizen |
| `ApplicationType` | Picklist | Change Of Circumstance, New, Recertification, Renewal — **picklist, not a lookup; there is no `ApplicationType` object** |
| `ApplicationCategory` | Picklist | Basic, Early Decision, Regular Decision, Special (grantmaking/education context values) |
| `Category` | Picklist | License, Permit, Grant Application, Letter of Intent — use this for PSS licensing/permit context |
| `ContactId` | Lookup(Contact) | Person applicant — **no `ApplicantId` field** |
| `AccountId` | Lookup(Account) | Organization applicant |
| `InternalStatus` | Picklist | PSS lifecycle: Invited, In Progress, Submitted, Application Accepted, Revision Requested, In Review, Approved, Denied |
| `Status` | Picklist | Org-extended values: Interest, Draft, In Progress, Submitted, Application Accepted, In Review, Approved, Complete, Denied, Distribution… — **neither field is `StatusCode`** |
| `AppliedDate` | DateTime | Date the application was received — **no `SubmittedDate` field** |
| `IsSubmitted` | Boolean | Whether the application has been submitted |

### `ApplicationForm`

- Confirm parent object in Object Manager for your PSS release — may attach to `IndividualApplication` or `BusinessLicenseApplication`
- Groups fields into sections via `ApplicationFormSection`

### `ApplicationFormField`

- Child of `ApplicationFormSection`
- Fields: `Label`, `DataType`, `FieldValue`
- **No `IsRequired` field** — requirement is configured on the assessment/form definition

---

## Benefit management

### `BenefitType`

- Reference; defines benefit category
- Fields: `Name`, `Type` (picklist: Goods, Monetary, Service), `Category` (picklist)
- **No `BenefitTypeCode` field, no `DeliveryMethodCode` field**

### `Benefit`

| Field | Type | Notes |
|-------|------|-------|
| `BenefitTypeId` | Master-Detail(BenefitType) | |
| `ProgramId` | Lookup(Program) | Program this benefit is associated with — **no `ProgramEnrollmentId` field**; enrollment-to-benefit link is via `BenefitAssignment` |
| `BenefitStatus` | Picklist | Active, Planned, Completed, Cancelled — **no `StatusCode` field; none of Pending/Active/Suspended/Closed** |
| `StartDateTime` | DateTime | **Not `StartDate`** |
| `EndDateTime` | DateTime | **Not `EndDate`** |

- **No `RecipientId` field** — benefit recipient is tracked via `BenefitAssignment.AccountId` / `ContactId`

### `BenefitDisbursement`

- Child of `Benefit`
- Fields: `DisbursementDate`, `Amount`, `PaymentMethodType` (not `PaymentMethodCode`)

### `BenefitAssignment`

- Maps `Benefit` to the enrollee (`AccountId` / `ContactId`)
- Links to `ProgramEnrollment` via `ProgramEnrollmentId`

---

## Program management

### `Program`

| Field | Type | Notes |
|-------|------|-------|
| `Name` | String(255) | |
| `Status` | Picklist | Active, Planned, Completed, Cancelled — **no `StatusCode`; no Inactive or Draft values** |
| `ParentProgramId` | Lookup(Program) | Optional program hierarchy |

- **No `ProgramTypeId` field — there is no `ProgramType` object in PSS v68.0**
- **No `OwningAgencyId` field**
- **No `FundingSourceCode` field**

### `ProgramEnrollment`

- Enrollee-to-program join
- Fields: `AccountId`, `ContactId`, `ProgramId` (Master-Detail), `StartDate`, `EndDate`, `ApplicationDate`, `IsActive`, `Status`
- `Status` picklist: Applied, Completed, Denied, In Progress, Waitlisted, Withdrawn — **no `EnrollmentStatusCode` field**
- **No `ProgramCohortId` field on this object** — cohort membership is via `ProgramCohortMember`

### `ProgramCohortMember`

- Junction between `ProgramCohort` (master) and `ProgramEnrollment`
- Fields: `ProgramCohortId`, `ProgramEnrollmentId`, `ProgramEnrolleeId` (polymorphic → Account/Contact)

---

## Grantmaking

> PSS v68.0 uses the Grantmaking model — **there is no `Grant`, `GrantApplication`, `GrantBudget`, `GrantAllocation`, or `GrantOpportunity` object**. Use the objects below.

### `FundingAward`

| Field | Type | Notes |
|-------|------|-------|
| `Name` | String | Auto-number |
| `Status` | Picklist | Active, Cancelled, Completed |
| `StartDate` | DateTime | |
| `EndDate` | DateTime | |
| `FundingOpportunityId` | Lookup(FundingOpportunity) | |

### `FundingOpportunity`

- Pre-award opportunity record (replaces `GrantOpportunity`)
- `IndividualApplication` is the filing record for grant applications under the Grantmaking license

### `FundingDisbursement`

- Actual spend/drawdown against a funding award (replaces `GrantAllocation`)

### `Budget` / `BudgetAllocation`

- `Budget`: high-level budget line (replaces `GrantBudget`)
- `BudgetAllocation`: allocation against a budget line

---

## Discovery Framework

### `AssessmentQuestion`

- Fields: `Name`, `DeveloperName`, `QuestionText`, `DataType` (picklist — 20+ values including Checkbox/Date/Text/Radio/etc.), `QuestionCategory` (Demographic/Financial)
- **No `IsRequired` field**
- **No `ResponseType` field** — question format is captured by `DataType`

### `AssessmentQuestionSet`

- Groups questions for a specific intake or eligibility purpose
- Fields: `Name`, `DeveloperName`

### `AssessmentQuestionVersion`

- Tracks a specific published version of an `AssessmentQuestion`
- Fields: `ActiveVersionId`, `IsActive`, `ActivationDateTime` (DateTime)
- **No `VersionNumber` field**
- **No `AssessmentQuestionSetId` field on this object** — the question-to-set link is via `AssessmentQuestionAssignment`

### `AssessmentQuestionAssignment`

- Junction between `AssessmentQuestion` and `AssessmentQuestionSet`
- Fields: `AssessmentQuestionId`, `AssessmentQuestionSetId`, `SequenceNumber`

### `AssessmentIndicator`

- Maps question responses to eligibility outcomes or score thresholds

---

## Party and identity

### `Individual`

> **This is the Salesforce GDPR/privacy management object, not a person-identity record.** It does NOT have `FirstName`, `LastName`, or `GenderIdentity`. Use `Contact` as the person record. Use `Individual` only for privacy consent tracking (`HasOptedOutProcessing`, `CanStorePiiElsewhere`, `ShouldForget`, etc.).

- Linked to a `Contact` record for privacy management
- Fields: `BirthDate`, `DeathDate`, `HasOptedOutProcessing`, `HasOptedOutTracking`, `HasOptedOutGeoTracking`, `HasOptedOutProfiling`, `ShouldForget`, `CanStorePiiElsewhere`

### `Contact` (person identity in PSS context)

- `FirstName`, `LastName`, `GenderIdentity` — **these are `Contact` fields**
- Linked to `ContactPointAddress`, `ContactPointEmail`, `ContactPointPhone` for structured channels
- `ContactId` is the applicant field on `IndividualApplication`, `ProgramEnrollment`, `BenefitAssignment`, etc.

### `PartyProfile`

- Optional extended profile data for a party (Account or Contact)

---

## Complaint management

> PSS v68.0 does **not** have `Complaint`, `ComplaintFinding`, or `ComplaintRemediation` as standalone objects.

### `PublicComplaint`

- Primary public-facing complaint record
- Use when a citizen files a complaint against an entity or service

### `ComplaintCase`

- Complaint tied to a Case record for internal tracking and resolution

### `ComplaintParticipant`

- Party associations on a complaint (replaces `ComplaintAssociation`)

---

## Enforcement

> PSS v68.0 does **not** have a standalone `EnforcementAction` or `Violation` object.

### `RegulatoryCodeViolation`

- Records a violation of a specific `RegulatoryCode`
- Child of the inspected or regulated entity

### `ViolationEnforcementAction`

- Enforcement record (fine, order, suspension) linked to a `RegulatoryCodeViolation`

### `ViolationType`

- Reference object defining the violation category (replaces `EnforcementActionType`)

---

## Appeals

> PSS v68.0 does **not** have `Appeal`, `AppealAssociation`, `AppealHearing`, or `AppealDecision` objects. Appeals are not modeled in the base PSS package. If needed, implement via custom objects or CaseProceedingComplaint / CaseProceedingResult (Investigative Case Management add-on).

---

## Visits and field activity

### `Visit`

- Schedules and records a field visit
- Child: `Visitor` (party attending the visit), `VisitedParty`
- **No `VisitQueue` object** — visit assignment is managed via `Visit` record queues and `Visitor` records

---

## Relationships quick reference

High-level parent/child patterns (verify field-level relationships in Object Manager):

- **Draft / resume:** `PreliminaryApplicationRef` → (on submit) `IndividualApplication` (person) or `BusinessLicenseApplication` (business)
- **Licensing / permitting:** `RegulatoryAuthorizationType` → `BusinessLicense` → `BusinessLicenseApplication`
- **Regulatory code chain:** `RegulatoryAuthority` → `RegulatoryCode` → `RegulatoryCodeViolation` → `ViolationEnforcementAction`
- **Benefits and programs:** `Program` → `ProgramEnrollment` → `BenefitAssignment` → `Benefit` → `BenefitDisbursement`; `ProgramCohortMember` links enrollment to `ProgramCohort`
- **Grantmaking:** `FundingOpportunity` → `IndividualApplication` (grant application) → `FundingAward` → `Budget`/`BudgetAllocation` → `FundingDisbursement`
- **Discovery Framework:** `AssessmentQuestionSet` → `AssessmentQuestionAssignment` → `AssessmentQuestion` → `AssessmentQuestionVersion`; `AssessmentIndicator` maps responses to outcomes
- **Complaints:** `PublicComplaint` / `ComplaintCase` → `ComplaintParticipant`
- **Identity:** `Contact` is the person record; `Individual` is the GDPR privacy companion; `Account` for organizations

Always confirm **polymorphic** fields in the official schema before writing SOQL or rollups.
