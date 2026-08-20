# Student Lab: Interview Application Automation

## Learning objectives

By completing this lab, you will:

- Create a Dataverse table with text, email, phone, URL, choice, and status fields.
- Create a Microsoft Form for external applicant intake.
- Use the Microsoft Forms trigger and Get response details action.
- Create a Dataverse row from dynamic content.
- Generate an applicant-facing reference code.
- Use a Dataverse trigger to orchestrate document processing.
- Populate a Word template using native content controls.
- Create a file in OneDrive for Business.
- Update the original Dataverse row after processing.
- Send an email containing the interview code.
- Add error-handling and status tracking.
- Prepare a Canvas App to retrieve a record using an exact code lookup.

## Prerequisites

- A Power Platform Development environment with Dataverse.
- Permission to create tables, columns, flows, apps, and connections.
- Microsoft Forms access.
- OneDrive for Business access.
- Word Online (Business) connector access.
- Office 365 Outlook connector access.
- A test mailbox that you control.
- A solution-aware maker experience is recommended.

Use a Development environment only. Do not use production applicant data for this exercise.

---

## Part 1 — Create the solution

1. Open `https://make.powerapps.com`.
2. Select the intended Development environment.
3. Open **Solutions**.
4. Select **New solution**.
5. Use values similar to:

```text
Display name: Interview Application Automation
Name: InterviewApplicationAutomation
Publisher: Your training publisher
Version: 1.0.0.0
```

6. Create the solution.

Keep the table, choice, flows, Canvas App, environment variables, and connection references inside this solution where possible.

---

## Part 2 — Create the Dataverse table

Create a table named:

```text
Interview Application
```

Suggested plural name:

```text
Interview Applications
```

Use a meaningful publisher prefix. Do not copy the `crf96_` prefix from another environment; logical names are environment-specific.

### Required columns

| Display name | Suggested type | Required | Purpose |
|---|---|---:|---|
| Applicant Name | Text | Yes | Full name from Forms |
| Email Address | Email | Yes | Applicant notification address |
| Phone Number | Phone | No | Applicant contact number |
| Working Experience | Multiple lines of text | No | Experience response |
| Application Comments | Multiple lines of text | No | Applicant comments |
| Form Response ID | Text | Yes | Idempotency and duplicate prevention |
| Interview Code | Text, maximum length 6 | Yes | Applicant-facing six-digit code |
| Application Status | Choice | Yes | Processing state |
| Document File Name | Text | No | Generated OneDrive path or filename |
| Document URL | URL | No | Optional secure document link |
| Error Details | Multiple lines of text | No | Failure diagnostics |

### Application Status choices

Create a choice column called `Application Status` with:

| Label | Example value |
|---|---:|
| Received | 100000000 |
| Document Created | 100000001 |
| Processing Failed | 100000002 |
| Interview Completed | Add your own value |
| Cancelled | Add your own value |

The numeric values are environment-specific. In Power Automate, use the dynamic choice value supplied by the designer instead of assuming numbers from another environment.

### Design notes

- Do not name the custom choice column simply `Status`; Dataverse already has system status fields.
- Keep the Dataverse row GUID as the true record identity.
- The six-digit Interview Code is a lookup key, not the primary key.
- Add an alternate key on `Form Response ID` if your environment and design support it.
- Consider an alternate key on `Interview Code` to prevent duplicates.

Save and publish the table before building the flows.

---

## Part 3 — Create the Microsoft Form

1. Open Microsoft Forms.
2. Select **New Form**.
3. Name it:

```text
Interview Application
```

4. Add these questions:

| Question | Type | Required |
|---|---|---:|
| Applicant Name | Text | Yes |
| Applicant Email Address | Text | Yes |
| Applicant Phone Number | Text | Yes |
| Enter your working Experience | Long text | Yes |
| Comment | Long text | No |

5. Add a privacy/consent question if required by the business:

```text
I consent to Bosch collecting and processing my application information for recruitment purposes.
```

Use a Choice question with `Yes` and `No`, and make it required.

6. Configure the response settings appropriately for the demonstration.
7. Copy the Form link for testing.

The Form is the intake surface only. Applicants do not receive Dataverse permissions.

