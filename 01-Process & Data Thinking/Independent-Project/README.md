# [Project Name]

AI Leave Request & Manager Approval

نظام أتمتة يحوّل طلبات إجازة الموظفين المكتوبة بحرية إلى بيانات منظمة، ويتحقق من اكتمالها، ثم يرسل الطلبات المكتملة إلى المدير ليوافق عليها أو يرفضها.

🎯 Problem
The Context
يرسل الموظفون طلبات الإجازة عبر الرسائل أو البريد أو المحادثات. يقرأ المدير كل طلب، ويحدد نوع الإجازة وتواريخها، ويتحقق من اكتمال المعلومات، ثم يقرر ويبلغ الموظف بالنتيجة.

The Problem
كل طلب يحتاج إلى العمل اليدوي نفسه: القراءة والاستخراج والتحقق والمتابعة والرد. ومع زيادة عدد الطلبات، قد يضيع بعضها، وتُتخذ قرارات بناءً على تواريخ غير واضحة، ويتأخر الرد، ولا يوجد سجل موثوق.

Why It Matters
لماذا هذا مهم؟

يجب أن يذهب وقت المدير إلى القرار نفسه (موافقة أو رفض)، لا إلى مطاردة التواريخ الناقصة. كما أن السجل المنظم يجعل كل طلب وقرار قابلًا للتتبع.
💡 Solution
سير عمل في Zapier يستقبل كل طلب من Google Form، ويستخدم AI لاستخراج الحقول المهمة، ثم تطبق قاعدة واضحة لتحديد اكتمال الطلب، ثم يوجّهه. الطلبات المكتملة تذهب إلى المدير ليتخذ قرارًا بشريًا. الطلبات الناقصة تعود إلى الموظف.

ماذا يفعل النظام؟

يستقبل طلب الإجازة من Google Forms. يستخرج نوع الإجازة وتاريخ البداية وتاريخ النهاية وملخص الطلب من النص الحر. يصنّف الطلب VALID أو NEEDS_REVIEW. يرسل الطلبات VALID إلى المدير، ويسجل قراره في Google Sheets. يعيد الطلبات NEEDS_REVIEW إلى الموظف مع بيان المعلومات الناقصة.
👤 Users
Primary Users:
المستخدمون الأساسيون: الموظفون الذين يطلبون الإجازة، والمدير الذي يوافق أو يرفض.

احتياجهم: يحتاج الموظف إلى طريقة بسيطة لتقديم الطلب والحصول على إجابة واضحة. ويحتاج المدير إلى رؤية الطلبات المكتملة فقط، واتخاذ قرار واحد لكل طلب. User Need:
[What do they need to accomplish?]

🧠 How I Designed the Solution
يفصل التصميم بين ثلاثة أدوار:

text AI → Understand (يفهم النص) Logic → Control (هل الطلب مكتمل؟) Manager → Judge (موافقة أو رفض)

لا يوافق AI على الإجازة ولا يرفضها. فالموافقة تعتمد على سياق لا يملكه النظام (ضغط العمل، تعارض الإجازات، رصيد الموظف)، لذلك يبقى هذا القرار بيد إنسان. والطلبات الناقصة لا تصل إلى المدير أبدًا، فلا يُطلب منه قرار على معلومات ناقصة.
⚙️ How It Works
Google Form (الموظف يرسل الطلب) ↓ Zapier Trigger: New Form Response ↓ AI by Zapier: استخراج الحقول المنظمة ↓ Paths: هل request_status = VALID ؟ ├─ VALID → Google Sheets (Pending) │ ↓ │ موافقة المدير (Human in the Loop) │ ↓ │ Google Sheets (تحديث الصف نفسه) │ ↓ │ بريد للموظف بالقرار │ └─ NEEDS_REVIEW → Google Sheets (Not applicable) ↓ بريد للموظف (نسخة للمدير)

🤖 Role of AI
ماذا يفعل AI في هذا الحل؟

