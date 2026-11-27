# Feature: Line Item

**Feature ID:** 10
**Branch pattern:** `feature/10-line-item`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Put service lines from approved time, and expense lines, on a pending invoice, then let the clerk issue it.
**Depends on:** [Feature 8 — Project Timeline](feature-8-project-timeline.md), [Feature 9 — Invoice](feature-9-invoice.md)
**Related:** —

## User Stories

### US-10.1: Add a service line from approved time

**As a** accounting clerk
**I want to** add an approved time entry to an invoice as a service line
**So that** the client is billed for that role, rate, and hours

**Priority:** P1
**Independent test:** An approved 60-hour Programmer entry at $100 becomes a service line totaling $600, and that time entry is then `Invoiced`.
**Acceptance scenarios:** see ### US-10.1 under Gherkin AC

### US-10.2: Add an expense line

**As a** accounting clerk
**I want to** add an expense with a date, description, and total
**So that** costs that are not hours still appear on the invoice

**Priority:** P1
**Independent test:** An authorized clerk adds an expense of `85.00` on a pending invoice and a later view shows that expense line.
**Acceptance scenarios:** see ### US-10.2 under Gherkin AC

### US-10.3: Issue the invoice

**As a** accounting clerk
**I want to** mark an approved invoice as invoiced after its lines are complete
**So that** those hours cannot be billed again

**Priority:** P1
**Independent test:** An authorized clerk sets an approved invoice that has at least one line to `invoiced`, and no further lines can be added.
**Acceptance scenarios:** see ### US-10.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to add a line item to an invoice whose `status` is `pending`.
- **FR-002**: System MUST assign a unique identifier (`id`) to each line item; the identifier MUST NOT change after create.
- **FR-003**: `type` MUST be `service` or `expense`. The sheet text "sev, expenses" means these two types.
- **FR-004**: A service line MUST be created from one time entry with `status` `Approved`, an empty `invoicedItemId`, and the same `projectId` as the invoice.
- **FR-005**: A service line MUST store `date` and `hours` from that time entry. `quantity` is those hours. `rate` is the `rate` on the assignment's project role. `roleName` is that role's name.
- **FR-006**: A service line MUST include a `description`.
- **FR-007**: A service line `total` MUST equal `quantity` multiplied by `rate`.
- **FR-008**: When a service line is saved, System MUST set that time entry's `invoicedItemId` to the line's `id` and its `status` to `Invoiced`.
- **FR-009**: The same time entry MUST NOT be placed on a second line.
- **FR-010**: An expense line MUST include `date`, `description`, and `total`. `quantity`, `rate`, and `roleName` MUST be empty. No time entry is linked.
- **FR-011**: An expense `total` MUST be greater than zero. An expense `date` MUST NOT be after today.
- **FR-012**: After any line is added or removed, System MUST set the invoice `hours` to the sum of service-line quantities on that invoice. An invoice with no service lines has hours `0`.
- **FR-013**: While the invoice is `pending`, an authorized owner/admin or accounting clerk MAY remove a line. Removing a service line MUST set the linked time entry back to `Approved` and clear its `invoicedItemId`.
- **FR-014**: System MUST NOT add or remove a line when the invoice `status` is `approved` or `invoiced`.
- **FR-015**: An authorized accounting clerk or owner/admin MAY set an `approved` invoice to `invoiced` only when it has at least one line.
- **FR-016**: System MUST NOT add or remove lines after `status` is `invoiced`.
- **FR-017**: System MUST allow an authorized owner/admin or accounting clerk to list line items for an invoice.
- **FR-018**: A project manager, a time-entry user, and any caller outside the business MUST NOT create, view, list, remove, or issue line items.
- **FR-019**: Unauthenticated callers MUST NOT create, view, list, remove, or issue line items.

## Assumptions

- The class service row is Date, Role, Description, Rate, Hours, Total. The sample is `12/1-12/15`, `Prgmr`, task info, `$100`, `60`, `$600`. This feature stores one service date, the role name, the clerk's description, the project-role rate, the approved hours, and the extended total.
- The class expense row is Date, Description, and Total.
- Rate comes from the project role, not from the standard role, so each client keeps the rate agreed on that project.
- Lines are added only while the invoice is `pending`. The owner approves in Feature 9 only after at least one line is on the invoice. The clerk then issues it.
- Login / session management is out of scope.
- Sales tax, payments, and a printed document are out of scope.

