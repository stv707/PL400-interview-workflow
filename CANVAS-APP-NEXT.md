# Student Lab: Build the Interviewer Canvas App

## Lab objective

Build an internal tablet Canvas App that allows an interviewer to:

1. Enter an applicant's six-digit Interview Code.
2. Retrieve exactly one application from Dataverse.
3. Review the applicant's details.
4. Open the generated Word document.
5. Print a clean Canvas summary using the browser's **Save as PDF** option.

The applicant does **not** use this app. The app is for authorized interviewers only.

## What you will build

```text
Screen1
  Search by six-digit Interview Code
       ↓
scrApplicationDetails
  Display applicant details
  Open Word document
  Go to print summary
       ↓
scrPrintSummary
  Print Canvas summary and save as PDF
```

## Prerequisites

Complete the Forms, Dataverse, and Power Automate parts of the interview workflow first.

You need:

- Access to the correct Development environment.
- A solution containing the interview workflow.
- The Dataverse table containing the interview applications.
- A working Microsoft Forms intake flow.
- A working document-generation flow.
- At least one test application whose status is `Document Created`.
- An interviewer account with read access to Dataverse and the OneDrive/SharePoint document location.

The Dataverse table should expose these Canvas-facing columns. Your publisher prefix may differ, so use the names displayed by Power Apps Studio:

| Column | Purpose |
|---|---|
| Applicant Name | Applicant's full name |
| Applicant Comments / Application Comments | Applicant's comments |
| Application Status | Processing status |
| Document File Name | Generated Word filename/path |
| Document URL | Authenticated Word viewer URL |
| Email Address | Applicant email |
| Interview Code | Six-digit lookup code |
| Phone Number | Applicant phone |
| Working Experience | Experience response |

## Checkpoint 0 — Verify the environment and backend

Before creating the app:

1. Open [https://make.powerapps.com](https://make.powerapps.com).
2. Select the intended **Development** environment from the environment switcher.
3. Open the solution containing the interview workflow.
4. Open the Dataverse table and confirm the columns above exist.
5. Open one completed application and confirm:
   - `Interview Code` contains six digits.
   - `Application Status` is `Document Created`.
   - `Document URL` is populated.
   - The URL is a Word viewer URL, normally containing `/:w:/`.

### Why this matters

A Canvas App can be built successfully but still point to the wrong environment or wrong table. Always verify the target before authoring.

### Student checkpoint

Stop here and confirm that one completed application and its six-digit code are visible.



## Checkpoint 1 — Create the blank Canvas App

1. From the solution, select **New**.
2. Select **App → Canvas app**.
3. Choose the **Tablet** layout.
4. Enter this name:

```text
Bosch Interview Canvas
```

5. Select **Create**.
6. Save the app in the current solution.
7. If the first screen is not named `Screen1`, rename it to:

```text
Screen1
```

### Student checkpoint

Confirm that the app opens in Power Apps Studio and contains a blank first screen.



## Checkpoint 2 — Add the Dataverse table

1. In the left navigation, select **Data**.
2. Select **Add data**.
3. Search for **Dataverse**.
4. Select the `Interview Applications` table.
5. Wait for the table to finish loading.
6. Expand the data source and confirm the Interview Code and Document URL fields are available.

If the table is not listed:

- Confirm the environment switcher is correct.
- Confirm the table is included in the solution or available in the environment.
- Do not create a second table with a similar name.

### Student checkpoint

Select the table in the Data pane and confirm its fields are available to formulas.



## Checkpoint 3 — Prepare the search screen

Select `Screen1` in the Tree view. Set its properties:

```text
Name: Screen1
Fill: RGBA(245, 246, 248, 1)
```

Select the screen's **OnVisible** property and enter:

```powerfx
Set(varApplication, First('Interview Applications'));
Set(varSearchAttempted, false)
```

### Why use `First` here?

This initializes the variable with the Dataverse record type. The Search button replaces it with the record matching the code. The app does not display the first record automatically.

### Add the page title

1. Select **Insert → Text label**.
2. Rename it to:

```text
txtPageTitle
```

3. Set its `Text` property to:

```powerfx
"Bosch Interviewer Lookup"
```

4. Set its `Size` to approximately `24`.
5. Set its `FontWeight` to `FontWeight.Bold`.
6. Set its `Color` to:

```powerfx
RGBA(35, 35, 35, 1)
```

### Add the instruction text

1. Insert another text label.
2. Rename it:

```text
txtInstructions
```

3. Set `Text` to:

```powerfx
"Enter the six-digit Interview Code provided to the applicant."
```

### Add the code input

1. Select **Insert → Input → Text input**.
2. Rename it:

```text
txtInterviewCode
```

3. Set these properties:

```text
HintText: For example, 675838
MaxLength: 6
AccessibleLabel: Six-digit interview code
```

If using a modern text input, set its `Type` to numeric when that property is available. Keep the regular `Text` property available because the lookup formula uses `txtInterviewCode.Text`.

### Add the Search button

1. Insert **Button**.
2. Rename it:

```text
btnSearch
```

3. Set its `Text` property to:

```powerfx
"Search"
```

4. Set its `OnSelect` property to:

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

### What the Search formula does

- Sets a flag showing that a search was attempted.
- Validates exactly six digits.
- Performs an exact Dataverse `LookUp`.
- Stores the matching record in `varApplication`.
- Navigates only when a record is found.

### Add the validation message

1. Insert a text label.
2. Rename it:

```text
txtSearchValidation
```

3. Set `Text` to:

```powerfx
"Use exactly six digits."
```

4. Set `Visible` to:

```powerfx
varSearchAttempted &&
!IsMatch(
    Trim(txtInterviewCode.Text),
    "^[0-9]{6}$"
)
```

5. Set its `Color` to:

```powerfx
RGBA(198, 0, 0, 1)
```

### Search-screen checkpoint test

Select **Preview** and test:

| Input | Expected result |
|---|---|
| Blank | Warning appears |
| Five digits | Warning appears |
| Letters | Warning appears |
| Unknown six-digit code | Not-found notification |
| Known completed code | Details screen navigation will work after the details screen is created |

Do not continue until invalid input is rejected.



## Checkpoint 4 — Create the applicant details screen

1. In Tree view, select **New screen**.
2. Choose **Blank**.
3. Rename the screen:

```text
scrApplicationDetails
```

4. Set `Fill` to:

```powerfx
RGBA(245, 246, 248, 1)
```

### Add the applicant name

1. Insert a text label.
2. Rename it:

```text
txtApplicantName
```

3. Set `Text` to:

```powerfx
Coalesce(
    varApplication.'Applicant Name',
    "Applicant"
)
```

Make this the largest text on the screen.

### Add the interview code

Insert a text label named `txtDetailsCode` and set `Text` to:

```powerfx
"Interview code: " &
Coalesce(
    varApplication.'Interview Code',
    ""
)
```

### Add the email

Insert a text label named `txtDetailsEmail` and set `Text` to:

```powerfx
"Email: " &
Coalesce(
    varApplication.'Email Address',
    "Not provided"
)
```

### Add the phone number

Insert a text label named `txtDetailsPhone` and set `Text` to:

```powerfx
"Phone: " &
Coalesce(
    varApplication.'Phone Number',
    "Not provided"
)
```

### Add working experience

Insert a text label named `txtDetailsExperience` and set `Text` to:

```powerfx
"Working experience: " &
Coalesce(
    varApplication.'Working Experience',
    "Not provided"
)
```

Set `AutoHeight` to `true` or make the label tall enough for several lines.

### Add comments

Insert a text label named `txtDetailsComments` and set `Text` to:

```powerfx
"Comments: " &
Coalesce(
    varApplication.'Application Comments',
    "None"
)
```

Set `AutoHeight` to `true` or make the label tall enough for several lines.

If your table uses the Canvas-facing name `Applicant Comments` instead, select that field from the formula suggestions rather than guessing the name.

### Add status

Insert a text label named `txtDetailsStatus` and set `Text` to:

```powerfx
"Status: " &
If(
    IsBlank(varApplication.'Application Status'),
    "Unknown",
    Text(varApplication.'Application Status')
)
```

If Studio reports a type error for the status choice, use the formula suggested by Studio for converting that live choice field to display text. Do not replace it with a guessed numeric value.

### Add the Back button

Insert a button named `btnBackToSearch`.

Set `Text` to:

```powerfx
"Back to search"
```

Set `OnSelect` to:

```powerfx
Navigate(
    Screen1,
    ScreenTransition.Fade
)
```

Set `AccessibleLabel` to:

```powerfx
"Return to interview code search"
```

## Checkpoint 5 — Add the Word document button

On `scrApplicationDetails`:

1. Insert a button.
2. Rename it:

```text
btnOpenDocument
```

3. Set `Text` to:

```powerfx
"Open Word document"
```

4. Set `OnSelect` to:

```powerfx
Launch(
    varApplication.'Document URL'
)
```

5. Set `DisplayMode` to:

```powerfx
If(
    IsBlank(varApplication.'Document URL'),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Important URL rule

Do not construct the URL from the raw OneDrive path:

```text
/Documents/bcp-interview/InterviewApplication_123456.docx
```

That path can open a blank page. The document flow must store the `WebUrl` returned by OneDrive **Create share link**:

```powerautomate
@body('Create_share_link')?['WebUrl']
```

A working viewer URL normally contains a route similar to:

```text
/:w:/...
```

### Document button checkpoint

Use a completed test record and select **Open Word document**.

**Expected result:** the generated Word document opens in the authenticated Microsoft viewer.



## Checkpoint 6 — Add the print-summary screen

1. Create another blank screen.
2. Rename it:

```text
scrPrintSummary
```

3. Set its `Fill` to:

```powerfx
RGBA(255, 255, 255, 1)
```

Design this screen as a simple portrait document:

- Heading: Bosch Interview Application Summary
- Applicant name
- Interview code
- Email
- Phone
- Application status
- Working experience
- Comments

Use the same formulas from the details screen. Keep the background white and use dark text so the PDF is readable.

### Add the Print Summary button to the details screen

On `scrApplicationDetails`, insert a button named:

```text
btnPrintSummary
```

Set `Text` to:

```powerfx
"Print summary PDF"
```

Set `OnSelect` to:

```powerfx
Navigate(
    scrPrintSummary,
    ScreenTransition.None
)
```

### Add the Print button to the print screen

On `scrPrintSummary`, insert a button named:

```text
btnPrint
```

Set `Text` to:

```powerfx
"Print / Save as PDF"
```

Set `OnSelect` to:

```powerfx
Print()
```

Set `Visible` to:

```powerfx
Not(scrPrintSummary.Printing)
```

### Add the print-screen Back button

Insert a button named `btnBackFromPrint`.

Set `Text` to:

```powerfx
"Back"
```

Set `OnSelect` to:

```powerfx
Navigate(
    scrApplicationDetails,
    ScreenTransition.None
)
```

Set `Visible` to:

```powerfx
Not(scrPrintSummary.Printing)
```

### Print checkpoint

1. Preview the app from `Screen1`.
2. Search for a valid interview code.
3. Select **Print summary PDF**.
4. Select **Print / Save as PDF**.
5. In the browser print dialog, choose **Save as PDF**.
6. Save the PDF locally and open it.

Canvas `Print()` prints the Canvas summary screen. It does not print the original Word binary directly and does not silently select a physical printer.



## Checkpoint 7 — Improve the layout

Apply a consistent light operational design:

```text
Page background: RGBA(245, 246, 248, 1)
Card background: RGBA(255, 255, 255, 1)
Dark text: RGBA(35, 35, 35, 1)
Secondary text: RGBA(80, 80, 80, 1)
Bosch red accent: RGBA(198, 0, 0, 1)
```

Recommendations:

- Use a light background and white information cards.
- Use dark text with strong contrast.
- Make buttons at least 44 pixels high.
- Give long experience and comment fields enough height.
- Do not use a gallery showing every applicant.
- Disable the Word button when `Document URL` is blank.
- Keep the print screen uncluttered.
- Add accessible labels to the input and action buttons.

Preview every screen, not only the search screen. Check for overlapping controls, clipped long text, and unreadable colors.

## Checkpoint 8 — Save, test, and publish

### Save the draft

1. Select **Save** in Power Apps Studio.
2. Wait for the save confirmation.

### Run the complete Canvas test

Use a known completed application:

| Test | Expected result |
|---|---|
| Blank code | Validation warning |
| Five-digit code | Validation warning |
| Unknown six-digit code | Not-found message |
| Valid six-digit code | Correct applicant details |
| Open Word document | Word viewer URL opens |
| Print summary | Print screen opens |
| Save as PDF | Readable PDF is created |
| Missing Document URL | Open button is disabled |

### Publish

After Preview tests pass:

1. Select **Save** again.
2. Select **Publish**.
3. Confirm the publish dialog.
4. Open the published app using the **Play** link.
5. Repeat one valid-code search from the published version.

Remember:

```text
Save = stores the draft
Publish = makes the version available to users
```



## Checkpoint 9 — Share securely

Share the app only with authorized interviewers.

1. From the app details, select **Share**.
2. Add the interviewer security group or named interviewer accounts.
3. Confirm they have the required Dataverse read permission.
4. Confirm they have permission to open the OneDrive/SharePoint document.
5. Do not share the app with public applicants.
6. Do not use anonymous document links.

## Full solution verification

The complete business process is:

```text
Microsoft Forms
  → Forms intake flow
  → Dataverse Interview Applications row
  → six-digit Interview Code
  → Word document generation
  → OneDrive/SharePoint file
  → organization-scoped Word viewer URL
  → applicant email
  → Canvas exact-code lookup
  → Word viewer launch
  → Canvas summary Save as PDF
```

Submit a new test Form response and verify:

1. The Dataverse row is created.
2. The Interview Code is generated.
3. The status becomes `Document Created`.
4. The Word file exists.
5. `Document URL` is populated.
6. The applicant email is sent.
7. The Canvas App retrieves the correct applicant.
8. The Word viewer opens.
9. The Canvas summary can be saved as PDF.

## Common student mistakes

### The table is missing

Check the environment switcher and confirm that the table exists in the current Development environment. Do not create a duplicate table.

### The lookup returns no record

Check that the code is entered as six digits, including leading zeroes if applicable. Confirm the Dataverse column is text and use the Canvas-facing field name shown by Studio.

### The Word button opens a blank page

The app is probably using a raw `/Documents/...docx` path. Store and launch the `WebUrl` returned by OneDrive **Create share link**.

### The status formula shows a type error

`Application Status` is a Dataverse choice, not ordinary text. Use the formula suggestion generated by Studio for the live field. Do not guess the option-set number.

### Print does not print the Word file

That is expected. `Print()` prints the Canvas print-summary screen. Use **Open Word document** for the original Word file, or select **Save as PDF** from the Canvas summary print dialog.

### Back does not work in Preview

Start Preview from `Screen1`. Navigation history may not exist if Preview starts directly on another screen.

## Completion evidence

A student has completed this lab when all of these are true:

- The app has three screens with the required names.
- The Dataverse table is connected.
- A six-digit code performs an exact lookup.
- Invalid input produces a friendly message.
- Applicant details display correctly.
- The generated Word document opens.
- The Canvas summary saves as PDF.
- The app is saved and published.
- The published app has been tested with a real completed application.
- The app is shared only with authorized interviewers.
