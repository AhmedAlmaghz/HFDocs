# النماذج المخصصة

تعد بعض تقنيات الضبط الدقيق، مثل ضبط الفواصل، خاصة بنماذج اللغة. وهذا يعني أنه في 🤗 PEFT، من المفترض استخدام نموذج 🤗 Transformers. ومع ذلك، لا تقتصر تقنيات الضبط الدقيق الأخرى - مثل [LoRA](../conceptual_guides/lora) - على أنواع نماذج محددة.

في هذا الدليل، سنرى كيف يمكن تطبيق LoRA على شبكة عصبية متعددة الطبقات، ونموذج رؤية حاسوبية من مكتبة [timm](https://huggingface.co/docs/timm/index)، أو بنية جديدة لـ 🤗 Transformers.

## الشبكة العصبية متعددة الطبقات

لنفترض أننا نريد ضبط شبكة عصبية متعددة الطبقات باستخدام LoRA. فيما يلي التعريف:

```python
from torch import nn


class MLP(nn.Module):
    def __init__(self, num_units_hidden=2000):
        super().__init__()
        self.seq = nn.Sequential(
            nn.Linear(20, num_units_hidden),
            nn.ReLU(),
            nn.Linear(num_units_hidden, num_units_hidden),
            nn.ReLU(),
            nn.Linear(num_units_hidden, 2),
            nn.LogSoftmax(dim=-1),
        )

    def forward(self, X):
        return self.seq(X)
```

هذه شبكة عصبية متعددة الطبقات مباشرة مع طبقة دخل وطبقة مخفية وطبقة إخراج.

<Tip>

بالنسبة لهذا المثال التوضيحي، نختار عددًا كبيرًا جدًا من الوحدات المخفية لإبراز مكاسب الكفاءة من PEFT، ولكن هذه المكاسب تتماشى مع الأمثلة الأكثر واقعية.

</Tip>

هناك بضع طبقات خطية في هذا النموذج يمكن ضبطها باستخدام LoRA. عندما نعمل مع نماذج 🤗 Transformers الشائعة، سيعرف PEFT الطبقات التي يجب تطبيق LoRA عليها، ولكن في هذه الحالة، يتوقف الأمر علينا كمستخدمين لاختيار الطبقات. لتحديد أسماء الطبقات التي سيتم ضبطها:

```python
print([(n, type(m)) for n, m in MLP().named_modules()])
```

يجب أن يطبع هذا ما يلي:

```
[('', __main__.MLP),
('seq', torch.nn.modules.container.Sequential),
('seq.0', torch.nn.modules.linear.Linear),
('seq.1', torch.nn.modules.activation.ReLU),
('seq.2', torch.nn.modules.linear.Linear),
('seq.3', torch.nn.modules.activation.ReLU),
('seq.4', torch.nn.modules.linear.Linear),
('seq.5', torch.nn.modules.activation.LogSoftmax)]
```

لنفترض أننا نريد تطبيق LoRA على طبقة الإدخال والطبقة المخفية، وهما "seq.0" و"seq.2". علاوة على ذلك، لنفترض أننا نريد تحديث طبقة الإخراج بدون LoRA، والتي ستكون "seq.4". سيكون التكوين المقابل كما يلي:

```python
from peft import LoraConfig

config = LoraConfig(
    target_modules=["seq.0", "seq.2"],
    modules_to_save=["seq.4"],
)
```

مع ذلك، يمكننا إنشاء نموذج PEFT الخاص بنا والتحقق من نسبة المعلمات المدربة:

```python
from peft import get_peft_model

model = MLP()
peft_model = get_peft_model(model, config)
peft_model.print_trainable_parameters()
# prints trainable params: 56,164 || all params: 4,100,164 || trainable%: 1.369798866581922
```

أخيرًا، يمكننا استخدام أي إطار تدريب نريده، أو كتابة حلقة التجهيز الخاصة بنا، لتدريب نموذج "peft_model".

لمثال كامل، راجع [دفتر الملاحظات هذا](https://github.com/huggingface/peft/blob/main/examples/multilayer_perceptron/multilayer_perceptron_lora.ipynb).

## نماذج timm

تحتوي مكتبة [timm](https://huggingface.co/docs/timm/index) على عدد كبير من نماذج الرؤية الحاسوبية المدربة مسبقًا. يمكن أيضًا ضبط تلك النماذج باستخدام PEFT. دعونا نلقي نظرة على كيفية عمل ذلك في الممارسة العملية.

للبدء، تأكد من تثبيت timm في بيئة Python:

```bash
python -m pip install -U timm
```

بعد ذلك، نقوم بتحميل نموذج timm لمهمة تصنيف الصور:

```python
import timm

num_classes = ...
model_id = "timm/poolformer_m36.sail_in1k"
model = timm.create_model(model_id, pretrained=True, num_classes=num_classes)
```

مرة أخرى، يتعين علينا اتخاذ قرار بشأن الطبقات التي سيتم تطبيق LoRA عليها. نظرًا لأن LoRA تدعم طبقات 2D conv، ونظرًا لأن تلك الطبقات هي عنصر بناء رئيسي في هذا النموذج، فيجب علينا تطبيق LoRA على طبقات 2D conv. لتحديد أسماء تلك الطبقات، دعونا نلقي نظرة على جميع أسماء الطبقات:

```python
print([(n, type(m)) for n, m in model.named_modules()])
```

سيقوم هذا بطباعة قائمة طويلة جدًا، وسنظهر فقط أول بضعة منها:

```
[('', timm.models.metaformer.MetaFormer),
('stem', timm.models.metaformer.Stem),
('stem.conv', torch.nn.modules.conv.Conv2d),
('stem.norm', torch.nn.modules.linear.Identity),
('stages', torch.nn.modules.container.Sequential),
('stages.0', timm.models.metaformer.MetaFormerStage),
('stages.0.downsample', torch.nn.modules.linear.Identity),
('stages.0.blocks', torch.nn.modules.container.Sequential),
('stages.0.blocks.0', timm.models.metaformer.MetaFormerBlock),
('stages.0.blocks.0.norm1', timm.layers.norm.GroupNorm1),
('stages.0.blocks.0.token_mixer', timm.models.metaformer.Pooling),
('stages.0.blocks.0.token_mixer.pool', torch.nn.modules.pooling.AvgPool2d),
('stages.0.blocks.0.drop_path1', torch.nn.modules.linear.Identity),
('stages.0.blocks.0.layer_scale1', timm.models.metaformer.Scale),
('stages.0.blocks.0.res_scale1', torch.nn.modules.linear.Identity),
('stages.0.blocks.0.norm2', timm.layers.norm.GroupNorm1),
('stages.0.blocks.0.mlp', timm.layers.mlp.Mlp),
('stages.0.blocks.0.mlp.fc1', torch.nn.modules.conv.Conv2d),
('stages.0.blocks.0.mlp.act', torch.nn.modules.activation.GELU),
('stages.0.blocks.0.mlp.drop1', torch.nn.modules.dropout.Dropout),
('stages.0.blocks.0.mlp.norm', torch.nn.modules.linear.Identity),
('stages.0.blocks.0.mlp.fc2', torch.nn.modules.conv.Conv2d),
('stages.0.blocks.0.mlp.drop2', torch.nn.modules.dropout.Dropout),
('stages.0.blocks.0.drop_path2', torch.nn.modules.linear.Identity),
('stages.0.blocks.0.layer_scale2', timm.models.metaformer.Scale),
('stages.0.blocks.0.res_scale2', torch.nn.modules.linear.Identity),
('stages.0.blocks.1', timm.models.metaformer.MetaFormerBlock),
('stages.0.blocks.1.norm1', timm.layers.norm.GroupNorm1),
('stages.0.blocks.1.token_mixer', timm.models.metaformer.Pooling),
('stages.0.blocks.1.token_mixer.pool', torch.nn.modules.pooling.AvgPool2d),
...
('head.global_pool.flatten', torch.nn.modules.linear.Identity),
('head.norm', timm.layers.norm.LayerNorm2d),
('head.flatten', torch.nn.modules.flatten.Flatten),
('head.drop', torch.nn.modules.linear.Identity),
('head.fc', torch.nn.modules.linear.Linear)]
]
```

عند الفحص الدقيق، نرى أن طبقات 2D conv لها أسماء مثل "stages.0.blocks.0.mlp.fc1" و"stages.0.blocks.0.mlp.fc2". كيف يمكننا مطابقة أسماء الطبقات هذه على وجه التحديد؟ يمكنك كتابة [التعبيرات العادية](https://docs.python.org/3/library/re.html) لمطابقة أسماء الطبقات. في حالتنا، يجب أن يؤدي التعبير العادي `r".*\.mlp\.fc\d"` إلى المهمة.

علاوة على ذلك، كما هو الحال في المثال الأول، يجب علينا التأكد من تحديث طبقة الإخراج، في هذه الحالة رأس التصنيف. بالنظر إلى نهاية القائمة المطبوعة أعلاه، يمكننا أن نرى أنها تسمى "head.fc". مع وضع ذلك في الاعتبار، فيما يلي تكوين LoRA الخاص بنا:

```python
config = LoraConfig(target_modules=r".*\.mlp\.fc\d", modules_to_save=["head.fc"])
```

بعد ذلك، نحتاج فقط إلى إنشاء نموذج PEFT عن طريق تمرير نموذج الأساس وتكوينه إلى `get_peft_model`:

```python
peft_model = get_peft_model(model, config)
peft_model.print_trainable_parameters()
# prints trainable params: 1,064,454 || all params: 56,467,974 || trainable%: 1.88505789139876
```

يظهر هذا أننا نحتاج فقط إلى تدريب أقل من 2% من جميع المعلمات، وهو مكسب كبير في الكفاءة.

لمثال كامل، راجع [دفتر الملاحظات هذا](https://github.com/huggingface/peft/blob/main/examples/image_classification/image_classification_timm_peft_lora.ipynb).

## بنيات Transformers الجديدة

عندما يتم إصدار بنيات Transformers جديدة وشعبية، نبذل قصارى جهدنا لإضافتها بسرعة إلى PEFT. إذا صادفت نموذج Transformers غير مدعوم بشكل افتراضي، فلا تقلق، فمن المحتمل أن يعمل على أي حال إذا تم تعيين التكوين بشكل صحيح. على وجه التحديد، يجب عليك تحديد الطبقات التي يجب تكييفها وتعيينها بشكل صحيح عند تهيئة فئة التكوين المقابلة، على سبيل المثال `LoraConfig`. فيما يلي بعض النصائح التي تساعد في ذلك.

كخطوة أولى، من الجيد التحقق من النماذج الموجودة للحصول على الإلهام. يمكنك العثور عليها داخل [constants.py](https://github.com/huggingface/peft/blob/main/src/peft/utils/constants.py) في مستودع PEFT. غالبًا ما ستجد بنية مماثلة تستخدم نفس الأسماء. على سبيل المثال، إذا كانت بنية النموذج الجديد هي تنوع لنموذج "mistral" وتريد تطبيق LoRA، فيمكنك أن ترى أن الإدخال الخاص بـ "mistral" في `TRANSFORMERS_MODELS_TO_LORA_TARGET_MODULES_MAPPING` يحتوي على `["q_proj"، "v_proj"]`. يخبرك هذا بأنه بالنسبة لنماذج "mistral"، يجب أن تكون `target_modules` لـ LoRA هي `["q_proj"، "v_proj"]`:

```python
from peft import LoraConfig, get_peft_model

my_mistral_model = ...
config = LoraConfig(
    target_modules=["q_proj", "v_proj"],
    ...,  # other LoRA arguments
)
peft_model = get_peft_model(my_mistral_model, config)
```

إذا لم يساعد ذلك، فتحقق من الوحدات النمطية الموجودة في بنية النموذج الخاص بك باستخدام طريقة `named_modules` وحاول تحديد طبقات الاهتمام، خاصة طبقات المفتاح والاستعلام والقيمة. غالبًا ما يكون لهذه الطبقات أسماء مثل `c_attn` و`query` و`q_proj`، وما إلى ذلك. طبقة المفتاح ليست دائمًا مُكيَّفة، ومن الأفضل التحقق مما إذا كان تضمينها يؤدي إلى أداء أفضل.

بالإضافة إلى ذلك، تعد الطبقات الخطية أهدافًا شائعة للتكيف (على سبيل المثال، في [ورقة QLoRA](https://arxiv.org/abs/2305.14314)، يقترح المؤلفون تكييفها أيضًا). غالبًا ما تحتوي أسماء هذه الطبقات على سلاسل الأحرف "fc" أو "dense".

إذا كنت تريد إضافة نموذج جديد إلى PEFT، فيرجى إنشاء إدخال في [constants.py](https://github.com/huggingface/peft/blob/main/src/peft/utils/constants.py) وإرسال طلب سحب على [المستودع](https://github.com/huggingface/peft/pulls). لا تنس تحديث [README](https://github.com/huggingface/peft#models-support-matrix) أيضًا.

## التحقق من المعلمات والطبقات

يمكنك التحقق مما إذا كنت قد طبقت طريقة PEFT على نموذجك بشكل صحيح بعدة طرق.

* تحقق من نسبة المعلمات التي يمكن تدريبها باستخدام طريقة [`~PeftModel.print_trainable_parameters`] . إذا كان هذا الرقم أقل أو أعلى مما هو متوقع، فتحقق من تمثيل النموذج عن طريق طباعة النموذج. يُظهر هذا أسماء جميع أنواع الطبقات في النموذج. تأكد من استبدال الطبقات المستهدفة فقط بطبقات المحول. على سبيل المثال، إذا تم تطبيق LoRA على طبقات `nn.Linear`، فيجب ألا ترى سوى استخدام طبقات `lora.Linear`.

```py
peft_model.print_trainable_parameters()
```

* طريقة أخرى لعرض الطبقات المكيفة هي استخدام السمة `targeted_module_names` لإدراج اسم كل وحدة نمطية تم تكييفها.

```python
print(peft_model.targeted_module_names)
```

## أنواع الوحدات النمطية غير المدعومة

تعمل الطرق مثل LoRA فقط إذا كانت الوحدات النمطية المستهدفة مدعومة بواسطة PEFT. على سبيل المثال، من الممكن تطبيق LoRA على طبقات `nn.Linear` و`nn.Conv2d`، ولكن ليس، على سبيل المثال، على `nn.LSTM`. إذا وجدت أن فئة الطبقة التي تريد تطبيق PEFT عليها غير مدعومة، فيمكنك:

- تحديد تعيين مخصص للتفويض الديناميكي للوحدات النمطية المخصصة في LoRA
- فتح [قضية](https://github.com/huggingface/peft/issues) وطلب الميزة حيث سينفذها المشرفون أو يرشدونك حول كيفية تنفيذها بنفسك إذا كان الطلب على هذا النوع من الوحدات النمطية مرتفعًا بدرجة كافية
### دعم تجريبي للتفريق الديناميكي للوحدات النمطية المخصصة في LoRA

> [!WARNING]
> هذه الميزة تجريبية وقد تتغير حسب استقبال المجتمع لها. سنقدم واجهة برمجة تطبيقات عامة ومستقرة إذا كان هناك طلب كبير عليها.

يدعم PEFT واجهة برمجة تطبيقات تجريبية لأنواع الوحدات النمطية المخصصة لـ LoRA. لنفترض أن لديك تنفيذ LoRA لـ LSTMs. عادة، لن تتمكن من إخبار PEFT باستخدامه، حتى إذا كان من الناحية النظرية يعمل مع PEFT. ومع ذلك، فإن هذا ممكن مع التفريق الديناميكي للطبقات المخصصة.

تبدو واجهة برمجة التطبيقات التجريبية حاليًا على النحو التالي:

```python
class MyLoraLSTMLayer:
...

base_model = ...  # تحميل نموذج الأساس الذي يستخدم LSTMs

# إضافة أسماء طبقة LSTM إلى target_modules
config = LoraConfig(..., target_modules=["lstm"])
# تحديد خريطة من نوع طبقة الأساس إلى نوع طبقة LoRA
custom_module_mapping = {nn.LSTM: MyLoraLSTMLayer}
# تسجيل خريطة جديدة
config._register_custom_module(custom_module_mapping)
# بعد التسجيل، قم بإنشاء نموذج PEFT
peft_model = get_peft_model(base_model, config)
# قم بالتدريب
```

<Tip>

عند استدعاء [`get_peft_model`]`[`get_peft_model`]`، سترى تحذيرًا لأن PEFT لا يتعرف على نوع الوحدة النمطية المستهدفة. في هذه الحالة، يمكنك تجاهل هذا التحذير.

</Tip>

من خلال توفير خريطة مخصصة، يتحقق PEFT أولاً من طبقات نموذج الأساس مقابل الخريطة المخصصة ويقوم بالتفريق إلى نوع طبقة LoRA المخصص إذا كان هناك تطابق. إذا لم يكن هناك تطابق، يتحقق PEFT من أنواع طبقات LoRA المدمجة للتطابق.

لذلك، يمكن أيضًا استخدام هذه الميزة لتجاوز منطق التفريق الموجود، على سبيل المثال إذا كنت تريد استخدام طبقة LoRA الخاصة بك لـ `nn.Linear` بدلاً من استخدام الطبقة التي يوفرها PEFT.

عند إنشاء وحدة LoRA المخصصة الخاصة بك، يرجى اتباع نفس القواعد كما في [وحدات LoRA الموجودة](https://github.com/huggingface/peft/blob/main/src/peft/tuners/lora/layer.py). هناك بعض القيود المهمة التي يجب مراعاتها:

- يجب أن ترث الوحدة المخصصة من `nn.Module` و`peft.tuners.lora.layer.LoraLayer`.
- يجب أن تحتوي طريقة `__init__` للوحدة المخصصة على وسيطين موضعيين `base_layer` و`adapter_name`. بعد ذلك، هناك وسيطات `**kwargs` إضافية يمكنك استخدامها أو تجاهلها.
- يجب تخزين المعلمات القابلة للتعلم في `nn.ModuleDict` أو `nn.ParameterDict`، حيث يتوافق المفتاح مع اسم المحول المحدد (تذكر أنه يمكن أن يكون للنموذج أكثر من محول في نفس الوقت).
- يجب أن يبدأ اسم سمات المعلمات القابلة للتعلم بـ `"lora_"``، على سبيل المثال `self.lora_new_param = ...`.
- بعض الطرق اختيارية، على سبيل المثال، أنت بحاجة فقط إلى تنفيذ `merge` و`unmerge` إذا كنت تريد دعم دمج الأوزان.

حاليًا، لا تستمر معلومات الوحدة النمطية المخصصة عند حفظ النموذج. عند تحميل النموذج، يجب عليك تسجيل الوحدات النمطية المخصصة مرة أخرى.

```python
# يعمل الحفظ دائمًا كما هو ويشمل معلمات الوحدات النمطية المخصصة
peft_model.save_pretrained(<model-path>)

# تحميل النموذج لاحقًا:
base_model = ...
# تحميل تكوين LoRA الذي قمت بحفظه سابقًا
config = LoraConfig.from_pretrained(<model-path>)
# تسجيل الوحدة النمطية المخصصة مرة أخرى، بنفس طريقة المرة الأولى
custom_module_mapping = {nn.LSTM: MyLoraLSTMLayer}
config._register_custom_module(custom_module_mapping)
# تمرير مثيل التكوين إلى from_pretrained:
peft_model = PeftModel.from_pretrained(model, tmp_path / "lora-custom-module"، config=config)
```

إذا كنت تستخدم هذه الميزة وتجدها مفيدة، أو إذا واجهتك مشكلات، فأخبرنا من خلال إنشاء مشكلة أو مناقشة على GitHub. يسمح لنا ذلك بتقدير الطلب على هذه الميزة وإضافة واجهة برمجة تطبيقات عامة إذا كان الطلب مرتفعًا بدرجة كافية.