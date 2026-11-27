# Feature: Customers

**Feature ID:** 3
**Branch pattern:** `feature/3-customers`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Keep each client the firm bills, including address, account number, main contact, and payment terms.
**Depends on:** [Feature 1 — Company](feature-1-company.md)
**Related:** [Feature 4 — Projects](feature-4-projects.md)

## User Stories

### US-3.1: Add a customer

**As a** owner/admin or accounting clerk
**I want to** add a client with an address, account number, and billing terms
**So that** projects and invoices can be tied to the right client

**Priority:** P1
**Independent test:** An authorized accounting clerk enters a client name, address, unique account number, and terms of 30 days, and the system persists a customer with a unique identifier.
**Acceptance scenarios:** see ### US-3.1 under Gherkin AC

### US-3.2: View a customer

**As a** owner/admin or accounting clerk
**I want to** view and list customers
**So that** I can confirm the client name, address, and account number before opening a project

**Priority:** P1
**Independent test:** An authorized accounting clerk opens an existing customer and sees the stored identity, account number, main contact, and billing terms.
**Acceptance scenarios:** see ### US-3.2 under Gherkin AC

### US-3.3: Update a customer

**As a** owner/admin or accounting clerk
**I want to** correct a client's address, main contact, or billing terms
**So that** later invoices use current client information without creating a second customer

**Priority:** P1
**Independent test:** An authorized accounting clerk changes the billing terms from 30 to 15 and a subsequent view of the same identifier shows 15.
**Acceptance scenarios:** see ### US-3.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to create a customer record.
- **FR-002**: System MUST assign a unique identifier (`id`) to each customer; the identifier MUST NOT change after create.
- **FR-003**: A customer record MUST include `name`, `address`, `accountNumber`, and `billingTerms`.
- **FR-004**: `mainContactName` and `mainContactEmail` MAY be omitted on create and update.
- **FR-005**: `billingTerms` MUST be `15` or `30`, meaning the number of days until an invoice is due.
- **FR-006**: System MUST persist the customer so it can be retrieved after the create operation completes.
- **FR-007**: System MUST NOT create or update a customer when `name`, `address`, or `accountNumber` is missing or blank after trim.
- **FR-008**: System MUST NOT create or update a customer when another customer already has the same `accountNumber` (compared after trim).
- **FR-009**: System MUST allow an authorized owner/admin or accounting clerk to retrieve a customer by unique identifier and to list customers.
- **FR-010**: System MUST allow an authorized owner/admin or accounting clerk to update `name`, `address`, `accountNumber`, `mainContactName`, `mainContactEmail`, and `billingTerms` on an existing customer.
- **FR-011**: A project manager, a user who only enters time, and any caller outside the business MUST NOT create, view, list, or update customers.
- **FR-012**: Unauthenticated callers MUST NOT create, view, list, or update customers.

## Assumptions

- A customer is a client company, not the consulting firm in Feature 1.
- `accountNumber` is the client account number from the class notes. The sheet calls this column `number`.
- The sheet's "main content (name, email)" is the office main contact. This feature stores it as `mainContactName` and `mainContactEmail`.
- The project sponsor is not the customer's main contact. The sponsor belongs on the project.
- Different rates for each client are Feature 6, not this feature.
- Login / session management is out of scope.
- Deleting a customer is out of scope.

## Edge Cases

- Missing or blank `name`, `address`, or `accountNumber` → `ValidationException` or equivalent
- `billingTerms` other than `15` or `30` → `ValidationException` or equivalent
- Duplicate account number → `DuplicateException` or equivalent
- Unknown customer `id` on view/update → `NotFoundException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Empty customer list → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A valid new customer can be created and then retrieved by the assigned unique identifier
- **SC-003**: A duplicate account number never creates a second customer
- **SC-004**: After an update, view of the same `id` returns the new billing terms
- **SC-005**: Billing terms other than 15 or 30 are rejected

## Data Ownership

- Customer records are the consulting firm's client master data.
- Only an authorized owner/admin or accounting clerk may create, view, list, or update customers.
- Callers outside the business MUST NOT read or change customer data.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/customers` | Yes | Create a customer |
| `GET` | `/invoice/customers` | Yes | List customers |
| `GET` | `/invoice/customers/:id` | Yes | Retrieve one customer by `id` |
| `PUT` | `/invoice/customers/:id` | Yes | Update an existing customer |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may call them. This feature does not add a delete route.

