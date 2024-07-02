# جولة سريعة

يوفر PEFT طرقًا فعالة من حيث التكلفة لضبط دقة النماذج المسبقة الضخمة. النموذج التقليدي هو ضبط دقة جميع معلمات النموذج لكل مهمة من مهام التدفق السفلي، ولكن هذا أصبح مكلفًا للغاية وغير عملي بسبب العدد الهائل من المعلمات في النماذج اليوم. بدلاً من ذلك، من الأكثر كفاءة تدريب عدد أقل من معلمات الإشارة أو استخدام طريقة إعادة المعلمية مثل التكيف منخفض الرتبة (LoRA) لتقليل عدد المعلمات القابلة للتدريب.

ستعرض لك هذه الجولة السريعة الميزات الرئيسية لـ PEFT وكيفية تدريبك أو تشغيل الاستدلال على النماذج الكبيرة التي عادة ما يتعذر الوصول إليها على الأجهزة الاستهلاكية.

## تدريب

يتم تعريف كل طريقة PEFT بواسطة فئة [`PeftConfig`] التي تخزن جميع المعلمات المهمة لبناء [`PeftModel`]. على سبيل المثال، لتدريب مع LoRA، قم بتحميل وإنشاء فئة [`LoraConfig`] وحدد المعلمات التالية:

- `task_type`: المهمة التي سيتم التدريب عليها (نمذجة اللغة من التسلسل إلى التسلسل في هذه الحالة)
- `inference_mode`: ما إذا كنت تستخدم النموذج للاستدلال أم لا
- `r`: بعد المصفوفات منخفضة الرتبة
- `lora_alpha`: عامل القياس للمصفوفات منخفضة الرتبة
- `lora_dropout`: احتمال إسقاط طبقات LoRA

```python
from peft import LoraConfig, TaskType

peft_config = LoraConfig(task_type=TaskType.SEQ_2_SEQ_LM, inference_mode=False, r=8, lora_alpha=32, lora_dropout=0.1)
```

<Tip>
راجع مرجع [`LoraConfig`] لمزيد من التفاصيل حول المعلمات الأخرى التي يمكنك ضبطها، مثل الوحدات المستهدفة أو نوع الانحياز.
</Tip>

بمجرد إعداد [`LoraConfig`]، قم بإنشاء [`PeftModel`] باستخدام دالة [`get_peft_model`]. يأخذ نموذجًا أساسيًا - يمكنك تحميله من مكتبة المحولات - و [`LoraConfig`] الذي يحتوي على المعلمات الخاصة بكيفية تكوين نموذج للتدريب مع LoRA.

قم بتحميل النموذج الأساسي الذي تريد ضبط دقته.

```python
from transformers import AutoModelForSeq2SeqLM

model = AutoModelForSeq2SeqLM.from_pretrained("bigscience/mt0-large")
```

قم بتغليف النموذج الأساسي و `peft_config` مع دالة [`get_peft_model`] لإنشاء [`PeftModel`]. للحصول على فكرة عن عدد المعلمات القابلة للتدريب في نموذجك، استخدم طريقة [`print_trainable_parameters`].

```python
from peft import get_peft_model

model = get_peft_model(model, peft_config)
model.print_trainable_parameters()
"output: trainable params: 2359296 || all params: 1231940608 || trainable%: 0.19151053100118282"
```

