# Feature: Invoice

**Feature ID:** 9
**Branch pattern:** `feature/9-invoice`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Let an accounting clerk open an invoice for one project so the firm can bill the client from the system instead of a Word template.
**Depends on:** [Feature 1 — Company](feature-1-company.md), [Feature 4 — Projects](feature-4-projects.md)
**Related:** [Feature 10 — Line Item](feature-10-line-item.md)

## User Stories

### US-9.1: Open an invoice

**As a** owner/admin or accounting clerk
**I want to** open an invoice for one customer project
**So that** the invoice has a number, a date, payment terms, and a due date

**Priority:** P1
**Independent test:** An authorized accounting clerk opens an invoice for ABC's project and the system stores a unique invoice number, today's date, terms of 30 days, and a due date 30 days later.
**Acceptance scenarios:** see ### US-9.1 under Gherkin AC

### US-9.2: View an invoice

**As a** owner/admin or accounting clerk
**I want to** view the invoice header
**So that** I can see who is billed, the project, the terms, and the status before adding lines

**Priority:** P1
**Independent test:** An authorized accounting clerk opens an existing invoice and sees the customer, project, dates, terms, hours, and status.
**Acceptance scenarios:** see ### US-9.2 under Gherkin AC

### US-9.3: Approve an invoice

**As a** owner/admin
**I want to** approve a pending invoice
**So that** the clerk can issue it after the lines are on it

**Priority:** P1
**Independent test:** An authorized owner sets a pending invoice to `approved` and a subsequent view shows `approved`.
**Acceptance scenarios:** see ### US-9.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to create an invoice for an existing project.
- **FR-002**: System MUST assign a unique identifier (`id`). That identifier is the invoice number and MUST NOT change after create.
- **FR-003**: An invoice MUST include `projectId`, `customerId`, `date`, and `term`.
- **FR-004**: `customerId` MUST be the `customerId` on that project.
- **FR-005**: `term` MUST be `15` or `30`. When the clerk omits `term`, System MUST copy the customer's `billingTerms`.
- **FR-006**: System MUST set `dueDate` to `date` plus `term` days.
- **FR-007**: System MUST set `status` to `pending` when an invoice is created.
- **FR-008**: System MUST set `hours` to `0` when an invoice is created. Feature 10 maintains `hours` from service lines.
- **FR-009**: `date` MUST NOT be after today.
- **FR-010**: System MUST NOT create an invoice for an unknown project.
- **FR-011**: While `status` is `pending`, an authorized owner/admin or accounting clerk MAY update `date` and `term`, and System MUST recalculate `dueDate`.
- **FR-012**: System MUST NOT change `projectId` or `customerId` after create.
- **FR-013**: System MUST NOT update `date` or `term` when `status` is `approved` or `invoiced`.
- **FR-014**: An authorized owner/admin MUST be able to set a `pending` invoice to `approved` only when that invoice has at least one line. A `pending` invoice with no lines MUST stay `pending`.
- **FR-015**: An accounting clerk MUST NOT set `status` to `approved`.
- **FR-016**: System MUST NOT set `status` to `invoiced` in this feature.
- **FR-017**: System MUST allow an authorized owner/admin or accounting clerk to retrieve an invoice by invoice number and to list invoices for a project.
- **FR-018**: A project manager, a time-entry user, and any caller outside the business MUST NOT create, view, list, update, or approve invoices.
- **FR-019**: Unauthenticated callers MUST NOT create, view, list, update, or approve invoices.

## Assumptions

- The class invoice shows Invoice Number, Invoice Date, Term of 15 or 30 days, Invoice Due Date, and the client project. This feature is that header.
- Remit-to check address and wire number stay on the company `billingInfo` from Feature 1. Copying them onto the invoice is out of scope.
- Bill-to name and address, and the project sponsor, are read from the customer and the project. They are not copied onto the invoice in this feature.
- Service rows and expense rows are Feature 10.
- Several invoices MAY exist for one project.
- The sheet status values are `pending`, `approved`, and `invoiced`. Owner/admin approves. The clerk issues it in Feature 10, which sets `invoiced`.
- Login / session management is out of scope.
- Deleting an invoice is out of scope.

