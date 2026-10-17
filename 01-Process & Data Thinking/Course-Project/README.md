AI Client Intake & Request Routing System

نظام أتمتة يستقبل طلبات العملاء المحتملين، ويستخرج المعلومات المهمة منها باستخدام AI، ثم يوجّه كل طلب إلى المسار المناسب: متابعة مباشرة أو مراجعة بشرية.

🎯 Problem
السياق

تستقبل شركة خدمات طلبات من عملاء محتملين، وتراجعها لفهم احتياجاتهم وتحديد الخطوة التالية المناسبة لكل طلب.

المشكلة

يعالج الموظفون الطلبات يدويًا: يقرأ الموظف كل طلب، ويفهمه، وينظم معلوماته، ويتحقق من اكتمالها، ثم يحدد الإجراء التالي. ومع زيادة عدد الطلبات، تصبح هذه العملية مستهلكة للوقت والجهد.

لماذا هذا مهم؟

تكرار المهام اليدوية مع كل طلب جديد قد يؤدي إلى تأخر المتابعة أو إغفال معلومات مهمة. وتنظيم الطلبات وتوجيهها آليًا يوفّر وقت الموظفين ويجعل التعامل مع الطلبات أكثر اتساقًا.

💡 Solution

سير عمل في Zapier يبدأ من Google Form، ويستخدم AI by Zapier لتحويل نص الطلب الحر إلى بيانات منظمة، ثم يطبّق قاعدة واضحة لتحديد هل الطلب مكتمل. الطلب المكتمل يذهب إلى مسار المتابعة، والطلب الناقص أو غير الواضح يذهب إلى المراجعة البشرية. وتُسجَّل النتيجة في Google Sheets ويُرسل إشعار مناسب.

ماذا يفعل النظام؟

يستقبل طلب العميل من Google Forms.
يستخرج نوع الخدمة والمدة الزمنية والميزانية وملخص الطلب من النص الحر.
يحدد المعلومات الناقصة ويصنّف الطلب VALID أو NEEDS_REVIEW.
يوجّه الطلب المكتمل إلى web_team والطلب الناقص إلى human_review.
يسجل النتيجة في Google Sheets ويرسل البريد المناسب.
👤 Users
Primary Users:
[Who will use this solution?]

User Need:
[What do they need to accomplish?]

🧠 How I Designed the Solution
[Briefly explain the main thinking/design decisions behind the solution.]

⚙️ How It Works
Input
  ↓
[Step]
  ↓
AI / Logic
  ↓
[Step]
  ↓
Output
ماذا يفعل AI في هذا الحل؟

يفهم نص الطلب الحر.
يستخرج نوع الخدمة، ويلخص الطلب.
يستخرج الميزانية والمدة الزمنية إن ذُكرتا صراحةً.
يحدد المعلومات المطلوبة التي تبدو ناقصة أو غير واضحة.

ما الذي يجب ألا يفعله AI؟

اختراع ميزانية أو مدة زمنية أو أي تفاصيل لم يذكرها العميل. المعلومة غير الموجودة تُسجَّل null ولا تُخمَّن.
تحديد مسار الطلب أو قرار المتابعة بنفسه. المسار تتحكم فيه قاعدة ثابتة (request_status).
🔄 Automation

ما الأجزاء المؤتمتة من العملية؟

استقبال الطلب وبدء سير العمل عند وصول رد جديد.
استخراج المعلومات وتنظيمها في حقول.
تحديد حالة الطلب (VALID أو NEEDS_REVIEW) واختيار المسار.
تسجيل النتائج في Google Sheets.
إرسال البريد المناسب لكل مسار.

وتبقى المراجعة البشرية للحالات الناقصة أو الغامضة. الأتمتة هنا تقلل الجهد المتكرر ولا تلغي الحكم البشري عند الحاجة.

🗂️ Data

Input حقول النموذج: client_name وemail وcompany وrequest_text. ويمثل request_text النص الحر الذي يكتبه العميل.

Output معلومات منظمة من AI: service_type وrequest_summary وtimeline وbudget وmissing_information. ونتائج المعالجة: validation_status (VALID أو NEEDS_REVIEW) وrequires_human (true أو false) وrouting (web_team أو human_review) وprocessed_at.

قيم service_type المسموح بها: web_design وdigital_marketing وgraphic_design وapp_development وconsulting وai_services وother وunknown.


🛠️ Tools & Technologies
الأداة	الغرض
Google Forms	استقبال طلبات العملاء
Zapier	تشغيل سير العمل وربط الخطوات
AI by Zapier	استخراج بيانات منظمة من النص الحر
Paths by Zapier	توجيه الطلب بحسب حالته
Google Sheets	تسجيل الطلبات ونتائج المعالجة
Email	إشعار الموظفين بنتيجة كل طلب
🔑 Key Design Decisions
Decision 1

القرار: AI يستخرج المعلومات فقط، والقواعد هي التي تحدد حالة الطلب ومساره.

السبب: قدرة AI على فهم الطلب لا تعني أنه يجب أن يملك القرار النهائي. القاعدة الثابتة يمكن اختبارها والتنبؤ بها، بينما قد يخطئ AI في قرار التوجيه.

Decision 2

القرار: الطلب لا يكون VALID إلا إذا توفرت الميزانية والمدة الزمنية ونوع الخدمة. وأي نقص أو غموض يحوّله إلى NEEDS_REVIEW.