من بين 1.2 مليار معلمة من [bigscience/mt0-large](https://huggingface.co/bigscience/mt0-large)، فإنك تدرب فقط 0.19٪ منهم!

هذا كل شيء 🎉! الآن يمكنك تدريب النموذج باستخدام محولات [`~transformers.Trainer`] أو تسريع أو أي حلقة تدريب PyTorch مخصصة.

على سبيل المثال، لتدريب باستخدام فئة [`~transformers.Trainer`]، قم بإعداد فئة [`~transformers.TrainingArguments`] بمعلمات تدريب مختلفة.

```py
training_args = TrainingArguments(
    output_dir="your-name/bigscience/mt0-large-lora",
    learning_rate=1e-3,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=32,
    num_train_epochs=2,
    weight_decay=0.01,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
)
```

مرر النموذج، وحجج التدريب، ومجموعة البيانات، ومعالج التحليل، وأي مكون ضروري آخر إلى [`~transformers.Trainer`]، واستدعاء [`~transformers.Trainer.train`] لبدء التدريب.

```py
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["test"],
    tokenizer=tokenizer,
    data_collator=data_collator,
    compute_metrics=compute_metrics,
)

trainer.train()
```

### حفظ النموذج

بعد انتهاء النموذج من التدريب، يمكنك حفظ نموذجك في دليل باستخدام وظيفة [`~transformers.PreTrainedModel.save_pretrained`].

```py
model.save_pretrained("output_dir")
```

يمكنك أيضًا حفظ نموذجك على Hub (تأكد من تسجيل الدخول إلى حساب Hugging Face الخاص بك أولاً) باستخدام وظيفة [`~transformers.PreTrainedModel.push_to_hub`].

```python
from huggingface_hub import notebook_login

notebook_login()
model.push_to_hub("your-name/bigscience/mt0-large-lora")
```

تقوم كلتا الطريقتين بحفظ أوزان PEFT الإضافية التي تم تدريبها فقط، مما يعني أنه من الكفء جدًا تخزينها ونقلها وتحميلها. على سبيل المثال، يحتوي هذا النموذج [facebook/opt-350m](https://huggingface.co/ybelkada/opt-350m-lora) المدرب باستخدام LoRA على ملفين فقط: `adapter_config.json` و`adapter_model.safetensors`. ملف `adapter_model.safetensors` هو فقط 6.3MB!

<div class="flex flex-col justify-center">
<img src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/peft/PEFT-hub-screenshot.png"/>
<figcaption class="text-center">أوزان المحول لنموذج opt-350m المخزنة على Hub هي فقط ~6MB مقارنة بالحجم الكامل لأوزان النموذج، والتي يمكن أن تكون ~700MB.</figcaption>
</div>

## الاستدلال

<Tip>
الق نظرة على مرجع [AutoPeftModel](package_reference/auto_class) للحصول على قائمة كاملة من فئات `AutoPeftModel` المتاحة.
</Tip>

يمكنك تحميل أي نموذج مدرب على PEFT للاستدلال بسهولة باستخدام فئة [`AutoPeftModel`] وطريقة [`~transformers.PreTrainedModel.from_pretrained`]:

```py
from peft import AutoPeftModelForCausalLM
from transformers import AutoTokenizer
import torch

model = AutoPeftModelForCausalLM.from_pretrained("ybelkada/opt-350m-lora")
tokenizer = AutoTokenizer.from_pretrained("facebook/opt-350m")

model = model.to("cuda")
model.eval()
inputs = tokenizer("سخن الفرن إلى 350 درجة وضع عجينة الكوكيز"، return_tensors="pt")

outputs = model.generate(input_ids=inputs["input_ids"].to("cuda"), max_new_tokens=50)
print(tokenizer.batch_decode(outputs.detach().cpu().numpy(), skip_special_tokens=True)[0])

"سخن الفرن إلى 350 درجة وضع عجينة الكوكيز في وسط الفرن. في وعاء كبير، اخلطي الدقيق والبيكنج باودر وصودا الخبز والملح والقرفة. في وعاء منفصل، اخفقي صفار البيض والسكر والفانيليا."
```

بالنسبة للمهام الأخرى التي لا يتم دعمها بشكل صريح باستخدام فئة `AutoPeftModelFor` - مثل التعرف التلقائي على الكلام - يمكنك استخدام فئة [`AutoPeftModel`] الأساسية لتحميل نموذج للمهمة.

```py
from peft import AutoPeftModel

model = AutoPeftModel.from_pretrained("smangrul/openai-whisper-large-v2-LORA-colab")
```

## الخطوات التالية

الآن بعد أن رأيت كيفية تدريب نموذج باستخدام إحدى طرق PEFT، نشجعك على تجربة بعض الطرق الأخرى مثل ضبط دقة الإشارة. الخطوات مماثلة لتلك الموضحة في الجولة السريعة:

1. قم بإعداد [`PeftConfig`] لطريقة PEFT
2. استخدم طريقة [`get_peft_model`] لإنشاء [`PeftModel`] من التكوين والنموذج الأساسي

بعد ذلك، يمكنك تدريبه بالطريقة التي تريدها! لتحميل نموذج PEFT للاستدلال، يمكنك استخدام فئة [`AutoPeftModel`].

لا تتردد في إلقاء نظرة على أدلة المهام إذا كنت مهتمًا بتدريب نموذج باستخدام طريقة PEFT أخرى لمهمة محددة مثل التجزئة الدلالية أو التعرف التلقائي على الكلام متعدد اللغات أو DreamBooth أو تصنيف الرموز، وغير ذلك.