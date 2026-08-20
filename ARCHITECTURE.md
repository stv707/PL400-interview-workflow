# Architecture and Design Notes

## Business requirements

| Requirement | Implementation |
|---|---|
| External applicant intake | Microsoft Forms |
| Durable system of record | Dataverse Interview Application table |
| Unique applicant reference | Six-digit Interview Code |
| Document generation | Word Online (Business) template |
| Document storage | OneDrive for Business for the lab; SharePoint recommended for production |
| Applicant notification | Office 365 Outlook |
| Interviewer retrieval | Canvas App exact code lookup |
| Processing visibility | Application Status choice |
| Failure diagnostics | Error Details column and Catch scope |

## Data ownership

Dataverse owns the application record and workflow state. Microsoft Forms is only the intake surface. OneDrive owns the generated document for this exercise. The Dataverse row stores the document path or URL so the application and artifact remain associated.

## Tables and relationships

The MVP uses one custom table:

```text
Interview Application
```

A future production design may separate:

- Applicant
- Job Position
- Interview Application
- Interview Session
- Interviewer
- Interview Document

The single-table design is intentional for a time-boxed PL-400 demonstration.

## Six-digit code design

The classroom MVP uses:

```powerautomate
string(rand(100000,999999))
```

This demonstrates expressions and data transformation, but random values are not collision-proof. A production design should use one of these approaches:

1. Dataverse autonumber with a six-digit format.
2. A dedicated sequence table with concurrency control.
3. Random generation plus a Dataverse uniqueness check and retry loop.
4. A longer reference such as `BOS-100001` to increase the addressable range.

The Dataverse GUID remains the technical primary identity. Interview Code is only an applicant-facing lookup key.

## Flow separation

Two flows are used instead of one large flow:

### Flow 1 — Capture Form Submission

Responsibilities:

- Receive the Forms event.
- Retrieve response details.
- Validate or deduplicate the response.
- Create the Dataverse application row.
- Generate the interview code.

### Flow 2 — Generate Document and Notify

Responsibilities:

- React to a new Dataverse application.
- Populate the Word template.
- Create the OneDrive document.
- Update the Dataverse row.
- Send the applicant notification.

This separation means a document failure does not lose the original application.

## Word binary handling

A Word template action returns the generated document content. The OneDrive Create file action must receive that generated binary output.

Correct:

```powerautomate
@body('Populate_a_Microsoft_Word_template')
```

Incorrect:

```powerautomate
@triggerOutputs()?['body/crf96_applicant_name']
```

The incorrect expression creates a file with a `.docx` extension but text content, so Word reports the file as corrupt.

## Document content controls

The Word connector requires native content controls. The recommended template exposes six separate fields. If the connector exposes one generic field such as `sdt`, the template controls are not being recognized distinctly and the same value may appear in every location.

## Status lifecycle

```text
Received
   |
   v
Document Created
   |
   v
Interview Completed
```

Failure path:

```text
Received → Processing Failed
```

The flow should never silently discard an application.

## Storage recommendation

OneDrive is acceptable for the student lab because it makes the connector setup simple. For a production recruitment solution, use a SharePoint document library because it provides:

- Team ownership.
- Central permissions.
- Retention and compliance features.
- Better continuity when the original flow owner leaves.
- Easier auditing and document management.

## Security model

### Applicant

- Microsoft Forms only.
- No Dataverse license or table permission.
- Receives only the interview code by email.

### Interviewer

- Canvas App access.
- Read access to application data.
- Read access to the document location.
- Optional write access to interview outcome columns.

### Administrator/Maker

- Solution and environment administration.
- Flow ownership and connection management.

## Canvas App contract

The Canvas App should:

1. Accept a six-digit code.
2. Validate that the input contains six digits.
3. Perform an exact Dataverse lookup.
4. Display the application information.
5. Open the associated document using a secured URL or controlled flow.

Example Power Fx pattern when `Interview Code` is stored as text:

```powerfx
Set(
    varApplication,
    LookUp(
        'Interview Applications',
        'Interview Code' = txtInterviewCode.Text
    )
)
```

Do not use an unfiltered gallery exposing every applicant to every interviewer.

## ALM notes

Use a solution containing:

- Dataverse table.
- Choice column.
- Canvas App.
- Both cloud flows.
- Connection references.
- Environment variables.
- Optional Word template reference documentation.

The Microsoft Form itself may need to be recreated or reselected per environment. Form IDs are environment- and tenant-specific and should not be hard-coded into public documentation.
