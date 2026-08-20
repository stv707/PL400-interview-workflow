# Canvas App: Interviewer Lookup and PDF Printing

This guide explains how a student can build the internal interviewer Canvas App manually, or study the coauthored YAML reference implementation in `BoschInterviewCanvas/`.

The app is built after the Forms, Dataverse, Word, OneDrive, email, and `Document URL` backend is working.

## Completed reference implementation

The verified app contains three screens:

```text
Screen1                 — six-digit interview code search
scrApplicationDetails   — applicant details and document actions
scrPrintSummary         — printable Canvas summary and Save as PDF
```

The reference YAML files are:

```text
BoschInterviewCanvas/App.pa.yaml
BoschInterviewCanvas/Screen1.pa.yaml
BoschInterviewCanvas/scrApplicationDetails.pa.yaml
BoschInterviewCanvas/scrPrintSummary.pa.yaml
```

Do not commit or edit `_EditorState.pa.yaml`; Power Apps Studio owns that file.

## Prerequisites

- The `Interview Applications` Dataverse table exists.
- The table contains `Interview Code` and `Document URL`.
- `Document URL` contains the OneDrive/SharePoint Word viewer URL returned by the document flow.
- The interviewer has read access to the Dataverse table.
- The interviewer has permission to open the document location.
- The document-generation flow has been tested successfully.

## Part 1 — Create the Canvas App

1. Open `https://make.powerapps.com`.
2. Select the correct Development environment.
3. Open the solution containing the interview workflow.
4. Select **New → App → Canvas**.
5. Choose **Tablet** layout.
6. Name the app:

```text
Bosch Interview Canvas
```

7. Save the app inside the solution.

## Part 2 — Add the Dataverse data source

1. In Power Apps Studio, select **Data**.
2. Select **Add data**.
3. Choose **Microsoft Dataverse**.
4. Select the `Interview Applications` table.
5. Confirm the following fields appear:

```text
Applicant Name
Application Comments
Application Status
Document File Name
Document URL
Email Address
Interview Code
Phone Number
Working Experience
```

The display name may be different if the student uses another publisher prefix. Always use the Canvas-facing names shown by Studio.

## Part 3 — Build the search screen

Rename the first screen to `Screen1` or retain the default name if the lab package expects it.

Add these controls:

| Control | Name | Purpose |
|---|---|---|
| Text input | `txtInterviewCode` | Six-digit code input |
| Button | `btnSearch` | Performs the lookup |
| Label/Text | `txtSearchStatus` | Shows validation feedback |
| Header/Text | `txtPageTitle` | Page heading |

### Text input properties

Set:

```text
Name: txtInterviewCode
Hint text: For example, 675838
Max length: 6
Accessible label: Six-digit interview code
```

### Search button formula

Set `btnSearch.OnSelect` to:

```powerfx
Set(varSearchAttempted, true);
If(
    !IsMatch(
        Trim(txtInterviewCode.Text),
        "^[0-9]{6}$"
    ),
    Notify(
        "Enter a valid six-digit interview code.",
        NotificationType.Warning
    ),
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
            "No interview application was found for this code.",
            NotificationType.Error
        ),
        Navigate(
            scrApplicationDetails,
            ScreenTransition.Fade
        )
    )
)
```

### Validation message formula

Set the validation label or text control `Visible` property to:

```powerfx
varSearchAttempted &&
!IsMatch(
    Trim(txtInterviewCode.Text),
    "^[0-9]{6}$"
)
```

Set its `Text` property to:

```powerfx
"Use exactly six digits."
```

### Search screen initialization

Set the screen `OnVisible` property to:

```powerfx
Set(varApplication, First('Interview Applications'));
Set(varSearchAttempted, false)
```

The initial typed record assignment helps Power Fx infer the variable type. The Search button replaces it with the matching record.

## Part 4 — Build the applicant details screen

Add a new blank screen named:

```text
scrApplicationDetails
```

Add text controls for:

- Applicant Name
- Interview Code
- Email Address
- Phone Number
- Working Experience
- Application Comments
- Application Status

### Example formulas

Applicant name:

```powerfx
Coalesce(
    varApplication.'Applicant Name',
    "Applicant"
)
```

Interview code:

```powerfx
"Interview code: " &
Coalesce(
    varApplication.'Interview Code',
    ""
)
```

Email:

```powerfx
"Email: " &
Coalesce(
    varApplication.'Email Address',
    "Not provided"
)
```

Phone:

```powerfx
"Phone: " &
Coalesce(
    varApplication.'Phone Number',
    "Not provided"
)
```

Working experience:

```powerfx
"Working experience: " &
Coalesce(
    varApplication.'Working Experience',
    "Not provided"
)
```

Comments:

```powerfx
"Comments: " &
Coalesce(
    varApplication.'Application Comments',
    "None"
)
```

Status:

```powerfx
"Status: " &
If(
    IsBlank(varApplication.'Application Status'),
    "Unknown",
    Text(varApplication.'Application Status')
)
```

### Back button

Add a button with:

```powerfx
Navigate(
    Screen1,
    ScreenTransition.Fade
)
```

## Part 5 — Open the generated Word document

Add a button named:

```text
btnOpenDocument
```

Set its `OnSelect` property to:

```powerfx
Launch(
    varApplication.'Document URL'
)
```

Set its `DisplayMode` property to:

```powerfx
If(
    IsBlank(varApplication.'Document URL'),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Important URL rule

Do not use the raw OneDrive path:

```text
/Documents/bcp-interview/InterviewApplication_123456.docx
```

That is a file path and may open a blank page. `Document URL` must contain the Word viewer URL returned by the OneDrive **Create share link** action, for example a URL containing:

```text
/:w:/...
```

The document flow stores the returned `WebUrl` value:

```powerautomate
@body('Create_share_link')?['WebUrl']
```

## Part 6 — Build the print summary screen

Add a new blank screen named:

```text
scrPrintSummary
```

This screen is deliberately designed for printing. It should use:

- White background.
- Clear heading.
- Applicant name.
- Interview code.
- Email and phone.
- Experience and comments.
- Status.
- Minimal navigation controls.

Use a portrait print screen template when Studio provides one. Otherwise use a fixed portrait-style layout with a white background.

### Navigate to the print screen

On the details screen, add a button named:

```text
btnPrintSummary
```

Set `OnSelect` to:

```powerfx
Navigate(
    scrPrintSummary,
    ScreenTransition.None
)
```

### Print button

Add a button named:

```text
btnPrint
```

Set `OnSelect` to:

```powerfx
Print()
```

Set the button `Visible` property to:

```powerfx
Not(scrPrintSummary.Printing)
```

When the interviewer selects the button, the browser print dialog opens. Select:

```text
Save as PDF
```

This prints the Canvas summary as a PDF. It does not silently control a physical printer.

### Back button

Set `OnSelect` to:

```powerfx
Navigate(
    scrApplicationDetails,
    ScreenTransition.None
)
```

Set its `Visible` property to:

```powerfx
Not(scrPrintSummary.Printing)
```

## Part 7 — Visual design guidance

Use a professional light operational layout:

```text
Page background: RGBA(245, 246, 248, 1)
Card background: RGBA(255, 255, 255, 1)
Dark header: RGBA(35, 35, 35, 1)
Bosch red accent: RGBA(198, 0, 0, 1)
Primary text: RGBA(35, 35, 35, 1)
Secondary text: RGBA(80, 80, 80, 1)
```

Recommendations:

- Use explicit text colors on every text control.
- Use readable font sizes.
- Keep buttons at least 44px high.
- Use responsive containers where possible.
- Keep the print screen uncluttered.
- Do not expose a gallery containing every applicant.
- Use a clear disabled state when no Document URL exists.

## Part 8 — Share and secure the app

1. Save the app.
2. Share it with the interviewer security group.
3. Assign table permissions:
   - Read access to Interview Applications.
   - Read access to the document location.
4. Do not share the app with applicants.
5. Do not use anonymous OneDrive links.
6. Confirm each interviewer can open the organization-scoped Word URL.

## Part 9 — Test the Canvas App

Use a known completed application such as a recently submitted Form response.

| Test | Expected result |
|---|---|
| Enter valid six-digit code | Details screen opens |
| Enter unknown six-digit code | Not-found notification |
| Enter five digits | Validation warning |
| Leave input blank | Validation warning |
| Select Open Word document | Word viewer URL opens |
| Select Print summary PDF | Browser print dialog appears |
| Select Save as PDF | PDF is produced from Canvas summary |
| Record has no Document URL | Open button is disabled |
| Unauthorized user | App/table access is denied |

## Part 10 — Full end-to-end test

1. Submit Microsoft Forms response.
2. Verify Flow 1 creates the Dataverse row.
3. Note the generated Interview Code.
4. Verify Flow 2 creates the Word document.
5. Verify OneDrive file exists.
6. Verify `Document URL` contains the Word viewer URL.
7. Verify applicant email contains the same code.
8. Open the Canvas App.
9. Enter the code.
10. Verify all applicant details.
11. Open the Word document.
12. Print the Canvas summary and choose Save as PDF.

## Canvas source reference

The coauthored YAML reference was compiled successfully and passed App Checker. The YAML is a study/reference artifact, not a substitute for adding the Dataverse data source and saving/publishing through Studio in a student's environment.
