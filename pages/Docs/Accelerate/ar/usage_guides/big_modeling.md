# التعامل مع النماذج الكبيرة للاستنتاج

من أكبر التطورات التي توفرها مكتبة 🤗 Accelerate هي مفهوم [استنتاج النماذج الكبيرة](../concept_guides/big_model_inference) حيث يمكنك إجراء الاستنتاج على النماذج التي لا يمكن أن تناسب كليًا بطاقة الرسومات الخاصة بك.

سيتم تقسيم هذا البرنامج التعليمي إلى جزأين يظهران كيفية استخدام كل من 🤗 Accelerate و 🤗 Transformers (مستوى واجهة برمجة التطبيقات الأعلى) للاستفادة من هذه الفكرة.

## استخدام 🤗 Accelerate

بالنسبة لهذه البرامج التعليمية، سنفترض سير عمل نموذجي لتحميل نموذجك بحيث:

```py
import torch

my_model = ModelClass(...)
state_dict = torch.load(checkpoint_file)
my_model.load_state_dict(state_dict)
```

لاحظ أننا نفترض هنا أن `ModelClass` هو نموذج يشغل ذاكرة بطاقة الفيديو أكثر مما يمكن أن يناسب جهازك (سواء كان `mps` أو `cuda`).

الخطوة الأولى هي إنشاء نموذج فارغ لن يشغل أي ذاكرة RAM باستخدام مدير سياق [`init_empty_weights`]:

```py
from accelerate import init_empty_weights
with init_empty_weights():
    my_model = ModelClass(...)
```

مع هذا، فإن `my_model` حاليًا "بدون معلمات"، وبالتالي يترك بصمة أصغر مما قد يحصل عليه المرء عادةً عن طريق التحميل مباشرة إلى وحدة المعالجة المركزية.

بعد ذلك، نحتاج إلى تحميل الأوزان في نموذجنا حتى نتمكن من إجراء الاستدلال.

للقيام بذلك، سنستخدم [`load_checkpoint_and_dispatch`]، والذي كما يوحي الاسم، سيقوم بتحميل نقطة تفتيش داخل نموذجك الفارغ وإرسال الأوزان لكل طبقة عبر جميع الأجهزة المتوفرة لديك (GPU/MPS وذاكرة CPU RAM).

ولتحديد كيفية إجراء هذا "الإرسال"، عادةً ما يكون تحديد `device_map="auto"` كافيًا لأن 🤗 Accelerate
سيحاول ملء جميع المساحة في وحدة (وحدات) معالجة الرسومات الخاصة بك، ثم تحميلها إلى وحدة المعالجة المركزية، وأخيراً إذا لم تكن هناك ذاكرة RAM كافية، فسيتم تحميلها إلى القرص (أبطأ خيار).

<Tip>

