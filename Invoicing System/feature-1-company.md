# Feature: Company

**Feature ID:** 1
**Branch pattern:** `feature/1-company`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Keep the consulting firm's own company record so invoices later know who is billing and where payment is sent.
**Depends on:** —
**Related:** —

## User Stories

### US-1.1: Register the consulting firm

**As a** owner/admin or accounting clerk
**I want to** register the consulting firm with its name, address, and contact information
**So that** invoices can bill from a current company record

**Priority:** P1
**Independent test:** An authorized owner/admin or accounting clerk enters a valid company name and address and the system persists a company with a unique identifier.
**Acceptance scenarios:** see ### US-1.1 under Gherkin AC

### US-1.2: View company information

**As a** owner/admin or accounting clerk
**I want to** view the consulting firm's current information
**So that** I can confirm the name, address, and billing information before any invoice is produced

**Priority:** P1
**Independent test:** An authorized owner/admin or accounting clerk selects an existing company and sees the stored fields and unique identifier.
**Acceptance scenarios:** see ### US-1.2 under Gherkin AC

### US-1.3: Update company information

**As a** owner/admin or accounting clerk
**I want to** correct the consulting firm's information
**So that** invoices use the current address and billing information without creating a second company

**Priority:** P1
**Independent test:** An authorized owner/admin or accounting clerk changes allowed fields on an existing company and a subsequent view shows the new values on the same unique identifier.
**Acceptance scenarios:** see ### US-1.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to create a company record.
- **FR-002**: System MUST assign a unique identifier (`id`) to each company; the identifier MUST NOT change after create.
- **FR-003**: A company record MUST include `name` and `address`; both are required to save.
- **FR-004**: `websiteUrl`, `phone`, `email`, and `billingInfo` MAY be omitted on create and update.
- **FR-005**: System MUST persist the company record so it can be retrieved after the create operation completes.
- **FR-006**: System MUST NOT create a company record when `name` or `address` is missing or blank after trim.
- **FR-007**: System MUST NOT create a company record when another company already exists with the same `name` (compared case-insensitively after trim).
- **FR-008**: System MUST NOT update a company in a way that would match another existing company's `name`.
- **FR-009**: System MUST allow an authorized owner/admin or accounting clerk to retrieve a company by unique identifier and to list companies.
- **FR-010**: System MUST allow an authorized owner/admin or accounting clerk to update `name`, `address`, `websiteUrl`, `phone`, `email`, and `billingInfo` on an existing company.
- **FR-011**: A project manager, a user who only enters time, and any caller outside the business MUST NOT create, view, list, or update companies.
- **FR-012**: Unauthenticated callers MUST NOT create, view, list, or update companies.
- **FR-013**: `billingInfo` is the remit-to information for later invoices (check address and/or wire number). This feature stores it; it does not produce an invoice.

## Assumptions

- Callers who are allowed to use this feature are already authorized as owner/admin or accounting clerk. Login / session management is out of scope for this feature.
- Company is the consulting firm itself, not a client. Clients are Customers, a later feature.
- One current set of billing information per company, not a history of old addresses.
- Deleting a company is out of scope (correct the record with update).
- The sheet's "billing Info" is the invoice "Remit To: Check Address/Wire Number".

## Edge Cases

