# Feature: Projects

**Feature ID:** 4
**Branch pattern:** `feature/4-projects`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Give each client project a unique number, a budget, dates, and a sponsor so time and invoices attach to the right job.
**Depends on:** [Feature 3 — Customers](feature-3-customers.md)
**Related:** [Feature 6 — Project Role](feature-6-project-role.md)

## User Stories

### US-4.1: Open a project

**As a** owner/admin or accounting clerk
**I want to** open a project for a customer with a name, budget, and planned end date
**So that** the firm has a unique project number under that client

**Priority:** P1
**Independent test:** An authorized accounting clerk creates a project for an existing customer and the system persists it with a unique identifier and status `active`.
**Acceptance scenarios:** see ### US-4.1 under Gherkin AC

### US-4.2: View a project

**As a** owner/admin or accounting clerk
**I want to** view and list projects
**So that** I can see the project number, customer, budget, and sponsor

**Priority:** P1
**Independent test:** An authorized accounting clerk opens an existing project and sees the stored budget, customer, and sponsor.
**Acceptance scenarios:** see ### US-4.2 under Gherkin AC

### US-4.3: Update a project budget

**As a** owner/admin or accounting clerk
**I want to** revise the budget and record cost-control notes
**So that** the approved budget stays on the same project number

**Priority:** P1
**Independent test:** An authorized accounting clerk sets a revised budget and a subsequent view of the same project shows that amount.
**Acceptance scenarios:** see ### US-4.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to create a project for an existing customer.
- **FR-002**: System MUST assign a unique identifier (`id`) to each project. That identifier is the unique project number and MUST NOT change after create.
- **FR-003**: A project record MUST include `name`, `customerId`, `estEndDate`, and `budget`.
- **FR-004**: `revisedBudget`, `costControlInfo`, `sponsorName`, and `sponsorContact` MAY be omitted on create and update.
- **FR-005**: System MUST set `status` to `active` when a project is opened.
- **FR-006**: System MUST set `statusDate` to today's date when a project is created, and MUST set `statusDate` to today's date again whenever `status` changes.
- **FR-007**: System MUST NOT create a project when `customerId` does not match an existing customer.
- **FR-008**: System MUST NOT create or update a project when `name` is missing or blank after trim.
- **FR-009**: System MUST NOT create or update a project when another project for the same customer already has the same `name` (compared case-insensitively after trim).
- **FR-010**: `budget` MUST be zero or greater. When `revisedBudget` is present it MUST be zero or greater.
- **FR-011**: `status` MUST be `active` or `closed`.
- **FR-012**: System MUST allow an authorized owner/admin or accounting clerk to retrieve a project by unique identifier and to list projects.
- **FR-013**: System MUST allow an authorized owner/admin or accounting clerk to update `name`, `estEndDate`, `budget`, `revisedBudget`, `costControlInfo`, `sponsorName`, `sponsorContact`, and `status` on an existing project.
- **FR-014**: System MUST NOT change `customerId` after create.
- **FR-015**: A project manager, a user who only enters time, and any caller outside the business MUST NOT create, view, list, or update projects.
- **FR-016**: Unauthenticated callers MUST NOT create, view, list, or update projects.

## Assumptions

- Feature 3 customers exist. A project has exactly one customer.
- The class notes ask for a sponsor at the client for the project. The sheet has no sponsor column, so this feature adds optional `sponsorName` and `sponsorContact`.
- The class notes say the budget is approved. Owner/admin and the accounting clerk record the approved `budget` and any `revisedBudget`. A separate approval workflow is out of scope.
- `costControlInfo` is free text from the sheet. This feature stores it and does not calculate cost.
- The sheet lists `status` without values. This feature uses `active` and `closed`.
- The same project name MAY exist for two different customers. It MUST NOT be repeated for one customer.
- Login / session management is out of scope.
- Deleting a project is out of scope. Close it by setting `status` to `closed`.

## Edge Cases

- Unknown `customerId` → `NotFoundException` or equivalent
- Missing name, estimated end date, or budget → `ValidationException` or equivalent
- Negative budget or revised budget → `ValidationException` or equivalent
- Duplicate project name for the same customer → `DuplicateException` or equivalent
- `status` other than `active` or `closed` → `ValidationException` or equivalent
- Unknown project `id` on view/update → `NotFoundException` or equivalent
- Attempt to change `customerId` → `ValidationException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Empty project list → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A valid project can be created for an existing customer and then retrieved by its project number
- **SC-003**: Two projects for the same customer cannot share a name
- **SC-004**: A revised budget is visible on a later view of the same project number
- **SC-005**: A project cannot be created for an unknown customer

## Data Ownership

- Projects are the consulting firm's client work.
- Only an authorized owner/admin or accounting clerk may create, view, list, or update projects.
- A project MUST remain attached to the customer it was created for.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/projects` | Yes | Open a project for an existing customer |
| `GET` | `/invoice/projects` | Yes | List projects |
| `GET` | `/invoice/projects/:id` | Yes | Retrieve one project by project number |
| `PUT` | `/invoice/projects/:id` | Yes | Update an existing project |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may call them. This feature does not add a delete route.

