<!-- Parent: sf-pss-data-dev/SKILL.md -->

# Public Sector Solutions (PSS) core objects reference

Source: [PSS application programming interface (API) overview](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/api_psc_overview.htm)

---

## Licensing

### `BusinessLicense`

| Field | Type | Notes |
|-------|------|-------|
| `Id` | ID | |
| `LicenseNumber` | String | Auto-generated, displayed to citizen |
| `LicenseType` | Picklist | References `LicenseType.Name` |
| `LicensedEntityId` | Lookup(Account) | The business holding the license |
| `IssuingAgencyId` | Lookup(Account) | Government agency issuing the license |
| `StatusCode` | Picklist | Active, Suspended, Revoked, Expired |
| `EffectiveDate` | Date | |
| `ExpirationDate` | Date | |

### `BusinessLicenseApplication`

- Lookup → `BusinessLicense` (license being renewed or applied for)
- **Business applicant:** Use **`BusinessLicenseApplication`** as the **primary** application record when a **business** (for example **`LicensedEntity`** / Account) is applying or renewing. **Do not** model that intake on **`IndividualApplication`**—reserve **`IndividualApplication`** for **person**-centric filings (benefits, programs where the applicant is an **`Individual`**).
- PSS **`BusinessLicenseApplication`** includes **`AccountId`** → **`Account`** (B2B **Account** or **Person Account**) and **`ApplicantId`** / **`PrimaryOwnerId`** → **`Contact`** (including **PersonContact** for Person Accounts)—see [PSS object reference](https://developer.salesforce.com/docs/atlas.en-us.psc_api.meta/psc_api/sforce_api_objects_businesslicenseapplication.htm).
- A **person** acting for the business (owner, agent) may appear on related records (for example **`RegulatoryTxnParty`**, contact roles, or org-specific fields)—confirm relationships in Object Manager; the filing object for the **business** remains **`BusinessLicenseApplication`**.
- Status lifecycle: Draft → Submitted → Under Review → Approved/Rejected

### `LicenseType`

- Master reference; defines required fields and fee schedule
- Relationship: `BusinessLicense.LicenseType` (text foreign key pattern)

---

## Permitting

### `Permit`

| Field | Type | Notes |
|-------|------|-------|
| `PermitNumber` | String | Auto-generated |
| `PermitTypeId` | Lookup(PermitType) | |
| `StatusCode` | Picklist | Draft, Submitted, Issued, Expired, Revoked |
| `ApplicantId` | Lookup(Individual/Account) | |
| `IssuingAgencyId` | Lookup(Account) | |
| `EffectiveDate` | Date | |
| `ExpirationDate` | Date | |
| `ParcelId` | Lookup(Parcel) | Optional; links to land record |

### `PermitApplication`

- Parent: `Permit`
- Tracks application submission and review workflow
- Child: `PermitApplicationReview` (one per reviewer/department)

### `PermitType`

- Defines permit class (for example Building, Electrical, Grading)
- Drives required inspections and fee schedule

---

## Inspections

### `Inspection`

| Field | Type | Notes |
|-------|------|-------|
| `InspectionNumber` | String | |
| `InspectionTypeId` | Lookup(InspectionType) | |
| `StatusCode` | Picklist | Scheduled, In Progress, Completed, Failed, Passed |
| `ScheduledStartTime` | DateTime | |
| `CompletedDateTime` | DateTime | |
| `InspectorId` | Lookup(User) | Assigned field officer |
| `RelatedEntityId` | Polymorphic | Links to `Permit`, `Violation`, `BusinessLicense` |

### `InspectionChecklistItem`

- Child of `Inspection`
- Fields: `QuestionText`, `ResponseCode` (Pass/Fail/N/A), `Notes`

### `InspectionFinding`

- Child of `Inspection`
- Fields: `RegulatoryCodeId` (Lookup), `FindingType`, `Severity`, `Description`
- Triggers `Violation` creation on failure outcomes

### `InspectionType`

- Reference object; defines checklist template and pass/fail rules

---

## Regulatory framework

### `RegulatoryCode`

| Field | Type | Notes |
|-------|------|-------|
| `RegulatoryCodeNumber` | String | Code citation (for example "IBC 1001.1") |
| `Description` | LongTextArea | |
| `EffectiveDate` | Date | |
| `StatusCode` | Picklist | Active, Superseded, Repealed |
| `JurisdictionId` | Lookup(Account) | Owning agency/jurisdiction |

### `RegulatoryAuthorizationType`

- Defines what actions a `RegulatoryCode` authorizes
- Used by `RegulatoryTxn` to validate transaction type

### `RegulatoryTxn`

- The primary regulatory transaction record
- Fields: `RegulatoryAuthorizationTypeId`, `SubjectEntityId` (polymorphic), `StatusCode`
- Child: `RegulatoryTxnParty` (roles: Applicant, Agent, Co-Applicant)

---

## Applications

**Applicant type:** **`IndividualApplication`** = person/program intake where the applicant is an **`Individual`**. **`BusinessLicenseApplication`** = **business** license apply or renew. Do not use **`IndividualApplication`** as a stand-in for a **business** license application.

### `IndividualApplication`

| Field | Type | Notes |
|-------|------|-------|
| `ApplicationNumber` | String | Auto-generated |
| `ApplicationTypeId` | Lookup(ApplicationType) | |
| `ApplicantId` | Lookup(Individual) | |
| `StatusCode` | Picklist | Draft, Submitted, Under Review, Approved, Rejected |
| `SubmittedDate` | DateTime | |
| `ChannelCode` | Picklist | Online, In Person, Phone, Mail |

### `ApplicationForm`

- Typically a child of **`IndividualApplication`** in many PSS configurations; for **business license** flows, confirm in Object Manager whether **`ApplicationForm`** (or equivalent capture) attaches to **`BusinessLicenseApplication`** in your API version—either way, the **business** filing parent is **`BusinessLicenseApplication`**, not **`IndividualApplication`**.
- Groups fields into sections via `ApplicationFormSection`

### `ApplicationFormField`

- Child of `ApplicationFormSection`
- Fields: `Label`, `DataType`, `IsRequired`, `FieldValue`
- Do not replicate with custom objects; extend natively

---

## Benefit management

### `BenefitType`

- Reference; defines benefit category (cash, service, voucher)
- Fields: `Name`, `BenefitTypeCode`, `DeliveryMethodCode`

### `Benefit`

| Field | Type | Notes |
|-------|------|-------|
| `BenefitTypeId` | Lookup(BenefitType) | |
| `RecipientId` | Lookup(Individual) | |
| `ProgramEnrollmentId` | Lookup(ProgramEnrollment) | |
| `StatusCode` | Picklist | Pending, Active, Suspended, Closed |
| `StartDate` | Date | |
| `EndDate` | Date | |

### `BenefitDisbursement`

- Child of `Benefit`
- Fields: `DisbursementDate`, `Amount`, `StatusCode`, `PaymentMethodCode`
- Financial line-level record; integrate with Accounting Subledger for GL posting

### `BenefitAssignment`

- Maps `Benefit` to `ProgramEnrollment`; supports multiple benefits per enrollment

---

## Program management

### `Program`

| Field | Type | Notes |
|-------|------|-------|
| `ProgramTypeId` | Lookup(ProgramType) | |
| `StatusCode` | Picklist | Active, Inactive, Draft |
| `OwningAgencyId` | Lookup(Account) | |
| `FundingSourceCode` | Picklist | Federal, State, Local, Private |

### `ProgramEnrollment`

- Junction between `Individual` and `Program`
- Fields: `EnrollmentStatusCode`, `StartDate`, `EndDate`, `ProgramCohortId`
- Child: `ProgramEnrollmentStatusHistory` (full status audit trail)

---

## Grants management

### `Grant`

| Field | Type | Notes |
|-------|------|-------|
| `GrantNumber` | String | |
| `GrantTypeId` | Lookup(GrantType) | |
| `GrantorId` | Lookup(Account) | Funding agency |
| `GranteeId` | Lookup(Account) | Receiving organization |
| `TotalAmount` | Currency | |
| `StatusCode` | Picklist | |

### `GrantApplication`

- Pre-award object; links `Individual`/`Account` to `GrantOpportunity`
- Status: Draft → Submitted → Under Review → Awarded/Rejected

### `GrantBudget` / `GrantAllocation`

- `GrantBudget`: high-level budget line (category, amount)
- `GrantAllocation`: actual spend/drawdown against budget line

---

## Discovery Framework

### `AssessmentQuestion`

- Fields: `QuestionText`, `DataType`, `IsRequired`, `ResponseType`

### `AssessmentQuestionSet`

- Groups questions for a specific intake or eligibility purpose
- Fields: `Name`, `DeveloperName`, `StatusCode`

### `AssessmentQuestionVersion`

- Tracks versioning of question sets; enables in-flight assessment migration
- Fields: `AssessmentQuestionSetId`, `VersionNumber`, `ActiveFromDate`

### `AssessmentIndicator`

- Maps question responses to eligibility outcomes or score thresholds

---

## Party and identity

### `Individual`

- Core constituent record; supersedes standalone Contact for PSS
- Fields: `FirstName`, `LastName`, `BirthDate`, `GenderIdentity`
- Linked to `ContactPointAddress`, `ContactPointEmail`, `ContactPointPhone`

### `Party`

- Polymorphic parent for both `Individual` and `Account` (org/business)
- Used in `RegulatoryTxnParty`, `GrantApplication`, `Complaint` associations

---

## Action plans (Industries)

Native **Industries** action objects model **repeatable, assignable work** on a parent record (for example an application, regulatory transaction, or case—**supported target types depend on org configuration**):

- **`ActionPlanTemplate`** — Reusable definition; defines the shape of plans you attach to records. See the Salesforce [**ActionPlanTemplate** object reference](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_actionplantemplate.htm) for fields, relationships, and supported behaviors in your API version.
- **`ActionPlan`** — Runtime instance of a template against a specific parent record.
- **`RecordAction`** — Individual tasks / steps (due dates, owners, completion) that clerks or automation advance.

Use **`Inspection`** / **`InspectionVisit`** and **`Assessment*`** for **compliance findings and scored intake**; use **`ActionPlan`** for **procedural checklists and role-based follow-up** on the same overall visits/inspections/assessments story (see the PSS Data Model Gallery *Visits, Inspections & Dynamic Assessments* topic and the **Action plans (Industries)** row in the parent [`SKILL.md`](../SKILL.md) domain map). Do not replace **`InspectionType`**-driven checklist cloning with action plans alone.

**Metadata:** `ActionPlanTemplate` deploy XML is sensitive to structure and API version—prefer Setup authoring + retrieve, or shell templates completed in-org. If you use the **PSS DX template** repository, see its **`docs/adr-pss.md`** (Action Plan Template source format) for shell-only vs full-template guidance.

---

## Relationships quick reference

High-level parent/child patterns (verify field-level relationships in Object Manager and the PSS API guide for your API version):

- **Licensing (business):** `LicenseType` → `BusinessLicense` / `LicensedEntity` → **`BusinessLicenseApplication`** (business apply or renew; **not** `IndividualApplication` as the filing parent)
- **Permitting:** `PermitType` → `Permit` → `PermitApplication` → `PermitApplicationReview`
- **Inspections:** `InspectionType` → `Inspection` → `InspectionChecklistItem` / `InspectionFinding` → `RegulatoryCode`
- **Action plans:** `ActionPlanTemplate` → `ActionPlan` → `RecordAction` (parent target varies; confirm in Object Reference)
- **Regulatory:** `RegulatoryCode` / `RegulatoryAuthorizationType` → `RegulatoryTxn` → `RegulatoryTxnParty`
- **Benefits and programs:** `Program` → `ProgramEnrollment` → `Benefit` → `BenefitDisbursement`; `BenefitAssignment` ties benefit to enrollment
- **Grants:** `GrantOpportunity` → `GrantApplication` → `Grant` → `GrantBudget` / `GrantAllocation`
- **Complaints and appeals:** `Complaint` → `ComplaintFinding` → `ComplaintRemediation`; `Appeal` → `AppealHearing` → `AppealDecision`
- **Identity:** `Individual` and `Party` anchor person and organization context for transactions above

Always confirm **polymorphic** fields (`RelatedEntityId`, `SubjectEntityId`) in the official schema before writing **Salesforce Object Query Language (SOQL)** or rollups.
