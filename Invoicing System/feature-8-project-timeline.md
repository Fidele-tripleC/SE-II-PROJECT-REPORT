# Feature: Project Timeline

**Feature ID:** 8
**Branch pattern:** `feature/8-project-timeline`
**Status:** Draft
**Created:** 2026-11-26
**Input:** Replace the shared time spreadsheet with time entries that a project manager can approve or deny.
**Depends on:** [Feature 7 — Project Role Assign](feature-7-project-role-assign.md)
**Related:** [Feature 10 — Line Item](feature-10-line-item.md)

## User Stories

### US-8.1: Enter time

**As a** user assigned to a project role
**I want to** enter the date and hours for one of my assignments
**So that** my work is recorded against that project and role instead of a shared spreadsheet

**Priority:** P1
**Independent test:** ABC enters 8 hours on the Programmer assignment and the saved entry is `Pending` with those hours.
**Acceptance scenarios:** see ### US-8.1 under Gherkin AC

### US-8.2: Add missing time

**As a** accounting clerk
**I want to** enter time for an employee who missed the spreadsheet
**So that** the hours still exist before an invoice is generated

**Priority:** P1
**Independent test:** An authorized accounting clerk enters 4 hours for XYZ on that assignment and the entry is `Pending` for XYZ, not for the clerk.
**Acceptance scenarios:** see ### US-8.2 under Gherkin AC

### US-8.3: Review a timesheet

**As a** project manager
**I want to** approve or deny pending time
**So that** only reviewed hours can later be billed

**Priority:** P1
**Independent test:** A project manager approves a pending entry and a subsequent view shows `Approved`. A second pending entry that the manager denies shows `Denied`.
**Acceptance scenarios:** see ### US-8.3 under Gherkin AC

## Functional Requirements

- **FR-001**: System MUST allow a time-entry user to create a time entry for an assignment whose `employeeId` is that user.
- **FR-002**: System MUST allow an authorized owner/admin or accounting clerk to create a time entry for any existing assignment.
- **FR-003**: System MUST assign a unique identifier (`id`) to each time entry; the identifier MUST NOT change after create.
- **FR-004**: A time entry MUST include `projectRoleAssignId`, `projectId`, `date`, and `hours`.
- **FR-005**: `projectId` MUST match the project on that assignment.
- **FR-006**: `hours` MUST be greater than zero.
- **FR-007**: `date` MUST NOT be after today.
- **FR-008**: System MUST set `status` to `Pending` when a time entry is created.
- **FR-009**: System MUST leave `invoicedItemId` empty until a line item is created for this entry.
- **FR-010**: The employee on the assignment MUST be `active`, and the project MUST be `active`.
- **FR-011**: System MUST allow the owning user, while the entry is `Pending`, to update `date` and `hours`.
- **FR-012**: An authorized owner/admin or accounting clerk MAY update `date` and `hours` on a `Pending` entry.
- **FR-013**: System MUST NOT update `date` or `hours` when `status` is `Approved`, `Denied`, or `Invoiced`.
- **FR-014**: An authorized project manager or owner/admin MUST be able to set a `Pending` entry to `Approved` or `Denied`.
- **FR-015**: A time-entry user MUST NOT approve or deny time, including their own.
- **FR-016**: System MUST NOT set `status` to `Invoiced` in this feature.
- **FR-017**: System MUST allow a project manager, owner/admin, or accounting clerk to list time entries for a project.
- **FR-018**: A time-entry user MUST be able to list only their own time entries.
- **FR-019**: Unauthenticated callers, and callers outside the business, MUST NOT create, view, list, or update time entries.

## Assumptions

- The enter-time screen shows Person, Status, Project, Role, Date, and Hours. Person, project, and role come from the assignment. The user enters date and hours.
- The screen words Pending, Reviewed, and Approved describe the review. The stored statuses in this feature are `Pending`, `Approved`, and `Denied`. Review is the act of setting `Approved` or `Denied`. `Invoiced` is set by Feature 10.
- The sheet also lists `Invoiced` on the timeline. This feature creates the row and reviews it. It does not mark it invoiced.
- A date range such as `12/1-12/15` on the sample invoice is a billed line, not one timeline row. Each timeline row has one `date`.
- Hours may be greater than 24. The sample service line is 60 hours.
- Login / session management is out of scope.
- Deleting a time entry is out of scope. Deny it instead.