---

## Part 4 — Build Flow 1: Capture Form Submission

Create an automated cloud flow inside the solution.

Suggested name:

```text
Interview - Capture Form Submission
```

### Trigger

```text
Microsoft Forms — When a new response is submitted
```

Select the form you created.

### Action 1: Get response details

```text
Microsoft Forms — Get response details
```

Set:

- Form ID: the same Form selected in the trigger.
- Response ID: the trigger's `Response Id` dynamic value.

### Action 2: Optional duplicate check

Add a Dataverse **List rows** action before creating the row:

- Table: Interview Applications.
- Filter rows:

```text
Form Response ID equals the trigger Response Id
```

If a matching row exists, terminate as a duplicate. This prevents retries from creating multiple applications.

For a basic classroom MVP, this duplicate check can be added as an enhancement after the first successful run.

### Action 3: Add a new Dataverse row

Use **Microsoft Dataverse — Add a new row**.

Map the Form answers to Dataverse columns:

| Dataverse column | Form/dynamic value |
|---|---|
| Applicant Name | Applicant Name |
| Email Address | Applicant Email Address |
| Phone Number | Applicant Phone Number |
| Working Experience | Enter your working Experience |
| Application Comments | Comment |
| Form Response ID | Response Id |
| Application Status | Received |
| Interview Code | Six-digit expression described below |

### Six-digit code expression

For this classroom MVP, use:

```powerautomate
string(rand(100000,999999))
```

This produces a six-digit string suitable for the demonstration.

For production, do not rely on an unchecked random value. Use a Dataverse sequence/autonumber approach or add a uniqueness check and retry strategy.

### Save and test Flow 1

Submit the Form once using a test mailbox. Confirm that:

- The flow run succeeds.
- One Dataverse row is created.
- The Form Response ID is stored.
- The Interview Code contains exactly six digits.
- Application Status is `Received`.

---

## Part 5 — Prepare the Word template

Use the supplied file:

```text
InterviewApplicationTemplatev2.docx
```

Upload it to OneDrive in a folder such as:

```text
/bcp-interview/
```

### Required content controls

The template must contain six native Word plain-text content controls:

| Content control title | Data source |
|---|---|
| Applicant Name | Dataverse Applicant Name |
| Phone Number | Dataverse Phone Number |
| Working Experience | Dataverse Working Experience |
| Application Comments | Dataverse Application Comments |
| Interview Code | Dataverse Interview Code |
| Email Address | Dataverse Email Address |

### Creating controls manually

If the supplied template does not expose all six fields:

1. Open the document in Word desktop.
2. Enable the **Developer** tab.
3. Place the cursor where a value should be inserted.
4. Select **Plain Text Content Control**.
5. Open **Properties**.
6. Set a unique title and tag.
7. Repeat for every field.
8. Save the file.
9. Upload or replace the OneDrive copy.
10. Delete and re-add the Populate Word template action so the schema refreshes.

Do not use ordinary text placeholders. The Word connector requires real content controls.

---

## Part 6 — Build Flow 2: Generate Document and Notify

Create a second automated cloud flow inside the solution.

Suggested name:

```text
Interview - Generate Document and Notify
```

### Trigger

```text
Microsoft Dataverse — When a row is added, modified or deleted
```

Configure:

- Table: Interview Applications.
- Change type: Added.
- Scope: Organization, or the least scope appropriate for the exercise.

For an update-triggered design, add a filter so the flow only runs when Application Status is `Received`. This prevents a loop when the flow updates the same row.

### Action 1: Populate a Microsoft Word template

Use:

```text
Word Online (Business) — Populate a Microsoft Word template
```

Select:

- Location: OneDrive for Business.
- Document library: OneDrive.
- File: `InterviewApplicationTemplatev2.docx`.

Wait for the dynamic content schema to load. Confirm that six separate content-control fields appear. Map each field to the Dataverse trigger values.

### Action 2: Create the document file

Use:

```text
OneDrive for Business — Create file
```

Suggested values:

```text
Folder path: /bcp-interview
File name: InterviewApplication_<Interview Code>.docx
File content: output from Populate a Microsoft Word template
```

The file-content expression must be:

```powerautomate
@body('Populate_a_Microsoft_Word_template')
```

The action name may differ in your environment. Use the actual generated action name in the expression.

Do not map Applicant Name, Email Address, or any other text field to File content. Doing so creates a corrupt Word file.

### Action 3: Update the Dataverse row

Use:

```text
Microsoft Dataverse — Update a row
```

Set:

```text
Application Status = Document Created
Document File Name = Path or Name from Create file
```

Use the Dataverse row ID from the trigger as the record ID.

### Action 4: Send applicant email

Use:

```text
Office 365 Outlook — Send an email (V2)
```

To:

```text
Email Address from the Dataverse trigger
```

Subject example:

```text
Interview application received - reference <Interview Code>
```

Body example:

```text
Dear <Applicant Name>,

Thank you for submitting your interview application.

Your interview reference number is:

<Interview Code>

Please bring this number when you arrive for your interview.

Regards,
Bosch Recruitment Team
```

Only send the reference number to the applicant. Do not attach internal recruitment notes unless explicitly required.

### Recommended action order

```text
Populate Word template
        ↓
Create file in OneDrive
        ↓
Update Dataverse status and file path
        ↓
Send applicant email
```

Configure each `run after` condition to continue only after the preceding action succeeds.

---

## Part 7 — Error handling

Add Scopes for a production-style version:

```text
Try
  Populate Word template
  Create OneDrive file
  Update Dataverse row
  Send email

Catch
  Update Application Status = Processing Failed
  Write the error message to Error Details
  Notify the administrator
```

Configure the Catch scope to run after the Try scope has failed, timed out, or been skipped.

Do not hide failures by deleting the Dataverse row. The row is the durable audit record.

---

## Part 8 — End-to-end test

Use a controlled test mailbox.

1. Submit the Microsoft Form.
2. Confirm Flow 1 succeeds.
3. Confirm a Dataverse row appears.
4. Confirm a six-digit code is generated.
5. Confirm Flow 2 starts.
6. Confirm Word template population succeeds.
7. Confirm the `.docx` is created in OneDrive.
8. Open the document in Word.
9. Confirm each field contains the correct value.
10. Confirm Dataverse status changes to `Document Created`.
11. Confirm the applicant receives the email.
12. Confirm the email code matches the document code and Dataverse code.

### Expected evidence

```text
Application Status = Document Created
Document File Name = /bcp-interview/InterviewApplication_<code>.docx
Flow 1 run = Succeeded
Flow 2 run = Succeeded
Word file opens normally
Email received with the same code
```

---

## Part 9 — Security and ALM

### Security

- Applicants access only Microsoft Forms.
- Applicants do not receive Dataverse access.
- Interviewers receive the Canvas App and read access to the relevant Dataverse table.
- Interviewers need access to the OneDrive folder or SharePoint library.
- Do not create anonymous document-sharing links for applicant data.
- Use a team-owned SharePoint library for production instead of a personal OneDrive.

### Environment variables

Use environment variables for:

- Document folder path.
- Template file path or identifier.
- Recruitment notification mailbox.
- SharePoint site or OneDrive location.

### Connection references

Use solution connection references for:

- Microsoft Forms.
- Microsoft Dataverse.
- Word Online (Business).
- OneDrive for Business.
- Office 365 Outlook.

Students must rebind connections after importing into another environment.

### Deployment checklist

- Export an unmanaged solution for Development backup.
- Do not include real applicant data.
- Document all environment variables.
- Rebind connection references in the target environment.
- Replace the Form ID with the target Form.
- Replace the Word template location.
- Test with a controlled mailbox.
- Verify security roles before sharing the Canvas App.

---

## Student challenge extensions

1. Add a duplicate Form Response ID check.
2. Replace random code generation with a Dataverse autonumber sequence.
3. Add a Processing Failed status and Catch scope.
4. Add retry processing for failed applications.
5. Store the document in SharePoint instead of personal OneDrive.
6. Add Interview Date, Interviewer, Outcome, and Interviewer Comments.
7. Build a Canvas App for exact code lookup.
8. Add a security role for interviewers.
9. Add Power BI reporting.
10. Add retention and privacy controls.
