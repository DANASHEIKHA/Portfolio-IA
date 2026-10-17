# 🤖 AI Leave Request & Manager Approval System

## U1 — Process & Data Thinking

مشروع عملي لتحويل ما تعلمناه في **U1 — Process & Data Thinking** إلى **AI Automation** حقيقية، لمعالجة طلبات إجازة الموظفين وعرضها على المدير ليوافق أو يرفض.

سنستخدم:

- **Google Forms**
- **Zapier**
- **AI by Zapier**
- **Human in the Loop by Zapier**
- **Google Sheets**
- **Email**

---

# 🎯 هدف المشروع

هدفنا أن نمر بالرحلة الهندسية الكاملة:

> **Understand → Design → Build → Test → Refine**

وسنطبق الـMental Model التالي:

> **AI → Understand**  
> **Rule / Logic → Control**  
> **Human (Manager) → Judge**

أي:

- **AI** يفهم الطلب ويستخرج نوع الإجازة وتواريخها.
- **Automation / Logic** تتحكم في مسار العملية وتقرر هل الطلب مكتمل.
- **المدير** هو من يقرر الموافقة أو الرفض. لا يقرر AI ولا الـAutomation ذلك أبدًا.

---

# 01 — الصورة الكاملة

```text
Employee
   ↓
Google Form
   ↓
Trigger
   ↓
Unstructured Request
   ↓
AI by Zapier
   ↓
Structured Output
   ↓
Request Status
   ↓
        ┌──────────────────────┐
        │                      │
      VALID               NEEDS_REVIEW
        │                      │
        ↓                      ↓
  Google Sheets          Google Sheets
   (Pending)           (Not applicable)
        │                      │
        ↓                      ↓
 Manager Approval        Email → Employee
        │                 (cc Manager)
        ↓
 Approved / Rejected
        │
        ↓
 Update Google Sheets
        │
        ↓
  Email → Employee
```

## لماذا هذا هو التصميم؟

```text
AI       → Understand   (يفهم النص)
Logic    → Control      (هل الطلب مكتمل؟)
Manager  → Judge        (هل نوافق؟)
```

الطلب الناقص لا يصل إلى المدير أصلًا، فلا نطلب منه قرارًا على معلومات غير كاملة.

---

# 02 — إنشاء Google Form

أنشئ Google Form باسم:

**Leave Request Form**

أضف الحقول التالية:

| Field | Type |
|---|---|
| Employee Name | Short answer |
| Email | Email (أو فعّل Collect email addresses) |
| Department | Short answer |
| Request | Paragraph |

## لماذا؟

الـForm يمثل نقطة دخول الموظف إلى النظام (**Input**).

أما `Request` فهو **Unstructured Data** لأن الموظف يكتب طلبه بحرية.

مثال:

```text
I would like to request annual leave from December 20 to December 24 to travel with my family.
```

> لا نضع في النموذج حقولًا منفصلة لنوع الإجازة والتواريخ، لأن مهمة AI هنا هي استخراجها من النص.

---

# 03 — إنشاء Zap

**Create → New Zap**

سمِّ الـZap:

```text
AI Leave Request & Manager Approval
```

---

# 04 — Trigger

اختر:

**Google Forms → New Form Response**

(أو الاسم المطابق الذي يظهر في واجهة Zapier)

اربط Google Account والـForm، ثم نفّذ **Test Trigger**.

يجب أن تظهر بيانات مثل:

```text
Employee Name
Email
Department
Request
```

## Trigger vs Input

- **Trigger:** الحدث الذي يبدأ الـAutomation → `New Leave Request Submission`
- **Input:** البيانات التي وصلت مع الحدث → `Employee Name, Email, Department, Request`

> **Trigger ≠ Input**

---

# 05 — AI by Zapier

أضف Action: **AI by Zapier**

## Data In

أنشئ Input باسم:

```text
request_text
```

وقيمته: **Request** من Google Forms.

نرسل إلى AI النص غير المنظم فقط، ولا نرسل له بيانات الموظف الأخرى.

---

# 06 — AI Task

استخدم الـPrompt التالي:

```text
Analyze the employee leave request and extract structured information.

Use ONLY information explicitly provided by the employee. Never invent, assume, infer, estimate, or guess.

### Required information
- leave_type
- start_date
- end_date

### Extraction rules

**leave_type**
- Allowed values: `annual`, `sick`, `emergency`, `unpaid`, `other`, `unknown`
- Use the type the employee explicitly names or clearly describes:
  - "annual leave", "vacation" → `annual`
  - "sick leave", "I am sick" → `sick`
  - "emergency", "urgent family matter" → `emergency`
  - "unpaid leave" → `unpaid`
- Use `other` when the employee clearly names a type outside these categories (for example study leave or marriage leave).
- Use `unknown` when the type cannot be determined (for example "I need some time off").
- Never guess a type from the reason alone.

**start_date / end_date**
- Extract explicit calendar dates exactly as stated.
  - "from December 20 to December 24" → start `December 20`, end `December 24`
  - "on December 20 only" → start `December 20`, end `December 20`
- Relative or vague expressions ("next week", "tomorrow", "after the holidays", "soon") → `null`.
- A duration alone ("3 days") → `null` for both dates.
- A start date plus a duration ("from December 20 for 3 days") → start `December 20`, end `null`. Never calculate dates.
- Never add a year that was not stated.
- If end_date is earlier than start_date, return both as stated and list `end_date` in missing_information.

**request_summary**
- Summarize only what the employee explicitly requested.
- Include the reason only if the employee stated one.
- Do not add assumptions.

**missing_information**
- List required fields that are missing or ambiguous.
- Allowed values: `leave_type`, `start_date`, `end_date`
- Separate multiple values with commas.
- If nothing is missing, return exactly `NONE`.

### Decision rules

**request_status**
- `VALID` only if leave_type is not `unknown`, and start_date and end_date are both explicit and consistent.
- Otherwise return `NEEDS_REVIEW`.
- Return only `VALID` or `NEEDS_REVIEW`.
- Never approve or reject the leave. You only validate completeness.

### Output

Return these fields separately:

- leave_type
- start_date
- end_date
- request_summary
- missing_information
- request_status
```

---

# 07 — AI Output

أنشئ Outputs التالية:

| Output | Type |
|---|---|
| `leave_type` | Text |
| `start_date` | Text |
| `end_date` | Text |
| `request_summary` | Text |
| `missing_information` | Text |
| `request_status` | Text |

هنا يحدث التحول الأساسي:

```text
Unstructured Data → AI → Structured Data
```

### Before

```text
I would like to request annual leave from December 20 to December 24 to travel with my family.
```

### After

```text
leave_type = annual
start_date = December 20
end_date = December 24
```

---

# 08 — اختبار AI قبل بناء الـPaths

⚠️ **لا نبني الـPaths قبل التأكد من أن AI يعطي النتيجة الصحيحة.**

استخدم:

```text
I would like to request annual leave from December 20 to December 24 to travel with my family.
```

يجب أن تكون النتيجة:

```text
leave_type = annual
start_date = December 20
end_date = December 24
request_summary = Annual leave from December 20 to December 24 to travel with family
missing_information = NONE
request_status = VALID
```

إذا حصلنا عليها: **AI PASS ✅**

إذا لم نحصل عليها: **Stop → Debug AI**

> **Garbage In → Garbage Out**

---

# 09 — Paths by Zapier

أضف **Paths by Zapier** بمسارين:

```text
VALID        → Ready for Manager
NEEDS_REVIEW → Needs Clarification
```

---

# 10 — Path A: Ready for Manager

الشرط:

```text
request_status
Exactly matches
VALID
```

هذا هو الـ**Normal Path**.

---

# 11 — Path B: Needs Clarification

اجعله **Fallback**: أي طلب لا يحقق شرط `VALID` ينتقل إلى هنا.

```text
VALID        → Normal Path
Anything else → Clarification
```

> **Route the exception. Don't stop it.**

---

# 12 — إنشاء Google Sheet

أنشئ Google Sheet باسم:

**AI Leave Requests Records**

الأعمدة:

```text
employee_name
email
department
request_text
leave_type
start_date
end_date
request_summary
missing_information
request_status
routing
manager_decision
processed_at
decided_at
```

