# 03 — Data Design

## 1. Input Data

سيستقبل النظام طلبات الإجازة من Google Forms.

تتضمن بيانات النموذج الحقول التالية:

| Field | Purpose | Required |
|---|---|---|
| employee_name | اسم الموظف | Yes |
| email | البريد الإلكتروني للموظف | Yes |
| department | القسم الذي يعمل فيه الموظف | No |
| request_text | تفاصيل طلب الإجازة بكلام الموظف | Yes |

قد تكون بعض المعلومات غير موجودة في الطلب، أو قد لا تكون واضحة بما يكفي لاتخاذ القرار المناسب.

## 2. Unstructured Data

يمثل `request_text` نص الطلب الذي يكتبه الموظف بطريقته الخاصة.

مثال:

"I would like to request annual leave from December 20 to December 24 to travel with my family."

هذا النص يحتوي على معلومات مهمة، لكنها ليست منظمة في حقول منفصلة.

## 3. Structured Data

سيستخدم AI by Zapier لاستخراج المعلومات التالية من الطلب:

| Field | Description |
|---|---|
| leave_type | نوع الإجازة التي يطلبها الموظف |
| start_date | تاريخ بداية الإجازة كما ذكره الموظف |
| end_date | تاريخ نهاية الإجازة كما ذكره الموظف |
| request_summary | ملخص واضح لطلب الموظف |
| missing_information | المعلومات المطلوبة التي لم تتوفر أو لم تتضح |
| request_status | `VALID` أو `NEEDS_REVIEW` وفق القواعد |

### Allowed leave_type Values

يجب أن تكون قيمة `leave_type` واحدة من القيم التالية:

- `annual`
- `sick`
- `emergency`
- `unpaid`
- `other`
- `unknown`

تُستخدم `other` عندما يذكر الموظف نوعًا واضحًا لا ينتمي إلى الفئات المحددة (مثل إجازة دراسية أو إجازة زواج).

وتُستخدم `unknown` عندما لا يمكن تحديد النوع من النص (مثل: "I need some time off").

## 4. Missing Information

سنستخدم القيم التالية للتمييز بين الحالات:

- `null`: المعلومة لم يذكرها الموظف صراحةً (تاريخ البداية أو النهاية).
- `unknown`: لم يتمكن النظام من تحديد نوع الإجازة.
- `other`: النوع مفهوم، لكنه خارج الفئات المحددة.
- `NONE`: لا توجد معلومات ناقصة ضمن الحقول التي جرى التحقق منها.

لا تعني هذه القيم الشيء نفسه، ويجب استخدامها وفق معناها المحدد.

### ما هو التاريخ الصريح؟

| النص | النتيجة |
|---|---|
| "from December 20 to December 24" | start = December 20, end = December 24 |
| "on December 20 only" | start = end = December 20 |
| "next week" / "tomorrow" / "after the holidays" | `null` (تعبير نسبي غير محدد) |
| "for 3 days" بدون تاريخ بداية | `null` للتاريخين |
| "from December 20 for 3 days" | start = December 20, end = `null` |

لا يحسب النظام تاريخ النهاية ولا يخمّن السنة ولا يحوّل تعبيرًا نسبيًا إلى تاريخ.

## 5. Processing Results

بعد التحقق من المعلومات وتحديد مسار الطلب، سيضيف النظام النتائج التالية إلى سجل الطلب:

| Field | Purpose | Example |
|---|---|---|
| request_status | حالة التحقق من الطلب | VALID |
| routing | المسار الذي حُدد للطلب | manager_approval |
| manager_decision | قرار المدير | Approved |
| processed_at | وقت معالجة الطلب | Timestamp |
| decided_at | وقت قرار المدير | Timestamp |

القيم المسموح بها:

**request_status**
- `VALID`
- `NEEDS_REVIEW`

**routing**
- `manager_approval`
- `clarification`

**manager_decision**
- `Pending`: الطلب أُرسل إلى المدير ولم يقرر بعد.
- `Approved`
- `Rejected`
- `Not applicable`: الطلب لم يصل إلى المدير لأنه يحتاج إلى استكمال.

## 6. Google Sheets Structure

سيحتفظ Google Sheets ببيانات النموذج الأصلية، مع نتائج المعالجة وقرار المدير.

الأعمدة:

- employee_name
- email
- department
- request_text
- leave_type
- start_date
- end_date
- request_summary
- missing_information
- request_status
- routing
- manager_decision
- processed_at
- decided_at

يمر كل طلب في مسار واحد فقط، فيُسجَّل مرة واحدة. وعند صدور قرار المدير يُحدَّث السجل نفسه بدلًا من إنشاء سجل جديد.

## Expected Outcome

تحديد شكل البيانات التي تدخل إلى النظام، والمعلومات التي يستخرجها AI، والنتائج وقرار المدير التي يجب حفظها بعد المعالجة.