## Edge Cases

- Unknown project → `NotFoundException` or equivalent
- `customerId` that is not the project's customer → `ValidationException` or equivalent
- `term` other than `15` or `30` → `ValidationException` or equivalent
- Invoice date after today → `ValidationException` or equivalent
- Update of date or term when status is not `pending` → `ValidationException` or equivalent
- Clerk attempts to approve → `NotAuthorizedException` or equivalent
- Owner attempts to approve a pending invoice with no lines → `ValidationException` or equivalent
- Attempt to mark `invoiced` in this feature → `ValidationException` or equivalent
- Unknown invoice `id` → `NotFoundException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Project with no invoices → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A clerk can open an invoice whose customer matches the project and whose due date is the invoice date plus the terms
- **SC-003**: Omitting the term stores the customer's billing terms
- **SC-004**: An owner can approve a pending invoice that has at least one line, and a clerk cannot. A pending invoice with no lines cannot be approved
- **SC-005**: Date and term cannot be changed after approval

## Data Ownership

- An invoice belongs to one project and that project's customer.
- Only an authorized owner/admin or accounting clerk may create and view invoices.
- Only an owner/admin may approve an invoice, and only when it has at least one line.
- Callers outside the business MUST NOT read or change invoices.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/invoices` | Yes | Open an invoice for one project |
| `GET` | `/invoice/projects/:projectId/invoices` | Yes | List invoices for one project |
| `GET` | `/invoice/invoices/:id` | Yes | Retrieve one invoice by invoice number |
| `PUT` | `/invoice/invoices/:id` | Yes | Change `date` and `term` while `pending` |
| `PUT` | `/invoice/invoices/:id/status` | Yes | Set a `pending` invoice that has at least one line to `approved` |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may create, list, view, and update the date and term. Only an owner/admin may approve, and only when the invoice has at least one line. This feature does not add a delete route and does not set `invoiced`. `hours` stays `0` until Feature 10.

**Create request body:**

```json
{
  "projectId": 4,
  "customerId": 1,
  "date": "2026-11-26"
}
```

`projectId`, `customerId`, and `date` are required. `term` may be omitted. When it is omitted, the system copies the customer's `billingTerms`. When it is sent, it must be `15` or `30`. `customerId` must be the customer on that project. `date` must not be after today. The system sets `status` to `pending`, `hours` to `0`, and `dueDate` to `date` plus `term` days. For date `2026-11-26` and term `30`, `dueDate` is `2026-12-26`. For date `2026-11-01` and term `15`, `dueDate` is `2026-11-16`. A body `id`, `hours`, `dueDate`, or `status` is ignored.

**Update date and term while `pending`:**

```json
{
  "date": "2026-11-01",
  "term": 15
}
```

The system recalculates `dueDate`. `projectId` and `customerId` cannot change. This update is rejected when `status` is `approved` or `invoiced`.

**Approve body:**

```json
{
  "status": "approved"
}
```

The invoice must already be `pending` and must have at least one line. `invoiced` is rejected.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 20,
  "date": "2026-11-26",
  "term": 30,
  "dueDate": "2026-12-26",
  "hours": 0,
  "customerId": 1,
  "customerName": "ABC",
  "projectId": 4,
  "status": "pending",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

