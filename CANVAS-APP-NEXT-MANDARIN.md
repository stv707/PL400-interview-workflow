# 学生实验：创建面试官 Canvas App

## 实验目标

创建一个供内部面试官使用的 Tablet Canvas App，使面试官能够：

1. 输入申请人的六位数 Interview Code。
2. 从 Dataverse 中准确取得一条申请记录。
3. 查看申请人的资料。
4. 打开系统生成的 Word 文件。
5. 使用浏览器中的 **Save as PDF** 选项打印 Canvas 摘要。

申请人不使用此应用。本应用只提供给获得授权的面试官使用。

## 你将创建的内容

```text
Screen1
  使用六位数 Interview Code 搜索
       ↓
scrApplicationDetails
  显示申请人资料
  打开 Word 文件
  进入打印摘要页面
       ↓
scrPrintSummary
  打印 Canvas 摘要并保存为 PDF
```

## 前提条件

请先完成面试 workflow 中的 Microsoft Forms、Dataverse 和 Power Automate 部分。

你需要具备：

- 正确 Development environment 的访问权限。
- 包含 interview workflow 的 solution。
- 包含面试申请记录的 Dataverse table。
- 已经可以运行的 Microsoft Forms intake flow。
- 已经可以运行的 document-generation flow。
- 至少一条状态为 `Document Created` 的测试申请记录。
- 具备 Dataverse 读取权限以及 OneDrive/SharePoint 文档位置访问权限的面试官账号。

Dataverse table 应该在 Canvas 中提供以下 columns。你的 publisher prefix 可能不同，因此请使用 Power Apps Studio 中显示的名称：

| Column | 用途 |
|---|---|
| Applicant Name | 申请人全名 |
| Applicant Comments / Application Comments | 申请人备注 |
| Application Status | 处理状态 |
| Document File Name | 生成的 Word 文件名或路径 |
| Document URL | 已验证的 Word viewer URL |
| Email Address | 申请人邮箱 |
| Interview Code | 六位数搜索代码 |
| Phone Number | 申请人电话 |
| Working Experience | 工作经验回答 |

## Checkpoint 0 — 验证环境和后端

创建应用之前：

