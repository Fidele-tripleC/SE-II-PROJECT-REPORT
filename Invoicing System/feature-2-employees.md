# Feature: Employees

**Feature ID:** 2
**Branch pattern:** `feature/2-employees`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Let an accounting clerk add the people who will enter time, and keep each person on one employee record.
**Depends on:** [Feature 1 — Company](feature-1-company.md)
**Related:** [Feature 7 — Project Role Assign](feature-7-project-role-assign.md)

## User Stories

### US-2.1: Add an employee

**As a** owner/admin or accounting clerk
**I want to** add a person who can be timed
**So that** the firm has a unique employee record before that person is assigned to a project

**Priority:** P1
**Independent test:** An authorized accounting clerk enters a name and email and the system persists an employee with status `active` and a unique identifier.
**Acceptance scenarios:** see ### US-2.1 under Gherkin AC

### US-2.2: View an employee

**As a** owner/admin or accounting clerk
**I want to** view and list employees
**So that** I can pick the right person when assigning work

**Priority:** P1
**Independent test:** An authorized accounting clerk opens an existing employee and sees the stored name, title, email, and status.
**Acceptance scenarios:** see ### US-2.2 under Gherkin AC

### US-2.3: Update an employee

**As a** owner/admin or accounting clerk
**I want to** correct an employee's information or mark the employee inactive
**So that** someone who has left is not available for new time entry, without deleting the record

**Priority:** P1
**Independent test:** An authorized accounting clerk sets an employee to `inactive` and a subsequent view shows `inactive` on the same identifier.
**Acceptance scenarios:** see ### US-2.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin or accounting clerk to create an employee record.
- **FR-002**: System MUST assign a unique identifier (`id`) to each employee; the identifier MUST NOT change after create.
- **FR-003**: An employee record MUST include `name` and `email`; both are required to save.
- **FR-004**: `title` MAY be omitted on create and update.
- **FR-005**: System MUST set `status` to `active` when an employee is first added.
- **FR-006**: System MUST persist the employee so it can be retrieved after the create operation completes.
- **FR-007**: System MUST NOT create or update an employee when `name` or `email` is missing or blank after trim.
- **FR-008**: System MUST NOT create or update an employee when another employee already has the same `email` (compared case-insensitively after trim).
- **FR-009**: System MUST allow an authorized owner/admin or accounting clerk to retrieve an employee by unique identifier and to list employees.
- **FR-010**: System MUST allow an authorized owner/admin or accounting clerk to update `name`, `title`, `email`, and `status` on an existing employee.
- **FR-011**: `status` MUST be `active` or `inactive`.
- **FR-012**: A project manager, a user who only enters time, and any caller outside the business MUST NOT create, view, list, or update employees.
- **FR-013**: Unauthenticated callers MUST NOT create, view, list, or update employees.

## Assumptions

- Feature 1 company record exists. Employees belong to that firm. The sheet does not put `companyId` on the employee, so this feature does not add one.
- Login / session management is out of scope. The caller is already known as owner/admin, accounting clerk, project manager, or user.
- `title` is the person's job title. It is not the billing role used on an invoice.
- One person has one employee record. Several billing roles for that person are Feature 7.
- Deleting an employee is out of scope. Set `status` to `inactive` instead.
- The sheet names the column `status` and does not list the values. This feature uses `active` and `inactive`.

## Edge Cases

- Missing or blank `name` or `email` → reject create/update (`ValidationException` or equivalent)
- Duplicate email → reject create/update (`DuplicateException` or equivalent)
- `status` other than `active` or `inactive` → `ValidationException` or equivalent
- Unknown employee `id` on view/update → `NotFoundException` or equivalent
- Project manager, time-entry user, or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Empty employee list → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A valid new employee can be created and then retrieved by the assigned unique identifier
- **SC-003**: A duplicate email never creates a second employee
- **SC-004**: After an update, view of the same `id` returns the new field values
- **SC-005**: A project manager cannot add an employee

## Key Entities

- **Employee**: a person who can be timed; identified by a unique system `id`; has name, optional title, email, and status
- **Accounting clerk**: authorized operator who adds the people to be timed
- **Owner/Admin**: authorized operator who can manage employees

## Initial Data Model

### `employees` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `name` | STRING(150) | Required; stored trimmed |
| `title` | STRING(100) | Optional; stored trimmed when present |
| `email` | STRING(255) | Required; stored trimmed; unique case-insensitively |
| `status` | STRING(20) | Required; `active` or `inactive`; default `active` on create |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

Unique business key: `email`, compared case-insensitively after trim.

## Gherkin AC

### US-2.1 — Add an employee

#### Scenario: Clerk adds an employee with valid information

* **Given** an authorized accounting clerk is using the system
* **When** the clerk enters name `ABC`, title `ABC`, and email `abc@abc.com` and saves
* **Then** the system creates an employee record
* **And** the record has a unique `id`
* **And** `status` is `active`
* **And** a subsequent retrieve by that `id` returns `ABC`, `ABC`, and `abc@abc.com`

#### Scenario: Clerk adds an employee without a title

* **Given** an authorized accounting clerk is using the system
* **When** the clerk enters name `XYZ` and email `xyz@xyz.com` with no title and saves
* **Then** the system creates an employee record
* **And** `title` is empty
* **And** `status` is `active`

#### Scenario: Clerk cannot add an employee with missing required fields

* **Given** an authorized accounting clerk is using the system
* **When** the clerk attempts to save an employee without a name or without an email
* **Then** the system does not create an employee record
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot add a duplicate email

* **Given** an employee exists with email `abc@abc.com`
* **When** the clerk attempts to add another employee with email `Abc@abc.com`
* **Then** the system does not create a second employee
* **And** the system rejects the save with a duplicate error