## Edge Cases

- Invoice is not `pending` when adding or removing a line → `ValidationException` or equivalent
- Time entry is not `Approved`, is already linked, or is on another project → `ValidationException` or equivalent
- Unknown invoice or unknown time entry → `NotFoundException` or equivalent
- Service description blank → `ValidationException` or equivalent
- Expense total of zero or less → `ValidationException` or equivalent
- Issue an invoice that is not `approved` → `ValidationException` or equivalent
- Issue an approved invoice with no lines → `ValidationException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Invoice with no lines → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: An approved time entry becomes one service line at the project-role rate, and that time entry is then `Invoiced`
- **SC-003**: The same time entry cannot be billed twice
- **SC-004**: An expense line stores date, description, and total without hours or a rate
- **SC-005**: Invoice hours equal the sum of service quantities
- **SC-006**: After the invoice is `invoiced`, its lines cannot be changed

## Key Entities

- **Line item**: one service row or one expense row on an invoice; identified by a unique system `id`
- **Service line**: billed time; quantity is hours; total is hours times the project-role rate
- **Expense line**: a non-hour cost; total is entered directly

## Initial Data Model

### `line_items` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `invoiceId` | INTEGER FK | Required; references `invoices.id`; never updated |
| `type` | STRING(20) | Required; `service` or `expense` |
| `date` | DATE | Required; for a service line, copied from the time entry |
| `description` | STRING(255) | Required; stored trimmed. The sheet has no description column; the class invoice template does |
| `roleName` | STRING(100) | Required on a service line; empty on an expense line |
| `quantity` | DECIMAL(8,2) | Required on a service line, where it is the approved hours; empty on an expense line |
| `rate` | DECIMAL(10,2) | Required on a service line; copied from the project role; empty on an expense line |
| `total` | DECIMAL(12,2) | Required; service total is `quantity` times `rate`; expense total is entered and greater than zero |
| `timeEntryId` | INTEGER FK | Required on a service line; references `project_timelines.id`; empty on an expense line |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

The sheet names this entity "Inline Item" and lists type, date, quantity, rate, and total. `invoiceId`, `description`, `roleName`, and `timeEntryId` are here because the invoice template and the timeline's `InvoicedItemId` need them.

## Gherkin AC

### US-10.1 — Add a service line from approved time

#### Scenario: Clerk bills approved programmer time

* **Given** invoice `20` is `pending` for project `4`
* **And** time entry `10` is `Approved` for 60 hours on `2026-12-01` for project `4`
* **And** that assignment's project role is `Programmer` at rate `100`
* **And** an authorized accounting clerk is using the system
* **When** the clerk adds time entry `10` to invoice `20` with description `ABC` and saves
* **Then** the system creates a service line
* **And** `roleName` is `Programmer`
* **And** `quantity` is `60`
* **And** `rate` is `100`
* **And** `total` is `600`
* **And** time entry `10` is `Invoiced`
* **And** time entry `10` `invoicedItemId` is the new line's `id`
* **And** invoice `20` `hours` is `60`

#### Scenario: Pending time cannot be billed

* **Given** invoice `20` is `pending` for project `4`
* **And** time entry `11` is `Pending` on project `4`
* **When** an authorized accounting clerk attempts to add time entry `11` to invoice `20`
* **Then** the system does not create a line
* **And** time entry `11` stays `Pending`

#### Scenario: The same time entry cannot be billed twice

* **Given** time entry `10` is already `Invoiced` on invoice `20`
* **And** invoice `21` is `pending` for the same project
* **When** an authorized accounting clerk attempts to add time entry `10` to invoice `21`
* **Then** the system does not create a second line
* **And** the system rejects the save with a validation error

#### Scenario: Clerk removes a service line from a pending invoice

* **Given** invoice `20` is `pending`
* **And** service line `5` is linked to time entry `10`, which is `Invoiced`
* **When** an authorized accounting clerk removes line `5`
* **Then** line `5` is no longer on the invoice
* **And** time entry `10` is `Approved`
* **And** time entry `10` `invoicedItemId` is empty
* **And** invoice `20` hours no longer include that line

### US-10.2 — Add an expense line

#### Scenario: Clerk adds an expense

* **Given** invoice `20` is `pending`
* **When** an authorized accounting clerk adds an expense dated `2026-11-20`, description `XYZ`, and total `85.00` and saves
* **Then** the system creates an expense line
* **And** `quantity`, `rate`, and `roleName` are empty
* **And** `total` is `85.00`
* **And** invoice `20` `hours` is unchanged

#### Scenario: An expense total must be greater than zero

* **Given** invoice `20` is `pending`
* **When** an authorized accounting clerk attempts to add an expense with total `0`
* **Then** the system does not create a line
* **And** the system rejects the save with a validation error

### US-10.3 — Issue the invoice

#### Scenario: Clerk issues an approved invoice that has a line

* **Given** invoice `20` is `approved`
* **And** invoice `20` has one service line
* **When** an authorized accounting clerk sets the status to `invoiced`
* **Then** a subsequent view of invoice `20` returns `invoiced`

#### Scenario: A pending invoice cannot be issued

* **Given** invoice `20` is `pending` and has one line
* **When** an authorized accounting clerk attempts to set the status to `invoiced`
* **Then** the system does not change the status
* **And** the status remains `pending`

#### Scenario: An approved invoice with no lines cannot be issued

* **Given** invoice `20` is `approved` and has no lines
* **When** an authorized accounting clerk attempts to set the status to `invoiced`
* **Then** the system does not change the status
* **And** the system rejects the save with a validation error

#### Scenario: No line can be added after the invoice is issued

* **Given** invoice `20` is `invoiced`
* **And** time entry `12` is `Approved` on the same project
* **When** an authorized accounting clerk attempts to add time entry `12` to invoice `20`
* **Then** the system does not create a line
* **And** time entry `12` stays `Approved`

## Data Ownership

- A line item belongs to one invoice.
- A service line also claims one approved time entry.
- Only an authorized owner/admin or accounting clerk may add, view, remove, or issue lines.
- Callers outside the business MUST NOT read or change invoice lines.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/invoices/:invoiceId/line-items` | Yes | Add a service line or an expense line |
| `GET` | `/invoice/invoices/:invoiceId/line-items` | Yes | List lines on one invoice |
| `DELETE` | `/invoice/line-items/:id` | Yes | Remove a line while the invoice is `pending` |
| `PUT` | `/invoice/invoices/:id/status` | Yes | Set an `approved` invoice to `invoiced` |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may call them. Lines can be added or removed only while the invoice is `pending`. Feature 9 owns this status route for `approved`. This feature allows `invoiced` on that same route when the invoice is already `approved` and has at least one line.

