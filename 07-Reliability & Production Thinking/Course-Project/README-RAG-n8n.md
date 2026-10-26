# RAG with n8n — Knowledge Pipeline & RAG AI Agent

نظام RAG مبني بالكامل على n8n وSupabase Vector Store: مسار يجهّز المعرفة من ملف PDF (استخراج، تقسيم، Embeddings، تخزين)، وAI Agent يبحث في هذه المعرفة ويجيب منها فقط.

---

## 🎯 Problem

### The Context
في Dify كانت تفاصيل RAG كثيرة مخفية خلف واجهة جاهزة (الاستخراج، التقسيم، Embeddings، البحث).

### The Problem
بدون فهم هذه المراحل يصعب تحديد سبب فشل الإجابة: هل المشكلة في الاستخراج أم التقسيم أم الاسترجاع أم التوليد؟

### Why It Matters
بناء RAG بخطوات مرئية يتيح تتبع المعلومة من الوثيقة حتى الإجابة، ويمكّن من دمج المعرفة لاحقًا في حلول أتمتة حقيقية.

---

## 💡 Solution
Workflow من جزأين في n8n:
1. **Knowledge Ingestion Pipeline:** يجهّز المعرفة ويخزنها.
2. **RAG AI Agent:** يستخدم المعرفة عبر Vector Store كأداة.

### What does the system do?
- يلتقط ملفًا جديدًا من Google Drive، ويحمّله، ويستخرج نصه.
- يقسّم النص إلى Chunks ويحوّلها إلى Embeddings ويخزنها في Supabase.
- يستقبل سؤالًا من المحادثة، فيحوّله إلى Embedding ويبحث عن أقرب المقاطع.
- يجيب اعتمادًا على المقاطع المسترجعة، ويصرّح بعدم كفاية المعلومات عند غيابها.

---

## 👤 Users

**Primary Users:**
المتدربات والمستخدمون الذين يسألون عن محتوى وثيقة محددة (مثل مساعد لمحتوى الدورة).

**User Need:**
إجابات مبنية على محتوى الوثيقة، وليس على معرفة النموذج العامة.

---

## 🧠 How I Designed the Solution
- فصلت **تجهيز المعرفة** (Ingestion) عن **استخدامها** (Agent)، لأن الأول ليس Agent بل Pipeline.
- جعلت الـ Vector Store **أداة** للـ Agent، ليقرر هو متى يحتاج البحث (Tool Selection) وليس كل رسالة تستدعيه (مثل "Hello").
- استخدمت **Embeddings** للبحث الدلالي ليجد المعنى حتى لو اختلفت الكلمات.
- فصلت **Knowledge** (Supabase) عن **Memory** (سياق المحادثة).
- عاملت التقسيم (Chunk Size وOverlap) وTop K كقرارات هندسية تُختبر، لا كأرقام سحرية.

---

## ⚙️ How It Works

### 1) Knowledge Ingestion Pipeline

```
Google Drive Trigger
      ↓
Edit Fields (File ID)
      ↓
Google Drive: Download File
      ↓
Extract From File (PDF → text)
      ↓
Supabase Vector Store (Add documents)
   ├── Default Data Loader
   ├── Recursive Character Text Splitter
   └── OpenAI Embeddings
```

### 2) RAG AI Agent

```
When Chat Message Received
      ↓
AI Agent  ←── Gemini Chat Model
          ←── Simple Memory
          ←── Supabase Vector Store (Tool)
                  └── OpenAI Embeddings
      ↓
Answer
```

---

## 🤖 Role of AI

**What does AI do in this solution?**
- Embeddings: تحويل المقاطع والسؤال إلى متجهات للبحث الدلالي.
- LLM: صياغة الإجابة من المقاطع المسترجعة.
- الـ Agent: يقرر متى يستخدم أداة البحث.

**What AI Should NOT Do**
- لا يخترع معلومات غير مدعومة بالمحتوى المسترجع.
- لا يستدعي أداة المعرفة بلا حاجة (تحية مثلًا).
- لا يجيب من معرفته العامة عندما لا توجد المعلومة في الـ Knowledge Base.

---

## 🔄 Automation
- التقاط الملف الجديد في Google Drive تلقائيًا.
- التنزيل والاستخراج والتقسيم والتضمين والتخزين دون تدخل.
- البحث الدلالي عند كل سؤال يتطلب معرفة.
- الاحتفاظ بسياق المحادثة عبر Simple Memory.

---

## 🗂️ Data

**Input**
- ملف PDF في Google Drive:AI Automation Starter Guide).
- أسئلة المستخدم في المحادثة.

**Output**
- Records في جدول Supabase (المحتوى، Metadata، Embedding).
- إجابات مبنية على المقاطع المسترجعة.

**التخزين:** Supabase مع دعم Vector، وجدول المستندات ودالة `match_documents` للبحث بالتشابه.

> مفاتيح Supabase والـ Secret تُحفظ في Credentials ولا تُكتب في التوثيق أو المستودع.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| n8n | بناء المسارين وتنسيقهما |
| Google Drive | مصدر الملفات (Trigger وDownload) |
| Extract From File | استخراج النص من PDF |
| Default Data Loader + Recursive Character Text Splitter | تحويل النص إلى Documents وتقسيمه |
| OpenAI Embeddings | تحويل النص إلى متجهات |
| Supabase Vector Store | تخزين الـ Embeddings والبحث بالتشابه |
| Gemini Chat Model | توليد الإجابة |
| Simple Memory | سياق المحادثة |