`customerName` is read from the customer. It is not a column on `invoices`. A list is `200` with an array of those objects. A project with no invoices returns `[]`. Each list row includes `id`, `date`, `customerName`, `status`, and `hours`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `projectId`, `customerId`, or `date` missing | `400` | `"Project, customer, and date are required."` |
| `customerId` is not the project's customer | `400` | `"The customer does not match the project."` |
| `term` other than `15` or `30` | `400` | `"Term must be 15 or 30."` |
| `date` is after today | `400` | `"Invoice date cannot be after today."` |
| Date or term change when status is not `pending` | `400` | `"Date and term can be changed only while the invoice is pending."` |
| Update tries to change `projectId` or `customerId` | `400` | `"The project and customer on an invoice cannot be changed."` |
| Approve when status is not `pending`, or status is not `approved` | `400` | `"Only a pending invoice can be approved."` |
| Approve when the invoice has no lines | `400` | `"An invoice with no lines cannot be approved."` |
| Caller tries to set `invoiced` | `400` | `"An invoice cannot be marked invoiced here."` |
| No project with that `id` | `404` | `"Project with id=999 not found."` |
| No invoice with that `id` | `404` | `"Invoice with id=999 not found."` |
| An accounting clerk approves | `403` | `"You are not allowed to approve an invoice."` |
| A project manager, a time-entry user, or a caller outside the business | `403` | `"You are not allowed to manage invoices."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Invoices] — route name `invoices`

* Route param: `projectId`. The page loads `GET /invoice/projects/:projectId/invoices`.
* Heading: **Invoices**
* Shown in `MenuBar` only for an owner/admin or accounting clerk. The **Project** screen links here with **Invoices**.
* Rows show **invoice number**, **date**, **customer**, **status**, and **hours**. Status text is `pending`, `approved`, or `invoiced`.
* Primary action: **Open invoice** (`oc-cta`). It opens a dialog with **Customer**, **Date**, and **Term**. **Customer** must be the customer on this project. **Term** offers `15`, `30`, or blank. Blank uses that customer's billing terms. **Save** (`oc-cta`) calls `POST /invoice/invoices`. **Cancel** closes the dialog.
* **Customer** and **Date** are required. The inline message for a future date is **"Invoice date cannot be after today."**
* Each row has a **View** action with `aria-label` **View invoice**. It opens route `invoice` for that `id`.
* **Empty state:** **"No invoices for this project yet."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A project manager, a time-entry user, or a caller outside the business sees **"You are not allowed to manage invoices."** and no invoice data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Invoice] — route name `invoice`

* Route param: `id`. The page loads `GET /invoice/invoices/:id`.
* Heading: **Invoice** plus the invoice number. The number, **customer**, **project**, **due date**, and **hours** are shown and cannot be edited. Hours shows `0` until Feature 10.
* While `status` is `pending`, an owner/admin or accounting clerk can change **Date** and **Term**. **Term** offers only `15` and `30`. **Save** (`oc-cta`) calls `PUT /invoice/invoices/:id`. The page then shows the recalculated **due date**.
* While `status` is `pending` and the invoice has at least one line, an owner/admin sees **Approve**. It calls `PUT /invoice/invoices/:id/status` with `approved`. An accounting clerk does not see **Approve**. **Approve** is hidden when the invoice has no lines.
* When status is `approved` or `invoiced`, **Date** and **Term** are read-only.
* **Loading state:** a skeleton while the invoice is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Date and term can be changed only while the invoice is pending."**, **"You are not allowed to approve an invoice."**, **"An invoice with no lines cannot be approved."**, and **"Invoice with id=999 not found."**
* No delete control and no control that sets `invoiced`.

## Key Entities

- **Invoice**: a bill for one project; identified by the invoice number (`id`); has a date, a 15- or 30-day term, a due date, total service hours, and a status

## Initial Data Model

### `invoices` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; invoice number; never updated |
| `date` | DATE | Required; not after today; editable only while `pending` |
| `term` | INTEGER | Required; `15` or `30`; defaults from the customer's `billingTerms` when omitted; editable only while `pending` |
| `dueDate` | DATE | Required; `date` plus `term` days; recalculated when `date` or `term` changes |
| `hours` | DECIMAL(8,2) | Required; `0` on create; Feature 10 sets the sum of service-line quantities |
| `customerId` | INTEGER FK | Required; references `customers.id`; must match the project; never updated |
| `projectId` | INTEGER FK | Required; references `projects.id`; never updated |
| `status` | STRING(20) | Required; `pending` on create; owner/admin may set `approved`; Feature 10 may set `invoiced` |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

`dueDate` is stored because the class invoice prints Invoice Due Date. The sheet does not have a separate due-date column.

## Gherkin AC

### US-9.1 — Open an invoice

#### Scenario: Clerk opens an invoice using the customer's terms

* **Given** customer `1` ABC has billing terms `30`
* **And** project `4` belongs to customer `1`
* **And** an authorized accounting clerk is using the system
* **And** today is `2026-11-26`
* **When** the clerk opens an invoice for project `4` and customer `1` dated `2026-11-26` with no term and saves
* **Then** the system creates an invoice
* **And** the invoice has a unique invoice number
* **And** `term` is `30`
* **And** `dueDate` is `2026-12-26`
* **And** `hours` is `0`
* **And** `status` is `pending`

#### Scenario: Clerk chooses a 15-day term

* **Given** project `4` belongs to customer `1`
* **When** an authorized accounting clerk opens an invoice for project `4` dated `2026-11-01` with term `15`
* **Then** `term` is `15`
* **And** `dueDate` is `2026-11-16`

#### Scenario: Clerk cannot bill a different customer

* **Given** project `4` belongs to customer `1`
* **When** an authorized accounting clerk attempts to open an invoice for project `4` and customer `2`
* **Then** the system does not create an invoice
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot use a term other than 15 or 30

* **Given** project `4` belongs to customer `1`
* **When** an authorized accounting clerk attempts to open an invoice with term `45`
* **Then** the system does not create an invoice
* **And** the system rejects the save with a validation error

### US-9.2 — View an invoice

#### Scenario: Clerk views an invoice

* **Given** invoice `20` is pending for customer `1` and project `4`, dated `2026-11-26`, term `30`, due `2026-12-26`, hours `0`
* **When** an authorized accounting clerk views invoice `20`
* **Then** the system displays those stored values
* **And** the invoice number `20` is shown

#### Scenario: Clerk lists invoices for a project

* **Given** project `4` has invoices `20` and `21`
* **When** an authorized accounting clerk lists invoices for project `4`
* **Then** the list includes both invoices
* **And** each row includes invoice number, date, customer, status, and hours

#### Scenario: A project manager cannot view an invoice

* **Given** invoice `20` exists
* **When** an authorized project manager views invoice `20`
* **Then** the system does not display the invoice
* **And** the system rejects the request as not authorized

### US-9.3 — Approve an invoice

#### Scenario: Owner approves a pending invoice that has a line

* **Given** invoice `20` is `pending`
* **And** invoice `20` has one line
* **And** an authorized owner/admin is using the system
* **When** the owner sets the status to `approved`
* **Then** a subsequent view of invoice `20` returns `approved`

#### Scenario: Owner cannot approve an invoice with no lines

* **Given** invoice `20` is `pending` and has no lines
* **And** an authorized owner/admin is using the system
* **When** the owner attempts to set the status to `approved`
* **Then** the system does not change the status
* **And** the status remains `pending`
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot approve an invoice

* **Given** invoice `20` is `pending`
* **When** an authorized accounting clerk attempts to set the status to `approved`
* **Then** the system does not change the status
* **And** the status remains `pending`
* **And** the system rejects the save as not authorized

#### Scenario: Approved invoice dates cannot be changed

* **Given** invoice `20` is `approved` with term `30`
* **When** an authorized accounting clerk attempts to change the term to `15`
* **Then** the system does not save the change
* **And** the term remains `30`

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-019**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and both `PUT` routes under `/invoice/invoices` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] Omitting `term` stores the customer's `billingTerms`, and `dueDate` is `date` plus `term` days
* [ ] A new invoice is `pending` with `hours` `0`, and this feature cannot set `invoiced`
* [ ] The **Invoices** and **Invoice** screens match **Screen Requirements**, including no **Approve** for a clerk
* [ ] The `invoices` table matches **Initial Data Model**, and `projectId` and `customerId` do not change after create
* [ ] This feature does not add lines, issuing, printing, or delete behavior listed in **Out of Scope**

## Out of Scope

- Service and expense lines — [Feature 10](feature-10-line-item.md)
- Marking the invoice `invoiced`
- Summing hours from time entries
- Printing or mailing the invoice
- Remit-to snapshot
- Deleting an invoice
- Login / role management
