# PlanCraft AI

تطبيق ويب يحوّل فكرة المشروع إلى خطة تنفيذية منظمة: واجهة على Lovable، وقاعدة بيانات على Supabase، وWorkflow في n8n Cloud يستخدم AI Agent لتوليد الخطة وحفظها.

> الأجزاء بين `[ ]` تحتاج تعبئتها بنتائجك الفعلية قبل التسليم. جدول الاختبار فيه الاختبارات الثمانية من دليل المشروع والسلوك المتوقع، أما النتائج فتُكتب من تجاربك فقط.

---

## 🎯 Problem

### The Context
كثيرون لديهم فكرة مشروع، لكنهم يجدون صعوبة في تحويلها إلى خطوات واضحة.

### The Problem
من الصعب على صاحب الفكرة تحديد الهدف الفعلي، وتقسيم العمل إلى مراحل، وتحديد الإجراءات، واكتشاف المخاطر، وتقدير المدة، ومعرفة الخطوة التالية.

### Why It Matters
الإجابة العامة داخل محادثة لا تُحفظ ولا تُنظَّم. خطة منظمة ومحفوظة يمكن مراجعتها والعودة إليها تجعل الفكرة قابلة للتنفيذ.

---

## 💡 Solution
تطبيق يستقبل بيانات المشروع ويحفظها في Supabase، ثم يرسل الطلب إلى Workflow في n8n. الـ Workflow يسترجع الطلب، ويستخدم AI Agent مع Structured Output Parser لإنشاء خطة، ويحفظها ويحدّث حالة الطلب، ثم تعرض الواجهة الخطة المحفوظة.

### What does the system do?
- يستقبل `title` و`description` و`goal` و`additional_context` من المستخدم.
- يحفظ الطلب في `project_requests` بحالة `pending`.
- يشغّل Workflow عبر Production Webhook من طبقة آمنة على الخادم.
- يولّد خطة تتضمن: الملخص، الهدف، مراحل التنفيذ، الإجراءات الموصى بها، المخاطر وطرق الحد منها، والمدة التقديرية.
- يحفظ الخطة في `project_plans` ويحدّث الحالة إلى `completed`.
- يعرض الخطة الحقيقية المحفوظة، وتبقى ظاهرة بعد إعادة تحميل الصفحة.

---

## 👤 Users

**Primary Users:**
أشخاص لديهم فكرة مشروع ويريدون تحويلها إلى خطة عملية.

**User Need:**
الحصول على خطة منظمة (مراحل، إجراءات، مخاطر، مدة) يمكن مراجعتها والرجوع إليها.

---

## 🧠 How I Designed the Solution
- **ثلاث طبقات منفصلة:** Lovable للواجهة، Supabase للبيانات، n8n للأتمتة، وكل طبقة لها مسؤولية واضحة.
- **الطلب يُحفظ قبل المعالجة:** فيحصل كل طلب على `id` يربطه بالـ Workflow وبالخطة الناتجة.
- **حالة الطلب** (`pending`, `processing`, `completed`, `failed`) تتيح للواجهة متابعة التنفيذ وعدم عرض طلب فاشل كأنه مكتمل.
- **اتصال آمن:** التطبيق يستدعي n8n من طبقة خادم، ولا تُوضع المفاتيح السرية في كود المتصفح.
- **مخرجات منظمة:** Structured Output Parser يضمن أن الخطة تأتي بصيغة تُحفظ في الأعمدة المخصصة.
- [أضيفي أي قرار تصميم آخر اتخذتِه فعلًا]

---

## ⚙️ How It Works

```
User
  |
Lovable Application
  |  (يحفظ الطلب)
Supabase: project_requests  (status = pending)
  |
Server-Side Integration
  |
n8n Production Webhook (POST)
  |
Edit Fields
  |
Supabase: Get Request
  |
AI Agent  +  Structured Output Parser
  |
Supabase: Save Project Plan
  |
Supabase: Update Request Status  (completed)
  |
Lovable: Fetch the saved plan  →  Display the actual result
```

