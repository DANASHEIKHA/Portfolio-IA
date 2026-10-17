# 02 — Workflow Decomposition

## 1. Process

**Leave Request Intake & Approval Process**

هي عملية استقبال طلبات الإجازة وفهمها وتنظيمها والتحقق منها، ثم عرض الطلب المكتمل على المدير لاتخاذ القرار، وتسجيل النتيجة وإبلاغ الموظف.

## 2. Workflow

يتكون سير العمل من الخطوات التالية:

1. Receive Request
2. Understand Request
3. Structure Information
4. Validate Information
5. Route Request
6. Request Manager Decision
7. Record Results
8. Notify Employee

## 3. Tasks

| Task | Description |
|---|---|
| Receive Request | استقبال طلب الموظف وتسجيل بياناته. |
| Understand Request | فهم محتوى الطلب ونوع الإجازة المطلوبة. |
| Structure Information | استخراج المعلومات المهمة وتنظيمها. |
| Validate Information | التحقق من اكتمال المعلومات ووضوحها. |
| Route Request | تحديد المسار المناسب وفق قواعد محددة. |
| Request Manager Decision | إرسال الطلب المكتمل إلى المدير وانتظار قراره. |
| Record Results | تسجيل نتائج المعالجة وقرار المدير. |
| Notify Employee | إبلاغ الموظف بالنتيجة. |

## 4. Subtasks

### Task 1: Receive Request
- استقبال بيانات الموظف من Google Forms.
- بدء تدفق الأتمتة عند وصول طلب جديد.

### Task 2: Understand Request
- قراءة نص الطلب.
- تحديد نوع الإجازة التي يطلبها الموظف.
- تلخيص الطلب.

### Task 3: Structure Information
- استخراج نوع الإجازة.
- استخراج تاريخ البداية وتاريخ النهاية إن ذُكرا صراحةً.
- استخراج ملخص الطلب.
- تحديد المعلومات الناقصة أو غير الواضحة.

### Task 4: Validate Information
- التحقق من وجود نوع الإجازة وتاريخيها.
- التحقق من أن التواريخ صريحة وغير متناقضة.
- تحديد ما إذا كان الطلب جاهزًا لقرار المدير أو يحتاج إلى استكمال.

### Task 5: Route Request
- توجيه الطلب المستوفي للشروط إلى المدير.
- توجيه الطلب الذي لا يستوفي الشروط إلى الموظف لاستكماله.

### Task 6: Request Manager Decision
- إرسال طلب الموافقة إلى المدير مع المعلومات المنظمة.
- انتظار قرار المدير: موافقة أو رفض.

### Task 7: Record Results
- تسجيل الطلب والمعلومات المستخرجة وحالة التحقق في Google Sheets.
- تحديث السجل نفسه بقرار المدير.

### Task 8: Notify Employee
- إبلاغ الموظف بقرار المدير.
- إبلاغ الموظف بالمعلومات التي يجب استكمالها، إن كان الطلب ناقصًا.

## 5. Sequence

يجب تنفيذ المهام بالترتيب التالي:

Receive Request
→ Understand Request
→ Structure Information
→ Validate Information
→ Route Request
→ Request Manager Decision
→ Record Results
→ Notify Employee

لا يُرسل الطلب إلى المدير قبل التحقق من اكتماله.

لا يُبلَّغ الموظف بالقرار قبل أن يصدر قرار المدير فعلًا.

## 6. Dependencies

| Task | Depends On |
|---|---|
| Understand Request | Receive Request |
| Structure Information | Understand Request |
| Validate Information | Structure Information |
| Route Request | Validate Information |
| Request Manager Decision | Route Request (المسار المكتمل فقط) |
| Record Results | Route Request |
| Notify Employee | Record Results |

## Expected Outcome

تحديد خطوات العمل وترتيبها واعتماد بعضها على بعض قبل تحويلها إلى نظام أتمتة.