**Create request body:**

```json
{
  "name": "ABC",
  "customerId": 1,
  "estEndDate": "2026-12-31",
  "budget": 50000,
  "sponsorName": "ABC",
  "sponsorContact": "abc@abc.com"
}
```

`name`, `customerId`, `estEndDate`, and `budget` are required. `revisedBudget`, `costControlInfo`, `sponsorName`, and `sponsorContact` may be omitted. `status` and `statusDate` are not accepted on create. The system sets `status` to `active` and `statusDate` to today's date.

**Update request body:**

```json
{
  "name": "ABC",
  "estEndDate": "2026-12-31",
  "budget": 50000,
  "revisedBudget": 62000,
  "costControlInfo": "ABC",
  "sponsorName": "ABC",
  "sponsorContact": "abc@abc.com",
  "status": "closed"
}
```

On update, `id` comes from the path. A body `id` is ignored. `customerId` cannot change. `status` must be `active` or `closed`. When `status` changes, the system sets `statusDate` to today's date. `budget` and a present `revisedBudget` must be zero or greater. The same `name` may exist for two customers. It must not repeat for one customer, compared case-insensitively after trim.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 4,
  "name": "ABC",
  "customerId": 1,
  "customerName": "ABC",
  "statusDate": "2026-11-26",
  "estEndDate": "2026-12-31",
  "budget": 50000,
  "revisedBudget": null,
  "costControlInfo": "",
  "status": "active",
  "sponsorName": "ABC",
  "sponsorContact": "abc@abc.com",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

