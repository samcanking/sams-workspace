---
notion-id: 262dd034-25a1-80ee-becb-d536c2283567
---
### Requester Experience

- New Workflow to terminate an Employee. 
- Task Definition/Setup
    1. Settings
        1. Task Name: Employee Termination Request
        2. Category: Employee changes
        3. Request initiation: Manager & Admin-level users
            1. Does NOT require rates access
    2. Content
        4. Read-only preview of request form (see below) 
    3. Approvers
        5. Same approver options as other employee request tasks
        6. Default “Employee Acknowledgement” is FALSE
    4. Notifications
        7. Same available/default notifications
    5. Review
        8. Same UX as other employee request tasks
- Task Request
    - Add “Employee Termination Request” to Employee Profile > + Change Request landing page (visible when task definition = ACTIVE)
    - Employee Termination Form
        - (REQUIRED) Effective date = “When is the employee’s last day” 
            - Must support past and future dates
        - (REQUIRED) “What kind of termination is this?”
            - VOLUNTARY (Default)
            - INVOLUNTARY
            - Responding one way or the other does not change the remaining form or change any validation logic
        - (REQUIRED) “What is the termination reason” is a user-defined list (likely the [dbo].[TerminationReasonCodeMapping] HRPremier table, but configured on the front-end via Maintenance > Human Resources > Termination Reasons) 
        - Notes
        - Attachments
            - Would be great to include especially in this workflow if possible
            - Expectation is that upon task instance approval the document is routed to document center (will provide appropriate tags/mapping details if doable for hackathon)
            - Document will also always be download-able directly from the task instance
            - Not required, but would be amazing to include — will be applicable for all request task types
        - (REQUIRED) Do you want to include this employee in payroll?
            - If the effective date is **in a past pay period**, the “Do you want to include this employee in payroll” will default to “No” and become disabled
            - If the effective date is in** the current or future pay period**, the include in payroll question is still active
            - Default value = No
        - (REQUIRED) Do you want to keep deductions active for Payroll? 
            - Question only visible if the employee has any <u>active</u> deductions
            - Returns list of all ACTIVE deductions, the employee and employer deductions
            - IF YES: Automatically make the following updates:
                - Status = One-time
                - End date = First pay check date AFTER termination date
            - IF NO: Automatically make the following updates: 
                - Status = Inactive
                - End date = Termination date
                - Default selection
        - (REQUIRED) Is this employee eligible for COBRA?
            - Yes (Default)
            - No
        - (REQUIRED) (If Employee is eligible for COBRA) COBRA event date
            - Default value = Termination Date
            - Users may choose any date (past or future)
        - Manager: If the employee is a manager, identify how many employees the manager is over and indicate that they should be reassigned as part of a separate process 