## Edge Cases

- Unknown assignment → `NotFoundException` or equivalent
- Assignment belongs to another employee and the caller is a time-entry user → `NotAuthorizedException` or equivalent
- Inactive employee or closed project → `ValidationException` or equivalent
- Hours of zero or less → `ValidationException` or equivalent
- Date after today → `ValidationException` or equivalent
- Update when status is not `Pending` → `ValidationException` or equivalent
- Approve or deny when status is not `Pending` → `ValidationException` or equivalent
- Time-entry user attempts to approve → `NotAuthorizedException` or equivalent
- Unknown time entry `id` → `NotFoundException` or equivalent
- Unauthenticated caller → `NotAuthenticatedException` or equivalent
- Project with no time entries → empty list, not an error

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: A user can save hours only against their own assignment, and the entry starts as `Pending`
- **SC-003**: A project manager can approve or deny pending time
- **SC-004**: The person who entered the time cannot approve it
- **SC-005**: Approved, denied, and invoiced hours cannot be edited

## Data Ownership

- A time entry belongs to one assignment and one project.
- The employee who owns the assignment may create and view their own pending and past entries.
- Accounting clerks may add missing time for any assignment.
- Project managers and the owner may review pending time.
- Callers outside the business MUST NOT read or change time entries.

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `POST` | `/invoice/time-entries` | Yes | Enter time against an assignment |
| `GET` | `/invoice/projects/:projectId/time-entries` | Yes | List time entries for one project |
| `GET` | `/invoice/time-entries/:id` | Yes | Retrieve one time entry |
| `PUT` | `/invoice/time-entries/:id` | Yes | Change `date` and `hours` while `Pending` |
| `PUT` | `/invoice/time-entries/:id/status` | Yes | Set a `Pending` entry to `Approved` or `Denied` |

Every route requires a Bearer session. A time-entry user may create, list, view, and correct only entries for their own assignments. An owner/admin or accounting clerk may create and correct time for any assignment. A project manager, owner/admin, or accounting clerk may list every entry on a project. Only a project manager or owner/admin may set `Approved` or `Denied`. This feature does not add a delete route and does not set `Invoiced` or `invoicedItemId`.

**Create request body:**

```json
{
  "projectRoleAssignId": 3,
  "projectId": 4,
  "date": "2026-11-20",
  "hours": 8
}
```

`projectRoleAssignId`, `projectId`, `date`, and `hours` are required. `projectId` must match the assignment. `hours` must be greater than zero and may be greater than 24. `date` must not be after today. The employee must be `active` and the project must be `active`. The system sets `status` to `Pending` and leaves `invoicedItemId` empty. A body `status` or `invoicedItemId` is ignored.

**Correct pending time:**

```json
{
  "date": "2026-11-20",
  "hours": 7.5
}
```

This is allowed only while `status` is `Pending`. The assignment and project do not change.

**Review body:**

```json
{
  "status": "Approved"
}
```

`status` must be `Approved` or `Denied`, and the entry must already be `Pending`. `Invoiced` is rejected.

**Success response** (`201` on create, `200` on view and update):

```json
{
  "id": 10,
  "projectRoleAssignId": 3,
  "projectId": 4,
  "employeeName": "ABC",
  "roleName": "Programmer",
  "date": "2026-11-20",
  "hours": 8,
  "invoicedItemId": null,
  "status": "Pending",
  "createdAt": "2026-11-26T12:00:00.000Z",
  "updatedAt": "2026-11-26T12:00:00.000Z"
}
```

`employeeName` and `roleName` are read from the assignment. They are not columns on `project_timelines`. A list is `200` with an array of those objects. A project with no entries returns `[]`. Each list row includes the person, `status`, project, role, `date`, and `hours`. A time-entry user's list contains only their own entries.

**Error response:** `{ "message": "Human-readable explanation." }`

