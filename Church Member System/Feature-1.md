# Feature: <Member Registration>

**Feature ID:** 1
**Branch pattern:** `feature/1-Member-registration`  
**Status:** Draft  
**Created:** 2026-09-10 
**Input:** Provide Church with a digital way to register and maintain individaul member records. 
**Depends on:** None
**Related:**No link yet 

---

## User Stories

### US-N.1: Register Achurch Member
**As a** Church Administrator  
**I want to**  register a new Church member in the System.
**So that** So that the Church can have the approprite and accurate record of members.

**Priority:** P1  
**Independent test:** An authorised church Administrator can entere a valid member  information and save a new member record.
**Acceptance scenarios:** see ### US-N.1 under Acceptance Criteria

### US-N.2: View Member Information
**As a**           A church Administator  
**I want to**      View a registered member's information 
**So that**        I can access the member's current information when needed.

**Priority:**             P1  
**Independent test:**     An authorised user can select a registered                  member                       and viewthe information stored for that member.
**Acceptance scenarios:** see ### US-N.2 under Acceptance Criteria

### US-N.3: Update Member Information
**As a**           A church Administator  
**I want to**      a member's information 
**So that**        church's record remain uodate and current.

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST Allow the church an authorised church administrator to to    create  a member record…
- **FR-001**: System MUST assign  a unique identifier to each registered member 

- **FR-001**: System MUST Store the required member information in a digital record

- **FR-001**: System MUST prevent duplicate member records when the system can determine that the member is arleady registered.

- **FR-003**: … MUST NOT create a member record when required information is missing …

---


## Data Model Requirements

### `table_name` table
--------------------------------------------------------------------------
| Field             | Type              | Rules                          |
|-------------------|------             |---------------------           |
| ID                | INTEGER PK        | Auto-increment                 |
| ..................| …..........       |.…..................            |
|first_name         | VARCHAR	        |   Required                     |
|...................|...................|................................|
|last_name          |VARCHAR	        |   Required                     |
|...................|...................|................................|
|date_of_birth      |DATE	            |   Required                     |
|...................|...................|................................|
|phone	            |VARCHAR	        |   Optional                     |
|...................|...................|................................|
|address	        |VARCHAR            |	Optional                     |
|...................|...................|................................|
|membership_status	|VARCHAR	        |   Required; active by default  |
|...................|...................|................................|
|created_at	DATETIME|Required           |................................|
|updated_at	DATETIME|Updated when information changes|...................|
--------------------------------------------------------------------------

### Associations (if known)
- …•	One member can have one current demographic record.
   •	A demographic record belongs to one member.
---

## Acceptance Criteria

### US-N.2.1 —  Record Member Demographics
#### Scenario:  Add demographic information
*   **Given** Given the system creates a unique number records
*   **When** When an authorized user enters valid demographic information
*   **Then**  then the system saves the information
*   **And**  And the information is associated with the correct member



### US.2.2 —Record Children's School Information
#### Scenario:Record a child's school
*   **Given** … a registered member is member is identified as a child
*   **When** … an authorized user enters the child's school
*   **Then** …  the system stores the school information and school information
                is displayed in the child's record


#### Scenario: Update Member information
*   **Given** … a member already exsts in the system
*   **When** … an authrised user changes valid member information and and saves it
*   **Then** … the system stores the update 


# Member Demographics
# Feature: <Member Demographic Information>

**Feature ID:** 2
**Branch pattern:** feature/2-member-demographics  
**Status:** Draft  
**Created:** 2026-09-10 
**Input:** Allow the church to maintain useful demographic information about members, including children's schools, residence and member's professions
**Depends on:** Feature 1 -Memnber Registration
**Related:**I is found above in this document

---

## User Stories

### US-N.2.1:Record Member Demographics
**As a** Church Administrator  
**I want to** To record demograpghicinformation about members 
**So that** The church can understan and maintain accurate information about its membership

**Priority:** P1  
**Independent test:** An authorised church Administrator can enter the details on demographic and background of any church member.
**Acceptance scenarios:** see US-2.1 under Gherkin AC

### US-N.2.2:Record Chldren's School Information

**As a**Church Administrator   
**I want to**  the School Children Attends.
**So that** The church can mantain the relevant information about the childre in the congregation



**Priority:** P1  
**Independent test:** An authorized Church Administrator can add and view a children's school information.
**Acceptance scenarios:** see ### US-2.2 Under Gherkin AC

### US-N.2.3:  Record Proffession
**As a** A church Administator  
**I want to** Record a members's proffession  
**So that**  The church can maintain information about member's occupations and skills

**Priority:** P1  
**Independent test:** 
An authorized user can add and view a member's proffession

**Acceptance scenarios:** see ### US-2.3 Under Gherkin AC

---

## Requirements

### Functional Requirements

- **FR-001**:The system Must allow demographic information to be associated with registered member.
- **FR-001**: **Acceptance scenarios:** see ### US-2.2 Under Gherkin AC
- **FR-001**: : The system MUST allow demographic information to be associated with a registered member.

- **FR-002**:: The system MUST allow demographic information to be associated with a registered member.

- **FR-003**:•	The system MUST allow the church to record a member's profession. 
**FR-004**:The system MUST allow the church to record the school attended by a child.
**FR-005**:The system MUST allow the church to record the school attended by a child.
**FR-006**:: The system MUST allow information that is no longer applicable to be updated rather than requiring creation of a duplicate member record.


---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: <measurable outcome for this feature>

---

## Data Model Requirements

### `member_demographics` table
--------------------------------------------------------------------------
| Field             | Type              | Rules                          |
|-------------------|------             |---------------------           |
| member_id         | INTEGER PK        | required                       |
| ..................| …..........       |.…..................            |
|residence          | VARCHAR	        |   Optional                     |
|...................|...................|................................|
|profession         |VARCHAR	        |      Optional                  |
|...................|...................|................................|
|school             |VARCHAr            |  Optional                      |
|...................|...................|................................|
|updated_at	DATETIME|Updated when information changes|...................|
--------------------------------------------------------------------------

---

## Acceptance Criteria

### US-N.1 — Record Member Demographics
#### Scenario 2.1: Add demographic information
*   **Given** a registered member exists
*   **When**  an authorized user enters valid demographic information.
*   **Then** the system saves the information
*   **And**  And the information is associated with the correct member

#### Scenario.2.2: Record Children's School Information
*   **Given** a registered member is identified as a child …
*   **When**  an authorized user enters the child's school …
*   **Then** the system rejects the registration …
*   **And** the school information is displayed in the child's record


### US-N.2.3 —Record Profession

#### Scenario:Record a member's profession
*   **Given** … a registered member exists
*   **When** … an authorized user enters the member's profession
*   **Then** …the system stores the profession
*   **And** … the profession is associated with that member
 
