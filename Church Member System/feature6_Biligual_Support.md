# Feature: <General management>
**Feature ID:** N06
**Branch pattern:** `feature/5-General Management`
**Status:** Draft
**Created:** 2026-10-04
**Input:** Comprehensive reporting dashboards, attendance analytics across the service dates/types, member demographics, and role management for Church Administratiors in both English and Spanish
**Depends on** [feature](\Feature1_Authentication.md),

**Related to**
 [Authentication](\Feature1_Authentication.md),
 [Member Registration](\feature2_Adult_Member_Registration.md),
 [Childrent Registration](\feature3_children_Registration.md),
 [service checkin](\feature4_ServiceCheck_inand_Security.md),
**Related:** [General Management](\feature5_GeneralManagement)
---
## User Stories
### US-N.6.1: Session Language Persistence & Global Switcher
**As a**Church Administrator, Usher, or Staff Member
**I want to** toggle the application language between English and Spanish from any screen or during logingroup (adults vs. children)
**So that**I can work comfortably in my preferred language without losing context or active form data.
**Priority:** P1
**Independent test:** Change the active language preference from English to Spanish on the main dashboard, navigate through multiple screens, and verify that all UI elements remain in Spanish until explicitly toggled back.
**Acceptance scenarios:** see ### US-N.1 under Acceptance
Criteria
### US-N.6.1: Localized Form Validation and Error Messaging
**As a**    receive system feedback, error messages, and validation prompts in my currently active session language
**I want to** I can immediately understand and resolve input errors without language barriers.
**So that**  I can immediately understand and resolve input errors without language barriers.
**Priority:** P1
**Independent test:** Trigger an inline validation error (e.g., missing required field) while in Spanish mode and confirm the error message renders in grammatically correct Spanish.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria

---
## Requirements
### Functional Requirements
- **FR-001**: System MUST store the active language selection (EN or ES) at the user profile level and persist it across session restarts.
- **FR-002**:ystem MUST provide a visible language toggle UI component accessible on the login screen and header navigation of all screens.
- **FR-003**:System MUST dynamically translate all static labels, navigation links, form headers, table columns, button actions, and tooltips based on dictionary translation keys.
- **FR-004**:System MUST return localized backend validation messages and error strings matching the requesting client's active language header or profile preference
- **FR-004**:System MUST fall back gracefully to English (EN) if a specific localization key is missing or undefined in Spanish (ES).

Data Model Re
## Aceptance criteria
### US-N.5.1 — User Login
#### Scenario: Successful login
* **Given** a Church Administrator is on the Management Dashboard
* **When**  they select a date range (e.g., "2026-10-01" to "2026-10-05")
* **Then** the dashboard renders overall attendance totals, percentage of adults vs. children, and attendance graph trends.

### US-B.2 — Localized System Feedback
#### Scenario:Displaying localized validation errors
* **Given**a user is operating the system in Spanish (ES)
* **when**an error occurs during child check-in or profile creation
* **Then** all dialog modals, field validation warnings, and toast notifications render entirely in Spanish.
