**Version:** 1.0 **Status:** Draft **Author:** Sam **Date:** 2026-03-13 **Type:** Enhancement **Domain:** Employee / Workflows **Parent System:** Netchex Onboarding & Workflows

---

## 1. Overview & Problem Statement

HR Administrators and managers at Netchex client organizations currently lack a structured, system-enforced mechanism for initiating ad hoc or bonus payments for employees. These payments — by nature one-time, often time-sensitive, and compensation-sensitive — are frequently coordinated through informal channels (email, verbal authorization), creating approval gaps, audit risk, and payroll processing friction.

This enhancement introduces a One-Time Payment Request as a new request type within the existing Onboarding & Workflows framework. It is scoped to eligible initiators (managers and above within the employee's hierarchy), routes through configurable approval chains, and integrates with payroll on approval. The request type is per-organization and inherits the feature flag, enablement model, and notification infrastructure already established in Workflows Phase 1.

_(Source: User and customer feedback identifying one-time bonus payments as a commonly occurring, currently unstructured workflow. Compensation sensitivity and audit requirements necessitate a defined approval path.)_

---

## 2. Goals & Success Metrics

### Goals

- Provide a formalized, auditable workflow for ad hoc/bonus payment requests within Netchex.
- Enforce position-based and hierarchy-based initiation controls to protect compensation data.
- Reduce manual coordination overhead by integrating approval routing and payroll submission.
- Maintain consistency with the existing Workflows configuration model to minimize implementation and training lift.

### Success Metrics

|Metric|Target|Measurement Method|Source|
|---|---|---|---|
|Adoption rate|≥60% of enabled orgs submit ≥1 request within 60 days of GA|Request creation events by org, 60-day cohort|Assumed — unverified|
|Approval chain completion rate|≥80% of submitted requests reach terminal state within 5 business days|Median time-to-terminal by org|Assumed — unverified|
|Payroll integration error rate|<2% of Approved requests result in a payroll integration failure|Integration error events / total approved requests|Assumed — unverified|
|Audit log completeness|100% of state transitions and edits captured in audit log|Log coverage audit post-launch|Required by design|

> **Note:** All metric targets are assumed pending baseline data from existing Workflows usage. Targets should be validated with Analytics prior to GA.

---

## 3. Request Lifecycle & State Machine

The One-Time Payment Request follows a defined lifecycle. All state transitions are enforced by the system; no out-of-band transitions are permitted.

|State|Description|Allowed Transitions|
|---|---|---|
|Draft|Request created but not yet submitted by initiator.|→ Submitted, → Canceled|
|Submitted|Request submitted; pending routing to first approval step.|→ In Review, → Canceled|
|In Review|Request is actively traversing the configured approval chain.|→ Approved (all steps cleared), → Rejected (any step), → Canceled (by initiator)|
|Approved|All approval steps completed. Payroll integration triggered automatically.|Terminal — no further transitions|
|Rejected|Rejected at any approval step. Request is terminal; a new request must be created.|Terminal — no further transitions|
|Canceled|Canceled by the initiating user before final approval.|Terminal — no further transitions|

**Edit behavior:** A request may be edited by the initiator at any point prior to the final approval step. Any edit after Submission restarts the approval chain from Step 1. Editing is disabled once a request reaches Approved, Rejected, or Canceled state.

---

## 4. Functional Requirements

> **Notation:** **SHALL** = mandatory. **SHOULD** = preferred. All requirements are numbered for traceability.

### 4.1 Initiator Eligibility

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-01|The system SHALL restrict initiation of a One-Time Payment Request to users whose Position is classified as Manager, Department Director, VP, or Executive of the target employee as recorded in the Netchex org chart.|Must Have|User feedback|
|FR-02|The system SHALL validate that the target employee exists within the initiating user's direct-report chain or broader reporting hierarchy at the time of submission. Requests targeting employees outside the initiator's hierarchy SHALL be rejected with a clear error message.|Must Have|User feedback|
|FR-03|The system SHALL prevent any user who initiated a request from serving as an approver at any step of the same request's approval chain.|Must Have|Standard compensation governance; assumed — unverified|

### 4.2 Request Form

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-04|The system SHALL require an Amount field (currency) on the request form. The field SHALL accept positive numeric values only. The system SHALL enforce the configured currency format of the organization.|Must Have|User feedback|
|FR-05|The system SHALL require a Payment Date field on the request form. The date SHALL not be in the past at time of submission and SHALL be validated against the target payroll period.|Must Have|User feedback|
|FR-06|The system SHALL provide a Notes field (free text, optional) with a maximum of 2,400 characters. The system SHALL display a live character counter and reject submission if the limit is exceeded.|Must Have|User feedback|
|FR-07|The system SHALL allow attachment of up to 5 files per request with no restriction on file type. The combined total size of all attachments SHALL NOT exceed 25 MB. The system SHALL validate both constraints and surface a specific error for each violation.|Must Have|User feedback|

### 4.3 Approval Chain

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-08|The system SHALL support a configurable, multi-step approval chain for One-Time Payment Requests, using the same approval configuration model already established in Onboarding & Workflows. Each step SHALL independently define its approver(s).|Must Have|Existing Workflows framework|
|FR-09|The system SHALL allow any assigned approver to be reassigned to another eligible user at any point before that step is acted upon, by an authorized administrator. Reassignment SHALL be logged in the request audit trail.|Must Have|PM clarification|
|FR-10|The system SHALL allow the initiating user to edit request fields (Amount, Payment Date, Notes, Attachments) at any time prior to the final approval step being completed. Edits after submission SHALL restart the approval chain from the beginning.|Must Have|PM clarification|

### 4.4 Request Lifecycle

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-11|The system SHALL enforce the following request lifecycle states: Draft, Submitted, In Review, Approved, Rejected, and Canceled. Valid transitions are defined in Section 3.|Must Have|PM clarification|
|FR-12|The system SHALL allow the initiating user to cancel a request that is in Draft, Submitted, or In Review state. Cancellation SHALL NOT be permitted once the request reaches Approved or Rejected state.|Must Have|PM clarification|
|FR-13|The system SHALL treat a Rejected request as terminal. No resubmission or revision of a rejected request SHALL be permitted. A new request must be created.|Must Have|PM clarification|

### 4.5 Payroll Integration

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-14|Upon reaching Approved state, the system SHALL automatically submit the One-Time Payment Request to the payroll processing pipeline with the specified Amount and Payment Date. The system SHALL surface an integration confirmation or error status to the HR Administrator.|Must Have|PM clarification|

### 4.6 Audit Log & Notifications

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-15|The system SHALL maintain a full, immutable audit log for each request, capturing: all field edits (before/after values), all state transitions (actor, timestamp in UTC), approver reassignments, and cancellation events.|Must Have|PM clarification|
|FR-16|The system SHALL send notifications to relevant stakeholders (initiator, approver(s), HR Administrator) on state transitions — submission, approval at each step, final approval, rejection, and cancellation — using the existing Workflows notification framework.|Must Have|Assumed — consistent with existing Workflows behavior|

### 4.7 Org Enablement & Multi-Company

|ID|Requirement|Priority|Source|
|---|---|---|---|
|FR-17|One-Time Payment Request SHALL be enabled per organization via the existing feature flag and org-level configuration model used by Workflows Phase 1. The feature SHALL be disabled by default and require explicit activation per org.|Must Have|Assumed — consistent with Workflows Phase 1 enablement model|
|FR-18|In multi-company organizations, One-Time Payment Request configuration (approval chain, enablement) SHALL be managed independently per company, consistent with existing multi-org Workflows behavior.|Must Have|Assumed — consistent with existing Workflows multi-org model|

---

## 5. Acceptance Criteria

> Written in Given / When / Then format. Keyed to Functional Requirement IDs for QA traceability.

|ID|Acceptance Criteria|
|---|---|
|FR-01|Given a user whose Position is 'Manager' in the org chart, when they initiate a One-Time Payment Request for a direct report, then the request form is presented and submission is permitted.|
|FR-01|Given a user whose Position is 'Coordinator' (not in the eligible set), when they attempt to initiate a One-Time Payment Request, then the system denies access and displays an appropriate permission error.|
|FR-02|Given an initiating Manager, when they select a target employee who is not in their direct or extended reporting hierarchy, then submission is blocked and the system displays a hierarchy validation error.|
|FR-03|Given a request where the initiator is also configured as an approver at any step, when the approval chain is evaluated, then the system removes or flags that approver assignment and requires a different approver to be designated.|
|FR-04|Given a valid request form, when the Amount field is left blank or contains a non-positive value, then submission is blocked and a field-level validation error is displayed.|
|FR-05|Given a valid request form, when the Payment Date field contains a past date, then submission is blocked and an error is displayed indicating the date must be current or future.|
|FR-06|Given a user typing in the Notes field, when the character count reaches 2,400, then no additional input is accepted and the counter displays '2400 / 2400'. When the field contains 2,401 or more characters at submission, then submission is blocked.|
|FR-07|Given an attachment upload, when the user attempts to attach a 6th file, then the upload is rejected and an error states the 5-file maximum has been reached. When the total size of attached files exceeds 25 MB, then submission is blocked and a size error is displayed.|
|FR-08|Given an org with a 2-step approval chain configured, when a request is submitted, then it enters In Review state and routes to Step 1 approver. Upon Step 1 approval, it routes to Step 2 approver. Upon Step 2 approval, it transitions to Approved.|
|FR-09|Given an In Review request pending Step 1 approval, when an HR Administrator reassigns the approver to another eligible user, then the new approver receives a notification and the reassignment is recorded in the audit log with actor and timestamp.|
|FR-10|Given a Submitted or In Review request, when the initiator edits the Amount field and saves, then the request returns to Submitted state, the approval chain restarts from Step 1, and the edit is recorded in the audit log with before/after values.|
|FR-10|Given a request that has received its final approval, when the initiating user attempts to edit any field, then the edit controls are disabled and no edits are accepted.|
|FR-11|Given any request, when a state transition is triggered, then the system only permits the transitions defined in the state machine (Draft → Submitted; Submitted → In Review; In Review → Approved \| Rejected \| Canceled; Submitted → Canceled; Draft → Canceled). All other transitions are rejected.|
|FR-12|Given a request in In Review state, when the initiating user cancels the request, then it transitions to Canceled, all pending approver notifications are suppressed, and the cancellation is recorded in the audit log.|
|FR-13|Given a Rejected request, when the initiating user attempts to resubmit or edit it, then the system does not allow any action and displays a message indicating a new request must be created.|
|FR-14|Given an Approved request, when the system submits it to payroll, then the payroll pipeline receives the Amount and Payment Date, and a confirmation status (success or error) is surfaced to the HR Administrator within the request record.|
|FR-15|Given any edit, state transition, or reassignment event on a request, when the event occurs, then a record is added to the audit log containing: event type, actor (user ID + display name), timestamp (UTC), and relevant field values (before/after where applicable). The log is read-only and cannot be modified.|
|FR-16|Given a request transitioning to Submitted state, when the transition completes, then the assigned Step 1 approver receives a notification via the existing Workflows notification channel within the defined SLA.|
|FR-17|Given an organization where the One-Time Payment Request feature flag is disabled, when any user accesses the Workflows creation flow, then the One-Time Payment Request type is not visible or selectable.|
|FR-18|Given a multi-company org, when Company A enables the feature and configures a 3-step approval chain, and Company B has the feature disabled, then requests initiated under Company A follow its configured chain, and no One-Time Payment Request option is available under Company B.|

---

## 6. Out of Scope / Exclusions

The following are explicitly excluded from this phase:

- **Amount-based approval routing** (e.g., escalation to a secondary approver for requests above a configured threshold). Deferred to Phase 2.
- **Self-service submission by the recipient employee.** This is an HR/manager-initiated flow only.
- **Recurring or scheduled payment requests.** This feature covers one-time, ad hoc payments only.
- **Direct integration with third-party payroll or ERP systems** beyond the existing Netchex payroll pipeline.
- **In-app editing of rejected requests.** Rejected requests are terminal; initiators must create a new request.
- **Role-based attachment type restrictions.** All file types are permitted within the size and count constraints.
- **Bulk initiation** (submitting a One-Time Payment Request for multiple employees simultaneously).

> **Note on Phase 2:** Dollar-threshold-based approval routing is a confirmed Phase 2 item. PRD and requirements will be defined separately.

---

## 7. Open Questions

- **Payroll integration error handling:** Who owns the error state? Should the initiating user or HR Administrator be notified, and what is the expected remediation path? _[TBD — Payroll/Engineering to confirm]_
- **Inactive approver:** What is the expected behavior if the configured approver is inactive or has left the org at the time the request reaches their step? _[TBD — Product to confirm]_
- **Payment Date validation:** Should the Payment Date field be validated against the actual payroll schedule of the target employee's pay group, or is it a free date picker with advisory validation? _[TBD — Payroll/Engineering to confirm]_
- **Compliance & data residency:** Are there SOC 2 or data residency considerations specific to compensation data stored in request Notes or Attachments? _[TBD — Legal/Compliance to confirm]_