- Missing or blank `name` or `address` → reject create/update (`ValidationException` or equivalent)
- Duplicate company name → reject create/update (`DuplicateException` or equivalent)
- Unknown company `id` on view/update → `NotFoundException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Empty company list → empty list, not an error
- Empty `websiteUrl`, `phone`, `email`, or `billingInfo` → allowed; stored empty

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A valid new company can be created and then retrieved by the assigned unique identifier
- **SC-003**: A duplicate company name never creates a second company
- **SC-004**: After an update, view of the same `id` returns the new field values
- **SC-005**: A project manager and an unauthenticated caller cannot create or view a company

## Key Entities

- **Company**: the consulting firm that bills clients; identified by a unique system `id`; has name, address, optional website, phone, email, and billing information
- **Owner/Admin**: authorized operator who can manage everything, including the company record (not a persisted entity in this feature)
- **Accounting clerk**: authorized operator who can add and maintain the company record (not a persisted entity in this feature)

## Initial Data Model

### `companies` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `name` | STRING(150) | Required; stored trimmed; unique case-insensitively |
| `address` | STRING(255) | Required; stored trimmed |
| `websiteUrl` | STRING(255) | Optional; stored trimmed when present |
| `phone` | STRING(30) | Optional; stored trimmed when present |
| `email` | STRING(255) | Optional; stored trimmed when present |
| `billingInfo` | STRING(255) | Optional; remit-to check address and/or wire number; stored trimmed when present |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

Unique business key: `name`, compared case-insensitively after trim.

## Gherkin AC

### US-1.1 — Register the consulting firm

#### Scenario: Accounting clerk registers the firm with valid information

* **Given** an authorized accounting clerk is using the system
* **When** the clerk enters name `ABC`, address `ABC`, website `https://abc.com`, phone `ABC`, email `abc@abc.com`, and billing info `ABC` and saves
* **Then** the system creates a company record
* **And** the record has a unique `id`
* **And** a subsequent retrieve by that `id` returns those stored values

#### Scenario: Owner registers the firm without optional contact fields

* **Given** an authorized owner/admin is using the system
* **When** the owner enters name `ABC` and address `ABC` with no website, phone, email, or billing info and saves
* **Then** the system creates a company record
* **And** `websiteUrl`, `phone`, `email`, and `billingInfo` are empty

#### Scenario: Clerk cannot register a company with missing required fields

* **Given** an authorized accounting clerk is using the system
* **When** the clerk attempts to save a company without a name or without an address
* **Then** the system does not create a company record
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot register a duplicate company name

* **Given** a company exists with name `ABC`
* **When** the clerk attempts to register another company with name `abc`
* **Then** the system does not create a second company
* **And** the system rejects the save with a duplicate error

#### Scenario: Project manager cannot register a company

* **Given** an authorized project manager is using the system
* **When** the project manager attempts to save a company named `ABC`
* **Then** the system does not create a company record
* **And** the system rejects the save as not authorized

### US-1.2 — View company information

#### Scenario: Clerk views the firm

* **Given** a company exists with `id` `1`, name `ABC`, address `ABC`, and billing info `ABC`
* **When** an authorized accounting clerk views company `1`
* **Then** the system displays those stored values
* **And** the unique identifier `1` is shown

#### Scenario: Clerk lists companies

* **Given** companies `ABC` and `XYZ` exist
* **When** an authorized accounting clerk lists companies
* **Then** the list includes both companies
* **And** each row includes unique identifier, name, and address

#### Scenario: Clerk cannot view an unknown company

* **Given** no company exists with `id` `999`
* **When** an authorized accounting clerk views company `999`
* **Then** the system reports that the company was not found

#### Scenario: A caller outside the business cannot view a company

* **Given** a company exists with `id` `1`
* **When** a caller who is not an owner/admin or accounting clerk views company `1`
* **Then** the system does not display the company
* **And** the system rejects the request as not authorized

### US-1.3 — Update company information

#### Scenario: Clerk updates billing information

* **Given** a company exists with `id` `1`, name `ABC`, and billing info `ABC`
* **When** an authorized accounting clerk changes billing info to `XYZ` and saves
* **Then** the company is updated
* **And** a subsequent view of `id` `1` returns billing info `XYZ`
* **And** the name is unchanged
* **And** the unique identifier remains `1`

#### Scenario: Clerk cannot update a company to match another company name

* **Given** company `1` is `ABC`
* **And** company `2` is `XYZ`
* **When** an authorized accounting clerk changes company `2` to name `ABC`
* **Then** the system does not save the change
* **And** the system rejects the save with a duplicate error