يقرأ طلب الإجازة المكتوب بحرية. يستخرج leave_type وstart_date وend_date كما كُتبت تمامًا. يكتب ملخصًا قصيرًا ويحدد المعلومات الناقصة أو غير الواضحة. يرجع request_status (VALID أو NEEDS_REVIEW) وفق قواعد صارمة في الـPrompt.

ما الذي يجب ألا يفعله AI؟

الموافقة على الإجازة أو رفضها. اختراع التواريخ أو تخمينها أو حسابها، أو إضافة سنة لم تُذكر. تحويل تعبيرات غامضة مثل "next week" إلى تواريخ.

🔄 Automation
ما الأجزاء المؤتمتة من العملية؟

استقبال الطلب وبدء سير العمل. استخراج المعلومات وتنظيمها. تحديد اكتمال الطلب واختيار المسار. تسجيل الطلب وقرار المدير في Google Sheets. إشعار المدير، وإبلاغ الموظف بالنتيجة أو بالمعلومات الناقصة.

🗂️ Data
Input
🗂️ Data

Input حقول النموذج: employee_name وemail وdepartment وrequest_text.

Output أعمدة الـSheet: employee_name وemail وdepartment وrequest_text وleave_type وstart_date وend_date وrequest_summary وmissing_information وrequest_status وrouting وmanager_decision وprocessed_at وdecided_at. لقيم المسموح بها:

الحقل القيم leave_type annual, sick, emergency, unpaid, other, unknown request_status VALID, NEEDS_REVIEW routing manager_approval, clarification manager_decision Pending، أو القرار الذي تعيده خطوة الموافقة، أو Not applicable

🛠️ Tools & Technologies
الأداة الغرض Google Forms استقبال طلبات الإجازة Zapier تشغيل سير العمل وربط الخطوات AI by Zapier استخراج بيانات منظمة من النص الحر Paths by Zapier التوجيه بحسب request_status Human in the Loop by Zapier طلب موافقة المدير أو رفضه Google Sheets تسجيل الطلبات والقرارات Gmail إشعار الموظفين (والمدير بنسخة)
🔑 Key Design Decisions
Decision 1

القرار: المدير هو من يتخذ قرار الموافقة أو الرفض دائمًا. النظام يقرر فقط هل الطلب مكتمل.

السبب: الموافقة تعتمد على سياق عمل لا يملكه النظام. والطلب VALID يعني فقط أنه جاهز لعرضه على المدير.

Decision 2

القرار: تُقبل التواريخ الصريحة فقط. التعبيرات النسبية ("next week") والمدة وحدها تصبح null.

السبب: يجب ألا يفترض النظام معلومات لم يقدمها الموظف، ولا ينبغي أن يقرر المدير على تاريخ مخمَّن.

Decision 3

القرار: الطلبات الناقصة تعود إلى الموظف ولا تصل إلى المدير. ومسار Needs Clarification هو الـFallback لأي حالة غير VALID.

السبب: نوجّه الاستثناء بدل إيقافه، ونبعد الطلبات الناقصة عن المدير.

Decision 4

القرار: إذا رفض المدير يكمل التشغيل. وإذا انتهت مهلة الموافقة يتوقف التشغيل.

السبب: الرفض قرار عمل صحيح يجب تسجيله وإبلاغ الموظف به. أما عدم الرد فليس قرارًا، فلا يجوز تسجيله موافقة ولا رفضًا.

Decision 5

القرار: حقول التواريخ في مخرجات AI من نوع Text وليست Date.

السبب: نوع Date يُجبر AI على إرجاع تاريخ كامل حتى لو لم يذكر الموظف أي تاريخ، وهذا يكسر قاعدة عدم اختراع المعلومات.

🧪 Testing
Test 1

Input:

text I would like to request annual leave from December 20 to December 24 to travel with my family.