| Condition | Status | Message |
|-----------|--------|---------|
| Assignment, project, date, or hours missing | `400` | `"Assignment, project, date, and hours are required."` |
| `hours` is zero or less | `400` | `"Hours must be greater than zero."` |
| `date` is after today | `400` | `"Date cannot be after today."` |
| `projectId` does not match the assignment | `400` | `"The project does not match the assignment."` |
| Employee is not `active`, or the project is not `active` | `400` | `"Time can be entered only for an active employee on an active project."` |
| Date or hours change when status is not `Pending` | `400` | `"Only pending time can be changed."` |
| Review when status is not `Pending`, or status is not `Approved` or `Denied` | `400` | `"Only pending time can be approved or denied."` |
| Caller tries to set `Invoiced` | `400` | `"Time cannot be marked invoiced here."` |
| No assignment with that `id` | `404` | `"Assignment with id=999 not found."` |
| No time entry with that `id` | `404` | `"Time entry with id=999 not found."` |
| A time-entry user uses someone else's assignment or entry | `403` | `"You are not allowed to enter time for this assignment."` |
| A time-entry user, or an accounting clerk, reviews time | `403` | `"You are not allowed to review time."` |
| A project manager enters or corrects time | `403` | `"You are not allowed to enter time."` |
| A caller outside the business | `403` | `"You are not allowed to manage time entries."` |
| No session | `401` | `"Authentication is required."` |

## Screen Requirements

### [View: Time entries] — route name `time-entries`

* Route param: `projectId`. The page loads `GET /invoice/projects/:projectId/time-entries`.
* Heading: **Time entries**
* Shown in `MenuBar` for an owner/admin, an accounting clerk, a project manager, or a time-entry user.
* Rows show **person**, **status**, **project**, **role**, **date**, and **hours**. Status text is `Pending`, `Approved`, or `Denied`.
* A time-entry user sees only their own rows. An owner/admin, accounting clerk, or project manager sees every row on the project.
* Primary action: **Enter time** (`oc-cta`), shown for a time-entry user, an owner/admin, or an accounting clerk. It is not shown for a project manager. The dialog has **Assignment**, **Date**, and **Hours**. A time-entry user can choose only their own assignments. **Save** (`oc-cta`) calls `POST /invoice/time-entries`. **Cancel** closes the dialog.
* **Date** and **Hours** are required. Hours must be greater than zero. The inline message for zero hours is **"Hours must be greater than zero."**
* Each row has a **View** action with `aria-label` **View time entry**. It opens route `time-entry` for that `id`.
* **Empty state:** **"No time entries for this project yet."** For a time-entry user with no row: **"You have no time entries on this project."**
* **Loading state:** a skeleton while the list is loading.
* **Error state:** `<v-alert type="error">` with the API `message`.
* A caller outside the business sees **"You are not allowed to manage time entries."** and no time data.
* A caller with no session sees **"Authentication is required."** Login itself is out of scope for this feature.
* No delete control.

### [View: Time entry] — route name `time-entry`

* Route param: `id`. The page loads `GET /invoice/time-entries/:id`.
* Heading: the **date**. The **id**, **person**, **project**, **role**, and **status** are shown. Person, project, and role cannot be edited.
* While `status` is `Pending`, the owning user, an owner/admin, or an accounting clerk can change **Date** and **Hours**. **Save** (`oc-cta`) calls `PUT /invoice/time-entries/:id`. The status stays `Pending`.
* While `status` is `Pending`, a project manager or owner/admin sees **Approve** and **Deny**. Those call `PUT /invoice/time-entries/:id/status` with `Approved` or `Denied`.
* An accounting clerk does not see **Approve** or **Deny**. A time-entry user does not see them, including on their own entry.
* When status is `Approved`, `Denied`, or `Invoiced`, **Date** and **Hours** are read-only.
* **Loading state:** a skeleton while the entry is loading.
* **Error state:** `<v-alert type="error">` with the API `message`, including **"Only pending time can be changed."**, **"You are not allowed to review time."**, and **"Time entry with id=999 not found."**
* No delete control. Denied time stays denied.

## Key Entities

- **Time entry**: hours worked on one date for one project-role assignment; identified by a unique system `id`; status is `Pending`, `Approved`, `Denied`, or later `Invoiced`

## Initial Data Model

