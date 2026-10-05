# Feature: <children Registration>
**Feature ID:** N03
**Branch pattern:** `feature/3-child_Registration`
**Status:** Draft
**Created:** 2026-10-04
**Input:**   Enable Church Administration and staff to manage child member profiles.
**Depends on** [feature](\feature2_Adult_Member_Registration.md)
**Related:** optional links to ADRs or reference docs
---
## US-A.2.1: child Profile Creation
**As a** a Church Administrator
**I want to** register a new child member with their parent's name, Schools they go to and  Disability if any
**So that** the church maintains accurate membership records and can link family members togather
**Priority:** P1
**Independent test:** Enteer details on the registration form and submit; the new child member profile appears in the directory with the assigned ID 
**Acceptance scenarios:** see ### US-N.1 under Acceptance
Criteria

### US-N.2.2: View the child profile 
**As a**       Logged-in Staff Member
**I want to**  search and filter child member records by name or School.
**So that**    Ican quickly locate a child details during services or pastoral visits
**Priority:** P1
**Independent test:**  Search for an existing child member using a partial name ; matching records are returned in real time.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria

### US-N.2.3: Update the child profile 
**As a**       Logged-in Staff Member/ Church Administrator
**I want to**  Update child memebr information and link to his or her parents
**So that**     so that the system will know the updted information on the childrens and their families
**Priority:**  P1
**Independent test:**  Edit an child profile, and the changes appears on the list.
**Acceptance scenarios:** see ### US-N.2 under Acceptance

### US-C.3: School  and Grade Info Tracking for a Child
**As a** Sunday School Teacher or Administrator

**I want to** track school grade, age, and date of birth for each child

**So that** children can be assigned to the correct age-appropriate Sunday school class or grade division.

**Priority:** P2

**Independent Test:** Update a child's date of birth and grade level, verifying the system accurately maps them to the matching classroom group.


---
## Requirements
### Functional Requirements
- **FR-001**:System MUST allow staff to create, view, update, and soft-delete child member profiles.
- **FR-002**: System MUST enforce mandatory fields for profile creation:schoool atteded, name, and Primary Language.
- **FR-003**:System MUST automatically generate or assign a child ID upon registering a new primary child member.
- **FR-004**:System MUST provide real-time search functionality by First Name, Last Name, or school.

### US-N.1 — child profile creation
#### Scenario: Successful login
* **Given** a Church Administrator is on the child Registration form
* **When** they enter valid member details (First Name: "stine", Last Name: "AMIZERO", School: "Stars of christ", Language: "Spanidh", Disability: "None")
* **And** click "Save Member" / "Guardar Miembro"
* **then** the system creates a new record in child member and generates a corresponding fimily_id
* **And** displays a success confirmation message in the active session language.

#### Scenario: Missing required field
* **Given** a user is filling out the A child Registration form
* **Then**  they omit the mandatory School and submit the form
* **when**  the system prevents form submissioncredentials
* **And**   highlights the field with an inline validation error in  English or spanish depending on the language selectd


