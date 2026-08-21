# Bosch Interview Application Automation

A PL-400 end-to-end demonstration and student lab using Microsoft Forms, Dataverse, Power Automate, Word Online (Business), OneDrive for Business, Outlook, and Power Apps Canvas Apps.

## Scenario

An applicant completes a Microsoft Form to apply for an interview. Power Automate stores the submission in Dataverse, generates a unique six-digit interview code, populates a Word document, stores the document in OneDrive, and emails the code to the applicant. When the applicant arrives, an internal interviewer uses a Canvas App to enter the code and retrieve the application and document.

```text
Microsoft Forms
      |
      v
Power Automate: Capture Form Submission
      |
      v
Dataverse: Interview Application
      |
      v
Power Automate: Generate Document and Notify
      |
      +--> Populate Word template
      +--> Create .docx in OneDrive
      +--> Update Dataverse status and file path
      +--> Email applicant the interview code

Interviewer Canvas App
      |
      v
Search Dataverse by six-digit interview code
      |
      v
Open the generated document
```

## Repository contents

- `README.md` — overview and quick start.
- `STUDENT-LAB.md` — complete student implementation guide.
- `ARCHITECTURE.md` — design decisions, data model, security, and ALM.
- `TROUBLESHOOTING.md` — common failures and recovery steps.
- `CANVAS-APP-NEXT.md` — Canvas App implementation and testing guide.

- `InterviewApplicationTemplatev2.docx` — recommended Word template with six native content controls.
- `InterviewApplicationTemplate.docx` — earlier template retained for comparison only; use `v2` for the lab.
- `create_template.py` — original local template-generation experiment; not required for the student lab.

## Recommended implementation order

1. Create a solution in a Development environment.
2. Create the Dataverse table and columns.
3. Create the Microsoft Form.
4. Build the Forms-to-Dataverse intake flow.
5. Create/upload the Word template.
6. Build the Dataverse-to-document-and-email flow.
7. Test the backend end to end.
8. Build the interviewer Canvas App.
9. Test the Canvas lookup, Word viewer link, and Save as PDF print screen.
10. Export the solution and document environment-specific configuration.

## Important implementation rule

In the OneDrive **Create file** action, the file body must be the output of the Word action:

```powerautomate
@body('Populate_a_Microsoft_Word_template')
```

Do not use an applicant text field as the file body. For example, this is wrong and creates an unreadable `.docx` file:

```powerautomate
@triggerOutputs()?['body/crf96_applicant_name']
```

## Environment portability

This folder is intended for GitHub distribution. Do not commit:

- Tenant IDs, environment IDs, form IDs, connection IDs, or connection-reference IDs.
- Personal email addresses or real applicant data.
- Exported flow definitions containing live connection names.
- `.env` files, access tokens, passwords, or screenshots containing personal data.

Students must replace all environment-specific values in their own environment.

## Verified demonstration outcome

The complete demonstration has been tested successfully:

```text
Microsoft Forms submission
  → Dataverse application row
  → six-digit Interview Code
  → populated Word document
  → OneDrive file
  → organization-scoped Word viewer URL in Dataverse
  → applicant email
  → Canvas App exact-code lookup
  → Open Word document
  → Canvas print summary and browser Save as PDF
```

The Canvas App uses the Dataverse `Document URL` value returned by the OneDrive
**Create share link** action. Do not construct a launch URL from the raw
`/Documents/...docx` path; that path can open a blank page instead of the Word viewer.
