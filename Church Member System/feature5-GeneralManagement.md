# Feature: <General management>
**Feature ID:** N01
**Branch pattern:** `feature/5-General Management`
**Status:** Draft
**Created:** 2026-10-04
**Input:** Comprehensive reporting dashboards, attendance analytics across the service dates/types, member demographics, and role management for Church Administratiors in both English and Spanish
**Depends on**
 [Authentication](\Feature1_Authentication.md),
 [Adult Registration](\feature2_Adult_Member_Registration.md),
 [Children Registration](\feature3_children_Registration.md),
 [Service check in](\feature4_ServiceCheck_inand_Security.md),
**Related:** [Bilingual Support](\feature6_BilingualSupport)
---
## User Stories
### US-N.1.1: Attendance Reporting and Filtering
**As a** a Church Administrator or pastor
**I want to** view attendance summaries filtered by date range, service type, or age group (adults vs. children)
**So that** I can track church growth trends, analyze service participation, and make informed ministry decisions.
**Priority:** P1
**Independent test:**  Select a specific date range on the reporting dashboard and verfy that the total attendance count, adult/child breakdowns, and trends match underlying check-in logs.
**Acceptance scenarios:** see ### US-N.1 under Acceptance
Criteria
### US-N.5.1: Staff User and Role Management
**As a**      Church Administrator
**I want to** create, update, and manage staff user accounts and assign roles (Admin, Pastor, Usher)
**So that**  system access is restricted strictly according to operational duties.
**Priority:** P1
**Independent test:** Create a new staff account with role Usher and verify they can perform check-ins but cannot access admin reporting screens.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria

### US-N.1.3: Demographic difference List Exporting
**As a** Church Administrator or Children's Ministry Leader
**I want to** generate and export member rosters and emergency contact lists
**So that** staff can maintain offline backups and accurate classroom headcounts.

**Priority:** P2

**Independent Test:** Filter children's attendance for a specific Sunday class and export the list, verifying all guardian contact numbers and medical flags are included
---
## Requirements
### Functional Requirements
- **FR-001**: System MUST provide an interactive administrative dashboard displaying overall attendance totals, breakdowns by age (Adult vs. Child), and date ranges
- **FR-002**:System MUST support dynamic filtering of check-in history by date, service time/type, and member category.
- **FR-003**: System MUST allow Administrators to manage user accounts (Create, Read, Update, Deactivate) and assign system roles (Admin, Pastor, Usher).
- **FR-004**:System MUST enforce Role-Based Access Control (RBAC) across all administrative management and reporting routes.
- **FR-004**: System MUST prevent unauthorized access to sensitive records when invalid credentials are provided

## Aceptance criteria
### US-N.5.1 — User Login
#### Scenario: Successful login
* **Given** a Church Administrator is on the Management Dashboard
* **When**  they select a date range (e.g., "2026-10-01" to "2026-10-05")
* **Then** the dashboard renders overall attendance totals, percentage of adults vs. children, and attendance graph trends.


#### Scenario: Exporting classroom attendance list
* **Given** an Administrator needs a hard copy of Sunday School check-ins
* **when** they apply the filter for "Children's Ministry - Pre-K" for the current date and click "Export" / "Exportar"
* **Then**  the system generates a downloadable file containing child names, guardian emergency contacts, and allergy badges.

### US-N.5.1 —Staff Role Management
#### Scenario: Assigning a new staff role
* **Given** a Church Administrator is on the User Management screen
* **When**  they create a new user account "j.doe" and select role "Usher"
* **And**   user "j.doe" can successfully log in but is restricted from accessing system user management or full administrative reporting.

####Scenario: Unauthorized access attempt by non-admin role
* **Given a** user is logged in with the role "Usher"

* **When**  they attempt to directly navigate to the User Management URL

* **Then** the system blocks access and displays an access denied message ("Unauthorized Access" / "Acceso No Autorizado").