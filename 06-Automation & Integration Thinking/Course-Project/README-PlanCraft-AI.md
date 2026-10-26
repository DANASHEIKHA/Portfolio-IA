# PlanCraft AI
PlanCraft AI هو تطبيق ويب مدعوم بالذكاء الاصطناعي، يساعد المستخدمين على تحويل أفكار المشاريع إلى خطط تنفيذية منظمة وقابلة للتطبيق.

يجمع المشروع بين ثلاثة مكونات رئيسية:

Lovable: لبناء واجهة التطبيق ووظائفه.
Supabase: لتخزين طلبات المشاريع والخطط الناتجة.
n8n Cloud: لبناء وإدارة سير العمل الآلي الذي يعالج الطلبات ويولّد الخطط باستخدام الذكاء الاصطناعي.
يستقبل التطبيق فكرة المشروع من المستخدم، ويحفظ بياناتها في قاعدة البيانات، ثم يرسل طلبًا إلى Workflow في n8n. يسترجع الـ Workflow بيانات المشروع، ويستخدم AI Agent لإنشاء خطة منظمة، ثم يحفظ النتائج في Supabase لتتمكن واجهة التطبيق من عرضها للمستخدم.
تطبيق ويب يحوّل فكرة المشروع إلى خطة تنفيذية منظمة: واجهة على Lovable، وقاعدة بيانات على Supabase، وWorkflow في n8n Cloud يستخدم AI Agent لتوليد الخطة وحفظها.


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
| AI Agent + Chat Model | توليد الخطة
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

##🧪 Testing


## 📸 Evidence

### Final Result
<img width="1892" height="912" alt="لقطة شاشة 2026-10-26 111647" src="https://github.com/user-attachments/assets/469d5bbb-2d2b-4d50-b469-e9712d1849c5" />

<img width="1275" height="721" alt="لقطة شاشة 2026-10-22 120952" src="https://github.com/user-attachments/assets/074a0c14-8004-4ef2-bfdb-ab6f2d5d2a9a" />
<img width="1780" height="727" alt="لقطة شاشة 2026-10-26 111156" src="https://github.com/user-attachments/assets/5ecd453f-c9bf-4285-b372-9ecd975d9cac" />
<img width="1045" height="878" alt="لقطة شاشة 2026-10-22 130616" src="https://github.com/user-attachments/assets/f0c4f524-2ef7-4a69-8dc5-1bc25aaf803d" />
<img width="1780" height="727" alt="لقطة شاشة 2026-10-26 111156" src="https://github.com/user-attachments/assets/d19176e0-9903-4eb7-9a6b-08ede5fa797e" />

### Workflow
<img width="1275" height="721" alt="لقطة شاشة 2026-10-22 120952" src="https://github.com/user-attachments/assets/e2c73ae4-68d0-45b7-a16d-5b20d67608a6" />

<img width="1892" height="912" alt="لقطة شاشة 2026-10-26 111647" src="https://github.com/user-attachments/assets/7c045b86-ebb7-433b-adbd-d12730a20d84" />

---

## ⚠️ Challenges & Failures

**What didn't work as expected?**
<img width="1502" height="886" alt="لقطة شاشة 2026-10-26 111816" src="https://github.com/user-attachments/assets/1756f9c8-e998-4093-898c-d3bba6d052ee" />

**How did I solve it?**
أعدنا الربط بشكل صحيح
---

## 📚 What I Learned

**Technical Lesson**
تعلمت أن التكامل بين الأدوات يعتمد على تتبع البيانات من مرحلة لمرحلة، وليس على نجاح كل أداة منفردة.

- **التنقل بين النظم عبر JSON:** يحفظ Lovable الطلب في Supabase، ثم يرسل `request_id` إلى n8n عبر Webhook، ويقرأه الـ Workflow من `body` الطلب الوارد ليسترجع بيانات المشروع كاملة من قاعدة البيانات.
- **المعرّف هو الرابط بين الأجزاء:** `request_id` يربط الطلب بالـ Workflow وبالخطة المحفوظة، وقيد `UNIQUE` عليه يمنع تكرار الخطة للطلب نفسه.
- **الحالة تعكس التقدم:** تحديث `status` من `pending` إلى `completed` يتيح للواجهة معرفة متى تعرض الخطة، ويمنع عرض طلب فاشل كأنه مكتمل.
- **Production URL وليس Test URL:** الاتصال الفعلي بين التطبيق وn8n يتطلب تفعيل الـ Workflow واستخدام الـ Production Webhook.
- **المفاتيح خارج المتصفح:** يتم الاستدعاء من طبقة خادم حتى لا تظهر الأسرار في كود الواجهة.
- **التحقق من الحفظ الفعلي:** نجاح عقدة في n8n لا يكفي، فأتحقق من الصف في Supabase ومن ظهور الخطة بعد إعادة تحميل الصفحة.
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

### حدود المشروع الحالية
- لا توجد مصادقة مستخدمين مكتملة، فلا ينبغي نشر التطبيق للاستخدام العام مع إتاحة غير مقيّدة للبيانات.
- الخطة اقتراح من الذكاء الاصطناعي وليست ضمانًا لنجاح المشروع.

---

## 🔗 Project Links

**Live Demo:** https://pixel-perfect-replay-1901.lovable.app

**Workflow:** <img width="1780" height="727" alt="لقطة شاشة 2026-10-26 111156" src="https://github.com/user-attachments/assets/26cb450d-f9f7-4479-9762-466ba9ee15de" />

## 👩‍💻 Author

[دانا خالد شيخة]
