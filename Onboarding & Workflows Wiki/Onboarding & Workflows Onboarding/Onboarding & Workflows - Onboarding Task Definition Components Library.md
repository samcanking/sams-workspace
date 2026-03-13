---
notion-id: 24ddd034-25a1-8002-be00-c23e2b586b66
---
To ensure users are able to effectively configure and customize Task Definitions such that Task Assignments are initiated only when and to whom the Task Assignment is relevant for, there are several Task Definition Components which may be setup by users. While most of these may be customized and may evolve in capability, certain defaults and limitations do exist for each Task Definition Component. 

This wiki provides insight into the various Task Definition Components, guidance on various functional requirements and insights on planned expansion where relevant. 

💡 **Note: **The guidance in this wiki is for custom or otherwise generalized Task Definitions, not pre-defined Tasks like Form I-9, Direct Deposit, etc. 

## 1. Settings

### Task name

| **Input type:** | User-defined text |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Blank |

Task setting which allows a user to label and identify what the Task is about in a few words. 

**⚠️ Note: **Task Definitions may not have the same name

### Category

| **Input type:** | System-defined drop-down selection |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Not currently |
| **Default value: ** | Onboarding or Employee Event |

Task Component which allows users to filter based on what the topic of the task is. Not configurable for now but will be in the future as additional capabilities and categories are added. 

### Task Assignment - Event

| **Input type:** | User-defined selection |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Hire date |

Task setting which drives WHEN a Task Assignment is assigned to a user. 

Note the additional “Assign:” selection which includes options for “On Event” (default) ,“Before,” and “After” allowing users to further refine when (in relation to the selected Event) a Task Assignment should be assigned to the defined Assignee.

### Task Assignment - Conditions

| **Input type:** | User-defined selections |
| --- | --- |
| **Required?** | No |
| **Configurable?** | Yes |
| **Default value: ** | Include rehires = TRUE<br>Include transfers = FALSE<br>All others = FALSE/NULL |

Task setting which allows users to refine the event such that it only applies to employees which satisfy the defined conditions. 

### Task Completion - Assign to

| **Input type:** | User-defined selection |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Employee |

Task setting which allows users to define WHO the Task Assignment should initially be assigned to. In the case of Onboarding, this will most frequently be the Employee, but users may choose to configure this to apply to other people related to the employee. Similar to Approver Groups, the options for Assignee Types include: 

- Employee
- Relationship-Type
- Position (TBA)
- Specific Individual (Non-Admin)
- Specific Individual (Admin)

Note the related “How long does the assignee have to complete the task?” field which drives the Task Assignee due date logic based on when the Task was first assigned. 

*🚀 Future capability will allow for more complicated due-date logic, such as specific calendar dates, relation to milestones, etc.)*

### Additional Settings - View Details

| **Input type:** | User-defined selections |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Employee = TRUE<br>Manager-level employees = TRUE<br>Company admins = TRUE<br>Organization admins = TRUE and INACTIVE |

Task setting which determines who else can view the details of a Task Assignment for the given Task Definition via the Task Dashboard. This only drives READ ONLY access to a Task Details, but it does include all details of the Task Assignment. 

User types for which this value is TRUE will see Task Assignments for their provisioned access (employees they manage, employees in their company, etc.) via the Task Dashboard. 

User types for which this value is FALSE will not see any Task Assignments in their dashboards for the given Task Definition. 

Note that the “Organization admins” option will always be selected as Organization Administrators are provisioned such that they are “super users” and must be able to manage and access all tasks for their Organization. 

The “Employee" option should become inactive when the “Assign To” selection is also Employee. 

⚠️ **Note: ** ASSIGNMENT to a Task overrides any visibility logic derived here. I.e., even if “Manager-level employees” is FALSE, if a manager-level employee is assigned to a Task as an approver, they will still be able to view the Task. 

*🚀 Future capability will allow users to further refine the nuance here, giving users the ability to allow users to see the Task Assignment on their “All Tasks” Dashboard, but be unable to open the Task Details.*

### Additional Settings - Rates access

| **Input type:** | User-defined selection |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | No |

Task setting which works together with the **Additional Settings - View Details** setting to give users further restriction capabilities to support use cases where a Task’s details may be particularly sensitive. 

