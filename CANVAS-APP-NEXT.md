# Canvas App Checkpoint: Interviewer Lookup

This document is the next implementation checkpoint after the Forms, Dataverse, Word, OneDrive, and email backend is verified.

## App purpose

An internal interviewer enters the applicant's six-digit Interview Code. The app retrieves exactly one Dataverse row and displays the application details. The interviewer can open the generated document.

## Screens

### Search screen

Controls:

- Header label.
- Text input: `txtInterviewCode`.
- Search button: `btnSearch`.
- Validation label.
- Optional recent lookup result.

### Details screen

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

If only the OneDrive path is stored, use a Power Automate flow or a controlled document-link strategy. Do not expose anonymous links to applicant documents.

## Security

- Share the app only with interviewers.
- Give the interviewer security role read access to Interview Applications.
- Give the interviewer read access to the document location.
- Do not share the app with public applicants.
- Avoid a gallery that lists all applicants by default.

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
