# Feature: Roles

**Feature ID:** 5
**Branch pattern:** `feature/5-roles`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Keep the firm's standard billing roles and the rate each role inherits when it is put on a project.
**Depends on:** [Feature 1 — Company](feature-1-company.md)
**Related:** [Feature 6 — Project Role](feature-6-project-role.md)

## User Stories

### US-5.1: Create a standard role

**As a** owner/admin, accounting clerk, or project manager
**I want to** create a billing role with a standard rate
**So that** a project can inherit that rate instead of inventing a new rate every time

**Priority:** P1
**Independent test:** An authorized project manager creates a role named `Programmer` at `100` and a later view shows that name and rate.
**Acceptance scenarios:** see ### US-5.1 under Gherkin AC

### US-5.2: View standard roles

**As a** owner/admin, accounting clerk, or project manager
**I want to** view and list standard roles
**So that** I can choose a role when setting up a project

**Priority:** P1
**Independent test:** An authorized accounting clerk lists roles and sees each role's name and standard rate.
**Acceptance scenarios:** see ### US-5.2 under Gherkin AC

### US-5.3: Update a standard rate

**As a** owner/admin or accounting clerk
**I want to** change a role's standard rate
**So that** new project roles inherit the current rate

**Priority:** P1
**Independent test:** An authorized accounting clerk changes the Programmer rate from `100` to `125` and a subsequent view shows `125` on the same role.
**Acceptance scenarios:** see ### US-5.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to create a role.
- **FR-002**: System MUST assign a unique identifier (`id`) to each role; the identifier MUST NOT change after create.
- **FR-003**: A role record MUST include `name` and `standardRate`.
- **FR-004**: `standardRate` MUST be zero or greater.
- **FR-005**: System MUST NOT create or update a role when `name` is missing or blank after trim.
- **FR-006**: System MUST NOT create or update a role when another role already has the same `name` (compared case-insensitively after trim).
- **FR-007**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to retrieve a role by unique identifier and to list roles.
- **FR-008**: System MUST allow an authorized owner/admin or accounting clerk to update `name` and `standardRate`.
- **FR-009**: A project manager MUST NOT update an existing role's name or standard rate.
- **FR-010**: A user who only enters time, and any caller outside the business, MUST NOT create, view, list, or update roles.
- **FR-011**: Unauthenticated callers MUST NOT create, view, list, or update roles.
- **FR-012**: Changing `standardRate` MUST NOT change a rate already stored on a project role.

## Assumptions

- A role here is a billing role such as Programmer or Project Manager. It is not the caller's system role (owner/admin, accounting clerk, project manager, or user).
- Project managers may create roles. Accounting clerks and the owner manage the standard rates that later projects inherit.
- Inheritance happens when a project role is saved in Feature 6. This feature only stores the standard rate.
- Login / session management is out of scope.
- Deleting a role is out of scope.

## Edge Cases

- Missing name → `ValidationException` or equivalent
- Negative standard rate → `ValidationException` or equivalent
- Duplicate role name → `DuplicateException` or equivalent
- Unknown role `id` on view/update → `NotFoundException` or equivalent
- Project manager attempts to update a rate → `NotAuthorizedException` or equivalent
- Time-entry user or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Empty role list → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A new role can be created and then retrieved with its standard rate
- **SC-003**: Two roles cannot share a name
- **SC-004**: An accounting clerk can change the standard rate, and a project manager cannot
- **SC-005**: A negative standard rate is rejected

## Data Ownership

- Standard roles are firm-wide master data.
- Owner/admin, accounting clerk, and project manager may view roles.
- Only owner/admin and accounting clerk may change a role after it is created.
- Callers outside the business MUST NOT read or change roles.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/roles` | Yes | Create a standard role |
| `GET` | `/invoice/roles` | Yes | List standard roles |
| `GET` | `/invoice/roles/:id` | Yes | Retrieve one role by `id` |
| `PUT` | `/invoice/roles/:id` | Yes | Update a role's name or standard rate |

Every route requires a Bearer session. An owner/admin, accounting clerk, or project manager may create, list, and view. Only an owner/admin or accounting clerk may update. This feature does not add a delete route. Updating `standardRate` changes only this role. It does not change a rate already stored on a project role.

**Create or update request body:**

```json
{
  "name": "Programmer",
  "standardRate": 100
}
```

`name` and `standardRate` are required. `standardRate` must be zero or greater. On update, `id` comes from the path. A body `id` is ignored. `name` is unique case-insensitively after trim.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 1,
  "name": "Programmer",
  "standardRate": 100,
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

A list is `200` with an array of those objects. An empty list is `[]`, not an error. Each list row includes `id`, `name`, and `standardRate`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `name` missing or blank after trim, or `standardRate` missing | `400` | `"Role name and standard rate are required."` |
| `standardRate` below zero | `400` | `"Standard rate must be zero or greater."` |
| Another role already has this `name` | `400` | `"A role with this name already exists."` |
| No role with that `id` | `404` | `"Role with id=999 not found."` |
| A project manager calls `PUT` | `403` | `"You are not allowed to change a standard role."` |
| A time-entry user or a caller outside the business | `403` | `"You are not allowed to manage roles."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Roles] — route name `roles`