**Service line request body:**

```json
{
  "type": "service",
  "timeEntryId": 10,
  "description": "ABC"
}
```

`type`, `timeEntryId`, and `description` are required. The time entry must be `Approved`, have an empty `invoicedItemId`, and belong to the invoice's project. The system copies `date` and hours from that entry. `quantity` is those hours. `rate` is the project-role rate, not the standard rate. `roleName` is that role's name. `total` is `quantity` multiplied by `rate`. For 60 hours at `100`, `total` is `600`. The system then sets that time entry's `status` to `Invoiced` and its `invoicedItemId` to the new line `id`. A sent `rate`, `quantity`, or `total` is ignored.

**Expense line request body:**

```json
{
  "type": "expense",
  "date": "2026-11-20",
  "description": "XYZ",
  "total": 85.00
}
```

`date`, `description`, and `total` are required. `total` must be greater than zero. `date` must not be after today. `quantity`, `rate`, `roleName`, and `timeEntryId` are empty. No time entry is linked.

After a line is added or removed, invoice `hours` becomes the sum of service-line quantities on that invoice. An invoice with no service lines has hours `0`. An expense does not change `hours`.

**Issue body:**

```json
{
  "status": "invoiced"
}
```

The invoice must be `approved` and have at least one line. A `pending` invoice is rejected.

**Success response** (`201` on create, `200` on list):