**Create or update request body:**

```json
{
  "name": "ABC",
  "address": "ABC",
  "accountNumber": "ABC",
  "mainContactName": "ABC",
  "mainContactEmail": "abc@abc.com",
  "billingTerms": 30
}
```

`name`, `address`, `accountNumber`, and `billingTerms` are required. `mainContactName` and `mainContactEmail` may be omitted. `billingTerms` must be `15` or `30`. On update, `id` comes from the path. A body `id` is ignored. `accountNumber` is unique after trim, and the comparison keeps case.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 1,
  "name": "ABC",
  "address": "ABC",
  "accountNumber": "ABC",
  "mainContactName": "ABC",
  "mainContactEmail": "abc@abc.com",
  "billingTerms": 30,
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

Omitted main-contact fields are returned as empty strings. A list is `200` with an array of those objects. An empty list is `[]`, not an error. Each list row includes `id`, `name`, `accountNumber`, and `billingTerms`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `name`, `address`, or `accountNumber` missing or blank after trim | `400` | `"Customer name, address, and account number are required."` |
| `billingTerms` other than `15` or `30` | `400` | `"Billing terms must be 15 or 30."` |
| Another customer already has this `accountNumber` after trim | `400` | `"A customer with this account number already exists."` |
| No customer with that `id` | `404` | `"Customer with id=999 not found."` |
| Signed in, but not an owner/admin or accounting clerk | `403` | `"You are not allowed to manage customers."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Customers] — route name `customers`

* Heading: **Customers**
* Shown in `MenuBar` only for an owner/admin or accounting clerk.
* The page loads `GET /invoice/customers`.
* Rows show **id**, **name**, **account number**, and **billing terms**.
* Primary action: **Add customer** (`oc-cta`). It opens a dialog with **Name**, **Address**, **Account number**, **Main contact name**, **Main contact email**, and **Billing terms**. **Save** (`oc-cta`) calls `POST /invoice/customers`. **Cancel** closes the dialog.
* **Name**, **Address**, and **Account number** are required. The inline message is **"Customer name, address, and account number are required."**
* **Billing terms** offers only `15` and `30`.
* **Main contact name** and **Main contact email** may be left blank.
* Each row has a **View** action with `aria-label` **View customer**. It opens route `customer` for that `id`.
* **Empty state:** **"No customers yet. Add a client."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A project manager, a time-entry user, or a caller outside the business who opens this route sees **"You are not allowed to manage customers."** and no customer data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Customer] — route name `customer`

* Route param: `id`. The page loads `GET /invoice/customers/:id`.
* Heading: the customer **name**. The **id** is shown and cannot be edited.
* Fields: **Name**, **Address**, **Account number**, **Main contact name**, **Main contact email**, and **Billing terms**. **Billing terms** offers only `15` and `30`.
* Primary action: **Save** (`oc-cta`). It calls `PUT /invoice/customers/:id`.
* **Name**, **Address**, and **Account number** use the same required message as the add dialog.
* After a successful save, the page shows the returned values, including billing terms `15`, on the same **id**.
* **Loading state:** a skeleton while the customer is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Customer with id=999 not found."**, **"A customer with this account number already exists."**, **"Billing terms must be 15 or 30."**, and **"You are not allowed to manage customers."**
* No delete control.

## Key Entities

- **Customer**: a client the firm bills; identified by a unique system `id`; has name, address, account number, optional main contact, and billing terms of 15 or 30 days
- **Main contact**: the office contact at the client; stored on the customer, not a separate table

## Initial Data Model

### `customers` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `name` | STRING(150) | Required; stored trimmed |
| `address` | STRING(255) | Required; stored trimmed |
| `accountNumber` | STRING(50) | Required; stored trimmed; unique |
| `mainContactName` | STRING(150) | Optional; stored trimmed when present |
| `mainContactEmail` | STRING(255) | Optional; stored trimmed when present |
| `billingTerms` | INTEGER | Required; `15` or `30` |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

Unique business key: `accountNumber`, compared after trim.

## Gherkin AC

### US-3.1 — Add a customer

#### Scenario: Clerk adds a customer with valid information

* **Given** an authorized accounting clerk is using the system
* **When** the clerk enters name `ABC`, address `ABC`, account number `ABC`, main contact name `ABC`, main contact email `abc@abc.com`, and billing terms `30` and saves
* **Then** the system creates a customer record
* **And** the record has a unique `id`
* **And** a subsequent retrieve by that `id` returns those stored values

#### Scenario: Clerk adds a customer without a main contact

* **Given** an authorized accounting clerk is using the system
* **When** the clerk enters name `ABC`, address `ABC`, account number `ABC`, and billing terms `15` with no main contact and saves
* **Then** the system creates a customer record
* **And** `mainContactName` and `mainContactEmail` are empty

#### Scenario: Clerk cannot add a customer with missing required fields

* **Given** an authorized accounting clerk is using the system
* **When** the clerk attempts to save a customer without a name, without an address, or without an account number
* **Then** the system does not create a customer record
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot add billing terms other than 15 or 30

* **Given** an authorized accounting clerk is using the system
* **When** the clerk attempts to save a customer with billing terms `45`
* **Then** the system does not create a customer record
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot add a duplicate account number

* **Given** a customer exists with account number `ABC`
* **When** the clerk attempts to add another customer with account number `ABC`
* **Then** the system does not create a second customer
* **And** the system rejects the save with a duplicate error

### US-3.2 — View a customer

#### Scenario: Clerk views a customer

* **Given** a customer exists with `id` `1`, name `ABC`, account number `ABC`, and billing terms `30`
* **When** an authorized accounting clerk views customer `1`
* **Then** the system displays those stored values
* **And** the unique identifier `1` is shown

#### Scenario: Clerk lists customers

* **Given** customers `ABC` and `XYZ` exist
* **When** an authorized accounting clerk lists customers
* **Then** the list includes both customers
* **And** each row includes unique identifier, name, account number, and billing terms

#### Scenario: Clerk cannot view an unknown customer

* **Given** no customer exists with `id` `999`
* **When** an authorized accounting clerk views customer `999`
* **Then** the system reports that the customer was not found

### US-3.3 — Update a customer

#### Scenario: Clerk changes billing terms

* **Given** a customer exists with `id` `1` and billing terms `30`
* **When** an authorized accounting clerk changes billing terms to `15` and saves
* **Then** the customer is updated
* **And** a subsequent view of `id` `1` returns billing terms `15`
* **And** the unique identifier remains `1`

#### Scenario: Clerk cannot change an account number to one that already exists

* **Given** customer `1` has account number `ABC`
* **And** customer `2` has account number `XYZ`
* **When** an authorized accounting clerk changes customer `2` account number to `ABC`
* **Then** the system does not save the change
* **And** the system rejects the save with a duplicate error

#### Scenario: A time-entry user cannot view a customer

* **Given** a customer exists with `id` `1`
* **When** a user who only enters time views customer `1`
* **Then** the system does not display the customer
* **And** the system rejects the request as not authorized

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-012**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/customers` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] The **Customers** and **Customer** screens match **Screen Requirements**
* [ ] The `customers` table matches **Initial Data Model**
* [ ] This feature does not add projects, sponsors, role rates, invoices, delete, or login behavior listed in **Out of Scope**

## Out of Scope

- Projects under the customer — [Feature 4](feature-4-projects.md)
- Project sponsor
- Per-client role rates
- Invoices
- Deleting a customer
- Login / role management