---

# 13 — Path A → Create Row

داخل **Ready for Manager** أضف:

**Google Sheets → Create Spreadsheet Row**

```text
employee_name        ← Google Form → Employee Name
email                ← Google Form → Email
department           ← Google Form → Department
request_text         ← Google Form → Request
leave_type           ← AI → leave_type
start_date           ← AI → start_date
end_date             ← AI → end_date
request_summary      ← AI → request_summary
missing_information  ← AI → missing_information
request_status       ← AI → request_status
processed_at         ← قيمة الوقت الحالي من Zapier إن توفرت
```

القيم الثابتة:

```text
routing = manager_approval
manager_decision = Pending
```

> سنحتاج **Row ID** الناتج من هذه الخطوة في الخطوة 15.

---

# 14 — Path A → Manager Approval

أضف:

**Human in the Loop by Zapier → Request Approval**

(أو الاسم المطابق في واجهة Zapier)

- **Approver:** بريد المدير.
- **Message / Details:**

```text
A leave request needs your decision.

Employee: {{employee_name}}
Department: {{department}}
Leave Type: {{leave_type}}
From: {{start_date}}
To: {{end_date}}

Summary:
{{request_summary}}

Original Request:
{{request_text}}
```

يتوقف الـZap هنا إلى أن يختار المدير **Approve** أو **Reject**.

## لماذا؟

هذه هي نقطة الـ**Human-in-the-Loop** الحقيقية. القرار هنا للمدير وحده.

---

# 15 — Path A → Update Row

أضف:

**Google Sheets → Update Spreadsheet Row**

- **Row:** Row ID من خطوة Create Row (الخطوة 13).

```text
manager_decision ← ناتج Manager Approval (Approved / Rejected)
decided_at       ← وقت القرار إن توفر
```

> نحدّث **السجل نفسه** ولا ننشئ سجلًا جديدًا.

---

# 16 — Path A → Email to Employee

أضف **Email by Zapier → Send Email**

- **To:** `{{email}}`

### Subject

```text
Your Leave Request: {{manager_decision}}
```

### Body

```text
Hello {{employee_name}},

Your manager has reviewed your leave request.

Decision: {{manager_decision}}
Leave Type: {{leave_type}}
From: {{start_date}}
To: {{end_date}}
```

> إذا أردت قالبين مختلفين (موافقة / رفض)، يمكنك لاحقًا إضافة Paths داخلية بحسب قرار المدير. في هذه النسخة نكتفي برسالة واحدة تعرض القرار.

---

# 17 — Path B → Create Row

داخل **Needs Clarification** أضف **Create Spreadsheet Row** بنفس الـMapping السابق، لكن:

```text
routing = clarification
manager_decision = Not applicable
```

لأن النظام اكتشف أن الطلب لا يصلح لقرار المدير بعد.

---

# 18 — Path B → Email

أضف **Send Email**

- **To:** `{{email}}`
- **CC:** بريد المدير

### Subject

```text
Your Leave Request Needs More Information
```

### Body

```text
Hello {{employee_name}},

We could not send your leave request to your manager because some information is missing or unclear.

Missing Information:
{{missing_information}}

Your Original Request:
{{request_text}}

Please submit a new request with the exact leave type, start date, and end date.
```

> يبقى هذا السجل في الـSheet بحالة `NEEDS_REVIEW` كسجل تاريخي، ويصل الطلب المصحَّح كسجل جديد.

---

# 19 — الاختبارات

---

## 🧪 TEST 01 — Complete Request → Approved

### Input

```text
I would like to request annual leave from December 20 to December 24 to travel with my family.
```

### Expected AI Output

```text
leave_type = annual
start_date = December 20
end_date = December 24
missing_information = NONE
request_status = VALID
```

### Expected Path

```text
Ready for Manager
```

### Manager Action

```text
Approve
```

### Expected Sheet

```text
request_status = VALID
routing = manager_approval
manager_decision = Approved
```

### Expected Email (to employee)

```text
Your Leave Request: Approved
```

### Result

**PASS ✅**

---

## 🧪 TEST 02 — Complete Request → Rejected

### Input

