# Feature: <Service check-in and security>
**Feature ID:** N04
**Branch pattern:** `feature/3-service-check_in`
**Status:** Draft
**Created:** 2026-10-04
**Input:**  Enables rapid Sunday service check-in for all church members (Aults and children) via phone number/ name search, track overall attendance.
**Depends on** [feature](\feature2_Adult_Member_Registration.md),[feature](\feature3_children_Registration.md)
**Related:** [feature](\feature5_GeneralManagement)
---
## US-A.4.1: General Church member Search Check-In
**As a**  Usher or Greater
**I want to** search church members by hone number, last name or first name.
**So that** I can check-in individuals adult members, visiting guests, or entire families in a single quick workflow as they arrive for the service.
**Priority:** P1
**Independent test:** Enter a registered family or adult member phone number at check-in terminal, select attending members(adults and/ or children), and confirm check-in complete.
**Acceptance scenarios:** see ### US-N.1 under Acceptance
Criteria

### US-N.4.2: Child Pickup Claim Code Generation
**As a**      Children's Ministry Leader or Volunteer
**I want to** automatically generate a unique matching 4- chracter security claim code when minor children are checked into children's classes.
**So that**    Children are sefely released after service only to the guardians prsesnting matching security tag.
**Priority:** P1
**Independent test:**  Complete check-in for a minors, verfy a 4- chracter code(like A7B28) is assigned to the minor child tags and adult claim tag, and confirm checkout secceeds only when codes match.
**Acceptance scenarios:** see ### US-N.2 under Acceptance
Criteria

### US-N.4.3: General Attendance Logging and Safety Alerts
**As a**       Administrator or Usher
**I want to**  log general service attendance while seeing dynamic allergy/safely warning for minors during check-in
**So that**     We maintain accurate headcounts for all attendee and ensure child safety protocols are triggered immediately.
**Priority:**  P1
**Independent test:**  Check-in an adults and child with allergies checked TRUE, verfying that adult atendence is recorded and the child's alleligy alert badge displays prominently.
**Acceptance scenarios:** see ### US-N.2 under Acceptance


---
## Requirements
### Functional Requirements
- **FR-001**: System MUST allow searching for all church members (adults and children) by full/partial phone number or name.
- **FR-002**: System MUST allow checking in both adult church members (for general attendance) and minor children (for classroom security/care).
- **FR-003**:System MUST generate a unique, upper-case 4-character alphanumeric security claim code whenever one or more minor children are checked in.
- **FR-004**:System MUST record attendance timestamps and attendee types (Adult, Child, Guest) per service event.
-**FR-005**:System MUST require matching security claim verification before marking a minor child as Checked Out.
 
## Acceptance Criteria

### US-N.1 — General Church and Family Check-In
#### Scenario: General family check-in (Adults & Children)
* **Given**  a registered adult member ("Amahoro Justin", phone: "555-0199") with a spouse ("Elena") and a child ("kelly")
* **When** an usher searches "555-0199" on the Check-In screen
* **then** the system displays the Rivera family card listing Amahoro (Adult), Elena (Adult), and Kelly (Child)
* **when** the usher selects all three members and taps "Check In Family" / "Registrar Familia"
* **Then** individual check-in records are created for Amahoro and Elena (General Attendance) and Kelly (Children's Ministry with Security Code).

#### Scenario: Individual adult check-in
* **Given** Give an adult member attending alone
* **Then**  The usher searches their phone number and cnfirms check-in
* **Then**  The syste mlogs the adult's  attendence without generating a child claim code.

### US-S.2 — Security Claim Verification for Minors
## Scenario: Child pickup verification with matching code
* **Given a** child ("Sofia Rivera") is checked in under claim code X8R2

* **When** the guardian presents code X8R2 at pickup and the volunteer enters it into the checkout screen

* **Then** the system updates Sofia's status to CHECKED_OUT with timestamp

* **And** displays "Pickup Authorized" / "Entrega Autorizada".

#### Scenario: Mismatched claim code on child pickup
* **Given** Achild is checked in under code E56BD 
* **Then**  Someone attempts checkout entering a code E56BC
* **Then**  The system blocks checkout and displays An error message in Spanish Or English according to the member choosen language