النتيجة المتوقعة: leave_type = annual وstart_date = December 20 وend_date = December 24 وmissing_information = NONE وrequest_status = VALID. المسار: Ready for Manager. يوافق المدير؛ يُحدَّث الصف بالموافقة ويصل الموظف قرار.

النتيجة الفعلية: [تعبّأ بعد التشغيل الكامل]

Status: ✅ / ❌

Test 2

Input: نفس طلب Test 1. المدير يرفض.

النتيجة المتوقعة: يُحدَّث الصف بقرار الرفض، وباقي الأعمدة كما هي، ويصل الموظف بريد فيه القرار.

النتيجة الفعلية: [تعبّأ بعد التشغيل الكامل]

Status: ✅ / ❌

Test 3

Input:

text I need sick leave starting December 20.

النتيجة المتوقعة: end_date = null وmissing_information = end_date وrequest_status = NEEDS_REVIEW. المسار: Needs Clarification. لا يُسأل المدير، ويصل الموظف بريد يطلب المعلومة الناقصة.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

Status: ✅ / ❌

Test 4

Input:

text I need some time off.

النتيجة المتوقعة: leave_type = unknown والتاريخان null وmissing_information = leave_type, start_date, end_date وrequest_status = NEEDS_REVIEW. المسار: Needs Clarification.

النتيجة الفعلية: [تعبّأ بعد التشغيل]

📸 Evidence
Final Result
q
Workflow
zap task
⚠️ Challenges & Failures
Challenges & Failures

ما الذي لم يعمل كما توقعت؟

ظهرت التواريخ فاضية في رسالة الموافقة. كان السبب أن حقول مخرجات AI عُرّفت بنوع Date وRequired؛ وتغييرها إلى Text حل المشكلة ووافق قاعدة "لا تخمّن". إعداد Stop run الافتراضي عند الرفض كان سينهي سير العمل مع كل رفض، فلا يُسجَّل شيء ولا يُبلَّغ الموظف. بريد الاختبار من خطوة الموافقة كان معاينة فقط، بدون زري Approve وDecline. والقيمة الفعلية للقرار لا تظهر إلا بعد نشر الـZap وتقديم طلب حقيقي. النشر كان محجوبًا بسبب خطوة فاضية في المسار الثاني. أظهر Zapier تحذيرًا أن الـZap ينتهي بخطوة Human in the Loop. وكان الحل إضافة خطوتي التحديث وإشعار الموظف بعدها.

كيف حللتها؟

اختبرت كل طبقة بالترتيب (Trigger ثم مخرجات AI ثم Path ثم Sheets ثم الموافقة ثم البريد)، وصلحت أول خطوة أعطت نتيجة خاطئة. أعدت ربط الحقول من الـData picker بدل كتابتها يدويًا، وتأكدت أن كل قيمة ظهرت كحقل مربوط.

📚 What I Learned
#Technical Lesson

نوع الحقل مهم. نوع خاطئ (Date بدل Text) غيّر بصمت ما يُسمح لـAI بإرجاعه.

Problem-Solving Lesson

اختبر طبقة واحدة في كل مرة، وابحث عن أول نتيجة خاطئة بدل تغيير عدة أشياء معًا.

AI Lesson

AI ممتاز في فهم النصوص غير المنظمة، لكن القواعد التي تتحكم في التوجيه والقرار النهائي يجب أن تبقى خارج AI.

🔮 Future Improvements
لو تابعت تطوير المشروع، فسأقوم بـ:

إرسال قالبين مختلفين للبريد: واحد للموافقة وواحد للرفض. إضافة تذكير عندما لا يرد المدير بعد مدة محددة. التحقق من رصيد إجازات الموظف وتعارض الإجازات قبل الإرسال إلى المدير. دعم أكثر من مدير بحسب القسم.

🔗 Project Links
Live Demo: [Link] https://docs.google.com/spreadsheets/d/1XUJOjfLwqCfScgVvMFamSxEmR1KeFBJpXeCjttNsnJM/edit?gid=0#gid=0

👩‍💻 Author
[Dana sheikha]