نفس طلب TEST 01.

### Manager Action

```text
Reject
```

### Expected Sheet

```text
request_status = VALID
routing = manager_approval
manager_decision = Rejected
```

### Expected Email (to employee)

```text
Your Leave Request: Rejected
```

### Result

**PASS ✅**

---

## 🧪 TEST 03 — Missing End Date

### Input

```text
I need sick leave starting December 20.
```

### Expected AI Output

```text
leave_type = sick
start_date = December 20
end_date = null
missing_information = end_date
request_status = NEEDS_REVIEW
```

### Expected Path

```text
Needs Clarification
```

### Expected Sheet

```text
request_status = NEEDS_REVIEW
routing = clarification
manager_decision = Not applicable
```

### Expected Email

```text
Your Leave Request Needs More Information
```

### Result

**PASS ✅**

---

## 🧪 TEST 04 — Ambiguous Dates

### Input

```text
I need annual leave next week.
```

### Expected AI Output

```text
leave_type = annual
start_date = null
end_date = null
missing_information = start_date, end_date
request_status = NEEDS_REVIEW
```

### Expected Path

```text
Needs Clarification
```

### Result

**PASS ✅**

---

## 🧪 TEST 05 — Multiple Missing Fields

### Input

```text
I need some time off.
```

### Expected AI Output

```text
leave_type = unknown
start_date = null
end_date = null
missing_information = leave_type, start_date, end_date
request_status = NEEDS_REVIEW
```

### Expected Path

```text
Needs Clarification
```

### Result

**PASS ✅**

---

## 🧪 TEST 06 — Contradictory Dates

### Input

```text
I need annual leave from December 24 to December 20.
```

### Expected AI Output

```text
leave_type = annual
start_date = December 24
end_date = December 20
missing_information = end_date
request_status = NEEDS_REVIEW
```

### Expected Path

```text
Needs Clarification
```

### Result

**PASS ✅**

---

# 20 — Test Matrix

| Test | Scenario | Expected Status | Expected Path | Manager Decision |
|---|---|---|---|---|
| 01 | Complete request | `VALID` | Ready for Manager | `Approved` |
| 02 | Complete request | `VALID` | Ready for Manager | `Rejected` |
| 03 | Missing end date | `NEEDS_REVIEW` | Clarification | `Not applicable` |
| 04 | Ambiguous dates | `NEEDS_REVIEW` | Clarification | `Not applicable` |
| 05 | Multiple missing fields | `NEEDS_REVIEW` | Clarification | `Not applicable` |
| 06 | Contradictory dates | `NEEDS_REVIEW` | Clarification | `Not applicable` |

---

# 21 — كيف نختبر الـAutomation؟

لا نعيد بناء الـZap لكل Test. نغيّر فقط الـInput.

1. افتح Google Form.
2. أدخل Test Case.
3. اضغط Submit.
4. راقب الـTrigger.
5. راقب AI Output.
6. راقب `request_status`.
7. راقب الـPath.
8. راقب Google Sheets.
9. إن كان Path A: افتح رسالة الموافقة بصفتك المدير واضغط Approve أو Reject.
10. راقب تحديث الـSheet والبريد.
11. قارن النتيجة.

```text
Expected
    VS
Actual
```

إذا تطابقا: **PASS ✅**

إذا لم يتطابقا: **FAIL ❌ → Debug**

> عند أول اختبار، تحقق من القيمة الفعلية التي يعيدها Zapier لقرار المدير (مثل `Approved` / `Rejected`) واعتمدها في الجدول والاختبارات.

---

# 22 — Debugging by Layer

```text
Trigger
   ↓
Input
   ↓
AI
   ↓
Structured Output
   ↓
Request Status
   ↓
Path
   ↓
Google Sheets (Create)
   ↓
Manager Approval
   ↓
Google Sheets (Update)
   ↓
Email
```

نسأل: **أين كانت أول نتيجة غير صحيحة؟**

