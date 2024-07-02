# استكشاف الأخطاء وإصلاحها

إذا واجهتك أي مشكلة عند استخدام PEFT، يرجى التحقق من قائمة المشكلات الشائعة وحلولها التالية.

## عدم عمل الأمثلة

غالباً ما تعتمد الأمثلة على أحدث إصدارات الحزم، لذا يرجى التأكد من تحديثها. وعلى وجه الخصوص، تحقق من إصدارات الحزم التالية:

- `peft`
- `transformers`
- `accelerate`
- `torch`

بشكل عام، يمكنك تحديث إصدار الحزمة بتشغيل هذا الأمر داخل بيئة Python الخاصة بك:

```bash
python -m pip install -U <package_name>
```

يمكن أن يكون تثبيت PEFT من المصدر مفيدًا لمواكبة أحدث التطورات:

```bash
python -m pip install git+https://github.com/huggingface/peft
```

## خطأ القيمة: محاولة إلغاء تحجيم تدرجات FP16

من المحتمل أن يكون هذا الخطأ قد حدث لأن النموذج تم تحميله باستخدام `torch_dtype=torch.float16` ثم تم استخدامه في سياق الدقة المتنقلة التلقائية (AMP)، على سبيل المثال، عن طريق تعيين `fp16=True` في فئة [`~transformers.Trainer`] من حزمة 🤗 Transformers. والسبب هو أنه عند استخدام AMP، لا ينبغي أبدًا استخدام الدقة العائمة fp16 في الأوزان القابلة للتدريب. لجعل هذا الأمر يعمل دون تحميل النموذج بالكامل في fp32، أضف ما يلي إلى كودك:

```python
peft_model = get_peft_model(...)

# add this:
for param in model.parameters():
    if param.requires_grad:
        param.data = param.data.float()

# proceed as usual
trainer = Trainer(model=peft_model, fp16=True, ...)
trainer.train()
```

بدلاً من ذلك، يمكنك استخدام وظيفة [`~utils.cast_mixed_precision_params`] لتحويل الأوزان بشكل صحيح:

```python
from peft import cast_mixed_precision_params

peft_model = get_peft_model(...)
cast_mixed_precision_params(peft_model, dtype=torch.float16)

# تابع كالمعتاد
trainer = Trainer(model=peft_model, fp16=True, ...)
trainer.train()
```

<Tip>

بدءًا من إصدار PEFT v0.11.0، يقوم PEFT تلقائيًا بترقية نوع بيانات أوزان المحول من `torch.float16` و`torch.bfloat16` إلى `torch.float32` عند الاقتضاء. لمنع هذا السلوك، يمكنك تمرير `autocast_adapter_dtype=False` إلى [`~get_peft_model` ]، إلى [`~PeftModel.from_pretrained` ]، وإلى [`~PeftModel.load_adapter`].

</Tip>

## نتائج سيئة من نموذج PEFT محمل