* Heading: **Roles**
* Shown in `MenuBar` for an owner/admin, an accounting clerk, or a project manager.
* The page loads `GET /invoice/roles`.
* Rows show **id**, **name**, and **standard rate**.
* Primary action: **Add role** (`oc-cta`). It opens a dialog with **Name** and **Standard rate**. **Save** (`oc-cta`) calls `POST /invoice/roles`. **Cancel** closes the dialog. A project manager may use this action.
* **Name** and **Standard rate** are required. The inline message is **"Role name and standard rate are required."**
* **Standard rate** must be zero or greater.
* Each row has a **View** action with `aria-label` **View role**. It opens route `role` for that `id`.
* **Empty state:** **"No roles yet. Add a billing role."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A time-entry user or a caller outside the business who opens this route sees **"You are not allowed to manage roles."** and no role data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Role] — route name `role`

* Route param: `id`. The page loads `GET /invoice/roles/:id`.
* Heading: the role **name**. The **id** is shown and cannot be edited.
* Fields: **Name** and **Standard rate**.
* An owner/admin or accounting clerk sees **Save** (`oc-cta`). It calls `PUT /invoice/roles/:id`.
* A project manager sees the name and rate and does not see **Save**.
* After a clerk saves, the page shows the returned rate, including `125`, on the same **id**.
* **Loading state:** a skeleton while the role is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Role with id=999 not found."**, **"A role with this name already exists."**, and **"You are not allowed to change a standard role."**
* No delete control.

## Key Entities

- **Role**: a standard billing role; identified by a unique system `id`; has a name and a standard hourly rate

## Initial Data Model

### `roles` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `name` | STRING(100) | Required; stored trimmed; unique case-insensitively |
| `standardRate` | DECIMAL(10,2) | Required; zero or greater |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

Unique business key: `name`, compared case-insensitively after trim.

## Gherkin AC

### US-5.1 — Create a standard role

#### Scenario: Project manager creates a role

* **Given** an authorized project manager is using the system
* **When** the project manager enters name `Programmer` and standard rate `100` and saves
* **Then** the system creates a role
* **And** the role has a unique `id`
* **And** a subsequent retrieve by that `id` returns `Programmer` and `100`

#### Scenario: Clerk cannot create a role with a negative rate

* **Given** an authorized accounting clerk is using the system
* **When** the clerk attempts to save a role named `Programmer` with standard rate `-5`
* **Then** the system does not create a role
* **And** the system rejects the save with a validation error

#### Scenario: Clerk cannot create a duplicate role name

* **Given** a role exists named `Programmer`
* **When** an authorized accounting clerk attempts to create a role named `programmer`
* **Then** the system does not create a second role
* **And** the system rejects the save with a duplicate error

#### Scenario: A time-entry user cannot create a role

* **Given** a user who only enters time is using the system
* **When** the user attempts to save a role named `Programmer`
* **Then** the system does not create a role
* **And** the system rejects the save as not authorized

### US-5.2 — View standard roles

#### Scenario: Clerk lists roles

* **Given** roles `Programmer` at `100` and `Project Manager` at `150` exist
* **When** an authorized accounting clerk lists roles
* **Then** the list includes both roles
* **And** each row includes unique identifier, name, and standard rate

#### Scenario: Clerk cannot view an unknown role

* **Given** no role exists with `id` `999`
* **When** an authorized accounting clerk views role `999`
* **Then** the system reports that the role was not found

### US-5.3 — Update a standard rate

#### Scenario: Clerk changes the standard rate

* **Given** role `1` is `Programmer` with standard rate `100`
* **When** an authorized accounting clerk changes the standard rate to `125` and saves
* **Then** a subsequent view of role `1` returns standard rate `125`
* **And** the unique identifier remains `1`

#### Scenario: Project manager cannot change the standard rate

* **Given** role `1` is `Programmer` with standard rate `100`
* **And** an authorized project manager is using the system
* **When** the project manager attempts to change the standard rate to `125`
* **Then** the system does not save the change
* **And** the standard rate remains `100`
* **And** the system rejects the save as not authorized

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-012**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/roles` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] The **Roles** and **Role** screens match **Screen Requirements**, including no **Save** for a project manager
* [ ] The `roles` table matches **Initial Data Model**
* [ ] This feature does not add project rates, assignments, time entry, invoices, or delete behavior listed in **Out of Scope**

## Out of Scope

- The rate used on one project — [Feature 6](feature-6-project-role.md)
- Assigning a person to a role
- Time entry and invoices
- Deleting a role
- Login / role management for system users
