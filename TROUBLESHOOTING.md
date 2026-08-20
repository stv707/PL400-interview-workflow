# Troubleshooting Guide

## 1. Word says the generated file is corrupt

### Symptom

The file has a `.docx` extension but Word cannot open it.

### Check

Inspect the OneDrive **Create file** action. The File content/body must be the output from the Word template action:

```powerautomate
@body('Populate_a_Microsoft_Word_template')
```

### Common cause

The body was mapped to Applicant Name or another text field. A text value saved as `.docx` is not a valid Word package.

### Recovery

1. Replace the Create file body with the Populate Word template output.
2. Save the flow.
3. Confirm the saved definition still contains the correct expression.
4. Generate a new file with a new interview code.
5. Do not reuse the corrupted file.

## 2. Every Word field contains the interview code

### Symptom

Applicant Name, email, phone, comments, and code all contain the same value.

### Cause

The Word connector exposed one generic content-control field such as `sdt` instead of separate fields.

### Recovery

1. Open the template in Word desktop.
2. Delete the old controls.
3. Insert separate native Plain Text Content Controls.
4. Set unique titles/tags.
5. Save as a new file.
6. Upload the new file.
7. Delete and re-add the Populate Word template action.
8. Confirm separate dynamic fields appear.
9. Map each field individually.

## 3. The Word field does not appear in Power Automate

### Checks

- Confirm the selected file is the actual `.docx`, not the folder.
- Confirm the file has been saved and closed.
- Reopen the Populate action and reselect the file.
- Delete and re-add the action to refresh its dynamic schema.
- Confirm the control is a native Word content control.

## 4. Flow asks for connection repair

Repair or select connected accounts for:

- Microsoft Forms.
- Microsoft Dataverse.
- Word Online (Business).
- OneDrive for Business.
- Office 365 Outlook.

Save and turn on the flow. Confirm the connection is associated with the correct Development environment.

## 5. Dataverse row is created but document flow does not run

Check:

- Document flow is turned on.
- Trigger table is the correct Interview Applications table.
- Trigger change type includes Added.
- The row is created after the flow was enabled.
- The trigger scope includes the row owner.

Rows created before the trigger was enabled will not automatically replay. Create a new test row or resubmit the Form.

## 6. Flow runs but status remains Received

Inspect the run history and action order:

```text
Populate Word template
Create file
Update application
Send email
```

The Update application action must run after Create file succeeds and must set:

```text
Application Status = Document Created
```

## 7. Applicant does not receive the email

Check:

- The Dataverse Email Address column contains a valid email.
- The Office 365 Outlook connection is connected.
- The Send email action succeeded.
- Junk/quarantine folders.
- Mailbox sending restrictions.

Use a controlled training mailbox during the lab.

## 8. Duplicate applications are created

Add an idempotency check using Form Response ID:

1. List existing Dataverse rows where Form Response ID equals the incoming Response ID.
2. If a row exists, terminate as duplicate.
3. Otherwise create the row.

## 9. Interview code is not six digits

Use:

```powerautomate
string(rand(100000,999999))
```

For production, use a collision-safe Dataverse sequence. Never expose a GUID to the applicant when the requirement is a six-digit reference.

## 10. Canvas App cannot find the application

Check:

- Text versus numeric data type.
- Leading zeros, if the design allows them.
- Exact comparison rather than partial search.
- Correct Dataverse data source.
- Delegation warnings.
- Application status filter.

Example for a text column:

```powerfx
LookUp(
    'Interview Applications',
    'Interview Code' = txtInterviewCode.Text
)
```

## 11. Wrong environment

Before making changes, verify:

- Maker portal environment selector.
- Power Automate environment selector.
- Dataverse table visible in the same environment.
- Flow run history belongs to the same environment.

Never assume the default environment is the intended Development environment.