#### Scenario: Clerk cannot update an unknown company

* **Given** no company exists with `id` `999`
* **When** an authorized accounting clerk attempts to update company `999`
* **Then** the system reports that the company was not found

## Data Ownership

- Company records are the consulting firm's own master data.
- Only an authorized owner/admin or accounting clerk may create, view, list, or update companies in this feature.
- Callers outside the business MUST NOT read or change company data.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/companies` | Yes | Create a company |
| `GET` | `/invoice/companies` | Yes | List companies |
| `GET` | `/invoice/companies/:id` | Yes | Retrieve one company by `id` |
| `PUT` | `/invoice/companies/:id` | Yes | Update an existing company |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may call them. This feature does not add a delete route.

**Create or update request body:**

```json
{
  "name": "ABC",
  "address": "ABC",
  "websiteUrl": "https://abc.com",
  "phone": "ABC",
  "email": "abc@abc.com",
  "billingInfo": "ABC"
}
```

`name` and `address` are required. `websiteUrl`, `phone`, `email`, and `billingInfo` may be omitted. On update, `id` comes from the path. A body `id` is ignored.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 1,
  "name": "ABC",
  "address": "ABC",
  "websiteUrl": "https://abc.com",
  "phone": "ABC",
  "email": "abc@abc.com",
  "billingInfo": "ABC",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

Omitted optional fields are returned as empty strings. A list is `200` with an array of those objects. An empty list is `[]`, not an error. Each list row includes `id`, `name`, and `address`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `name` or `address` missing or blank after trim | `400` | `"Company name and address are required."` |
| Another company already has this `name`, compared case-insensitively after trim | `400` | `"A company with this name already exists."` |
| No company with that `id` | `404` | `"Company with id=999 not found."` |
| Signed in, but not an owner/admin or accounting clerk | `403` | `"You are not allowed to manage companies."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Companies] — route name `companies`

* Heading: **Companies**
* Shown in `MenuBar` only for an owner/admin or accounting clerk.
* The page loads `GET /invoice/companies`.
* Rows show **id**, **name**, and **address**.
* Primary action: **Add company** (`oc-cta`). It opens a dialog with **Name**, **Address**, **Website**, **Phone**, **Email**, and **Billing info**. **Save** (`oc-cta`) calls `POST /invoice/companies`. **Cancel** closes the dialog.
* **Name** and **Address** are required. The inline message is **"Company name and address are required."**
* Each row has a **View** action with `aria-label` **View company**. It opens route `company` for that `id`.
* **Empty state:** **"No companies yet. Add the consulting firm."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A project manager, a time-entry user, or a caller outside the business who opens this route sees **"You are not allowed to manage companies."** and no company data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Company] — route name `company`

* Route param: `id`. The page loads `GET /invoice/companies/:id`.
* Heading: the company **name**. The **id** is shown and cannot be edited.
* Fields: **Name**, **Address**, **Website**, **Phone**, **Email**, **Billing info**.
* Primary action: **Save** (`oc-cta`). It calls `PUT /invoice/companies/:id`.
* **Name** and **Address** use the same required message as the add dialog.
* After a successful save, the page shows the returned values, including an updated **Billing info**, on the same **id**.
* **Loading state:** a skeleton while the company is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Company with id=999 not found."**, **"A company with this name already exists."**, and **"You are not allowed to manage companies."**
* No delete control.

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-013**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/companies` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] The **Companies** and **Company** screens match **Screen Requirements**
* [ ] The `companies` table matches **Initial Data Model**
* [ ] This feature does not add delete, login, or invoice behavior listed in **Out of Scope**

## Out of Scope

- Employees, customers, projects, roles, project roles, role assignments, time entry, invoices, and line items
- Login / role management
- Deleting a company
- Generating an invoice or a remit-to block on a document
- Client companies (those are Customers)
