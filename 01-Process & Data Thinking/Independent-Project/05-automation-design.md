# 05 — Automation Design

## 1. Automation Goal

بناء نظام يساعد الشركة على معالجة طلبات الإجازة بصورة منظمة، بدءًا من استقبال الطلب وحتى قرار المدير بالموافقة أو الرفض وإبلاغ الموظف به.

## 2. Automation Flow

Google Forms
→ AI by Zapier
→ Decision & Routing (Paths)

**Path A — Ready for Manager**
→ Google Sheets (Create Row, `Pending`)
→ Manager Approval (Human in the Loop by Zapier)
→ Google Sheets (Update Row)
→ Email to Employee

**Path B — Needs Clarification**
→ Google Sheets (Create Row)
→ Email to Employee (نسخة إلى المدير)

## 3. Step 1 — Receive Request

**Tool:** Google Forms

عندما يرسل الموظف النموذج:

1. يستقبل Zapier بيانات الموظف.
2. يبدأ تدفق الأتمتة عند وصول رد جديد.

## 4. Step 2 — Extract Information

**Tool:** AI by Zapier

يستقبل AI نص الطلب (`request_text`)، ثم يستخرج المعلومات المطلوبة:

- leave_type
- start_date
- end_date
- request_summary
- missing_information
- request_status

يجب أن يلتزم AI بالمعلومات التي قدمها الموظف، وألا يخترع نوع إجازة أو تاريخًا أو سببًا غير موجود، وألا يحوّل تعبيرًا نسبيًا مثل "next week" إلى تاريخ.

إذا لم يقدم الموظف معلومة، تُسجل بوصفها `null` بدلًا من تخمينها.

## 5. Step 3 — Validate & Route

تُطبّق قواعد التحقق المحددة في:

`04-decision-and-exception-design.md`

يوجد مساران:

### Path A — Ready for Manager

يُستخدم عندما `request_status = VALID`.

النتيجة:

- `routing = manager_approval`
- `manager_decision = Pending`

### Path B — Needs Clarification

يُستخدم لأي حالة أخرى (Fallback).

النتيجة:

- `routing = clarification`
- `manager_decision = Not applicable`

يجب أن تتطابق شروط المسارين، بحيث لا يُوجَّه الطلب نفسه إلى المسارين معًا.

## 6. Step 4 — Record the Request

**Tool:** Google Sheets

بعد تحديد المسار:

1. يُنشئ النظام سجلًا واحدًا للطلب.
2. تُحفظ بيانات النموذج والمعلومات المستخرجة.
3. تُحفظ حالة التحقق والمسار وحالة قرار المدير.
4. يُسجل وقت المعالجة عند توفر قيمة وقت مناسبة من Zapier.

يجب الاحتفاظ بمعرّف الصف (Row ID) في Path A حتى يتم تحديث السجل نفسه لاحقًا.

## 7. Step 5 — Manager Decision (Path A فقط)

**Tool:** Human in the Loop by Zapier — Request Approval

1. يُرسل Zapier إلى المدير رسالة تتضمن المعلومات المنظمة للطلب.
2. يختار المدير: موافقة أو رفض.
3. يتوقف الـZap إلى أن يصدر القرار.

بعد صدور القرار:

1. يُحدَّث سجل الطلب نفسه بـ `manager_decision` ووقت القرار `decided_at`.
2. يجب ألا يؤدي التحديث إلى إنشاء سجل مكرر.

## 8. Step 6 — Notify Employee

**Tool:** Email by Zapier

### For Path A

إرسال إشعار للموظف بقرار المدير (`Approved` أو `Rejected`) مع تفاصيل الطلب.

### For Path B

إرسال إشعار للموظف يوضح المعلومات الناقصة أو غير الواضحة ويطلب منه إرسال طلب جديد، مع إرسال نسخة إلى المدير.

## 9. Technical Failures

إذا فشلت خطوة تقنية، مثل استخراج المعلومات أو تحديث السجل أو إرسال البريد أو انتهت مهلة طلب الموافقة، فلا ينبغي تسجيل هذا الفشل تلقائيًا على أنه `NEEDS_REVIEW` أو `Rejected`.

في هذه النسخة، تتم مراجعة سجل التنفيذ في Zapier لتحديد الخطوة الفاشلة وتصحيحها.

## Expected Outcome

تصميم تدفق أتمتة واضح يربط استقبال الطلب بمعالجته والتحقق منه وقرار المدير وتسجيل النتيجة وإبلاغ الموظف.
