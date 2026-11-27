# Feature: Project Role

**Feature ID:** 6
**Branch pattern:** `feature/6-project-role`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Put a standard role on a project and keep the hourly rate that client will be billed, inheriting the standard rate when no project rate is entered.
**Depends on:** [Feature 4 — Projects](feature-4-projects.md), [Feature 5 — Roles](feature-5-roles.md)
**Related:** [Feature 7 — Project Role Assign](feature-7-project-role-assign.md)

## User Stories

### US-6.1: Add a role to a project

**As a** owner/admin, accounting clerk, or project manager
**I want to** add a billing role to a project with a rate
**So that** that client is billed at the agreed rate for that role

**Priority:** P1
**Independent test:** An authorized project manager adds Programmer to a project at `200` and a later view shows rate `200` for that project and role.
**Acceptance scenarios:** see ### US-6.1 under Gherkin AC

### US-6.2: Inherit the standard rate

**As a** accounting clerk
**I want to** add a role to a project without typing a rate
**So that** the project uses the role's current standard rate

**Priority:** P1
**Independent test:** Programmer's standard rate is `150`. An authorized clerk adds Programmer to a project with no rate, and the stored project rate is `150`.
**Acceptance scenarios:** see ### US-6.2 under Gherkin AC

### US-6.3: Change a project rate

**As a** owner/admin, accounting clerk, or project manager
**I want to** change the rate on a project role
**So that** later time on that project uses the new agreed rate

**Priority:** P1
**Independent test:** An authorized project manager changes a project role from `150` to `200` and a subsequent view shows `200` on the same project role.
**Acceptance scenarios:** see ### US-6.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to create a project role for an existing project and an existing role.
- **FR-002**: System MUST assign a unique identifier (`id`) to each project role; the identifier MUST NOT change after create.
- **FR-003**: A project role MUST include `projectId` and `roleId`.
- **FR-004**: When `rate` is omitted, System MUST store the role's current `standardRate` as this project role's `rate`.
- **FR-005**: When `rate` is provided, System MUST store that rate and MUST NOT replace it with `standardRate`.
- **FR-006**: The stored `rate` MUST be zero or greater.
- **FR-007**: System MUST NOT create a second project role for the same `projectId` and `roleId`.
- **FR-008**: System MUST NOT create a project role when the project or the role does not exist.
- **FR-009**: System MUST NOT create a project role on a project whose `status` is `closed`.
- **FR-010**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to retrieve a project role by unique identifier and to list project roles for a project.
- **FR-011**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to update `rate` on an existing project role.
- **FR-012**: System MUST NOT change `projectId` or `roleId` after create.
- **FR-013**: A later change to the role's `standardRate` MUST NOT change a rate already stored on a project role.
- **FR-014**: A user who only enters time, and any caller outside the business, MUST NOT create, view, list, or update project roles.
- **FR-015**: Unauthenticated callers MUST NOT create, view, list, or update project roles.

## Assumptions

- The class example is the same role at different rates: Project Manager at `150` on one project and `200` on another. That is two project-role rows, not two standard roles.
- The inherited rate is copied at save time. It is not a live link.
- Login / session management is out of scope.
- Deleting a project role is out of scope.
- Who is staffed on the role is Feature 7.

## Edge Cases

- Unknown project or unknown role → `NotFoundException` or equivalent
- Project status `closed` → `ValidationException` or equivalent
- Same role added twice to one project → `DuplicateException` or equivalent
- Negative rate → `ValidationException` or equivalent
- Unknown project role `id` → `NotFoundException` or equivalent
- Attempt to change `projectId` or `roleId` → `ValidationException` or equivalent
- Time-entry user or caller outside the business → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Project with no roles → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A project can store its own rate for a role
- **SC-003**: Omitting the rate stores the role's standard rate at that moment
- **SC-004**: A later change to the standard rate does not change the stored project rate
- **SC-005**: The same role cannot be added twice to one project

## Data Ownership

- A project role belongs to one project.
- Owner/admin, accounting clerk, and project manager may create, view, and update project rates.
- Callers outside the business MUST NOT read or change project rates.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/project-roles` | Yes | Add a role to a project |
| `GET` | `/invoice/projects/:projectId/project-roles` | Yes | List project roles for one project |
| `GET` | `/invoice/project-roles/:id` | Yes | Retrieve one project role by `id` |
| `PUT` | `/invoice/project-roles/:id` | Yes | Change the rate on a project role |

Every route requires a Bearer session. An owner/admin, accounting clerk, or project manager may call them. This feature does not add a delete route. A later `PUT /invoice/roles/:id` changes the standard rate only. It does not change `rate` on a project role already saved.

**Create request body:**

```json
{
  "projectId": 4,
  "roleId": 1,
  "rate": 200
}
```

`projectId` and `roleId` are required. `rate` may be omitted. When it is omitted, the system stores that role's current `standardRate`. When `rate` is sent, that value is stored. The project must exist, the role must exist, and the project `status` must not be `closed`. The same role cannot be added twice to one project.

**Update request body:**

```json
{
  "rate": 200
}
```

On update, `id` comes from the path. `projectId` and `roleId` cannot change. `rate` is required and must be zero or greater.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 8,
  "projectId": 4,
  "roleId": 1,
  "roleName": "Project Manager",
  "rate": 200,
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

`roleName` is read from the role. It is not a column on `project_roles`. A list is `200` with an array of those objects. A project with no roles returns `[]`, not an error. Each list row includes `id`, `roleName`, and `rate`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `projectId` or `roleId` missing on create | `400` | `"Project and role are required."` |
| `rate` below zero, or missing on update | `400` | `"Rate must be zero or greater."` |
| Project `status` is `closed` | `400` | `"A role cannot be added to a closed project."` |
| That role is already on that project | `400` | `"This role is already on the project."` |
| Update tries to change `projectId` or `roleId` | `400` | `"The project and role on a project role cannot be changed."` |
| No project with that `id` | `404` | `"Project with id=999 not found."` |
| No role with that `id` | `404` | `"Role with id=999 not found."` |
| No project role with that `id` | `404` | `"Project role with id=999 not found."` |
| A time-entry user or a caller outside the business | `403` | `"You are not allowed to manage project rates."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Project roles] — route name `project-roles`

