# RAG with n8n — Knowledge Pipeline & RAG AI Agent

نظام RAG مبني بالكامل على n8n وSupabase Vector Store: مسار يجهّز المعرفة من ملف PDF (استخراج، تقسيم، Embeddings، تخزين)، وAI Agent يبحث في هذه المعرفة ويجيب منها فقط.

> الأجزاء بين `[ ]` تعبئينها بنتائجك الفعلية. لم أكتب أي نتيجة اختبار لم تحدث.

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
- ملف PDF في Google Drive: [اسم الملف ومحتواه].
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

### Decision 3
**Decision:**
جعل الـ Vector Store أداة للـ Agent مع تعليمات "استخدمها عندما يتطلب السؤال معرفة".

**Why:**
لتعليم Tool Selection بدل الاستدعاء الأعمى.

### Decision 4
**Decision:**
نفس نموذج Embeddings في الإدخال والاسترجاع.

**Why:**
لأن اختلاف النموذج أو الأبعاد يسبب أخطاء Dimension وبحثًا غير صحيح.

---

## 🧪 Testing

| # | Test | Input | Expected Result | Actual Result | Status |
|---|------|-------|-----------------|---------------|--------|
| 1 | معلومة موجودة | `What is RAG?` | يستدعي الأداة ويجيب من المقاطع | [ ] | [ ✅ / ❌ ] |
| 2 | بحث دلالي | `Can an AI Agent interact with external systems?` (والنص يقول: AI Agents can use external tools) | يجد المعلومة رغم اختلاف الكلمات | [ ] | [ ✅ / ❌ ] |
| 3 | معلومة غير موجودة | `What is the admission policy of Harvard University?` | `I don't have enough information in the knowledge base to answer this.` دون اختلاق | [ ] | [ ✅ / ❌ ] |
| 4 | الذاكرة مع RAG | `What is RAG?` ثم `Why is it useful?` | يفهم "it" من السياق ويجيب من المعرفة | [ ] | [ ✅ / ❌ ] |

**تحقق إضافي:** افتحي Supabase وتأكدي من وجود Records بعد الـ Ingestion (المحتوى وMetadata وEmbedding).

---

## 📸 Evidence

### Final Result
![Final Result](screenshots/rag-answer.png)

### Workflow
![Workflow](screenshots/workflow.png)

[أضيفي: لقطة جدول Supabase بعد الإدخال، لقطة Executions، لقطة الاسترجاع الناجح، ولقطة سؤال بلا إجابة. أخفي المفاتيح]

---

## ⚠️ Challenges & Failures

**What didn't work as expected?**
- [اكتبي ما واجهتِه فعلًا. من المشاكل المحتملة في الدليل: لا Record في Supabase، خطأ في البحث، Dimension Error، الـ Agent يجيب معلومات غير موجودة، أو Retrieval سيئ]

**How did I solve it?**
- [السبب، والتعديل، ونتيجة إعادة الاختبار]

---

## 📚 What I Learned

**Technical Lesson**
[ما تعلمتِه عن الـ Chunking وEmbeddings وVector Store]

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

**Live Demo:** [ ]

**Workflow:** [ملف JSON بدون مفاتيح]

**Repository:** [رابط GitHub]

---

## 👩‍💻 Author

[اسمك]

[LinkedIn] · [GitHub]
