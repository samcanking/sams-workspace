---
notion-id: 277dd034-25a1-809c-9cd1-ca6487a4bd97
---
# Product Requirements Document

## Position/Compensation Change Request (PCCR) - Retro Pay Support

### Overview

This document outlines the requirements for adding Retro Pay support to the Position/Compensation Change Request feature in Tasks & Workflows. This enhancement will allow users to process compensation changes with effective dates in the past, ensuring proper calculation and payment of retroactive pay differences.

### Business Requirements

The primary goal is to enable retroactive compensation changes through the Tasks & Workflows system while maintaining data integrity and providing clear user guidance throughout the process.

### Eligibility Requirements

- Client must have Time & Attendance enabled
- Hourly employees must have time recorded in time cards for the retroactive period
    - System requires actual punch data to calculate retroactive differences
    - Cannot calculate mid-pay period adjustments without daily hour distribution
- Salary employees are supported without time card requirement

### Feature Requirements

### 1. Date Validation & Restrictions

- Disable past dates that are earlier than the employee's "Last Pay Change Date"
    - This date is stored on the Employee Profile > Compensation page
    - Validation occurs on save, not during date selection
- System will check for time card data availability for hourly employees
    - Return error message if no time exists in applicable date range

### 2. Employee Profile Alerts

- Introduce "Pending change(s)" alert on Employee Profile when an employee has an OPEN or PENDING Request
    - Clicking banner brings user to filtered "Open Tasks" dashboard for specific employee

### 3. Pending Compensation Changes Display

- Update Compensation "Pending Compensation Changes" drawer to show:
    - APPROVED (Pending) tasks
    - IN APPROVAL (Open) tasks
    - Update view to link directly to specific Task

### 4. Compensation View Restrictions

- DISABLE the "Save" button on Compensation view when there is an open PCCR with a PAST effective date
    - Add hover explanation to inform user they cannot make compensation changes while there is an open PCCR with a past effective date

### 5. Task Creation Restrictions

- Disable new Position/Compensation Change Request ability when the user has an OPEN PCCR with a PAST effective date
    - Prevents conflicts with existing retroactive requests
    - Preserves last pay change date integrity

### 6. Retro Pay Default Settings

- Add "Retro Pay - Default Settings" component to PCCR setup
    - Users may configure the default behavior for past effective dated tasks to either:
        - ALWAYS use retro pay
        - NEVER use retro pay

### 7. Retro Pay Handling Method

- Add "Retro Pay" handling experience to Task Assignment experience, visible only when:
    - The task type = PCCR
    - The effective date is <TODAY
    - The task includes a rate change
- This toggle will be:
    - VISIBLE to **all** approvers/viewers
    - Only editable by Org Admins
- Toggle remains visible in closed tasks as a record of how the past effective date was handled

### 8. User Notification & Alerts

- When a user selects a past effective date (in initial request or edit experience):
    - Display prominent alert with information about what is/is not included in effective dating processing
- If a date moves to the past during the approval process:
    - Display less prominent notation informing how the task will be handled
- These alerts appear regardless of whether a rate is being changed

### 9. Field Retroactive Processing

- ONLY rate fields will respect past effective dates in the audit history:
    - Hourly rate
    - Annual salary
    - Additional rates
- All other fields (i.e., Position) will process effective as of the date the last approver signs off

### Processing Logic

### Retro Pay Calculation Process

- System will calculate difference between old/new rates for time already paid
- Only recalculates through end of last completed pay period
    - Current pay period gets new rate automatically without retro calculation
- For requests that span pay periods:
    - Calculate retro through most recent completed pay period at approval time
    - System dynamically determines pay period end dates based on approval date
- Adds calculated retro amount to "Other Scheduled Earnings"
- Pays out on employee's next scheduled payroll
- Notification persists until amount is paid

### Technical Requirements

### API & Integration

- API needs retro pay effective date from Tasks & Workflows
- Payroll will handle internal calculations:
    - Fetch last pay change date
    - Determine applicable pay period range
    - Make workflow API call for approval date if needed
- Validation check on initial request creation for time card data

### Success Metrics

- Successful processing of retroactive pay changes through the PCCR workflow
- Accurate calculation and payment of retroactive amounts
- Clear user understanding of the retroactive process through UI guidance
- Reduction in compensation processing errors due to conflicting requests

### Open Questions

- Determine audit logging requirements for "declined retro pay" decisions
- Define notification system for failed retroactive requests