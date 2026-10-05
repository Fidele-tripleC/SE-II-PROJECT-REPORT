# Feature: <Adult Member Registration>
**Feature ID:** N02
**Branch pattern:** `feature/2-adult Registration`
**Status:** Draft
**Created:** 2026-10-04
**Input:**   Enable Church Administration and staff to manage adult member profiles.
**Depends on** [feature](\Feature1_Authentication.md)
**Related:** optional links to ADRs or reference docs
---
## US-A.2.1: Adult Profile Creation
**As a** a Church Administrator
**I want to** register a new adult member with their contact and demographic information
**So that** the church maintains accurate membership records and can link family members togather
**Priority:** P1
**Independent test:** Enteer details on the registration form and submi; the new adult member profile appears in the directory with the assigned ID 
**Acceptance scenarios:** see ### US-N.1 under Acceptance
Criteria

### US-N.2.2: View the Adult profile 
**As a** Logged-in Staff Member
**I want to**  search and filter adultsd member records by name, Phone number, or email
**So that** Ican quickly locate member details during services or pastoral visits
**Priority:** P1
**Independent test:**  Search for an existing adult member using a partial phone number or last name; matching records are returned in real time.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria

### US-N.2.3: Update the Adult profile 
**As a** Logged-in Staff Member/ Church Administrator
**I want to**  Update adult memebr information and link spouses or family members
**So that** family units stay connected for check-in and communication purpose
**Priority:** P1
**Independent test:**  Edit an adult profile, and the changes appears on the lists.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria


---
## Requirements
### Functional Requirements
- **FR-001**:System MUST allow staff to create, view, update, and soft-delete adult member profiles.
- **FR-002**: System MUST enforce mandatory fields for profile creation: First Name, Last Name, Phone Number, and Primary Language.
- **FR-003**:System MUST automatically generate or assign a family_id upon registering a new primary adult member.
- **FR-004**:System MUST provide real-time search functionality by First Name, Last Name, or Phone Number (last 4 digits or full string).


### US-N.1 — Adult profile creation
#### Scenario: Successful login
* **Given** a Church Administrator is on the Adult Registration form
* **When** they enter valid member details (First Name: "Maria", Last Name: "González", Phone: "555-0192", Language: "ES")
* **And** click "Save Member" / "Guardar Miembro"
* **then** the system creates a new record in adult_members and generates a corresponding family_id
* **And** displays a success confirmation message in the active session language.

#### Scenario: Missing required field
* **Given** a user is filling out the Adult Registration form
* **Then**  they omit the mandatory phone number and submit the form
* **when**  the system prevents form submissioncredentials
* **And**   highlights the field with an inline validation error in  English or spanish depending on the language selectd


