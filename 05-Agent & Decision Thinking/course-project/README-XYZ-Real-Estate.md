[My workflow.json](https://github.com/user-attachments/files/33923371/My.workflow.json)# XYZ Real Estate AI Agent

وكيل ذكاء اصطناعي على n8n يستقبل رسائل عملاء شركة XYZ Real Estate عبر Telegram، ويجمع احتياجاتهم العقارية تدريجيًا، ويحفظها في Google Sheets بعد تأكيد العميل الصريح فقط.

> الأجزاء بين `[ ]` تحتاج تعبئتها بنتائجك الفعلية قبل التسليم. لم أكتب نتيجة اختبار لم تحدث.

---

## 🎯 Problem

### The Context
تستقبل شركة XYZ Real Estate طلبات عملاء يرغبون في شراء عقارات أو استئجارها، ويحتاج فريق الشركة إلى معلومات أساسية عن كل عميل لفهم احتياجه ومتابعته.

### The Problem
جمع المعلومات يدويًا بطيء ويتكرر فيه السؤال نفسه، وقد تُسجَّل بيانات ناقصة أو غير مطابقة لقواعد العمل (مثل بريد إلكتروني غير صالح أو ميزانية غير واضحة).

### Why It Matters
بيانات العملاء الصحيحة والكاملة تسمح للفريق بالمتابعة السريعة. وحفظ بيانات غير مؤكدة أو الادعاء بحفظ لم يحدث يضيّع عملاء ويضر بثقة الشركة.

---

## 💡 Solution
Agent في n8n يدير المحادثة مع العميل على Telegram، ويفهم المعلومات التي يقدمها بأي ترتيب، ويسأل عن الناقص فقط، ثم يعرض ملخصًا للتأكيد قبل أن يحفظ الصف في Google Sheets ويتحقق من نتيجة الحفظ.

### What does the system do?
- يرحّب بالعميل ويبدأ بسؤال: شراء أم إيجار؟
- يستخرج المعلومات من الرسائل حتى لو جاءت عدة معلومات في رسالة واحدة.
- يتذكر ما قدّمه العميل خلال المحادثة.
- يطبّق القواعد الشرطية (الإيجار ← New/Resale وFirst Ourcase = Not Applicable).
- يعرض ملخصًا من تسعة حقول ويطلب تأكيدًا صريحًا.
- يحفظ في Google Sheets بعد التأكيد فقط، ولا يعلن النجاح إلا بعد التحقق من نتيجة الأداة.

---

## 👤 Users

**Primary Users:**
عملاء شركة XYZ Real Estate الراغبون في الشراء أو الإيجار، وفريق مبيعات الشركة الذي يستلم البيانات.

**User Need:**
العميل يريد تقديم طلبه بسهولة عبر محادثة طبيعية، والفريق يريد بيانات كاملة وصحيحة جاهزة للمتابعة.

---

## 🧠 How I Designed the Solution
- جعلت الـ Agent يقرر الخطوة التالية بنفسه (سؤال، توضيح، ملخص، حفظ) بدل مسار أسئلة ثابت.
- ربطت الحفظ بشرط صريح: اكتمال الحقول + اجتياز التحقق + تأكيد العميل على الملخص.
- فصلت قواعد العمل (الميزانية، البريد، الحقول الشرطية) في تعليمات النظام وطبّقتها قبل إعداد الملخص.
- اعتبرت التحقق من نتيجة Google Sheets جزءًا من السلوك المطلوب، حتى لا يدّعي الـ Agent حفظًا لم يتم.
- [أضيفي أي قرار تصميم آخر اتخذتِه فعلًا]

---

## ⚙️ How It Works

```
رسالة العميل على Telegram
          ↓
   Telegram Trigger
          ↓
       AI Agent  ←→  OpenRouter Chat Model
          │      ←→  Simple Memory
          │      ←→  Google Sheets Tool
          ↓
 Telegram Send Message (رد الـ Agent)
```

---

## 🤖 Role of AI

**What does AI do in this solution?**
- يفهم رسائل العميل ويستخرج منها الحقول المطلوبة.
- يحدد المعلومات الناقصة ويطرح السؤال المناسب فقط.
- يصنّف الميزانية ضمن إحدى الفئات الخمس ويطلب التوضيح عند الغموض.
- يعدّ الملخص النهائي ويحدّثه عند تصحيح العميل.
- يقرر متى يستدعي أداة Google Sheets.

**What AI Should NOT Do**
- لا يخمّن معلومة غامضة أو يخترع بيانات لم يقدمها العميل.
- لا يحفظ البيانات قبل التأكيد الصريح لمجرد اكتمالها.
- لا يسأل عن New/Resale وFirst Ourcase في طلبات الإيجار.
- لا يدّعي نجاح الحفظ إذا فشلت الأداة.
- لا يضيف أعمدة أخرى غير الأعمدة التسعة.

---

## 🔄 Automation
- استقبال رسائل Telegram تلقائيًا عبر Telegram Trigger.
- الاحتفاظ بسياق المحادثة عبر Simple Memory بـ Session ID ثابت لكل عميل.
- إضافة صف جديد إلى ورقة Leads في Google Sheets بعد التأكيد.
- إرسال رد الـ Agent إلى العميل عبر Telegram Send Message.
- [اكتبي أي ضوابط تنفيذ أضفتِها لمنع استدعاء أداة الحفظ قبل التأكيد، إن أضفتِ]

---

## 🗂️ Data

**Input**
رسائل نصية من العميل عبر Telegram.

**Output**
- ردود الـ Agent على Telegram (أسئلة، توضيح، ملخص، نتيجة الحفظ).
- صف في Google Sheets (`XYZ Real Estate Leads` ← ورقة `Leads`) بالأعمدة التسعة:

| Column | Header | القيم المسموحة |
|---|---|---|
| A | Property type | Apartment, Villa, Commercial |
| B | Buy/Rent | Buy, Rent |
| C | New/Resale | New, Resale, Not Applicable |
| D | First Ourcase | Yes, No, Not Applicable |
| E | Timeline | Immediately, Within 1 month, 1–3 months, 3–6 months, 6+ months |
| F | Budget | إحدى الفئات الخمس أدناه |
| G | Name | يقدمه العميل |
| H | Email | صيغة صحيحة |
| I | Phone Number | يقدمه العميل |

**فئات الميزانية:** Below $200,000 / $200,000 to below $400,000 / $400,000 to below $700,000 / $700,000 to $1,000,000 / Above $1,000,000

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| n8n | بناء الـ Workflow وتشغيله |
| Telegram Trigger / Send Message | استقبال رسائل العميل وإرسال الردود |
| AI Agent (n8n) | فهم الرسائل واتخاذ قرار الخطوة التالية واستخدام الأدوات |
| OpenRouter Chat Model | نموذج اللغة: [اسم النموذج الذي اخترتِه] |
| Simple Memory | الاحتفاظ بسياق المحادثة لكل جلسة |
| Google Sheets Tool | إضافة بيانات العميل المؤكدة |

---

## 🔑 Key Design Decisions

### Decision 1
**Decision:**
الحفظ في Google Sheets بعد تأكيد العميل الصريح على الملخص فقط.

**Why:**
حتى لا تُسجَّل بيانات لم يوافق عليها العميل أو لا تزال تتغير.

### Decision 2
**Decision:**
تطبيق القواعد الشرطية قبل إعداد الملخص (الإيجار ← Not Applicable في حقلين).

**Why:**
حتى لا يُسأل العميل أسئلة غير منطبقة، ولتكون القيم المحفوظة مطابقة للملخص.

### Decision 3
**Decision:**
طلب التوضيح عند الإجابات الغامضة بدل التخمين.

**Why:**
لأن تصنيفًا خاطئًا للميزانية أو الإطار الزمني يضر بجودة البيانات.

### Decision 4
**Decision:**
Session ID ثابت لكل محادثة في Simple Memory.

**Why:**
لضمان عدم اختلاط معلومات عميلين مختلفين.

---

## 🧪 Testing

<img width="862" height="842" alt="24" src="https://github.com/user-attachments/assets/555223e7-3fde-4ae7-b306-64e3ed6c40f8" />
<img width="987" height="727" alt="23" src="https://github.com/user-attachments/assets/d819a5a8-cc9a-4fb3-be76-decb92a10f0c" />
<img width="802" height="716" alt="25" src="https://github.com/user-attachments/assets/585e6f5a-6986-4f83-891e-0ec90e11b861" />



## 📸 Evidence
<img width="1462" height="347" alt="26" src="https://github.com/user-attachments/assets/f9fe5b43-1608-4324-9411-b0a2f3b3636b" />

### Final Result

### Workflow
<img width="1497" height="722" alt="X" src="https://github.com/user-attachments/assets/b4e0d767-9415-4683-a731-579b453396fd" />


---


## 📚 What I Learned

**Technical Lesson**
تعلمتُ كيفية ربط AI Agent بالأدوات في n8n لتنفيذ المهام والتفاعل مع الخدمات المختلفة بشكل تلقائي.
كما تعلمتُ ربطه بالذاكرة للاحتفاظ بسياق المحادثات السابقة وتقديم ردود أكثر ذكاءً ودقة.
**Problem-Solving Lesson**
رد الـ Agent لا يكفي كدليل؛ يجب التحقق من سجل التنفيذ ومحتوى Google Sheets.

**AI Lesson**
منع الإجراء الحساس (الحفظ) يحتاج ضوابط في تصميم الـ Workflow، وليس تعليمات في البرومبت فقط.

---

## 🔮 Future Improvements
If I continued developing this project, I would:

- منع استدعاء أداة Google Sheets تقنيًا قبل التأكيد.
- منع الصفوف المكررة بمفتاح فريد لكل طلب.
- دعم لغات إضافية في المحادثة.

---

## 🔗 Project Links

**Live Demo:** 
(https://web.telegram.org/k/#@XYZRealEstate1Assistant_bot)
**Workflow:**<img width="1497" height="722" alt="X" src="https://github.com/user-attachments/assets/2045c4f5-1e62-408a-8397-e64936a56780" />
https://docs.google.com/spreadsheets/d/1KZwjbv0he7xswZ5uTz-tUd8_iyOdGVySKIGBuaYM3lE/edit?gid=0#gid=0
  



---

## 👩‍💻 Author

[دانا خالد شيخة]

