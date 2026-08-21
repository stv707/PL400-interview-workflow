# Lab Pelajar: Bina Canvas App untuk Penemuduga

## Objektif lab

Bina sebuah tablet Canvas App dalaman yang membolehkan penemuduga:

1. Memasukkan Interview Code enam digit milik pemohon.
2. Mendapatkan satu rekod permohonan daripada Dataverse.
3. Menyemak maklumat pemohon.
4. Membuka dokumen Word yang telah dijana.
5. Mencetak ringkasan Canvas menggunakan pilihan **Save as PDF** dalam browser.

Pemohon tidak menggunakan aplikasi ini. Aplikasi ini hanya untuk penemuduga yang diberi kuasa.

## Apa yang akan dibina

```text
Screen1
  Carian menggunakan Interview Code enam digit
       ↓
scrApplicationDetails
  Paparkan maklumat pemohon
  Buka dokumen Word
  Pergi ke ringkasan cetakan
       ↓
scrPrintSummary
  Cetak ringkasan Canvas dan simpan sebagai PDF
```

## Prasyarat

Lengkapkan bahagian Microsoft Forms, Dataverse, dan Power Automate bagi workflow interview terlebih dahulu.

Anda memerlukan:

- Akses kepada Development environment yang betul.
- Solution yang mengandungi interview workflow.
- Jadual Dataverse yang mengandungi rekod permohonan interview.
- Microsoft Forms intake flow yang berfungsi.
- Document-generation flow yang berfungsi.
- Sekurang-kurangnya satu test application dengan status `Document Created`.
- Akaun penemuduga yang mempunyai read access kepada Dataverse dan lokasi dokumen OneDrive/SharePoint.

Jadual Dataverse perlu mempunyai column berikut pada Canvas. Publisher prefix mungkin berbeza, jadi gunakan nama yang dipaparkan oleh Power Apps Studio:

| Column | Tujuan |
|---|---|
| Applicant Name | Nama penuh pemohon |
| Applicant Comments / Application Comments | Komen pemohon |
| Application Status | Status pemprosesan |
| Document File Name | Nama/path fail Word yang dijana |
| Document URL | URL Word viewer yang disahkan |
| Email Address | Email pemohon |
| Interview Code | Kod carian enam digit |
| Phone Number | Telefon pemohon |
| Working Experience | Jawapan pengalaman kerja |

## Checkpoint 0 — Sahkan environment dan backend

Sebelum mencipta app:

1. Buka [https://make.powerapps.com](https://make.powerapps.com).
2. Pilih **Development** environment yang dimaksudkan pada environment switcher.
3. Buka solution yang mengandungi interview workflow.
4. Buka jadual Dataverse dan sahkan column di atas wujud.
5. Buka satu application yang telah selesai dan sahkan:
   - `Interview Code` mengandungi enam digit.
   - `Application Status` ialah `Document Created`.
   - `Document URL` telah diisi.
   - URL tersebut ialah Word viewer URL, biasanya mengandungi `/:w:/`.

### Mengapa langkah ini penting

Canvas App boleh dibina dengan berjaya tetapi masih menunjuk kepada environment atau jadual yang salah. Sentiasa sahkan target sebelum membuat authoring.

### Semakan pelajar

Berhenti di sini dan sahkan bahawa satu application yang telah selesai serta kod enam digitnya boleh dilihat.

## Checkpoint 1 — Cipta Canvas App kosong

1. Daripada solution, pilih **New**.
2. Pilih **App → Canvas app**.
3. Pilih layout **Tablet**.
4. Masukkan nama berikut:

```text
Bosch Interview Canvas
```

5. Pilih **Create**.
6. Simpan app dalam solution semasa.
7. Jika screen pertama bukan bernama `Screen1`, tukarkan namanya kepada:

```text
Screen1
```

### Semakan pelajar

Sahkan bahawa app dibuka dalam Power Apps Studio dan mengandungi satu screen kosong.

## Checkpoint 2 — Tambah Dataverse table

1. Dalam left navigation, pilih **Data**.
2. Pilih **Add data**.
3. Cari **Dataverse**.
4. Pilih jadual `Interview Applications`.
5. Tunggu sehingga table selesai dimuatkan.
6. Expand data source dan sahkan field Interview Code serta Document URL tersedia.

Jika table tidak disenaraikan:

- Sahkan environment switcher adalah betul.
- Sahkan table wujud dalam environment semasa atau tersedia dalam solution.
- Jangan cipta table kedua dengan nama yang hampir sama.

### Semakan pelajar

Pilih table dalam Data pane dan sahkan field-nya boleh digunakan dalam formula.

## Checkpoint 3 — Sediakan search screen

Pilih `Screen1` dalam Tree view. Tetapkan properties berikut:

```text
Name: Screen1
Fill: RGBA(245, 246, 248, 1)
```

Pilih property **OnVisible** untuk screen dan masukkan:

```powerfx
Set(varApplication, First('Interview Applications'));
Set(varSearchAttempted, false)
```

### Mengapa menggunakan `First`?

Formula ini initialize variable dengan Dataverse record type. Search button akan menggantikannya dengan record yang sepadan dengan kod. App tidak akan memaparkan first record secara automatik.

### Tambah page title

1. Pilih **Insert → Text label**.
2. Tukar nama kepada:

```text
txtPageTitle
```

3. Tetapkan property `Text` kepada:

```powerfx
"Bosch Interviewer Lookup"
```

4. Tetapkan `Size` kepada kira-kira `24`.
5. Tetapkan `FontWeight` kepada `FontWeight.Bold`.
6. Tetapkan `Color` kepada:

```powerfx
RGBA(35, 35, 35, 1)
```

### Tambah arahan kepada pengguna

1. Insert satu lagi text label.
2. Tukar nama kepada:

```text
txtInstructions
```

3. Tetapkan `Text` kepada:

```powerfx
"Enter the six-digit Interview Code provided to the applicant."
```

### Tambah code input

1. Pilih **Insert → Input → Text input**.
2. Tukar nama kepada:

```text
txtInterviewCode
```

3. Tetapkan properties berikut:

```text
HintText: For example, 675838
MaxLength: 6
AccessibleLabel: Six-digit interview code
```

Jika menggunakan modern text input, tetapkan `Type` kepada numeric jika property tersebut tersedia. Pastikan property `Text` kekal tersedia kerana formula lookup menggunakan `txtInterviewCode.Text`.

### Tambah Search button

1. Insert **Button**.
2. Tukar nama kepada:

```text
btnSearch
```

3. Tetapkan property `Text` kepada:

```powerfx
"Search"
```

4. Tetapkan property `OnSelect` kepada:

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

### Apa yang dilakukan oleh Search formula

- Menetapkan flag bahawa carian telah dibuat.
- Mengesahkan input mempunyai tepat enam digit.
- Menjalankan exact Dataverse `LookUp`.
- Menyimpan record yang sepadan dalam `varApplication`.
- Navigate hanya apabila record ditemui.

### Tambah validation message

1. Insert satu text label.
2. Tukar nama kepada:

```text
txtSearchValidation
```

3. Tetapkan `Text` kepada:

```powerfx
"Use exactly six digits."
```

4. Tetapkan `Visible` kepada:

```powerfx
varSearchAttempted &&
!IsMatch(
    Trim(txtInterviewCode.Text),
    "^[0-9]{6}$"
)
```

5. Tetapkan `Color` kepada:

```powerfx
RGBA(198, 0, 0, 1)
```

### Ujian checkpoint search screen

Pilih **Preview** dan uji nilai berikut:

| Input | Hasil yang dijangka |
|---|---|
| Kosong | Warning dipaparkan |
| Lima digit | Warning dipaparkan |
| Huruf | Warning dipaparkan |
| Kod enam digit yang tidak wujud | Not-found notification |
| Kod completed yang betul | Akan navigate ke details screen selepas screen tersebut dicipta |

Jangan teruskan sehingga input tidak sah ditolak dengan betul.

## Checkpoint 4 — Cipta applicant details screen

1. Dalam Tree view, pilih **New screen**.
2. Pilih **Blank**.
3. Tukar nama screen kepada:

```text
scrApplicationDetails
```

4. Tetapkan `Fill` kepada:

```powerfx
RGBA(245, 246, 248, 1)
```

### Tambah nama pemohon

1. Insert satu text label.
2. Tukar nama kepada:

```text
txtApplicantName
```

3. Tetapkan `Text` kepada:

```powerfx
Coalesce(
    varApplication.'Applicant Name',
    "Applicant"
)
```

Jadikan ini sebagai teks terbesar pada screen.

### Tambah interview code

Insert text label bernama `txtDetailsCode` dan tetapkan `Text` kepada:

```powerfx
"Interview code: " &
Coalesce(
    varApplication.'Interview Code',
    ""
)
```

### Tambah email

Insert text label bernama `txtDetailsEmail` dan tetapkan `Text` kepada:

```powerfx
"Email: " &
Coalesce(
    varApplication.'Email Address',
    "Not provided"
)
```

### Tambah nombor telefon

Insert text label bernama `txtDetailsPhone` dan tetapkan `Text` kepada:

```powerfx
"Phone: " &
Coalesce(
    varApplication.'Phone Number',
    "Not provided"
)
```

### Tambah pengalaman kerja

Insert text label bernama `txtDetailsExperience` dan tetapkan `Text` kepada:

```powerfx
"Working experience: " &
Coalesce(
    varApplication.'Working Experience',
    "Not provided"
)
```

Tetapkan `AutoHeight` kepada `true` atau jadikan label cukup tinggi untuk beberapa baris teks.

### Tambah komen

Insert text label bernama `txtDetailsComments` dan tetapkan `Text` kepada:

```powerfx
"Comments: " &
Coalesce(
    varApplication.'Application Comments',
    "None"
)
```

Tetapkan `AutoHeight` kepada `true` atau jadikan label cukup tinggi untuk beberapa baris teks.

Jika table anda menggunakan Canvas-facing name `Applicant Comments`, pilih field tersebut daripada formula suggestions dan jangan meneka nama field.

### Tambah status

Insert text label bernama `txtDetailsStatus` dan tetapkan `Text` kepada:

```powerfx
"Status: " &
If(
    IsBlank(varApplication.'Application Status'),
    "Unknown",
    Text(varApplication.'Application Status')
)
```

Jika Studio melaporkan type error untuk status choice, gunakan formula yang dicadangkan oleh Studio untuk menukar live choice field kepada display text. Jangan gantikan dengan nombor option-set yang diteka.

### Tambah Back button

Insert button bernama `btnBackToSearch`.

Tetapkan `Text` kepada:

```powerfx
"Back to search"
```

Tetapkan `OnSelect` kepada:

```powerfx
Navigate(
    Screen1,
    ScreenTransition.Fade
)
```

Tetapkan `AccessibleLabel` kepada:

```powerfx
"Return to interview code search"
```

## Checkpoint 5 — Tambah Word document button

Pada `scrApplicationDetails`:

1. Insert satu button.
2. Tukar nama kepada:

```text
btnOpenDocument
```

3. Tetapkan `Text` kepada:

```powerfx
"Open Word document"
```

4. Tetapkan `OnSelect` kepada:

```powerfx
Launch(
    varApplication.'Document URL'
)
```

5. Tetapkan `DisplayMode` kepada:

```powerfx
If(
    IsBlank(varApplication.'Document URL'),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Peraturan penting URL

Jangan bina URL daripada raw OneDrive path berikut:

```text
/Documents/bcp-interview/InterviewApplication_123456.docx
```

Path tersebut boleh membuka blank page. Document flow mesti menyimpan `WebUrl` yang dikembalikan oleh OneDrive **Create share link**:

```powerautomate
@body('Create_share_link')?['WebUrl']
```

Viewer URL yang berfungsi biasanya mengandungi route seperti:

```text
/:w:/...
```

### Ujian document button

Gunakan completed test record dan pilih **Open Word document**.

**Hasil yang dijangka:** generated Word document terbuka dalam authenticated Microsoft viewer.

## Checkpoint 6 — Tambah print-summary screen

1. Cipta satu lagi blank screen.
2. Tukar nama kepada:

```text
scrPrintSummary
```

3. Tetapkan `Fill` kepada:

```powerfx
RGBA(255, 255, 255, 1)
```

Reka screen ini seperti dokumen portrait yang ringkas:

- Heading: Bosch Interview Application Summary
- Nama pemohon
- Interview code
- Email
- Telefon
- Application status
- Working experience
- Komen

Gunakan formula yang sama seperti details screen. Pastikan background berwarna putih dan teks gelap supaya PDF mudah dibaca.

### Tambah Print Summary button pada details screen

Pada `scrApplicationDetails`, insert button bernama:

```text
btnPrintSummary
```

Tetapkan `Text` kepada:

```powerfx
"Print summary PDF"
```

Tetapkan `OnSelect` kepada:

```powerfx
Navigate(
    scrPrintSummary,
    ScreenTransition.None
)
```

### Tambah Print button pada print screen

Pada `scrPrintSummary`, insert button bernama:

```text
btnPrint
```

Tetapkan `Text` kepada:

```powerfx
"Print / Save as PDF"
```

Tetapkan `OnSelect` kepada:

```powerfx
Print()
```

Tetapkan `Visible` kepada:

```powerfx
Not(scrPrintSummary.Printing)
```

### Tambah Back button pada print screen

Insert button bernama `btnBackFromPrint`.

Tetapkan `Text` kepada:

```powerfx
"Back"
```

Tetapkan `OnSelect` kepada:

```powerfx
Navigate(
    scrApplicationDetails,
    ScreenTransition.None
)
```

Tetapkan `Visible` kepada:

```powerfx
Not(scrPrintSummary.Printing)
```

### Ujian cetakan

1. Preview app bermula daripada `Screen1`.
2. Cari menggunakan interview code yang sah.
3. Pilih **Print summary PDF**.
4. Pilih **Print / Save as PDF**.
5. Dalam browser print dialog, pilih **Save as PDF**.
6. Simpan PDF secara local dan buka fail tersebut.

Canvas `Print()` mencetak Canvas summary screen. Ia tidak mencetak Word binary asal secara terus dan tidak memilih physical printer secara senyap.

## Checkpoint 7 — Kemaskan layout

Gunakan design operational yang konsisten dan cerah:

```text
Page background: RGBA(245, 246, 248, 1)
Card background: RGBA(255, 255, 255, 1)
Dark text: RGBA(35, 35, 35, 1)
Secondary text: RGBA(80, 80, 80, 1)
Bosch red accent: RGBA(198, 0, 0, 1)
```

Cadangan:

- Gunakan background cerah dan white information cards.
- Gunakan teks gelap dengan contrast yang baik.
- Jadikan button sekurang-kurangnya 44 pixels tinggi.
- Berikan ruang tinggi yang mencukupi untuk pengalaman kerja dan komen yang panjang.
- Jangan gunakan gallery yang memaparkan semua pemohon.
- Disable Word button apabila `Document URL` kosong.
- Pastikan print screen ringkas dan tidak berserabut.
- Tambah accessible labels kepada input dan action buttons.

Preview setiap screen, bukan search screen sahaja. Periksa overlapping controls, teks panjang yang terpotong, dan warna yang sukar dibaca.

## Checkpoint 8 — Save, test, dan publish

### Save draft

1. Pilih **Save** dalam Power Apps Studio.
2. Tunggu sehingga save confirmation dipaparkan.

### Jalankan Canvas test lengkap

Gunakan completed application yang diketahui:

| Ujian | Hasil yang dijangka |
|---|---|
| Code kosong | Validation warning |
| Code lima digit | Validation warning |
| Code enam digit yang tidak wujud | Not-found message |
| Code enam digit yang sah | Maklumat pemohon yang betul |
| Open Word document | Word viewer URL terbuka |
| Print summary | Print screen terbuka |
| Save as PDF | PDF yang boleh dibaca berjaya dicipta |
| Document URL kosong | Open button disabled |

### Publish

Selepas ujian Preview berjaya:

1. Pilih **Save** sekali lagi.
2. Pilih **Publish**.
3. Sahkan publish dialog.
4. Buka published app menggunakan pautan **Play**.
5. Ulang satu valid-code search daripada published version.

Ingat:

```text
Save = menyimpan draft
Publish = menjadikan version tersedia kepada pengguna
```

## Checkpoint 9 — Share dengan selamat

Share app hanya dengan penemuduga yang diberi kuasa.

1. Daripada app details, pilih **Share**.
2. Tambah interviewer security group atau akaun penemuduga tertentu.
3. Sahkan mereka mempunyai Dataverse read permission yang diperlukan.
4. Sahkan mereka mempunyai permission untuk membuka dokumen OneDrive/SharePoint.
5. Jangan share app dengan pemohon awam.
6. Jangan gunakan anonymous document links.

## Full solution verification

Business process lengkap ialah:

```text
Microsoft Forms
  → Forms intake flow
  → Dataverse Interview Applications row
  → Interview Code enam digit
  → Word document generation
  → OneDrive/SharePoint file
  → organization-scoped Word viewer URL
  → applicant email
  → Canvas exact-code lookup
  → Word viewer launch
  → Canvas summary Save as PDF
```

Submit satu test Form response baharu dan sahkan:

1. Dataverse row dicipta.
2. Interview Code dijana.
3. Status berubah kepada `Document Created`.
4. Word file wujud.
5. `Document URL` telah diisi.
6. Applicant email dihantar.
7. Canvas App mendapatkan pemohon yang betul.
8. Word viewer terbuka.
9. Canvas summary boleh disimpan sebagai PDF.

## Kesilapan pelajar yang biasa berlaku

### Table tidak ditemui

Periksa environment switcher dan sahkan table wujud dalam Development environment semasa. Jangan cipta duplicate table.

### Lookup tidak menjumpai record

Pastikan code dimasukkan sebagai enam digit, termasuk leading zero jika ada. Sahkan Dataverse column ialah text dan gunakan Canvas-facing field name yang dipaparkan oleh Studio.

### Word button membuka blank page

App mungkin menggunakan raw `/Documents/...docx` path. Simpan dan launch `WebUrl` yang dikembalikan oleh OneDrive **Create share link**.

### Status formula menunjukkan type error

`Application Status` ialah Dataverse choice, bukan text biasa. Gunakan formula suggestion yang dijana oleh Studio untuk live field tersebut. Jangan teka option-set number.

### Print tidak mencetak Word file

Ini adalah behavior yang dijangka. `Print()` mencetak Canvas print-summary screen. Gunakan **Open Word document** untuk Word file asal, atau pilih **Save as PDF** daripada Canvas summary print dialog.

### Back tidak berfungsi dalam Preview

Mulakan Preview daripada `Screen1`. Navigation history mungkin tidak wujud jika Preview bermula terus pada screen lain.

## Completion evidence

Pelajar telah menyelesaikan lab apabila semua perkara berikut benar:

- App mempunyai tiga screen yang diperlukan.
- Dataverse table telah disambungkan.
- Code enam digit menjalankan exact lookup.
- Input tidak sah menghasilkan mesej yang sesuai.
- Maklumat pemohon dipaparkan dengan betul.
- Generated Word document boleh dibuka.
- Canvas summary boleh disimpan sebagai PDF.
- App telah disimpan dan dipublish.
- Published app telah diuji dengan completed application sebenar.
- App hanya dikongsi dengan penemuduga yang diberi kuasa.