#### Scenario: Project manager cannot add an employee

* **Given** an authorized project manager is using the system
* **When** the project manager attempts to save an employee named `ABC`
* **Then** the system does not create an employee record
* **And** the system rejects the save as not authorized

### US-2.2 — View an employee

#### Scenario: Clerk views an employee

* **Given** an employee exists with `id` `1`, name `ABC`, title `ABC`, email `abc@abc.com`, and status `active`
* **When** an authorized accounting clerk views employee `1`
* **Then** the system displays those stored values
* **And** the unique identifier `1` is shown

#### Scenario: Clerk lists employees

* **Given** employees `ABC` and `XYZ` exist
* **When** an authorized accounting clerk lists employees
* **Then** the list includes both employees
* **And** each row includes unique identifier, name, email, and status

#### Scenario: Clerk cannot view an unknown employee

* **Given** no employee exists with `id` `999`
* **When** an authorized accounting clerk views employee `999`
* **Then** the system reports that the employee was not found

### US-2.3 — Update an employee

#### Scenario: Clerk marks an employee inactive

* **Given** an employee exists with `id` `1`, name `ABC`, and status `active`
* **When** an authorized accounting clerk sets status to `inactive` and saves
* **Then** the employee is updated
* **And** a subsequent view of `id` `1` returns status `inactive`
* **And** the unique identifier remains `1`

#### Scenario: Clerk cannot change an employee email to one that already exists

* **Given** employee `1` has email `abc@abc.com`
* **And** employee `2` has email `xyz@xyz.com`
* **When** an authorized accounting clerk changes employee `2` email to `abc@abc.com`
* **Then** the system does not save the change
* **And** the system rejects the save with a duplicate error

#### Scenario: Clerk cannot update an unknown employee

* **Given** no employee exists with `id` `999`
* **When** an authorized accounting clerk attempts to update employee `999`
* **Then** the system reports that the employee was not found

## Data Ownership

- Employee records are the consulting firm's staff data.
- Only an authorized owner/admin or accounting clerk may create, view, list, or update employees.
- Callers outside the business MUST NOT read or change employee data.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/employees` | Yes | Create an employee |
| `GET` | `/invoice/employees` | Yes | List employees |
| `GET` | `/invoice/employees/:id` | Yes | Retrieve one employee by `id` |
| `PUT` | `/invoice/employees/:id` | Yes | Update an existing employee |

Every route requires a Bearer session. Only an owner/admin or accounting clerk may call them. This feature does not add a delete route.

**Create request body:**

```json
{
  "name": "ABC",
  "title": "ABC",
  "email": "abc@abc.com"
}
```

`name` and `email` are required. `title` may be omitted. `status` is not accepted on create. The system sets it to `active`.

**Update request body:**

```json
{
  "name": "ABC",
  "title": "ABC",
  "email": "abc@abc.com",
  "status": "inactive"
}
```

On update, `id` comes from the path. A body `id` is ignored. `status` must be `active` or `inactive`.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 1,
  "name": "ABC",
  "title": "ABC",
  "email": "abc@abc.com",
  "status": "active",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

An omitted `title` is returned as an empty string. A list is `200` with an array of those objects. An empty list is `[]`, not an error. Each list row includes `id`, `name`, `email`, and `status`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `name` or `email` missing or blank after trim | `400` | `"Employee name and email are required."` |
| Another employee already has this `email`, compared case-insensitively after trim | `400` | `"An employee with this email already exists."` |
| `status` other than `active` or `inactive` | `400` | `"Status must be active or inactive."` |
| No employee with that `id` | `404` | `"Employee with id=999 not found."` |
| Signed in, but not an owner/admin or accounting clerk | `403` | `"You are not allowed to manage employees."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Employees] — route name `employees`

* Heading: **Employees**
* Shown in `MenuBar` only for an owner/admin or accounting clerk.
* The page loads `GET /invoice/employees`.
* Rows show **id**, **name**, **email**, and **status**.
* Primary action: **Add employee** (`oc-cta`). It opens a dialog with **Name**, **Title**, and **Email**. **Save** (`oc-cta`) calls `POST /invoice/employees`. **Cancel** closes the dialog. The new employee is `active`. The dialog has no status field.
* **Name** and **Email** are required. The inline message is **"Employee name and email are required."**
* Each row has a **View** action with `aria-label` **View employee**. It opens route `employee` for that `id`.
* **Empty state:** **"No employees yet. Add a person who can be timed."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A project manager, a time-entry user, or a caller outside the business who opens this route sees **"You are not allowed to manage employees."** and no employee data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Employee] — route name `employee`

* Route param: `id`. The page loads `GET /invoice/employees/:id`.
* Heading: the employee **name**. The **id** is shown and cannot be edited.
* Fields: **Name**, **Title**, **Email**, and **Status**. **Status** offers only `active` and `inactive`.
* Primary action: **Save** (`oc-cta`). It calls `PUT /invoice/employees/:id`.
* **Name** and **Email** use the same required message as the add dialog.
* After a successful save, the page shows the returned values, including `inactive`, on the same **id**.
* **Loading state:** a skeleton while the employee is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Employee with id=999 not found."**, **"An employee with this email already exists."**, and **"You are not allowed to manage employees."**
* No delete control. An employee who has left is marked `inactive`.

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-013**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/employees` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] The **Employees** and **Employee** screens match **Screen Requirements**
* [ ] The `employees` table matches **Initial Data Model**
* [ ] This feature does not add login, project-role assignment, time entry, or delete behavior listed in **Out of Scope**

## Out of Scope

- Login accounts and passwords
- Assigning an employee to a project role — [Feature 7](feature-7-project-role-assign.md)
- Entering time
- Deleting an employee
- Customers, projects, invoices, and line items
