# Feature: Project Role Assign

**Feature ID:** 7
**Branch pattern:** `feature/7-project-role-assign`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Assign an employee to a role on a project so that person can enter time against that role.
**Depends on:** [Feature 2 — Employees](feature-2-employees.md), [Feature 6 — Project Role](feature-6-project-role.md)
**Related:** [Feature 8 — Project Timeline](feature-8-project-timeline.md)

## User Stories

### US-7.1: Assign an employee to a project role

**As a** owner/admin, accounting clerk, or project manager
**I want to** assign an active employee to a role already priced on a project
**So that** the employee can enter time for that project and role

**Priority:** P1
**Independent test:** An authorized project manager assigns ABC to Programmer on project `4` and a later view shows that assignment.
**Acceptance scenarios:** see ### US-7.1 under Gherkin AC

### US-7.2: Give one employee several roles

**As a** project manager
**I want to** assign the same employee to more than one role
**So that** a person who does more than one kind of work can be timed for each role

**Priority:** P1
**Independent test:** ABC is assigned to Programmer on project `4` and to Project Manager on project `2`, and both assignments exist.
**Acceptance scenarios:** see ### US-7.2 under Gherkin AC

### US-7.3: View assignments

**As a** owner/admin, accounting clerk, or project manager
**I want to** list who is assigned to a project
**So that** I know who may enter time before I review a timesheet

**Priority:** P1
**Independent test:** An authorized project manager lists assignments for a project and sees the employee and the project role on each row.
**Acceptance scenarios:** see ### US-7.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to create an assignment.
- **FR-002**: System MUST assign a unique identifier (`id`) to each assignment; the identifier MUST NOT change after create.
- **FR-003**: An assignment MUST include `projectId`, `employeeId`, and `projectRoleId`.
- **FR-004**: `projectRoleId` MUST reference a project role whose `projectId` is the same as the assignment's `projectId`.
- **FR-005**: The employee MUST exist and MUST have `status` `active`.
- **FR-006**: The project MUST exist and MUST have `status` `active`.
- **FR-007**: System MUST NOT create a second assignment for the same `projectRoleId` and `employeeId`.
- **FR-008**: System MUST allow an authorized owner/admin, accounting clerk, or project manager to retrieve an assignment by unique identifier and to list assignments for a project.
- **FR-009**: System MUST NOT change `projectId`, `employeeId`, or `projectRoleId` after create.
- **FR-010**: A user who only enters time MAY view the assignments for their own employee record and MUST NOT create assignments or view another employee's assignments.
- **FR-011**: Any other caller outside the business, and unauthenticated callers, MUST NOT create, view, or list assignments.

## Assumptions

- The sheet lists `projectId` and `EmployeeId` on Project Role Assign. Time entry also names the role, and this entity is an assignment of a project role, so this feature adds `id` and `projectRoleId`. `id` is what Feature 8 stores as `projectRoleAssignId`.
- One employee MAY hold several assignments. That is the class note "users can have several roles."
- Login / session management is out of scope. A time-entry user's own `employeeId` is already known.
- Removing an assignment is out of scope. Do not assign an inactive employee.

## Edge Cases

- Unknown employee, project, or project role → `NotFoundException` or equivalent
- Inactive employee → `ValidationException` or equivalent
- Closed project → `ValidationException` or equivalent
- Project role belongs to a different project → `ValidationException` or equivalent
- Same employee and project role assigned twice → `DuplicateException` or equivalent
- Time-entry user views another employee's assignment → `NotAuthorizedException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Project with no assignments → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: An active employee can be assigned to a priced role on an active project
- **SC-003**: One employee can hold two assignments on different roles
- **SC-004**: The same employee cannot be assigned twice to the same project role
- **SC-005**: An inactive employee cannot be assigned

## Data Ownership