للحصول على مزيد من التفاصيل حول تصميم خريطة الأجهزة الخاصة بك، راجع هذا القسم من [دليل المفاهيم](../concept_guides/big_model_inference#designing-a-device-map)

</Tip>

انظر المثال أدناه:

```py
from accelerate import load_checkpoint_and_dispatch

model = load_checkpoint_and_dispatch(
    model, checkpoint=checkpoint_file, device_map="auto"
)
```

<Tip>

إذا كانت هناك "قطع" معينة من الطبقات التي لا يجب تقسيمها، فيمكنك تمريرها على أنها `no_split_module_classes`. اقرأ المزيد عنها [هنا](../concept_guides/big_model_inference#loading-weights)

</Tip>

<Tip>

أيضًا لتوفير الذاكرة (مثل إذا لم يتم احتواء `state_dict` في ذاكرة الوصول العشوائي)، يمكن تقسيم أوزان النموذج وتقسيمها إلى عدة ملفات نقاط تفتيش. اقرأ المزيد عنها [هنا](../concept_guides/big_model_inference#sharded-checkpoints)

</Tip>

الآن بعد أن تم إرسال النموذج بالكامل، يمكنك إجراء الاستدلال كالمعتاد باستخدام النموذج:

```py
input = torch.randn(2,3)
input = input.to("cuda")
output = model(input)
```

ما سيحدث الآن هو أنه في كل مرة يتم فيها تمرير الإدخال عبر طبقة، فإنه يتم إرساله من وحدة المعالجة المركزية إلى وحدة معالجة الرسومات (أو القرص إلى وحدة المعالجة المركزية إلى وحدة معالجة الرسومات)، ويتم حساب الإخراج، ثم يتم سحب الطبقة مرة أخرى من وحدة معالجة الرسومات والعودة إلى الخط. في حين أن هذا يضيف بعض النفقات العامة للاستدلال الذي يتم إجراؤه، من خلال هذه الطريقة من الممكن تشغيل **أي حجم نموذج** على نظامك، طالما أن أكبر طبقة قادرة على احتواء وحدة معالجة الرسومات الخاصة بك.

<Tip>

يمكن استخدام وحدات معالجة الرسومات المتعددة، ولكن هذا يعتبر "توازي النماذج" ونتيجة لذلك، ستكون وحدة معالجة الرسومات واحدة فقط نشطة في أي وقت، في انتظار استلام الإخراج من الوحدة السابقة. يجب عليك تشغيل البرنامج النصي الخاص بك بشكل طبيعي مع `python`
وليس هناك حاجة إلى `torchrun`، `accelerate launch`، إلخ.

</Tip>

للحصول على تمثيل مرئي لهذا، تحقق من الرسوم المتحركة أدناه:

<Youtube id="MWCSGj9jEAo" />

### مثال كامل

فيما يلي المثال الكامل الذي يوضح ما قمنا به أعلاه:

```py
import torch
from accelerate import init_empty_weights, load_checkpoint_and_dispatch

with init_empty_weights():
model = MyModel(...)

model = load_checkpoint_and_dispatch(
    model, checkpoint=checkpoint_file, device_map="auto"
)

input = torch.randn(2,3)
input = input.to("cuda")
output = model(input)
```

## استخدام 🤗 Transformers و 🤗 Diffusers ومكتبات المصادر المفتوحة الأخرى لـ 🤗

تتضمن المكتبات التي تدعم استنتاج النماذج الكبيرة في 🤗 Accelerate كل المنطق السابق في برامجها `from_pretrained` البنائية.

تعمل هذه البرامج من خلال تحديد سلسلة تمثل النموذج الذي سيتم تنزيله من [🤗 Hub](https://hf.co/models) ثم الإشارة إلى `device_map="auto"` إلى جانب بعض المعلمات الإضافية.

كمثال موجز، سنلقي نظرة على استخدام `transformers` وتحميل نموذج T0pp الخاص بـ Big Science.

```py
from transformers import AutoModelForSeq2SeqLM

model = AutoModelForSeq2SeqLM.from_pretrained("bigscience/T0pp", device_map="auto")
```

بعد تحميل النموذج، يتم تنفيذ الخطوات الأولية من قبل لإعداد النموذج وقد تم تنفيذها جميعًا، والنموذج جاهز تمامًا للاستفادة من جميع الموارد الموجودة في جهازك. من خلال هذه البرامج البنائية، يمكنك أيضًا توفير *المزيد* من الذاكرة عن طريق
تحديد الدقة التي يتم تحميل النموذج بها أيضًا، من خلال معلمة `torch_dtype`، مثل:

```py
from transformers import AutoModelForSeq2SeqLM

model = AutoModelForSeq2SeqLM.from_pretrained("bigscience/T0pp", device_map="auto", torch_dtype=torch.float16)
```

لمعرفة المزيد حول هذا الموضوع، راجع وثائق 🤗 Transformers المتوفرة [هنا](https://huggingface.co/docs/transformers/main/en/main_classes/model#large-model-loading).

## إلى أين تذهب من هنا

للحصول على نظرة أكثر تفصيلاً على استنتاج النماذج الكبيرة، تأكد من مراجعة [الدليل المفاهيمي حوله](../concept_guides/big_model_inference)