* Route param: `projectId`. The page loads `GET /invoice/projects/:projectId/project-roles`.
* Heading: **Project roles**
* Shown in `MenuBar` for an owner/admin, an accounting clerk, or a project manager. The **Project** screen links here with **Project roles**.
* Rows show the project role **id**, **role**, and **rate**.
* Primary action: **Add role** (`oc-cta`), shown only when the project is `active`. It opens a dialog with **Role** and **Rate**. **Role** is a choice of standard roles. **Rate** may be left blank. **Save** (`oc-cta`) calls `POST /invoice/project-roles`. A blank rate is omitted so the standard rate is stored. **Cancel** closes the dialog.
* Each row has a **View** action with `aria-label` **View project role**. It opens route `project-role` for that `id`.
* **Empty state:** **"No roles on this project yet. Add a billing role."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A time-entry user or a caller outside the business who opens this route sees **"You are not allowed to manage project rates."** and no rates.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Project role] — route name `project-role`

* Route param: `id`. The page loads `GET /invoice/project-roles/:id`.
* Heading: the **role** name. The project role **id**, the **project**, and the **role** are shown and cannot be changed.
* Field: **Rate**. It must be zero or greater.
* Primary action: **Save** (`oc-cta`). It calls `PUT /invoice/project-roles/:id`. An owner/admin, accounting clerk, or project manager may save.
* After a successful save, the page shows the returned rate, including `200`, on the same **id**.
* **Loading state:** a skeleton while the project role is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Project role with id=999 not found."**, **"This role is already on the project."**, and **"You are not allowed to manage project rates."**
* No delete control.

## Key Entities

- **Project role**: one standard role priced for one project; identified by a unique system `id`; has a project, a role, and an hourly rate

## Initial Data Model

### `project_roles` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `projectId` | INTEGER FK | Required; references `projects.id`; never updated |
| `roleId` | INTEGER FK | Required; references `roles.id`; never updated |
| `rate` | DECIMAL(10,2) | Required; zero or greater; copied from `roles.standardRate` when omitted on create |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

Unique business key: (`projectId`, `roleId`).

## Gherkin AC

### US-6.1 — Add a role to a project

#### Scenario: Project manager sets a project-specific rate

* **Given** project `4` is active
* **And** role `1` is `Project Manager` with standard rate `150`
* **And** an authorized project manager is using the system
* **When** the project manager adds role `1` to project `4` with rate `200` and saves
* **Then** the system creates a project role
* **And** the stored rate is `200`
* **And** a subsequent retrieve returns project `4`, role `1`, and rate `200`

#### Scenario: The same role cannot be added twice

* **Given** project `4` already has a project role for role `1`
* **When** an authorized project manager attempts to add role `1` to project `4` again
* **Then** the system does not create a second project role
* **And** the system rejects the save with a duplicate error

#### Scenario: A role cannot be added to a closed project

* **Given** project `4` has status `closed`
* **And** role `1` exists
* **When** an authorized accounting clerk attempts to add role `1` to project `4`
* **Then** the system does not create a project role
* **And** the system rejects the save with a validation error

### US-6.2 — Inherit the standard rate

#### Scenario: Clerk omits the rate and the standard rate is stored

* **Given** role `2` is `Programmer` with standard rate `150`
* **And** project `3` is active
* **When** an authorized accounting clerk adds role `2` to project `3` with no rate and saves
* **Then** the stored project rate is `150`

#### Scenario: A later standard-rate change does not change the project rate

* **Given** project role `9` stores rate `150` for Programmer on project `3`
* **When** an authorized accounting clerk changes the Programmer standard rate to `175`
* **Then** project role `9` still has rate `150`

### US-6.3 — Change a project rate

#### Scenario: Project manager changes the project rate

* **Given** project role `8` is Project Manager on project `2` at rate `150`
* **When** an authorized project manager changes the rate to `200` and saves
* **Then** a subsequent view of project role `8` returns rate `200`
* **And** the project and role are unchanged

#### Scenario: A time-entry user cannot view project rates

* **Given** project role `8` exists
* **When** a user who only enters time views project role `8`
* **Then** the system does not display the project role
* **And** the system rejects the request as not authorized

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-015**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and `PUT` `/invoice/project-roles` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] Omitting `rate` on create stores the role's current `standardRate`, and a later standard-rate change leaves that stored rate in place
* [ ] The **Project roles** and **Project role** screens match **Screen Requirements**
* [ ] The `project_roles` table matches **Initial Data Model**, and `projectId` and `roleId` do not change after create
* [ ] This feature does not add assignments, time entry, invoices, or delete behavior listed in **Out of Scope**

## Out of Scope

- Assigning an employee to the project role — [Feature 7](feature-7-project-role-assign.md)
- Time entry and invoices
- Deleting a project role
- Changing the standard role catalog except by reading its current rate at create time