- An assignment connects one employee to one project role.
- Owner/admin, accounting clerk, and project manager may create and list assignments.
- A time-entry user may see only their own assignments.
- Callers outside the business MUST NOT read or change assignments.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/assignments` | Yes | Assign an active employee to a project role |
| `GET` | `/invoice/projects/:projectId/assignments` | Yes | List assignments for one project |
| `GET` | `/invoice/assignments/:id` | Yes | Retrieve one assignment by `id` |

Every route requires a Bearer session. An owner/admin, accounting clerk, or project manager may create and list. A time-entry user may retrieve only an assignment whose `employeeId` is their own, and the project list returns only those rows. This feature does not add an update route or a delete route. `projectId`, `employeeId`, and `projectRoleId` do not change after create.

**Create request body:**

```json
{
  "projectId": 4,
  "employeeId": 1,
  "projectRoleId": 8
}
```

`projectId`, `employeeId`, and `projectRoleId` are required. The employee must exist and be `active`. The project must exist and be `active`. The project role must exist and belong to that same project. The same employee cannot be assigned twice to the same project role. One employee may hold assignments on different project roles.

**Success response** (`201` on create, `200` on view):

```json
{
  "id": 3,
  "projectId": 4,
  "employeeId": 1,
  "employeeName": "ABC",
  "projectRoleId": 8,
  "roleName": "Programmer",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

`employeeName` and `roleName` are read from the employee and the role. They are not columns on `project_role_assignments`. A list is `200` with an array of those objects. A project with no assignments returns `[]`, not an error. Each list row includes `id`, `employeeName`, and `roleName`. For a time-entry user, that array contains only assignments for their `employeeId`.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| `projectId`, `employeeId`, or `projectRoleId` missing | `400` | `"Project, employee, and project role are required."` |
| Employee `status` is not `active` | `400` | `"Only an active employee can be assigned."` |
| Project `status` is not `active` | `400` | `"An employee cannot be assigned on a closed project."` |
| Project role belongs to a different project | `400` | `"The project role does not belong to this project."` |
| That employee is already assigned to that project role | `400` | `"This employee is already assigned to that project role."` |
| No employee, project, or project role with that `id` | `404` | `"Employee with id=999 not found."`, `"Project with id=999 not found."`, or `"Project role with id=999 not found."` |
| No assignment with that `id` | `404` | `"Assignment with id=999 not found."` |
| A time-entry user creates an assignment | `403` | `"You are not allowed to create an assignment."` |
| A time-entry user views another employee's assignment | `403` | `"You are not allowed to view this assignment."` |
| Any other caller outside the business |  `403` | `"You are not allowed to manage assignments."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Assignments] — route name `assignments`

* Route param: `projectId`. The page loads `GET /invoice/projects/:projectId/assignments`.
* Heading: **Assignments**
* Shown in `MenuBar` for an owner/admin, an accounting clerk, a project manager, or a time-entry user. The **Project** screen links here with **Assignments**.
* Rows show the assignment **id**, **employee**, and **project role**.
* An owner/admin, accounting clerk, or project manager sees every assignment on the project.
* A time-entry user sees only their own rows.
* Primary action: **Assign employee** (`oc-cta`), shown only for an owner/admin, accounting clerk, or project manager, and only when the project is `active`. It opens a dialog with **Employee** and **Project role**. **Employee** lists active employees. **Project role** lists roles already priced on this project. **Save** (`oc-cta`) calls `POST /invoice/assignments`. **Cancel** closes the dialog.
* A time-entry user does not see **Assign employee**.
* Each row has a **View** action with `aria-label` **View assignment**. It opens route `assignment` for that `id`.
* **Empty state:** **"No one is assigned to this project yet."** For a time-entry user with no row: **"You have no assignment on this project."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A caller outside the business who opens this route sees **"You are not allowed to manage assignments."** and no assignment data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control and no edit control.

### [View: Assignment] — route name `assignment`

* Route param: `id`. The page loads `GET /invoice/assignments/:id`.
* Heading: the **employee** name. The assignment **id**, **project**, **employee**, and **project role** are shown and cannot be changed.
* There is no **Save** action.
* An owner/admin, accounting clerk, or project manager may open any assignment.
* A time-entry user may open only their own assignment. Another employee's assignment shows **"You are not allowed to view this assignment."** and no assignment data.
* **Loading state:** a skeleton while the assignment is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Assignment with id=999 not found."**
* No delete control.

## Key Entities

- **Project role assignment**: one active employee staffed on one project role; identified by a unique system `id`

## Initial Data Model

### `project_role_assignments` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated; referenced by time entry as `projectRoleAssignId` |
| `projectId` | INTEGER FK | Required; references `projects.id`; must match the project role's project; never updated |
| `employeeId` | INTEGER FK | Required; references `employees.id`; employee must be `active`; never updated |
| `projectRoleId` | INTEGER FK | Required; references `project_roles.id`; never updated |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create |

Unique business key: (`projectRoleId`, `employeeId`).

## Gherkin AC

### US-7.1 — Assign an employee to a project role

#### Scenario: Project manager assigns an active employee

* **Given** employee `1` ABC is `active`
* **And** project role `8` is Programmer on active project `4` at rate `100`
* **And** an authorized project manager is using the system
* **When** the project manager assigns employee `1` to project role `8` on project `4` and saves
* **Then** the system creates an assignment
* **And** the assignment has a unique `id`
* **And** a subsequent retrieve returns employee `1`, project `4`, and project role `8`

#### Scenario: An inactive employee cannot be assigned

* **Given** employee `1` is `inactive`
* **And** project role `8` is on active project `4`
* **When** an authorized project manager attempts to assign employee `1` to project role `8`
* **Then** the system does not create an assignment
* **And** the system rejects the save with a validation error

#### Scenario: The project role must belong to that project

* **Given** project role `8` belongs to project `4`
* **And** employee `1` is `active`
* **When** an authorized project manager attempts to assign employee `1` to project role `8` with project `2`
* **Then** the system does not create an assignment
* **And** the system rejects the save with a validation error

### US-7.2 — Give one employee several roles

#### Scenario: The same employee is assigned to two roles

* **Given** employee `1` is assigned to Programmer on project `4`
* **And** project role `9` is Project Manager on active project `2`
* **When** an authorized project manager assigns employee `1` to project role `9` on project `2`
* **Then** both assignments exist
* **And** they have different identifiers

#### Scenario: The same employee and project role cannot be assigned twice

* **Given** employee `1` is already assigned to project role `8`
* **When** an authorized accounting clerk attempts to assign employee `1` to project role `8` again
* **Then** the system does not create a second assignment
* **And** the system rejects the save with a duplicate error

### US-7.3 — View assignments

#### Scenario: Project manager lists assignments for a project

* **Given** project `4` has ABC assigned as Programmer and XYZ assigned as Tester
* **When** an authorized project manager lists assignments for project `4`
* **Then** the list includes both assignments
* **And** each row includes the assignment identifier, employee, and project role

#### Scenario: A time-entry user can view only their own assignment

* **Given** the signed-in user is employee `1`
* **And** assignment `3` belongs to employee `1`
* **And** assignment `4` belongs to employee `2`
* **When** the user views assignment `3`
* **Then** the system displays assignment `3`
* **When** the user views assignment `4`
* **Then** the system does not display assignment `4`
* **And** the system rejects the request as not authorized

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-011**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST` and `GET` `/invoice/assignments` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] A time-entry user can view only their own assignments and cannot create one
* [ ] The **Assignments** and **Assignment** screens match **Screen Requirements**
* [ ] The `project_role_assignments` table matches **Initial Data Model**, and `projectId`, `employeeId`, and `projectRoleId` do not change after create
* [ ] This feature does not add time entry, rate changes, or removal behavior listed in **Out of Scope**

## Out of Scope

- Entering hours — [Feature 8](feature-8-project-timeline.md)
- Changing the project rate
- Removing an assignment
- Login / role management
