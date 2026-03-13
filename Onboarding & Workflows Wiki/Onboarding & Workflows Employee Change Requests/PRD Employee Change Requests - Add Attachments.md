---
notion-id: 2bedd034-25a1-8061-b2b9-e07e05f6f996
---
🎨 [Mockup](https://www.figma.com/design/U7BdiF9n5uIrwdiB6N2cDX/Workflows--Requests-?node-id=17900-111054&t=KS13dC6iuiTLpAqH-1)

🧠 [Document Center API documentation](https://dev-askhr-platform-api.azurewebsites.net/swagger/#/file_repository%20v2)

🎟️ Azure DevOps feature

# **1. Overview**

Employee Change Requests (ECRs) currently support notes but cannot include supporting documents. Customers frequently need to attach files that provide context or justification for a requested change — offer letters, promotion documents, written notices, performance assessments, resignation letters, etc.

This feature introduces **attachments** as an optional component of an ECR, allowing requesters and approvers with edit permissions to upload files. All approvers — including optional Employee Acknowledgement participants — will be able to view and download attachments. Attachments will immediately upload to Document Center for storage.

This will be the first consumer of a **reusable attachments component** that will later be shared across onboarding tasks and custom form-based tasks.

---

# **2. Goals**

### **Primary Goals**

- Allow requesters to attach one or multiple documents to an Employee Change Request.
- Allow approvers (including employee acknowledgement steps) to view and download attachments.
- Allow approvers with edit capabilities to upload or delete attachments prior to a request reaching its final approved state.
- Automatically store all attachments in Document Center immediately upon upload.
- Provide clear activity logging for upload and removal actions.
- Enable visibility of attachments in both active and closed tasks.

### **Secondary Goal**

- Establish a reusable attachment framework for future task types (e.g., onboarding tasks, custom task builder).

---

# **3. Non-Goals**

- No support for attachment previews (download only).
- No support for version history — replacement is treated as deletion + upload.
- No temporary storage or pre-approval storage logic (will be added later).
- No changes to Document Center UI or backend structure.
- No mobile UI redesign beyond ensuring base usability.

---

# **4. Users & Permissions**

### **Requester**

- Can upload attachments before submitting the request.
- Can remove attachments before submission.
- Can download attachments at any point (even after closure).

### **Approver (View-only)**

- Can view and download attachments.
- Cannot upload or delete attachments.

### **Approver (Edit permissions)**

- Can upload attachments until the request reaches final approval.
- Can delete attachments until final approval.

### **Employee Acknowledgement Step (if enabled)**

- May view and download attachments (no upload/delete).

### **Post-Approval**

- Once the request is fully approved, attachments become read-only for all (except cancel functionality).

---

# **5. Functional Requirements**

## **5.1 Uploading Attachments**

- Upload control appears within **Change Details → Attachments**.
- Attachments are optional.
- Users may upload **multiple attachments**, either:
    - In a single select-with-multi-upload operation, or
    - Repeated single uploads using “Add Attachment.”

### **Constraints**

- **Allowed file types:** Follows Document Center allowlist (PDF, images, MS Office files, etc.).
- **Max file size:** 25 MB per file.
- **Max total attachments:** 1,000 per request.

### **Behavior**

- On successful upload:
    - File appears in a simple list under “Attachments.”
    - File name is displayed as the download label.
    - Attachment immediately uploads to Document Center.
    - Activity log entry is created.
- On failed upload:
    - Inline UI error appears (e.g., invalid type, size too large).

---

## **5.2 Viewing & Downloading Attachments**

- Attachments displayed as a simple list of file names.
- Clicking a file name triggers download using a pre-signed URL from Document Center.
- Available in:
    - Draft request view
    - Approver task view
    - Closed tasks

---

## **5.3 Deleting Attachments**

- Only requesters or approvers with edit permissions may delete.
- Delete action uses a hard delete:
    - Attachment is removed from the task
    - Attachment is deleted from Document Center
- Deletion is allowed only until **final approval** is reached.
- Activity log entry is created upon deletion.

---

## **5.4 Activity Log Rules**

### **Required Entries**

- “{User} uploaded ‘{filename}’”
- “{User} removed ‘{filename}’”

### **Not Required**

- No log entries for:
    - Replacement events (implicitly handled by delete + upload)
    - Document Center storage or updates

---

## **5.5 Closed Tasks Behavior**

- Closed tasks must display the attachments list in read-only form.
- All users who can view the closed task may download attachments.
- No editing or deleting attachments is permitted.

---

# **6. User Experience Requirements**

### **Placement**

- Appears in “Change Details” section under “Notes.”

### **Attachment UI**

- Button: **Add Attachment**
- When files are present:
    - Show simple list of filenames, each as a download link.
    - For each file:
        - Download link (everyone)
        - Delete icon (if user has permission and task is not fully approved)

### **Error States**

- Invalid file type → inline error message.
- File too large → inline error with size limit.
- Upload failure → inline retry prompt.

### **Empty State**

- “No attachments added yet” until first file is uploaded.

---

# **7. Data Requirements**

(Engineering exploration required; PM will link Documentation Center wiki rather than prescribe storage details.)

### Required considerations (owned by engineering):

- Metadata to store (file name, uploadedBy, timestamps, docCenterId, etc.)
- Relationship between Workflows and Document Center IDs
- Handling deletion propagation

No strict schema requirements are imposed in the PRD.

---

# **8. Document Center Integration Requirements**

### Immediate Upload

- All attachments upload to Document Center **as soon as they are added** — not at approval.

### No Folder Structure

- Document Center does not support folders; default placement is acceptable.

### Deletions

- Removing an attachment from a workflow request must delete the corresponding Document Center asset.

### No Additional Logging

- No audit log entries required for Document Center operations.

---

# **9. Future-Proofing Requirements**

This component must be designed so that:

1. It can be reused by **onboarding tasks** (EEO, Emergency Contact, Direct Deposit, etc.).
2. It can be reused by **custom form-based tasks** in the future.
3. Permissions and UI layers are abstracted so different task types can inherit shared logic.

No onboarding-specific rules need implementation at this stage.

---

# **10. Success Metrics**

### **Primary Indicators**

- Percentage of ECRs created with attachments (baseline establishment).
- Number of support tickets related to missing supporting documents (should decrease).
- Number of successful attachment downloads with no Document Center errors.

### **Quality Indicators**

- Upload success/error rate.
- No regression in ECR creation flow performance.

---

# **11. Edge Cases & Constraints**

- **File name collisions:** Backend should handle uniqueness; UI shows only the visible name.
- **Deleting after approval:** Not allowed; delete button hidden.
- **Large batches:** System must support up to 1,000 attachments efficiently.
- **Concurrent uploads:** Multiple users adding attachments simultaneously should not override each other.

---

# **12. Open Questions (for Engineering/Design)**

4. Do we need a loading/progress indicator for large files?
5. Should we prevent navigation away from the page during active uploads?
6. Should the attachment list support sorting (by upload time, user)? Currently out of scope.

---

# **13. Timeline & Dependencies**

- **Dependencies:** Document Center API, Workflow task detail UI, permissions logic, activity log subsystem.
- **Design:** UX mockups from Rachel following this PRD.
- **Engineering:** Workflows team + Document Center team consultation.