```json
{
  "id": 5,
  "invoiceId": 20,
  "type": "service",
  "date": "2026-12-01",
  "description": "ABC",
  "roleName": "Programmer",
  "quantity": 60,
  "rate": 100,
  "total": 600,
  "timeEntryId": 10,
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

On an expense line, `quantity`, `rate`, `roleName`, and `timeEntryId` are `null`. A list is `200` with an array of those objects. An invoice with no lines returns `[]`. `DELETE` returns `200` with `{ "invoiceHours": 0 }` after the sum is recalculated. Removing a service line sets the linked time entry back to `Approved` and clears `invoicedItemId`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| Invoice is not `pending` when adding or removing a line | `400` | `"Lines can be changed only while the invoice is pending."` |
| `type` is not `service` or `expense` | `400` | `"Type must be service or expense."` |
| Service description is blank, or the time entry is missing | `400` | `"A service line requires a description and an approved time entry."` |
| Time entry is not `Approved`, is already linked, or is on another project | `400` | `"Only approved time on this project can be billed."` |
| Expense `total` is zero or less | `400` | `"Expense total must be greater than zero."` |
| Expense `date` is after today, or expense date or description is missing | `400` | `"An expense requires a date, a description, and a total greater than zero."` |
| Issue when status is not `approved` | `400` | `"Only an approved invoice can be issued."` |
| Issue when the approved invoice has no lines | `400` | `"An invoice with no lines cannot be issued."` |
| No invoice with that `id` | `404` | `"Invoice with id=999 not found."` |
| No time entry with that `id` | `404` | `"Time entry with id=999 not found."` |
| No line with that `id` | `404` | `"Line item with id=999 not found."` |
| A project manager, a time-entry user, or a caller outside the business | `403` | `"You are not allowed to manage invoice lines."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Invoice lines] — route name `invoice-lines`

* Route param: `invoiceId`. The page loads `GET /invoice/invoices/:invoiceId/line-items` and the invoice header.
* Heading: **Invoice lines**
* Shown from the **Invoice** screen with **Lines**. `MenuBar` shows it only for an owner/admin or accounting clerk.
* Rows show **type**, **date**, **role**, **description**, **rate**, **hours**, and **total**. An expense row leaves role, rate, and hours blank.
* The invoice **hours** shown on this page are the sum of the service quantities.
* While the invoice is `pending`, two actions are shown. **Add service line** (`oc-cta`) opens a dialog with **Time entry** and **Description**. **Time entry** lists `Approved` entries on this project that are not yet billed. The clerk does not type the rate or the hours. **Save** calls `POST` with `type` `service`. **Add expense** opens a dialog with **Date**, **Description**, and **Total**. **Save** calls `POST` with `type` `expense`. **Cancel** closes either dialog.
* While the invoice is `pending`, each row has **Remove** with `aria-label` **Remove line**. It calls `DELETE /invoice/line-items/:id`.
* While the invoice is `approved` and has at least one line, **Issue invoice** is shown. It calls `PUT /invoice/invoices/:id/status` with `invoiced`. An owner/admin or accounting clerk may use it.
* While the invoice is `approved` or `invoiced`, **Add service line**, **Add expense**, and **Remove** are hidden.
* **Empty state:** **"No lines on this invoice yet."**
* **Loading state:** a skeleton while the lines are loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Only approved time on this project can be billed."**, **"Expense total must be greater than zero."**, **"Only an approved invoice can be issued."**, and **"An invoice with no lines cannot be issued."**
* A project manager, a time-entry user, or a caller outside the business sees **"You are not allowed to manage invoice lines."** and no lines.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-019**
* [ ] **SC-001** through **SC-006** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `DELETE` for line items, and issue through `PUT /invoice/invoices/:id/status`, match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] A service line copies the project-role rate, sets the time entry to `Invoiced`, and invoice `hours` equal the sum of service quantities
* [ ] Removing a service line from a `pending` invoice sets that time entry back to `Approved` and clears `invoicedItemId`
* [ ] The **Invoice lines** screen matches **Screen Requirements**
* [ ] The `line_items` table matches **Initial Data Model**
* [ ] This feature does not add printing, payment, sales tax, or edits to approved hours listed in **Out of Scope**

## Out of Scope

- Printing, mailing, or faxing the invoice
- Recording a payment
- Sales tax
- Changing approved hours from this feature
- Login / role management