`customerName` is read from the customer. It is not a column on `projects`. An omitted `revisedBudget` is `null`. Omitted optional text fields are empty strings. `statusDate` in this example is the server's current date. A list is `200` with an array of those objects. An empty list is `[]`, not an error. Each list row includes `id`, `name`, `customerName`, `budget`, and `status`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `name`, `estEndDate`, or `budget` missing or blank | `400` | `"Project name, estimated end date, and budget are required."` |
| `budget` or `revisedBudget` below zero | `400` | `"Budget and revised budget must be zero or greater."` |
| Another project for the same customer already has this `name` | `400` | `"This customer already has a project with this name."` |
| `status` other than `active` or `closed` | `400` | `"Status must be active or closed."` |
| Update tries to change `customerId` | `400` | `"The customer on a project cannot be changed."` |
| `customerId` does not match a customer | `404` | `"Customer with id=999 not found."` |
| No project with that `id` | `404` | `"Project with id=999 not found."` |
| Signed in, but not an owner/admin or accounting clerk | `403` | `"You are not allowed to manage projects."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Projects] — route name `projects`

* Heading: **Projects**
* Shown in `MenuBar` only for an owner/admin or accounting clerk.
* The page loads `GET /invoice/projects`.
* Rows show **project number**, **name**, **customer**, **budget**, and **status**.
* Primary action: **Open project** (`oc-cta`). It opens a dialog with **Name**, **Customer**, **Estimated end date**, **Budget**, **Sponsor name**, and **Sponsor contact**. **Customer** is a choice of existing customers. **Save** (`oc-cta`) calls `POST /invoice/projects`. **Cancel** closes the dialog. The new project is `active`. The dialog has no status field.
* **Name**, **Customer**, **Estimated end date**, and **Budget** are required. The inline message is **"Project name, estimated end date, and budget are required."**
* **Budget** must be zero or greater.
* Each row has a **View** action with `aria-label` **View project**. It opens route `project` for that `id`.
* **Empty state:** **"No projects yet. Open a project for a client."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A project manager, a time-entry user, or a caller outside the business who opens this route sees **"You are not allowed to manage projects."** and no project data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Project] — route name `project`

* Route param: `id`. The page loads `GET /invoice/projects/:id`.
* Heading: the project **name**. The project number is shown and cannot be edited. The **customer** is shown and cannot be changed.
* Fields: **Name**, **Estimated end date**, **Budget**, **Revised budget**, **Cost-control info**, **Sponsor name**, **Sponsor contact**, and **Status**. **Status** offers only `active` and `closed`.
* Primary action: **Save** (`oc-cta`). It calls `PUT /invoice/projects/:id`.
* **Budget** and **Revised budget** must be zero or greater. **Revised budget** may be left blank.
* After a successful save, the page shows the returned values, including a revised budget of `62000` or status `closed`, on the same project number. Closing the project shows today's date as **status date**.
* **Loading state:** a skeleton while the project is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Project with id=999 not found."**, **"This customer already has a project with this name."**, and **"You are not allowed to manage projects."**
* No delete control. A finished project is marked `closed`.

## Key Entities

- **Project**: one job for one customer; identified by a unique project number (`id`); has dates, budget, optional revised budget, cost-control notes, status, and optional sponsor
- **Sponsor**: the client person responsible for that project; stored on the project, not a separate table

## Initial Data Model

### `projects` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique project number; never updated |
| `name` | STRING(150) | Required; stored trimmed; unique per `customerId`, case-insensitively |
| `customerId` | INTEGER FK | Required; references `customers.id`; never updated |
| `statusDate` | DATE | Required; set to today on create and whenever `status` changes |
| `estEndDate` | DATE | Required |
| `budget` | DECIMAL(12,2) | Required; zero or greater |
| `revisedBudget` | DECIMAL(12,2) | Optional; zero or greater when present |
| `costControlInfo` | STRING(500) | Optional; stored trimmed when present |
| `status` | STRING(20) | Required; `active` or `closed`; default `active` on create |
| `sponsorName` | STRING(150) | Optional; stored trimmed when present |
| `sponsorContact` | STRING(255) | Optional; stored trimmed when present |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

## Gherkin AC

### US-4.1 — Open a project

#### Scenario: Clerk opens a project for an existing customer

* **Given** customer `1` is `ABC`
* **And** an authorized accounting clerk is using the system
* **When** the clerk enters name `ABC`, customer `1`, estimated end date `2026-12-31`, budget `50000`, sponsor name `ABC`, and sponsor contact `abc@abc.com` and saves
* **Then** the system creates a project
* **And** the project has a unique `id`
* **And** `status` is `active`
* **And** `statusDate` is today's date
* **And** a subsequent retrieve by that `id` returns those stored values

#### Scenario: Clerk cannot open a project for an unknown customer

* **Given** no customer exists with `id` `999`
* **When** an authorized accounting clerk attempts to open a project for customer `999`
* **Then** the system does not create a project
* **And** the system reports that the customer was not found

#### Scenario: Clerk cannot open a project with a negative budget

* **Given** customer `1` exists
* **When** an authorized accounting clerk attempts to save a project for customer `1` with budget `-1`
* **Then** the system does not create a project
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot repeat a project name for the same customer

* **Given** customer `1` already has a project named `ABC`
* **When** an authorized accounting clerk attempts to open another project named `abc` for customer `1`
* **Then** the system does not create a second project
* **And** the system rejects the save with a duplicate error

### US-4.2 — View a project

#### Scenario: Clerk views a project

* **Given** project `4` is `ABC` for customer `1` with budget `50000` and sponsor `ABC`
* **When** an authorized accounting clerk views project `4`
* **Then** the system displays those stored values
* **And** the unique project number `4` is shown

#### Scenario: Clerk lists projects

* **Given** projects `ABC` and `XYZ` exist
* **When** an authorized accounting clerk lists projects
* **Then** the list includes both projects
* **And** each row includes project number, name, customer, budget, and status

#### Scenario: Clerk cannot view an unknown project

* **Given** no project exists with `id` `999`
* **When** an authorized accounting clerk views project `999`
* **Then** the system reports that the project was not found

### US-4.3 — Update a project budget

#### Scenario: Clerk records a revised budget

* **Given** project `4` has budget `50000` and no revised budget
* **When** an authorized accounting clerk sets revised budget to `62000` and cost-control info `ABC` and saves
* **Then** the project is updated
* **And** a subsequent view of project `4` returns revised budget `62000`
* **And** the project number remains `4`
* **And** the customer is unchanged

#### Scenario: Clerk closes a project

* **Given** project `4` has status `active`
* **When** an authorized accounting clerk sets status to `closed` and saves
* **Then** a subsequent view of project `4` returns status `closed`
* **And** `statusDate` is today's date

#### Scenario: Project manager cannot open a project

* **Given** customer `1` exists
* **And** an authorized project manager is using the system
* **When** the project manager attempts to open a project for customer `1`
* **Then** the system does not create a project
* **And** the system rejects the save as not authorized

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-016**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/projects` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] The **Projects** and **Project** screens match **Screen Requirements**
* [ ] The `projects` table matches **Initial Data Model**, and `customerId` does not change after create
* [ ] This feature does not add roles, assignments, time entry, invoices, or delete behavior listed in **Out of Scope**

## Out of Scope

- Standard roles and project rates — [Feature 5](feature-5-roles.md) and [Feature 6](feature-6-project-role.md)
- Assigning employees
- Time entry and approving time
- Comparing actual cost to the budget
- Invoices
- Deleting a project
- Contracts as a separate record