السبب: لا ينبغي اعتبار الطلب غير المكتمل جاهزًا للمتابعة لمجرد أن AI لخّصه. وعدم اختراع المعلومات الناقصة أهم من تمرير الطلب بسرعة.

Decision 3

القرار: التمييز بين القيم: null (لم يذكرها العميل) وunknown (لا يمكن تحديد الخدمة) وother (الخدمة مفهومة لكنها خارج الفئات) وNONE (لا نقص).

السبب: هذه القيم لا تعني الشيء نفسه، واستخدام كل قيمة بمعناها يجعل القرارات اللاحقة أدق.

Decision 4

القرار: مسار المراجعة البشرية هو الـFallback لأي حالة غير VALID، ولا نستخدم Filter يوقف الطلب.

السبب: نريد توجيه الاستثناء لا إيقافه، حتى لا يضيع طلب فيه نقص.

Decision 5

القرار: NEEDS_REVIEW حالة عمل مختلفة عن الفشل التقني. فشل AI أو Sheets أو البريد يُراجَع من سجل التنفيذ في Zapier ولا يُسجَّل تلقائيًا كمراجعة بشرية.

السبب: الطلب الناقص استثناء في العمل (Exception)، أما الفشل التقني فهو خطأ في النظام (Error) ويحتاج Debugging.
🧪 Testing
est 1 — طلب مكتمل

Input:

text
We need a website for our company.
Our budget is $5000.
We need it next month.

النتيجة المتوقعة: service_type = website وtimeline = next month وbudget = $5000 وmissing_information = NONE وrequest_status = VALID. المسار: Valid Request. في الـSheet: requires_human = false وrouting = web_team. البريد: New Valid Client Request.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

Status: ✅ / ❌

Test 2 — ميزانية ناقصة

Input:

text
We need a website for our company.
We need it next month.

النتيجة المتوقعة: budget = null وmissing_information = budget وrequest_status = NEEDS_REVIEW. المسار: Needs Human Review. في الـSheet: requires_human = true وrouting = human_review. البريد: Client Request Requires Human Review.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

Status: ✅ / ❌

Test 3 — مدة زمنية غامضة

Input:

text
We need a website soon.
Our budget is $5000.

النتيجة المتوقعة: timeline = null وmissing_information = timeline وrequest_status = NEEDS_REVIEW. المسار: Needs Human Review.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

Status: ✅ / ❌

Test 4 — عدة معلومات ناقصة

Input:

text
We need a website for our company.

النتيجة المتوقعة: timeline = null وbudget = null وmissing_information = timeline, budget وrequest_status = NEEDS_REVIEW. المسار: Needs Human Review.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

📸 Evidence
<img width="1846" height="262" alt="w" src="https://github.com/user-attachments/assets/908c949c-9397-471c-a47d-991d2d185bf3" />


<img width="16384" height="7049" alt="zap 1" src="https://github.com/user-attachments/assets/bff9187c-a401-41cd-baf4-4630168346ec" />


⚠️ Challenges & Failures
ما الذي لم يعمل كما توقعت؟
في الاختبار الثاني (الميزانية ناقصة)، رجّع AI قيمة $5000 في حقل budget مع أن العميل ما ذكر أي ميزانية. اخترع معلومة بدل أن يرجّع null.
بعد ذلك دخل الطلب مسار Valid Request بدل Needs Human Review، لأن الـPath قرأ request_status = VALID.
كيف حللتها؟
فحصت الطبقات بالترتيب، وكان أول نتيجة خاطئة في مخرجات AI، فما كان الخطأ في الـPath نفسه.
أضفت إلى الـPrompt قاعدة صريحة: "إذا لم تُذكر الميزانية، أرجع null ولا تخمّن".
أعدت اختبار الطلب نفسه وتأكدت أن النتيجة صارت budget = null وNEEDS_REVIEW.
📚 What I Learned

Technical Lesson

الأداة تنفّذ التصميم ولا تصمم الحل بدلًا عنا. وفهم العمل وتفكيكه قبل فتح Zapier جعل بناء الـZap أسهل وأوضح.

Problem-Solving Lesson

عند ظهور نتيجة خاطئة، أفحص الطبقات بالترتيب (Trigger ثم Input ثم AI ثم Status ثم Path ثم Sheets ثم Email) وأصلح أول طبقة فيها الخطأ.

AI Lesson

AI يفهم، والقاعدة تتحكم، والإنسان يحكم. و"Unknown is better than invented": المعلومة غير الموجودة تبقى ناقصة ولا تُخمَّن.

🔮 Future Improvements

لو تابعت تطوير المشروع، فسأقوم بـ:

إضافة مسارات لأنواع خدمات أخرى بدل مسار web_team فقط.
معالجة الفشل التقني تلقائيًا بتنبيه منفصل بدل مراجعة سجل التنفيذ يدويًا.
تحديث السجل الأصلي للطلب بدل إنشاء سجل جديد، وربط ذلك بـRow ID.
إضافة ردّ تلقائي للعميل يطلب المعلومات الناقصة.
🔗 Project Links
(https://docs.google.com/forms/d/e/1FAIpQLSfJhuwhSPMecQ_UMjD0bqXj9NtooPQnrKMqJzoBwjTBQPwv4w/viewform?usp=dialog)
https://docs.google.com/spreadsheets/d/15wXXQnJyXt2gYemSJ2_sl_v61lgCUmqK5mrGMz2ZwYI/edit?usp=sharing
Project Links



👩‍💻 Author
[Dana sheikha]