### `project_timelines` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment; unique identifier; never updated |
| `projectRoleAssignId` | INTEGER FK | Required; references `project_role_assignments.id`; never updated |
| `projectId` | INTEGER FK | Required; references `projects.id`; must match the assignment's project; never updated |
| `date` | DATE | Required; not after today; editable only while `Pending` |
| `hours` | DECIMAL(6,2) | Required; greater than zero; editable only while `Pending` |
| `invoicedItemId` | INTEGER | Empty until Feature 10; MUST NOT be set by this feature |
| `status` | STRING(20) | Required; `Pending` on create; review may set `Approved` or `Denied` |
| `createdAt` | DATETIME | Required; set on create |
| `updatedAt` | DATETIME | Required; set on create and whenever the row changes |

## Gherkin AC

### US-8.1 — Enter time

#### Scenario: User enters time on their own assignment

* **Given** assignment `3` is ABC as Programmer on active project `4`
* **And** the signed-in user is ABC
* **When** ABC enters date `2026-11-20` and hours `8` for assignment `3` and saves
* **Then** the system creates a time entry
* **And** `status` is `Pending`
* **And** `projectId` is `4`
* **And** `invoicedItemId` is empty
* **And** a subsequent retrieve returns those values

#### Scenario: User cannot enter time on someone else's assignment

* **Given** assignment `4` belongs to XYZ
* **And** the signed-in user is ABC
* **When** ABC attempts to enter 8 hours for assignment `4`
* **Then** the system does not create a time entry
* **And** the system rejects the save as not authorized

#### Scenario: User cannot enter zero hours

* **Given** assignment `3` belongs to the signed-in user
* **When** the user attempts to save hours `0`
* **Then** the system does not create a time entry
* **And** the system rejects the save with a validation error

#### Scenario: User can correct pending hours

* **Given** time entry `10` belongs to the signed-in user and is `Pending` with hours `8`
* **When** the user changes the hours to `7.5` and saves
* **Then** a subsequent view of time entry `10` returns hours `7.5`
* **And** the status remains `Pending`

### US-8.2 — Add missing time

#### Scenario: Clerk enters missing time for an employee

* **Given** assignment `4` is XYZ on project `2`
* **And** an authorized accounting clerk is using the system
* **When** the clerk enters date `2026-11-18` and hours `4` for assignment `4` and saves
* **Then** the system creates a time entry for assignment `4`
* **And** `status` is `Pending`

### US-8.3 — Review a timesheet

#### Scenario: Project manager approves pending time

* **Given** time entry `10` is `Pending`
* **And** an authorized project manager is using the system
* **When** the project manager sets the status to `Approved`
* **Then** a subsequent view of time entry `10` returns `Approved`

#### Scenario: Project manager denies pending time

* **Given** time entry `11` is `Pending`
* **When** an authorized project manager sets the status to `Denied`
* **Then** a subsequent view of time entry `11` returns `Denied`

#### Scenario: The user who entered the time cannot approve it

* **Given** time entry `10` is `Pending` and belongs to the signed-in user
* **When** the user attempts to set the status to `Approved`
* **Then** the system does not change the status
* **And** the status remains `Pending`
* **And** the system rejects the save as not authorized

#### Scenario: Approved hours cannot be edited

* **Given** time entry `10` is `Approved` with hours `8`
* **When** an authorized accounting clerk attempts to change the hours to `9`
* **Then** the system does not save the change
* **And** the hours remain `8`

#### Scenario: Pending time cannot be marked invoiced here

* **Given** time entry `10` is `Pending`
* **When** an authorized accounting clerk attempts to set the status to `Invoiced`
* **Then** the system does not change the status
* **And** the system rejects the save with a validation error

## Definition of Done

* [ ] Backend and frontend implement **FR-001** through **FR-019**
* [ ] **SC-001** through **SC-005** are met
* [ ] Every Gherkin scenario in this file has at least one automated test, and `npm test` passes
* [ ] `POST`, `GET`, and both `PUT` routes under `/invoice/time-entries` match **API Requirements**, including `400`, `401`, `403`, and `404`
* [ ] A new entry is `Pending`, with an empty `invoicedItemId`, and this feature cannot set `Invoiced`
* [ ] The **Time entries** and **Time entry** screens match **Screen Requirements**
* [ ] The `project_timelines` table matches **Initial Data Model**
* [ ] This feature does not add invoice lines, delete, or login behavior listed in **Out of Scope**

## Out of Scope

- Creating the invoice line and setting `invoicedItemId` — [Feature 10](feature-10-line-item.md)
- Generating an invoice
- Deleting a time entry
- Login / role management