1. 打开 [https://make.powerapps.com](https://make.powerapps.com)。
2. 在 environment switcher 中选择指定的 **Development** environment。
3. 打开包含 interview workflow 的 solution。
4. 打开 Dataverse table，确认上面的 columns 存在。
5. 打开一条已经完成的 application，并确认：
   - `Interview Code` 包含六位数字。
   - `Application Status` 为 `Document Created`。
   - `Document URL` 已填充。
   - 该 URL 是 Word viewer URL，通常包含 `/:w:/`。

### 为什么这一步重要

Canvas App 即使创建成功，也可能连接到错误的 environment 或错误的 table。开始 authoring 前必须先确认目标环境。

### 学生检查点

在这里暂停，确认可以看到一条已经完成的 application 和它的六位数代码。

## Checkpoint 1 — 创建空白 Canvas App

1. 在 solution 中选择 **New**。
2. 选择 **App → Canvas app**。
3. 选择 **Tablet** layout。
4. 输入以下名称：

```text
Bosch Interview Canvas
```

5. 选择 **Create**。
6. 将应用保存到当前 solution。
7. 如果第一个 screen 不是 `Screen1`，将它重命名为：

```text
Screen1
```

### 学生检查点

确认应用在 Power Apps Studio 中打开，并且包含一个空白 screen。

## Checkpoint 2 — 添加 Dataverse table

1. 在左侧导航中选择 **Data**。
2. 选择 **Add data**。
3. 搜索 **Dataverse**。
4. 选择 `Interview Applications` table。
5. 等待 table 加载完成。
6. 展开 data source，确认 Interview Code 和 Document URL fields 可用。

如果 table 没有显示：

- 确认 environment switcher 选择正确。
- 确认 table 存在于当前 environment，或已经可以在 solution 中使用。
- 不要创建一个名称相似的 duplicate table。

### 学生检查点

在 Data pane 中选择该 table，确认它的 fields 可以在公式中使用。

## Checkpoint 3 — 准备搜索 screen

在 Tree view 中选择 `Screen1`，设置以下 properties：

```text
Name: Screen1
Fill: RGBA(245, 246, 248, 1)
```

选择 screen 的 **OnVisible** property，输入：

```powerfx
Set(varApplication, First('Interview Applications'));
Set(varSearchAttempted, false)
```

### 为什么使用 `First`？

这个公式会使用 Dataverse record type 初始化 variable。Search button 会用符合代码的 record 替换它。应用不会自动显示第一条记录。

### 添加页面标题

1. 选择 **Insert → Text label**。
2. 将名称改为：

```text
txtPageTitle
```

3. 将 `Text` property 设置为：

```powerfx
"Bosch Interviewer Lookup"
```

4. 将 `Size` 设置为大约 `24`。
5. 将 `FontWeight` 设置为 `FontWeight.Bold`。
6. 将 `Color` 设置为：

```powerfx
RGBA(35, 35, 35, 1)
```

### 添加说明文字

1. 再 Insert 一个 text label。
2. 将名称改为：

```text
txtInstructions
```

3. 将 `Text` 设置为：

```powerfx
"Enter the six-digit Interview Code provided to the applicant."
```

### 添加代码输入框

1. 选择 **Insert → Input → Text input**。
2. 将名称改为：

```text
txtInterviewCode
```

3. 设置以下 properties：

```text
HintText: For example, 675838
MaxLength: 6
AccessibleLabel: Six-digit interview code
```

如果使用 modern text input，并且该 property 可用，可以将 `Type` 设置为 numeric。请保留 `Text` property，因为 lookup formula 使用 `txtInterviewCode.Text`。

### 添加 Search button

1. Insert **Button**。
2. 将名称改为：

```text
btnSearch
```

3. 将 `Text` property 设置为：

```powerfx
"Search"
```

4. 将 `OnSelect` property 设置为：

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

### Search formula 的作用

- 设置一个 flag，表示用户已经执行搜索。
- 验证输入是否正好包含六位数字。
- 执行精确的 Dataverse `LookUp`。
- 将匹配的 record 保存到 `varApplication`。
- 只有找到 record 时才进行 navigation。

### 添加验证提示

1. Insert 一个 text label。
2. 将名称改为：

```text
txtSearchValidation
```

3. 将 `Text` 设置为：

```powerfx
"Use exactly six digits."
```

4. 将 `Visible` 设置为：

```powerfx
varSearchAttempted &&
!IsMatch(
    Trim(txtInterviewCode.Text),
    "^[0-9]{6}$"
)
```

5. 将 `Color` 设置为：

```powerfx
RGBA(198, 0, 0, 1)
```

### 搜索 screen 测试

选择 **Preview**，测试以下输入：

| 输入 | 预期结果 |
|---|---|
| 空白 | 显示 warning |
| 五位数字 | 显示 warning |
| 字母 | 显示 warning |
| 不存在的六位数代码 | 显示 not-found notification |
| 有效的 completed code | 在 details screen 创建后导航到该 screen |

在无效输入能够被正确拒绝之前，不要继续下一步。

## Checkpoint 4 — 创建 applicant details screen

1. 在 Tree view 中选择 **New screen**。
2. 选择 **Blank**。
3. 将 screen 重命名为：

```text
scrApplicationDetails
```

4. 将 `Fill` 设置为：

```powerfx
RGBA(245, 246, 248, 1)
```

### 添加申请人姓名

1. Insert 一个 text label。
2. 将名称改为：

```text
txtApplicantName
```

3. 将 `Text` 设置为：

```powerfx
Coalesce(
    varApplication.'Applicant Name',
    "Applicant"
)
```

将它设置为 screen 上最大的文字。

### 添加 interview code

Insert 一个名为 `txtDetailsCode` 的 text label，并将 `Text` 设置为：

```powerfx
"Interview code: " &
Coalesce(
    varApplication.'Interview Code',
    ""
)
```

### 添加邮箱

Insert 一个名为 `txtDetailsEmail` 的 text label，并将 `Text` 设置为：

```powerfx
"Email: " &
Coalesce(
    varApplication.'Email Address',
    "Not provided"
)
```

### 添加电话号码

Insert 一个名为 `txtDetailsPhone` 的 text label，并将 `Text` 设置为：

```powerfx
"Phone: " &
Coalesce(
    varApplication.'Phone Number',
    "Not provided"
)
```

### 添加工作经验

Insert 一个名为 `txtDetailsExperience` 的 text label，并将 `Text` 设置为：

```powerfx
"Working experience: " &
Coalesce(
    varApplication.'Working Experience',
    "Not provided"
)
```

将 `AutoHeight` 设置为 `true`，或者让 label 有足够高度显示多行文字。

### 添加备注

Insert 一个名为 `txtDetailsComments` 的 text label，并将 `Text` 设置为：

```powerfx
"Comments: " &
Coalesce(
    varApplication.'Application Comments',
    "None"
)
```

将 `AutoHeight` 设置为 `true`，或者让 label 有足够高度显示多行文字。

如果你的 table 使用 Canvas-facing name `Applicant Comments`，请从 formula suggestions 中选择该 field，不要猜测 field 名称。

### 添加状态

Insert 一个名为 `txtDetailsStatus` 的 text label，并将 `Text` 设置为：

```powerfx
"Status: " &
If(
    IsBlank(varApplication.'Application Status'),
    "Unknown",
    Text(varApplication.'Application Status')
)
```

如果 Studio 对 status choice 报告 type error，请使用 Studio 针对该 live choice field 建议的转换 display text 的公式。不要自行猜测 option-set number。

### 添加 Back button

Insert 一个名为 `btnBackToSearch` 的 button。

将 `Text` 设置为：

```powerfx
"Back to search"
```

将 `OnSelect` 设置为：

```powerfx
Navigate(
    Screen1,
    ScreenTransition.Fade
)
```

将 `AccessibleLabel` 设置为：

```powerfx
"Return to interview code search"
```

## Checkpoint 5 — 添加 Word document button

在 `scrApplicationDetails` 中：

1. Insert 一个 button。
2. 将名称改为：

```text
btnOpenDocument
```

3. 将 `Text` 设置为：

```powerfx
"Open Word document"
```

4. 将 `OnSelect` 设置为：

```powerfx
Launch(
    varApplication.'Document URL'
)
```

5. 将 `DisplayMode` 设置为：

```powerfx
If(
    IsBlank(varApplication.'Document URL'),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### 重要的 URL 规则

不要使用以下 raw OneDrive path 来构建 URL：

```text
/Documents/bcp-interview/InterviewApplication_123456.docx
```

这个 path 可能会打开 blank page。Document flow 必须保存 OneDrive **Create share link** 返回的 `WebUrl`：

```powerautomate
@body('Create_share_link')?['WebUrl']
```

有效的 viewer URL 通常包含类似以下的 route：

```text
/:w:/...
```

### Document button 测试

使用一条已完成的测试记录，选择 **Open Word document**。

**预期结果：** 生成的 Word document 在 authenticated Microsoft viewer 中打开。

## Checkpoint 6 — 添加 print-summary screen

1. 创建另一个 blank screen。
2. 将名称改为：

```text
scrPrintSummary
```

3. 将 `Fill` 设置为：

```powerfx
RGBA(255, 255, 255, 1)
```

将这个 screen 设计成简洁的 portrait document：

- Heading: Bosch Interview Application Summary
- 申请人姓名
- Interview code
- Email
- 电话
- Application status
- Working experience
- Comments

使用与 details screen 相同的公式。保持白色背景和深色文字，让 PDF 易于阅读。

### 在 details screen 添加 Print Summary button

在 `scrApplicationDetails` 中 Insert 一个名为以下名称的 button：

```text
btnPrintSummary
```

将 `Text` 设置为：

```powerfx
"Print summary PDF"
```

将 `OnSelect` 设置为：

```powerfx
Navigate(
    scrPrintSummary,
    ScreenTransition.None
)
```

### 在 print screen 添加 Print button

在 `scrPrintSummary` 中 Insert 一个名为以下名称的 button：

```text
btnPrint
```

将 `Text` 设置为：

```powerfx
"Print / Save as PDF"
```

将 `OnSelect` 设置为：

```powerfx
Print()
```

将 `Visible` 设置为：

```powerfx
Not(scrPrintSummary.Printing)
```

### 在 print screen 添加 Back button

Insert 一个名为 `btnBackFromPrint` 的 button。

将 `Text` 设置为：

```powerfx
"Back"
```

将 `OnSelect` 设置为：

```powerfx
Navigate(
    scrApplicationDetails,
    ScreenTransition.None
)
```

将 `Visible` 设置为：

```powerfx
Not(scrPrintSummary.Printing)
```

### 打印测试

1. 从 `Screen1` 开始 Preview app。
2. 使用有效的 interview code 搜索。
3. 选择 **Print summary PDF**。
4. 选择 **Print / Save as PDF**。
5. 在 browser print dialog 中选择 **Save as PDF**。
6. 将 PDF 保存到本地并打开文件。

Canvas `Print()` 打印的是 Canvas summary screen。它不会直接打印原始 Word binary，也不会静默选择 physical printer。

## Checkpoint 7 — 改善 layout

使用一致的浅色 operational design：

```text
Page background: RGBA(245, 246, 248, 1)
Card background: RGBA(255, 255, 255, 1)
Dark text: RGBA(35, 35, 35, 1)
Secondary text: RGBA(80, 80, 80, 1)
Bosch red accent: RGBA(198, 0, 0, 1)
```

建议：

- 使用浅色背景和白色 information cards。
- 使用具有良好 contrast 的深色文字。
- Button 高度至少设置为 44 pixels。
- 为较长的工作经验和备注内容提供足够高度。
- 不要使用显示所有申请人的 gallery。
- 当 `Document URL` 为空时 disable Word button。
- 保持 print screen 简洁，不要放置过多内容。
- 为 input 和 action buttons 添加 accessible labels。

Preview 每一个 screen，不只是 search screen。检查 controls 是否重叠、长文字是否被截断，以及颜色是否清晰可读。

## Checkpoint 8 — Save、test 和 publish

### Save draft

1. 在 Power Apps Studio 中选择 **Save**。
2. 等待 save confirmation 出现。

### 执行完整 Canvas 测试

使用一条已知的 completed application：

| 测试 | 预期结果 |
|---|---|
| 空白 code | Validation warning |
| 五位数 code | Validation warning |
| 不存在的六位数 code | Not-found message |
| 有效的六位数 code | 显示正确的申请人资料 |
| Open Word document | Word viewer URL 打开 |
| Print summary | 打印 screen 打开 |
| Save as PDF | 成功创建可读 PDF |
| Document URL 为空 | Open button disabled |

### Publish

Preview 测试通过后：

1. 再次选择 **Save**。
2. 选择 **Publish**。
3. 确认 publish dialog。
4. 使用 **Play** link 打开 published app。
5. 在 published version 中重新进行一次 valid-code search。

请记住：

```text
Save = 保存 draft
Publish = 让 version 对用户可用
```

## Checkpoint 9 — 安全地 Share app

只与获得授权的面试官分享 app。

1. 从 app details 中选择 **Share**。
2. 添加 interviewer security group 或指定的面试官账号。
3. 确认他们拥有所需的 Dataverse read permission。
4. 确认他们有权限打开 OneDrive/SharePoint 文档。
5. 不要把 app 分享给公共申请人。
6. 不要使用 anonymous document links。

## 完整 solution verification

完整 business process 如下：

```text
Microsoft Forms
  → Forms intake flow
  → Dataverse Interview Applications row
  → 六位数 Interview Code
  → Word document generation
  → OneDrive/SharePoint file
  → organization-scoped Word viewer URL
  → applicant email
  → Canvas exact-code lookup
  → Word viewer launch
  → Canvas summary Save as PDF
```

提交一条新的 test Form response，并确认：

1. Dataverse row 已创建。
2. Interview Code 已生成。
3. Status 变为 `Document Created`。
4. Word file 存在。
5. `Document URL` 已填充。
6. Applicant email 已发送。
7. Canvas App 取得正确的申请人。
8. Word viewer 可以打开。
9. Canvas summary 可以保存为 PDF。

## 常见的学生错误

### 找不到 table

检查 environment switcher，并确认 table 存在于当前 Development environment。不要创建 duplicate table。

### Lookup 找不到 record

确认输入的是六位数字，包括可能存在的 leading zero。确认 Dataverse column 是 text，并使用 Studio 显示的 Canvas-facing field name。

### Word button 打开 blank page

App 可能使用了 raw `/Documents/...docx` path。请保存并 launch OneDrive **Create share link** 返回的 `WebUrl`。

### Status formula 出现 type error

`Application Status` 是 Dataverse choice，不是普通 text。使用 Studio 为当前 live field 生成的 formula suggestion。不要猜 option-set number。

### Print 没有打印 Word file

这是预期行为。`Print()` 打印的是 Canvas print-summary screen。需要原始 Word file 时使用 **Open Word document**，或者在 Canvas summary print dialog 中选择 **Save as PDF**。

### Preview 中 Back 不工作

从 `Screen1` 开始 Preview。若 Preview 直接从其他 screen 开始，可能没有 navigation history。

## 完成标准

当以下所有条件都满足时，学生完成本实验：

- App 具有所需的三个 screens。
- Dataverse table 已连接。
- 六位数 code 可以执行 exact lookup。
- 无效输入会显示适当提示。
- 申请人资料显示正确。
- Generated Word document 可以打开。
- Canvas summary 可以保存为 PDF。
- App 已保存并 publish。
- Published app 已使用真实的 completed application 进行测试。
- App 只分享给获得授权的面试官。
