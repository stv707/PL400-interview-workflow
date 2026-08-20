# Canvas App Checkpoint: Interviewer Lookup

This document describes the completed interviewer Canvas App after the Forms, Dataverse,
Word, OneDrive, email, and document URL backend was verified.

## Completion status

The coauthored app compiled successfully, passed App Checker, and was saved/published in
the Development environment. The full Forms-to-Canvas test was completed successfully.

## App purpose

An internal interviewer enters the applicant's six-digit Interview Code. The app retrieves exactly one Dataverse row and displays the application details. The interviewer can open the generated document.

## Screens

### Search screen — `Screen1`

Controls:

- Header label.
- Text input: `txtInterviewCode`.
- Search button: `btnSearch`.
- Validation label.
- Optional recent lookup result.

### Details screen — `scrApplicationDetails`

Display:

- Applicant Name.
- Email Address.
- Phone Number.
- Working Experience.
- Application Comments.
- Interview Code.
- Application Status.
- Document File Name.
- Open Document button.

## Power Fx search pattern

For a text Interview Code column:

```powerfx
Set(
    varApplication,
    LookUp(
        'Interview Applications',
        'Interview Code' = Trim(txtInterviewCode.Text)
    )
);

If(
    IsBlank(varApplication),
    Notify(
        "No application was found for this interview code.",
        NotificationType.Error
    ),
    Navigate(
        scrApplicationDetails,
        ScreenTransition.Fade
    )
)
```

## Validation pattern

```powerfx
If(
    !IsMatch(Trim(txtInterviewCode.Text), "^[0-9]{6}$"),
    Notify(
        "Enter a six-digit interview code.",
        NotificationType.Warning
    ),
    /* execute lookup */
)
```

## Open document

If Dataverse stores a secure URL:

```powerfx
Launch(varApplication.'Document URL')
```

`Document URL` must contain the Word viewer URL returned by the OneDrive **Create share
link** action. Do not use the raw OneDrive file path.

## Print summary — `scrPrintSummary`

The app contains a dedicated portrait print screen. Its button uses:

```powerfx
Print()
```

The interviewer selects **Save as PDF** in the browser print dialog. This creates a PDF
of the Canvas summary without requiring a physical printer.

The generated Word document remains available through **Open Word document** when the
interviewer needs the original document layout.

If only the OneDrive path is stored, use a Power Automate flow or a controlled document-link strategy. Do not expose anonymous links to applicant documents.

## Security

- Share the app only with interviewers.
- Give the interviewer security role read access to Interview Applications.
- Give the interviewer read access to the document location.
- Do not share the app with public applicants.
- Avoid a gallery that lists all applicants by default.

## Verified end-to-end test

The complete scenario was tested with a real Microsoft Forms submission:

```text
Forms response
  → Dataverse row
  → Interview Code
  → Word document
  → OneDrive file
  → Document URL
  → applicant email
  → Canvas exact-code lookup
  → Word viewer launch
  → Save as PDF print screen
```

## Test cases

| Test | Expected result |
|---|---|
| Valid code | Correct applicant displayed |
| Invalid six-digit code | Friendly not-found message |
| Five-digit value | Validation message |
| Blank value | Validation message |
| Document Created row | Open document available |
| Received row without document | Disable or hide Open Document |
| Unauthorized user | App/table access denied |