| الحالة | المراجعة |
|---|---|
| AI أعطى `VALID` لكن الـPath لم يعمل | Path Logic |
| `end_date` مذكور لكن الحالة `NEEDS_REVIEW` | AI Task / Prompt |
| الـPath صحيح لكن الـSheet فيه بيانات خاطئة | Data Mapping |
| قرار المدير لم يُسجَّل في الـSheet | Update Row (هل استُخدم Row ID الصحيح؟) |
| الـSheet صحيح لكن البريد لم يصل | Email Action |

---

# 23 — Exception vs Error vs Rejection

### Exception (Business)

الطلب نفسه يحتاج إلى معالجة مختلفة. مثال: `end_date` غير مذكور.

→ يذهب إلى **Clarification**.

### Rejection (Decision)

المدير نظر في الطلب المكتمل ورفضه.

هذا **قرار عمل صحيح** وليس خطأً ولا استثناءً.

### Error (Technical)

النظام نفسه لم يعمل كما يجب: فشل AI، فشل Sheets، فشل البريد، أو انتهت مهلة الموافقة.

→ يحتاج إلى **Debugging**.

> ⚠️ **عدم رد المدير ليس رفضًا.** لا نحوّل انتهاء المهلة إلى `Rejected`.

---

# 24 — لماذا لا يوافق AI على الإجازة؟

لأن الموافقة على الإجازة قرار يتعلق بالعمل والأشخاص (ضغط العمل، تعارض الإجازات، رصيد الموظف...) وهذه سياقات لا يملكها النظام.

```text
AI       → يفهم النص
Logic    → يتحقق من الاكتمال
Manager  → يحكم
```

---

# 25 — لماذا نستخدم `request_status`؟

نحتاج إلى Signal واضح تتعامل معه الـAutomation.

بدلًا من أن يفسر الـPath:

```text
start_date = null
end_date = null
missing_information = start_date, end_date
```

نعطيه قيمة واحدة: `VALID` أو `NEEDS_REVIEW`.

وهذا يجعل الـDecision Logic أبسط وأوضح وأسهل في الاختبار والـDebugging.

---

# 26 — لماذا لم نستخدم Filter by Zapier؟

نحن لا نريد إيقاف الطلبات التي فيها مشكلة، بل توجيهها إلى الموظف ليستكملها.

> **Route the exception. Don't stop it.**

---

# 27 — ما الذي تعلمناه من المشروع؟

| Concept | تطبيقه في المشروع |
|---|---|
| Work | معالجة طلبات الإجازة |
| Process | من تقديم الطلب حتى إبلاغ الموظف بالقرار |
| Trigger | New Form Response |
| Input | بيانات الـGoogle Form |
| Unstructured Data | Request |
| AI Task | استخراج نوع الإجازة وتواريخها |
| Structured Output | الحقول الناتجة من AI |
| Missing Information | تاريخ أو نوع ناقص أو غامض |
| Decision | اكتمال الطلب → من يتخذ القرار؟ |
| Rule / Logic | تحديد `VALID` أو `NEEDS_REVIEW` |
| Exception | طلب ناقص يعود إلى الموظف |
| Human-in-the-Loop | موافقة المدير أو رفضه |
| Automation | تنفيذ الخطوات تلقائيًا |
| Record | Google Sheets |
| Notification | Email |
| Testing | Expected vs Actual |
| Debugging | تحديد أول Layer فشل |

---

# 28 — Definition of Done

- [ ] Google Form يعمل.
- [ ] Trigger يستقبل الطلب.
- [ ] AI يفهم الطلب.
- [ ] Structured Output صحيح.
- [ ] `request_status` صحيح.
- [ ] Path A يعمل ويُنشئ سجلًا بحالة `Pending`.
- [ ] رسالة الموافقة تصل إلى المدير.
- [ ] قرار المدير يحدّث السجل نفسه.
- [ ] بريد القرار يصل إلى الموظف.
- [ ] Path B يعمل ويرسل بريد الاستكمال إلى الموظف مع نسخة للمدير.
- [ ] Test 01 → 06 = PASS.
- [ ] تستطيع شرح سبب وجود كل خطوة.

---

# 🧠 الفكرة الأساسية

> **Tools implement the design.  
> Tools do not replace the design.**

```text
UNDERSTAND → DESIGN → BUILD → TEST → REFINE
```

## 🚀 Think First. Build Later.
