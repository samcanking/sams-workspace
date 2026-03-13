# Tech Spec: One-Time Payment Request (v1.0)

**Status:** Draft **Date:** 2026-03-13 **Author:** Claude **Parent PRD:** PRD - One-Time Payment Request (Bonus Payment)

---

## System Context

The One-Time Payment Request (OTP) is a new request type within Netchex Onboarding & Workflows. It reuses the existing approval chain engine, notification framework, feature flag model, and audit logging infrastructure from Workflows Phase 1. The primary net-new surface is the payroll integration trigger on approval and the hierarchy-gated initiation control.

---

## Architecture Components

| Component | Approach |
|---|---|
| Request storage | New request type entity extending existing ECR data model |
| State machine | New state machine (Draft → Submitted → In Review → Approved/Rejected/Canceled); enforced server-side |
| Approval engine | Reuse existing configurable multi-step approval chain |
| Payroll integration | New async job triggered on `Approved` transition; pushes Amount + Payment Date to payroll pipeline |
| Audit log | Extend existing immutable audit log schema with OTP-specific event types |
| Attachment storage | Immediate upload to Document Center on file add (consistent with ECR attachment model) |
| Notifications | Reuse existing Workflows notification framework |
| Enablement | Per-org feature flag, disabled by default (consistent with Phase 1 model) |

---

## Critical Unknowns

**1. Payroll integration error ownership** _(Open — Payroll/Engineering)_
On payroll submission failure post-approval, the system is in a terminal `Approved` state with no defined recovery path. The PRD does not specify: who receives the error, what retry behavior (if any) is expected, whether the request can be resubmitted to payroll without creating a new request, and what prevents duplicate payment if a retry succeeds after a transient failure.

**2. Inactive approver handling** _(Open — Product)_
If a configured approver is inactive or has departed the org when a request reaches their step, the request stalls with no defined escalation path. The reassignment mechanism (FR-09) requires an HR Admin to intervene — but there is no SLA, auto-escalation rule, or timeout behavior defined.

**3. Payment Date validation depth** _(Open — Payroll/Engineering)_
FR-05 validates that Payment Date is not in the past at submission time, but does not specify whether the date must align with a valid payroll period for the target employee's pay group. If advisory-only, the payroll pipeline may reject the submitted date on approval — which loops back to the unresolved error handling path above.

**4. Hierarchy re-validation on edit**
FR-02 validates the target employee's reporting relationship "at the time of submission." FR-10 allows field edits after submission (which restart the approval chain). The PRD does not specify whether hierarchy is re-validated when an edit triggers a chain restart — creating a window where a previously valid request remains active after a reporting change.

**5. Notification SLA**
FR-16 references "within the defined SLA" but no SLA value is defined in this PRD or in the Workflows Phase 1 documentation. This is a dependency that must be resolved before acceptance criteria for FR-16 can be verified.

---

## Conflicts with Existing Documented Capabilities

**Conflict 1: Attachment constraints (high severity)**
The OTP PRD (FR-07) specifies: max **5 files**, max **25 MB combined total**, **no file type restrictions**.

The existing ECR Attachment Feature documentation specifies: up to **1,000 attachments per request**, **25 MB per file** (not combined), file types restricted to the **Document Center allowlist**.

These are materially inconsistent on all three dimensions (count, size semantics, type restrictions). If the OTP attachment component is built as a variant of the shared ECR attachment component, one of the two specifications must be treated as authoritative. If OTP uses a custom implementation, the shared component cannot be advertised as reusable across request types without a configuration surface.

**Conflict 2: Approver edit permissions**
The ECR Attachment model defines an approver permission tier — approvers with "edit" access can upload/delete attachments until final approval. The broader Workflows system also defines an "Initial Reviewer" role (admin-only) with the ability to edit request detail fields.

The OTP PRD assigns edit capability exclusively to the **initiating user** and does not define an Initial Reviewer role or approver-with-edit-rights for this request type. It is unresolved whether: (a) OTP intentionally omits the Initial Reviewer role, (b) approvers with "edit" access can modify OTP attachments per the ECR model, and (c) admin edits to an OTP request (e.g., via reassignment) would also trigger the chain-restart behavior defined in FR-10.

**Conflict 3: Past effective dates**
Both the Position/Compensation Change Request and the Employee Termination Request explicitly support past effective dates with defined payroll handling (retroactive pay calculation, payroll inclusion toggle). The OTP PRD (FR-05) explicitly disallows past Payment Dates. This is likely an intentional product decision, but it diverges from the pattern established by every other request type in the system and should be confirmed — particularly given the "time-sensitive" framing in the problem statement (Section 1), where a manager may need to back-date a bonus payment to a prior pay period.

**Conflict 4: Approver reassignment eligibility**
FR-09 allows reassignment to "another eligible user" but does not define eligibility criteria for reassignment targets. The initiator eligibility rules (FR-01/FR-02) require a specific position classification and hierarchy relationship. It is unresolved whether those same constraints apply to reassigned approvers or whether any admin-level user qualifies — which is the convention elsewhere in the approval chain model.

---

## Recommended Pre-Build Resolutions

1. Align OTP attachment spec with the shared ECR attachment component or formally fork it with documented divergence.
2. Define payroll integration error handling: owner, retry policy, idempotency key strategy.
3. Confirm whether past Payment Dates are intentionally excluded (vs. PCCR/Termination precedent).
4. Define approver reassignment eligibility criteria explicitly.
5. Confirm hierarchy re-validation behavior on edit-triggered chain restart.
6. Assign a concrete notification delivery SLA.