---

## 🔑 Key Design Decisions

### Decision 1
**Decision:**
Chunk Size = 500 وChunk Overlap = 50 كنقطة بداية. [اكتبي القيم التي استخدمتِها فعلًا]

**Why:**
حجم صغير مناسب للتجربة، وOverlap يمنع انقطاع المعنى عند حدود الـ chunks. لا يوجد حجم مثالي، والقرار يعتمد على المحتوى وجودة الاسترجاع.

### Decision 2
**Decision:**
Top K (Limit) = 4 كنقطة بداية. [القيمة الفعلية]

**Why:**
يوازن بين كفاية السياق وتجنب مقاطع غير مرتبطة، ويُختبر مع 3 و5.




## 📸 Evidence

### Final Result
<img width="1842" height="842" alt="لقطة شاشة 2026-10-26 115505" src="https://github.com/user-attachments/assets/aca36fab-7445-47fa-8123-79fa61a2c9a0" />

### Workflow
<img width="1297" height="727" alt="ص8" src="https://github.com/user-attachments/assets/16810db7-1b6d-4017-b02a-c821b7f5795f" />

<img width="1736" height="737" alt="لقطة شاشة 2026-10-26 112553" src="https://github.com/user-attachments/assets/80951b65-f34d-4199-b532-3beff900817b" />

---

## ⚠️ Challenges & Failures
**What didn't work as expected?**
- ظهر Dimension Error أثناء الإدخال أو البحث.

**How did I solve it?**
- حددت أن المشكلة في التوافق بين نموذج Embeddings وأبعاد الـ Vector في جدول Supabase.
- راجعت أبعاد العمود وأبعاد النموذج المستخدم، وجعلتهما متطابقين، واستخدمت النموذج نفسه في الإدخال والبحث.
- أعدت تشغيل الإدخال (وأفرغت السجلات القديمة عند الحاجة) ثم اختبرت من جديد.
---

## 📚 What I Learned

**Technical Lesson**
تعلمت كيف يعمل RAG خطوة بخطوة بدل أن يبقى مخفيًا خلف واجهة جاهزة:

- **Chunking:** النموذج لا يتعامل مع الوثيقة كاملة، فنقسّمها إلى مقاطع صغيرة. حجم المقطع وOverlap قرار هندسي وليس رقمًا سحريًا؛ فالمقاطع الصغيرة جدًا تفقد السياق، والكبيرة جدًا تضعف دقة البحث، والـ Overlap يمنع انقطاع المعنى عند حدود المقاطع. بدأت بـ [500] و[50] واختبرت أثرهما.
- **Embeddings:** هي تحويل النص إلى متجهات رقمية تمثل المعنى، فيعمل البحث الدلالي حتى لو اختلفت الكلمات بين السؤال والنص. ويجب استخدام نموذج Embeddings نفسه في الإدخال والبحث، وإلا حدث خطأ في الأبعاد أو تراجعت جودة النتائج.
- **Vector Store (Supabase):** يخزن المقاطع مع متجهاتها وMetadata، ثم يجد أقرب المقاطع للسؤال بالتشابه (عبر `match_documents`)، ويحدد Top K عدد المقاطع المعادة للنموذج.
- **Retrieval وGeneration:** مرحلتان منفصلتان، فقد يُسترجع مقطع صحيح وتأتي الإجابة ناقصة، وقد تكون الإجابة خاطئة لأن الاسترجاع لم يجد المقطع المناسب. لذلك أفحص ما استُرجع قبل أن أحكم على النموذج.
- **Knowledge وMemory:** المعرفة في Supabase تُسترجع عند الحاجة، أما Memory فتحفظ سياق المحادثة فقط، ولا يصح الخلط بينهما.

**Problem-Solving Lesson**
عند سوء الإجابة يجب تحديد المرحلة: الاستخراج، التقسيم، التخزين، الاسترجاع، أم التوليد.

**AI Lesson**
الـ Retrieval غير الـ Generation: قد تُسترجع مقاطع صحيحة ومع ذلك تأتي الإجابة ناقصة، والعكس.

---

## 🔮 Future Improvements
If I continued developing this project, I would:

- تجربة قيم مختلفة لـ Chunk Size وTop K وتسجيل أثرها على الاسترجاع.
- إضافة مجموعة أسئلة اختبار ثابتة وإعادة تشغيلها بعد كل تعديل.
- دعم أنواع ملفات ومصادر أخرى، وإضافة Metadata للتصفية.
- إضافة ذكر المصدر في الإجابة.

---

## 🔗 Project Links


**Workflow:** <img width="1297" height="727" alt="لقطة شاشة 2026-10-26 112644" src="https://github.com/user-attachments/assets/063e05af-6514-44ea-a5a9-2559e3b41bc7" />


<img width="1736" height="737" alt="لقطة شاشة 2026-10-26 112553" src="https://github.com/user-attachments/assets/d123eed1-f41b-4ba6-893e-c878edf9c4fb" />

---

## 👩‍💻 Author

[دانا خالد شيخة]