[عدّلي المخطط إذا كان تصميمك للاستجابة (Respond to Webhook) مختلفًا: مبكرة أو نهائية]

---

## 🤖 Role of AI

**What does AI do in this solution?**
- يقرأ بيانات المشروع (العنوان والوصف والهدف والسياق الإضافي).
- ينشئ خطة منظمة تتضمن الملخص والهدف والمراحل والإجراءات والمخاطر وطرق الحد منها والمدة التقديرية.
- يعيد المخرجات بالبنية التي يحددها Structured Output Parser.

**What AI Should NOT Do**
- لا يضمن نجاح المشروع أو صحة كل تقديراته؛ الخطة اقتراح قابل للمراجعة.
- لا يحفظ البيانات بنفسه ولا يحدّث الحالة؛ هذه عقد منفصلة في الـ Workflow.
- لا يعتمد على بيانات تجريبية أو نتيجة وهمية في الواجهة.

---

## 🔄 Automation
- تشغيل الـ Workflow تلقائيًا عند إرسال الطلب عبر Webhook.
- استرجاع الطلب من Supabase بمعرّفه.
- توليد الخطة وحفظها وتحديث الحالة دون تدخل المستخدم.
- متابعة الحالة في الواجهة وعرض الخطة عند اكتمالها.

---

## 🗂️ Data

**Input**
بيانات مشروع من المستخدم: `title`, `description`, `goal`, `additional_context`.

**Output**
خطة محفوظة ومعروضة، مع تحديث حالة الطلب.

**الجداول في Supabase:**

| الجدول | الأعمدة الرئيسية |
|---|---|
| `project_requests` | id, title, description, goal, additional_context, status, created_at, updated_at |
| `project_plans` | id, request_id (UNIQUE ومرتبط بالطلب), summary, project_goal, phases (JSONB), recommended_actions (JSONB), risks (JSONB), estimated_duration, created_at, updated_at |

> تحققي من الأنواع والقيود الفعلية في Supabase قبل التسليم.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Lovable | بناء الواجهة ووظائف التطبيق |
| Supabase | تخزين الطلبات والخطط وإدارة الحالة |
| n8n Cloud | تنسيق Workflow المعالجة |
| Webhook / Edit Fields | استقبال الطلب واستخراج `request_id` |
| Supabase nodes | استرجاع الطلب وحفظ الخطة وتحديث الحالة |
| AI Agent + Chat Model | توليد الخطة: [اسم النموذج الذي استخدمتِه] |
| Structured Output Parser | فرض بنية الخرج |

---

## 🔑 Key Design Decisions

### Decision 1
**Decision:**
حفظ الطلب في Supabase قبل استدعاء n8n.

**Why:**
ليحصل الطلب على `id` يربط الواجهة بالـ Workflow وبالخطة، ويمكن تتبعه عند الفشل.

### Decision 2
**Decision:**
استدعاء n8n من طبقة خادم باستخدام Production URL.

**Why:**
لعدم كشف المفاتيح وأسرار Webhook في المتصفح، ولأن Test URL لا يصلح للاستخدام الفعلي.

### Decision 3
**Decision:**
قيد `UNIQUE` على `request_id` في `project_plans`.

**Why:**
لضمان خطة واحدة لكل طلب وعدم تكرارها عند إعادة التنفيذ.

### Decision 4
**Decision:**
عرض الخطة من Supabase وليس من استجابة مؤقتة.

**Why:**
ليثبت أن النتيجة محفوظة فعلًا، وتبقى ظاهرة بعد إعادة تحميل الصفحة.

---

## 🧪 Testing

افحصي كل نتيجة في Supabase وn8n Executions (وليس فقط في الواجهة).