When the value of this field is TRUE (yes), the Task Assignment will only be viewable on the Tasks & Workflows dashboards of users who have Rates access. 

When the value of this field is FALSE (no), the Task Assignment will be viewable on the Tasks & Workflows dashboards of all employees provisioned to view details in the previous step. 

**⚠️ Note**: This setting does NOT apply to any assigned Task Roles (Assignees, Approvers), which will always have visibility to the full Task Details for the Task Assignments they are assigned to. 

## 2. Task Content

### Task name

| **Input type:** | User-defined text |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Blank |

Task setting which allows a user to label and identify what the Task is about in a few words. 

### Instructions

| **Input type:** | User-defined text |
| --- | --- |
| **Required?** | No |
| **Configurable?** | Yes |
| **Default value: ** | Blank |

Task setting which allows a user to input additional instructions on how a Task Assignment should be completed. 

### 🚀 Task Type

| **Input type:** | User-defined selection |
| --- | --- |
| **Required?** | Yes |
| **Configurable?** | Yes |
| **Default value: ** | Custom-built Task |

Task setting which allows a user to decide what kind of Task content the Task Assignment will include. 

🚀 *The Custom-built Task type is not part of the MVP — as such this selection will not be visible until this capability is added as we will default users to the “Use a Form” experience. *

### 🚀 *Task Type - Custom-built Task Task Actions*

| **Input type:** | User-defined builder |
| --- | --- |
| **Required?** | Yes (one field) |
| **Configurable?** | Yes |
| **Default value: ** | Blank |

Custom task builder which allows users to build custom tasks by combining and customizing available components, which include: 

- Download Attachment
- Upload Attachment
- Check to Complete
- External Link
- E-Signature

🚀 *The Custom-built Task type is not part of the MVP— as such this functionality will not exist as part of initial release phasing. *

### Task Type - Form Selection & Configuration

| **Input type:** | User-defined setup |
| --- | --- |
| **Required?** | Yes (link a form) |
| **Configurable?** | Yes |
| **Default value: ** | Empty |

The Form Selection and Configuration component allows users to select, view and edit information about the Form (setup outside of the Task Definition setup) they would like to associate with and assign via the Task. 

Generally this component will allow users to: 

1. Choose the Form Definition the Assignee(s) will complete
2. View a preview of the Form Definition
3. Assign any additional Assignees which may be necessitated by additional Recipients (as configured via Forms setup), including additional due date logic
4. View all fields assigned to each individual Recipient

🚀 *In future versions, users will be able to create new Forms directly from within the Task Setup experience. *

## 3. Approvers

### Add Approvers

| **Input type:** | User-defined selections |
| --- | --- |
| **Required?** | No |
| **Configurable?** | Yes |
| **Default value: ** | Blank (No) |

Unlike Request Type Tasks which require at least one Approver (the Initial Reviewer), Onboarding Tasks do not require Approvers, though users may optionally choose to include them. 

If the administrator chooses to add Approvers to a Task, they may do so by selecting “Yes” and then choosing the Approver Group(s) (configured outside of the Task Definition setup) they would like to include on the Task Definition. 

Task Approvers ALWAYS follow any and all Task Assignees, proceeding in the order identified on the Approvers Task Setup view. 

🚀 *In future versions, users will be able to create new Approver Groups directly from within the Task Setup experience. *

## 4. Notifications

### Configure Notifications

| **Input type:** | User-defined selections |
| --- | --- |
| **Required?** | No |
| **Configurable?** | Yes |
| **Default value: ** | See mockups |

Like the Approvers setup, the Notifications setup also functions similarly to the established protocols in the initial Tasks & Workflows releases. 

A series of default settings are assigned to each individual role on the Task.

🚀* Future enhancements to this component include the ability to add additional notification recipients to users outside of the assigned Task Roles, as well as support for additional Alert delivery methods, including SMS and push notifications*

## 5. Review

### Review & Complete

| **Input type:** | System-defined information |
| --- | --- |
| **Required?** | N/A |
| **Configurable?** | No |
| **Default value: ** | Responsive to setup configured in other Task Setup steps |

A visual representation of the task setup configured as part of the other Task Setup steps. No action can be taken by the users from this screen beyond saving their Task Setup. 