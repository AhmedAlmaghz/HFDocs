# فهم حجم النموذج الذي يمكن أن يتسع لجهازك

هناك جانب صعب للغاية عند استكشاف النماذج المحتملة لاستخدامها على جهازك وهو معرفة مدى حجم النموذج الذي سيتم *تثبيته* في الذاكرة مع بطاقة الرسومات الحالية لديك (مثل تحميل النموذج على CUDA).

وللمساعدة في التخفيف من ذلك، يوفر 🤗 Accelerate واجهة سطر أوامر من خلال `accelerate estimate-memory`. سيساعدك هذا البرنامج التعليمي على استخدامه، وما يمكن توقعه، وفي النهاية، سيتم ربطه بالتجربة التفاعلية المستضافة على 🤗 Hub والتي ستسمح لك حتى بنشر هذه النتائج مباشرة في مستودع النموذج!

حاليًا، ندعم البحث عن النماذج التي يمكن استخدامها في `timm` و`transformers`.

<Tip>

ستقوم واجه برمجة التطبيقات هذه بتحميل النموذج في الذاكرة على الجهاز `meta`، لذلك فإننا لا نقوم فعليًا بتنزيل وتحميل الأوزان الكاملة للنموذج في الذاكرة، ولا نحتاج إلى ذلك. ونتيجة لذلك، من الآمن تمامًا قياس نماذج المعلمات التي تبلغ 8 مليارات (أو أكثر)، دون القلق بشأن ما إذا كان بإمكان وحدة المعالجة المركزية التعامل معها!

</Tip>

## عروض Gradio

فيما يلي بعض عروض Gradio المتعلقة بما تم وصفه أعلاه. الأول هو مساحة تقدير الذاكرة الرسمية لـ Hugging Face، والتي تستخدم Accelerate مباشرة:

<div class="block dark:hidden">
<iframe
src="https://hf-accelerate-model-memory-usage.hf.space?__theme=light"
width="850"
height="1600"
></iframe>
</div>
<div class="hidden dark:block">
<iframe
src="https://hf-accelerate-model-memory-usage.hf.space?__theme=dark"
width="850"
height="1600"
></iframe>
</div>

قام أحد أعضاء المجتمع بتطوير الفكرة وتوسيعها، مما يتيح لك تصفية النماذج مباشرة ومعرفة ما إذا كان بإمكانك تشغيل LLM معين نظرًا لقيود GPU وتكوينات LoRA. للعب به، راجع [هنا](https://huggingface.co/spaces/Vokturz/can-it-run-llm) لمزيد من التفاصيل.

## الأمر

عند استخدام `accelerate estimate-memory`، يجب عليك تمرير اسم النموذج الذي تريد استخدامه، وإطار العمل المحتمل
الذي يستخدمه النموذج (إذا لم يتم العثور عليه تلقائيًا)، وأنواع البيانات التي تريد تحميل النموذج بها.

على سبيل المثال، إليك كيفية حساب بصمة الذاكرة لـ `bert-base-cased`:

```bash
accelerate estimate-memory bert-base-cased
```

سيقوم هذا الأمر بتنزيل `config.json` لـ `bert-based-cased`، وتحميل النموذج على الجهاز `meta`، والإبلاغ عن مقدار المساحة
سيتم استخدامه:

Memory Usage for loading `bert-base-cased`:

| dtype   | Largest Layer | Total Size | Training using Adam |
|---------|---------------|------------|---------------------|
| float32 | 84.95 MB      | 418.18 MB  | 1.61 GB             |
| float16 | 42.47 MB      | 206.59 MB  | 826.36 MB           |
| int8    | 21.24 MB      | 103.29 MB  | 413.18 MB           |
| int4    | 10.62 MB      | 51.65 MB   | 206.59 MB           |

افتراضيًا، ستتم إعادتها لجميع أنواع البيانات المدعومة (`int4` من خلال `float32`)، ولكن إذا كنت مهتمًا بأنواع محددة، فيمكن تصفيتها.

### مكتبات محددة

إذا لم يتم تحديد مكتبة المصدر تلقائيًا (مثلما حدث في حالة `bert-base-cased`)، فيمكن تمرير اسم المكتبة.

```bash
accelerate estimate-memory HuggingFaceM4/idefics-80b-instruct --library_name transformers
```

Memory Usage for loading `HuggingFaceM4/idefics-80b-instruct`:

| dtype   | Largest Layer | Total Size | Training using Adam |
|---------|---------------|------------|---------------------|
| float32 | 3.02 GB       | 297.12 GB  | 1.16 TB             |
| float16 | 1.51 GB       | 148.56 GB  | 594.24 GB           |
| int8    | 772.52 MB     | 74.28 GB   | 297.12 GB           |
| int4    | 386.26 MB     | 37.14 GB   | 148.56 GB           |

```bash
accelerate estimate-memory timm/resnet50.a1_in1k --library_name timm
```

Memory Usage for loading `timm/resnet50.a1_in1k`:

| dtype   | Largest Layer | Total Size | Training using Adam |
|---------|---------------|------------|---------------------|
| float32 | 9.0 MB        | 97.7 MB    | 390.78 MB           |
| float16 | 4.5 MB        | 48.85 MB   | 195.39 MB           |
| int8    | 2.25 MB       | 24.42 MB   | 97.7 MB             |
| int4    | 1.12 MB       | 12.21 MB   | 48.85 MB            |

### أنواع بيانات محددة

كما ذكرنا سابقًا، في حين أننا نعيد `int4` من خلال `float32` بشكل افتراضي، يمكن استخدام أي نوع بيانات من `float32` و`float16` و`int8` و`int4`.

للقيام بذلك، قم بتمريرها بعد تحديد `--dtypes`:

```bash
accelerate estimate-memory bert-base-cased --dtypes float32 float16
```

Memory Usage for loading `bert-base-cased`:

| dtype   | Largest Layer | Total Size | Training using Adam |
|---------|---------------|------------|---------------------|
| float32 | 84.95 MB      | 413.18 MB  | 1.61 GB             |
| float16 | 42.47 MB      | 206.59 MB  | 826.36 MB           |

## التحذيرات مع هذه الآلة الحاسبة

ستخبرك هذه الآلة الحاسبة بكيفية كمية الذاكرة اللازمة لتحميل النموذج فقط، *وليس* لأداء الاستدلال.

هذه الحسابات دقيقة في حدود بضع % من القيمة الفعلية، لذا فهي توفر نظرة جيدة جدًا عن مقدار الذاكرة التي ستستغرقها. على سبيل المثال، يستغرق تحميل `bert-base-cased` في الواقع `413.68 MB` عند تحميله على CUDA بدقة كاملة، وتقدر الآلة الحاسبة `413.18 MB`.

عند إجراء الاستدلال، يمكنك توقع إضافة ما يصل إلى 20% إضافية كما وجدتها [EleutherAI](https://blog.eleuther.ai/transformer-math/). سنقوم بإجراء أبحاث لإيجاد تقدير أكثر دقة لهذه القيم، وسنقوم بتحديث
هذه الآلة الحاسبة بمجرد الانتهاء.