| # | Test | Expected Result | Actual Result | Status |
|---|------|-----------------|---------------|--------|
| 1 | إرسال طلب جديد من التطبيق المنشور | يقبل النموذج الطلب | [ ] | [ ✅ / ❌ ] |
| 2 | التحقق من حفظ الطلب | صف جديد في `project_requests` بحالة `pending` | [ ] | [ ✅ / ❌ ] |
| 3 | التحقق من تشغيل n8n | الطلب يصل إلى Production Webhook وتتنفذ العقد بالتسلسل | [ ] | [ ✅ / ❌ ] |
| 4 | التحقق من إنشاء الخطة | خطة في `project_plans` مرتبطة بالطلب الصحيح | [ ] | [ ✅ / ❌ ] |
| 5 | التحقق من تحديث الحالة | الحالة `completed` (وإذا فشلت المعالجة فلا تُعرض مكتملة) | [ ] | [ ✅ / ❌ ] |
| 6 | عرض الخطة داخل التطبيق | تظهر الخطة الحقيقية المحفوظة وليس بيانات ثابتة | [ ] | [ ✅ / ❌ ] |
| 7 | إعادة تحميل صفحة الخطة | تبقى الخطة ظاهرة | [ ] | [ ✅ / ❌ ] |
| 8 | اختبار فشل (`request_id` غير موجود في بيئة اختبار) | لا تُنشأ خطة غير مرتبطة بطلب صحيح، ويُعالج الخطأ بوضوح | [ ] | [ ✅ / ❌ ] |

---

## 📸 Evidence

### Final Result
![Final Result](screenshots/plan-details.png)

### Workflow
![Workflow](screenshots/n8n-workflow.png)

[أضيفي أيضًا: لقطة الجدولين في Supabase بعد الحفظ، لقطة Executions الناجحة، لقطة الحالة `completed`، ولقطة قبل/بعد لأي إصلاح. أخفي المفاتيح والروابط السرية قبل التصوير]

---

## ⚠️ Challenges & Failures

**What didn't work as expected?**
- [اكتبي ما فشل فعلًا: مثل request_id فارغ، تعبير Edit Fields، فشل حفظ الخطة، رسالة فشل رغم وجود الخطة، أو الاستجابة المبكرة للـ Webhook]

**How did I solve it?**
- [اكتبي السبب، والتعديل بالضبط، ونتيجة إعادة الاختبار]

---

## 📚 What I Learned

**Technical Lesson**
[ما تعلمتِه عن ربط Lovable وSupabase وn8n ونقل البيانات بينها بـ JSON]

**Problem-Solving Lesson**
عند فشل النظام يجب تتبع البيانات في كل مرحلة: الواجهة، قاعدة البيانات، سجل Executions.

**AI Lesson**
المخرجات المنظمة تحتاج Parser وتحققًا من الحفظ، وليس الاكتفاء بأن النموذج أعاد نصًا مقنعًا.

---

## 🔮 Future Improvements
If I continued developing this project, I would:

- إضافة مصادقة مستخدمين وربط كل طلب بمالكه، مع سياسات RLS تمنع الاطلاع على بيانات الآخرين.
- حماية Webhook بآلية مصادقة.
- إضافة إعادة محاولة آمنة بعد التحقق من وجود خطة سابقة.
- [أضيفي أفكارًا أخرى من تجربتك]

### حدود المشروع الحالية
- لا توجد مصادقة مستخدمين مكتملة، فلا ينبغي نشر التطبيق للاستخدام العام مع إتاحة غير مقيّدة للبيانات.
- الخطة اقتراح من الذكاء الاصطناعي وليست ضمانًا لنجاح المشروع.

---

## 🔗 Project Links

**Live Demo:** [رابط التطبيق المنشور]

**Workflow:** [ملف JSON للـ Workflow، بدون مفاتيح]

**Repository:** [رابط GitHub]

---

## 👩‍💻 Author

[اسمك]

[LinkedIn] · [GitHub]