قد يكون هناك عدة أسباب للحصول على نتيجة سيئة من نموذج PEFT محمل، وهي مدرجة أدناه. إذا كنت لا تزال غير قادر على استكشاف المشكلة وإصلاحها، تحقق مما إذا كان أي شخص آخر قد واجه مشكلة مماثلة على [GitHub](https://github.com/huggingface/peft/issues)، وإذا لم تتمكن من العثور على أي منها، فقم بفتح مشكلة جديدة.

عند فتح مشكلة، سيكون من المفيد جدًا إذا قدمت مثالًا برمجيًا بسيطًا يتكاثر المشكلة. أيضًا، يرجى الإبلاغ عما إذا كان النموذج المحمل يعمل بنفس مستوى النموذج قبل الضبط الدقيق، أو إذا كان يعمل بمستوى عشوائي، أو إذا كان أسوأ قليلاً من المتوقع. تساعدنا هذه المعلومات على تحديد المشكلة بشكل أسرع.

### الانحرافات العشوائية

إذا كانت نتائج النموذج غير مطابقة تمامًا للتشغيل السابق، فقد تكون هناك مشكلة في العناصر العشوائية. على سبيل المثال:

1. يرجى التأكد من أنه في وضع `.eval()`، وهو أمر مهم، على سبيل المثال، إذا كان النموذج يستخدم الإسقاط

2. إذا كنت تستخدم [`~transformers.GenerationMixin.generate`] على نموذج اللغة، فقد يكون هناك عينات عشوائية، لذا يتطلب الحصول على نفس النتيجة تعيين بذرة عشوائية

3. إذا كنت قد استخدمت التكميم ودمجت الأوزان، فمن المتوقع حدوث انحرافات طفيفة بسبب أخطاء التقريب

### نموذج محمل بشكل غير صحيح

يرجى التأكد من تحميل النموذج بشكل صحيح. أحد الأخطاء الشائعة هو محاولة تحميل نموذج _trained_ باستخدام [`get_peft_model`] وهو أمر غير صحيح. بدلاً من ذلك، يجب أن تبدو كود التحميل على النحو التالي:

```python
from peft import PeftModel, PeftConfig

base_model = ...  # لتحميل النموذج الأساسي، استخدم نفس الكود كما فعلت عند تدريبه
config = PeftConfig.from_pretrained(peft_model_id)
peft_model = PeftModel.from_pretrained(base_model, peft_model_id)
```

### طبقات تم تهيئتها بشكل عشوائي

بالنسبة لبعض المهام، من المهم تهيئة `modules_to_save` في التهيئة بشكل صحيح لحساب الطبقات التي تم تهيئتها بشكل عشوائي.

كمثال على ذلك، هذا أمر ضروري إذا كنت تستخدم LoRA لضبط نموذج اللغة بشكل دقيق لتصنيف التسلسل لأن 🤗 Transformers تضيف رأس تصنيف تم تهيئته بشكل عشوائي أعلى النموذج. إذا لم تضف هذه الطبقة إلى `modules_to_save`، فلن يتم حفظ رأس التصنيف. في المرة التالية التي تقوم فيها بتحميل النموذج، ستحصل على رأس تصنيف _different_ تم تهيئته بشكل عشوائي، مما يؤدي إلى نتائج مختلفة تمامًا.

يحاول PEFT تخمين `modules_to_save` بشكل صحيح إذا قدمت حجة `task_type` في التهيئة. يجب أن يعمل هذا مع نماذج المحولات التي تتبع مخطط التسمية القياسي. من الجيد دائمًا التحقق المزدوج على الرغم من أننا لا نستطيع ضمان اتباع جميع النماذج لمخطط التسمية.

عندما تقوم بتحميل نموذج محولات له طبقات تم تهيئتها بشكل عشوائي، يجب أن تشاهد تحذيرًا على النحو التالي:

```
لم يتم تهيئة بعض أوزان <MODEL> من نقطة تفتيش النموذج في <ID> وتم تهيئتها حديثًا: [<LAYER_NAMES>].
يجب عليك على الأرجح تدريب هذا النموذج على مهمة لأسفل لتتمكن من استخدامه للتنبؤات والاستدلال.
```

يجب إضافة الطبقات المذكورة إلى `modules_to_save` في التهيئة لتجنب المشكلة الموضحة.

### توسيع المفردات

بالنسبة للعديد من مهام ضبط اللغة الدقيق، من الضروري توسيع مفردات النموذج حيث يتم تقديم رموز جديدة. يتطلب ذلك توسيع طبقة التضمين لحساب الرموز الجديدة، وكذلك تخزين طبقة التضمين بالإضافة إلى أوزان المحول عند حفظ المحول.

احفظ طبقة التضمين عن طريق إضافتها إلى `target_modules` من التهيئة. يجب أن يتبع اسم طبقة التضمين مخطط التسمية القياسي من Transformers. على سبيل المثال، يمكن أن تبدو تهيئة Mistral على النحو التالي:

```python
config = LoraConfig(..., target_modules=["embed_tokens"، "lm_head"، "q_proj"، "v_proj"])
```

بمجرد إضافته إلى `target_modules`، يقوم PEFT تلقائيًا بتخزين طبقة التضمين عند حفظ المحول إذا كان للنموذج [`~transformers.PreTrainedModel.get_input_embeddings`] و [`~transformers.PreTrainedModel.get_output_embeddings`]. هذا هو الحال بشكل عام لنماذج Transformers.

إذا لم تتبع طبقة تضمين النموذج مخطط التسمية الخاص بـ Transformers، فيمكنك حفظها عن طريق تمرير `save_embedding_layers=True` يدويًا عند حفظ المحول:

```python
model = get_peft_model(...)
# تدريب النموذج
model.save_pretrained("my_adapter"، save_embedding_layers=True)
```

للعرض، قم بتحميل النموذج الأساسي أولاً وقم بتغيير حجمه بنفس الطريقة التي قمت بها قبل تدريب النموذج. بعد أن قمت بتغيير حجم النموذج الأساسي، يمكنك تحميل نقطة تفتيش PEFT.

للحصول على مثال كامل، يرجى الاطلاع على [هذا الدفتر](https://github.com/huggingface/peft/blob/main/examples/causal_language_modeling/peft_lora_clm_with_additional_tokens.ipynb).

### التحقق من حالة الطبقة والنموذج

في بعض الأحيان، قد ينتهي نموذج PEFT في حالة سيئة، خاصة عند التعامل مع محولات متعددة. قد يكون هناك بعض الارتباك حول المحولات الموجودة، وأيها نشط، وأيها تم دمجه، وما إلى ذلك. للمساعدة في التحقيق في هذه المشكلة، قم باستدعاء طرق [`~peft.PeftModel.get_layer_status`] و [`~peft.PeftModel.get_model_status`].

توفر طريقة [`~peft.PeftModel.get_layer_status`] نظرة عامة مفصلة عن المحولات النشطة والمدمجة والمتاحة لكل طبقة مستهدفة.

```python
>>> from transformers import AutoModel
>>> from peft import get_peft_model, LoraConfig

>>> model_id = "google/flan-t5-small"
>>> model = AutoModel.from_pretrained(model_id)
>>> model = get_peft_model(model, LoraConfig())

>>> model.get_layer_status()
[TunerLayerStatus(name='model.encoder.block.0.layer.0.SelfAttention.q',
                  module_type='lora.Linear',
                  enabled=True,
                  active_adapters=['default'],
                  merged_adapters=[],
                  requires_grad={'default': True},
                  available_adapters=['default']),
 TunerLayerStatus(name='model.encoder.block.0.layer.0.SelfAttention.v',
                  module_type='lora.Linear',
                  enabled=True,
                  active_adapters=['default'],
                  merged_adapters=[],
                  requires_grad={'default': True},
                  available_adapters=['default']),
...]

>>> model.get_model_status()
TunerModelStatus(
    base_model_type='T5Model',
    adapter_model_type='LoraModel',
    peft_types={'default': 'LORA'},
    trainable_params=344064,
    total_params=60855680,
    num_adapter_layers=48,
    enabled=True,
    active_adapters=['default'],
    merged_adapters=[],
    requires_grad={'default': True},
    available_adapters=['default'],
)
```

في إخراج حالة النموذج، يجب الانتباه إلى الإدخالات التي تقول "irregular". وهذا يعني أن PEFT اكتشف حالة غير متسقة في النموذج. على سبيل المثال، إذا كان `merged_adapters="irregular"`، فهذا يعني أنه بالنسبة لمحول واحد على الأقل، تم دمجه في بعض وحدات الطبقة المستهدفة ولكن ليس في غيرها. من المحتمل أن تكون نتائج الاستدلال غير صحيحة نتيجة لذلك.

أفضل طريقة لحل هذه المشكلة هي إعادة تحميل نقطة تفتيش النموذج ومحول (ات) بالكامل. تأكد من أنك لا تقوم بأي عمليات غير صحيحة على النموذج، على سبيل المثال، دمج المحولات يدويًا في بعض الوحدات ولكن ليس في وحدات أخرى.

قم بتحويل حالة الطبقة إلى إطار بيانات Pandas للفحص المرئي الأسهل.

```python
from dataclasses import asdict
import pandas as pd

df = pd.DataFrame(asdict(layer) for layer in model.get_layer_status())
```

من الممكن الحصول على هذه المعلومات لنماذج غير PEFT إذا كانت تستخدم طبقات PEFT تحت الغطاء، ولكن بعض المعلومات مثل `base_model_type` أو `peft_types` لا يمكن تحديدها في هذه الحالة. كمثال على ذلك، يمكنك استدعاء هذا على نموذج [diffusers](https://huggingface.co/docs/diffusers/index) مثل هذا:

```python
>>> import torch
>>> from diffusers import StableDiffusionPipeline
>>> from peft import get_model_status, get_layer_status

>>> path = "runwayml/stable-diffusion-v1-5"
>>> lora_id = "takuma104/lora-test-text-encoder-lora-target"
>>> pipe = StableDiffusionPipeline.from_pretrained(path, torch_dtype=torch.float16)
>>> pipe.load_lora_weights(lora_id, adapter_name="adapter-1")
>>> pipe.load_lora_weights(lora_id, adapter_name="adapter-2")
>>> pipe.set_lora_device(["adapter-2"], "cuda")
>>> get_layer_status(pipe.text_encoder)
[TunerLayerStatus(name='text_model.encoder.layers.0.self_attn.k_proj',
                  module_type='lora.Linear',
                  enabled=True,
                  active_adapters=['adapter-2'],
                  merged_adapters=[],
                  requires_grad={'adapter-1': False, 'adapter-2': True},
                  available_adapters=['adapter-1', 'adapter-2'],
                  devices={'adapter-1': ['cpu'], 'adapter-2': ['cuda']}),
 TunerLayerStatus(name='text_model.encoder.layers.0.self_attn.v_proj',
                  module_type='lora.Linear',
                  enabled=True,
                  active_adapters=['adapter-2'],
                  merged_adapters=[],
                  requires_grad={'adapter-1': False, 'adapter-2': True},
                  devices={'adapter-1': ['cpu'], 'adapter-2': ['cuda']}),
...]

>>> get_model_status(pipe.unet)
TunerModelStatus(
    base_model_type='other',
    adapter_model_type='None',
    peft_types={},
    trainable_params=797184,
    total_params=861115332,
    num_adapter_layers=128,
    enabled=True,
    active_adapters=['adapter-2'],
    merged_adapters=[],
    requires_grad={'adapter-1': False, 'adapter-2': True},
    available_adapters=['adapter-1', 'adapter-2'],
    devices={'adapter-1': ['cpu'], 'adapter-2': ['cuda']},
)
```
## إمكانية إعادة الإنتاج

### النماذج التي تستخدم معيار الدفعة

عند تحميل نموذج PEFT مدرب حيث يستخدم النموذج الأساسي معيار الدفعة (على سبيل المثال، `torch.nn.BatchNorm1d` أو `torch.nn.BatchNorm2d`)، قد تجد أنك لا تستطيع إعادة إنتاج نفس المخرجات بالضبط. ويرجع ذلك إلى أن طبقات معيار الدفعة تحتفظ بإحصائيات التشغيل أثناء التدريب، ولكن هذه الإحصائيات ليست جزءًا من نقطة تفتيش PEFT. لذلك، عند تحميل نموذج PEFT، يتم استخدام الإحصائيات التشغيلية للنموذج الأساسي (أي من قبل التدريب مع PEFT).

اعتمادًا على حالتك الاستخدامية، قد لا يكون هذا الأمر مهمًا. إذا كنت بحاجة إلى أن تكون المخرجات قابلة لإعادة الإنتاج بنسبة 100٪، فيمكنك تحقيق ذلك عن طريق إضافة طبقات معيار الدفعة إلى `modules_to_save`. وفيما يلي مثال على ذلك باستخدام resnet وLoRA. لاحظ أننا قمنا بتعيين `modules_to_save=["classifier"، "normalization"]`. نحتاج إلى وسيط "المصنف" لأن مهمتنا هي تصنيف الصور، ونضيف وسيط "التطبيع" لضمان حفظ طبقات معيار الدفعة في نقطة تفتيش PEFT.

```python
from transformers import AutoModelForImageClassification
from peft import LoraConfig, get_peft_model

model_id = "microsoft/resnet-18"
base_model = AutoModelForImageClassification.from_pretrained(self.model_id)
config = LoraConfig(
    target_modules=["convolution"],
    modules_to_save=["classifier", "normalization"],
),
```

اعتمادًا على نوع النموذج الذي تستخدمه، قد يكون لطبقات معيار الدفعة أسماء مختلفة عن "التطبيع"، لذا يرجى التأكد من أن الاسم يتطابق مع بنية نموذجك.