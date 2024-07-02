# الاستدلال الموزع باستخدام 🤗 Accelerate

يمكن أن يقع الاستدلال الموزع في ثلاث فئات:

1. تحميل نموذج كامل على كل GPU وإرسال أجزاء من دفعة عبر نسخة النموذج لكل GPU في وقت واحد
2. تحميل أجزاء من نموذج على كل GPU ومعالجة إدخال واحد في وقت واحد
3. تحميل أجزاء من نموذج على كل GPU واستخدام ما يسمى موازاة الأنابيب المجدولة لدمج الطريقتين السابقتين.

سنمر عبر القوس الأول والأخير، ونعرض كيفية القيام بكل منهما لأنهما سيناريوهات أكثر واقعية.

## إرسال أجزاء من دفعة تلقائيًا إلى كل نموذج محمل

هذا هو الحل الأكثر كثافة في الذاكرة، لأنه يتطلب من كل GPU الاحتفاظ بنسخة كاملة من النموذج في الذاكرة في وقت معين.

عادةً عند القيام بذلك، يقوم المستخدمون بإرسال النموذج إلى جهاز محدد لتحميله من وحدة المعالجة المركزية، ثم نقل كل موجه إلى جهاز مختلف.

قد يبدو خط الأنابيب الأساسي باستخدام مكتبة `diffusers` شيئًا ما على النحو التالي:

```python
import torch
import torch.distributed as dist
from diffusers import DiffusionPipeline

pipe = DiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5", torch_dtype=torch.float16)
```

ثم يليه إجراء الاستدلال بناءً على الموجه المحدد:

```python
def run_inference(rank, world_size):
    dist.init_process_group("nccl", rank=rank, world_size=world_size)
    pipe.to(rank)

    if torch.distributed.get_rank() == 0:
        prompt = "a dog"
    elif torch.distributed.get_rank() == 1:
        prompt = "a cat"

    result = pipe(prompt).images[0]
    result.save(f"result_{rank}.png")
```

سيُلاحظ المرء كيف يتعين علينا التحقق من الترتيب لمعرفة الموجه الذي سيتم إرساله، والذي قد يكون مرهقًا بعض الشيء.

قد يفكر المستخدم أيضًا أنه باستخدام 🤗 Accelerate، فإن استخدام `Accelerator` لإعداد برنامج تحميل البيانات لمهمة كهذه قد يكون أيضًا طريقة بسيطة لإدارتها. (لمعرفة المزيد، راجع القسم ذي الصلة في [الجولة السريعة](../quicktour#distributed-evaluation))

هل يمكنه إدارتها؟ نعم. ولكن هل يضيف أيضًا رمزًا إضافيًا غير ضروري: نعم أيضًا.

مع 🤗 Accelerate، يمكننا تبسيط هذه العملية باستخدام مدير سياق [`Accelerator.split_between_processes`] (الذي يوجد أيضًا في `PartialState` و`AcceleratorState`).

ستقوم هذه الدالة تلقائيًا بتقسيم أي بيانات تقوم بإمرارها إليها (سواء كان موجهًا أو مجموعة من المنسوجات أو قاموسًا للبيانات السابقة، إلخ.) عبر جميع العمليات (مع إمكانية التبطين) لتتمكن من استخدامها على الفور.

دعنا نعيد كتابة المثال أعلاه باستخدام مدير السياق هذا:

```python
from accelerate import PartialState  # Can also be Accelerator or AcceleratorState
from diffusers import DiffusionPipeline

pipe = DiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5", torch_dtype=torch.float16)
distributed_state = PartialState()
pipe.to(distributed_state.device)

# Assume two processes
with distributed_state.split_between_processes(["a dog", "a cat"]) as prompt:
    result = pipe(prompt).images[0]
    result.save(f"result_{distributed_state.process_index}.png")
```

ولإطلاق الكود، يمكننا استخدام 🤗 Accelerate:

إذا قمت بتوليد ملف تكوين لاستخدامه باستخدام `accelerate config`:

```bash
accelerate launch distributed_inference.py
```

إذا كان لديك ملف تكوين محدد تريد استخدامه:

```bash
accelerate launch --config_file my_config.json distributed_inference.py
```

أو إذا كنت لا ترغب في إنشاء أي ملفات تكوين وتشغيلها على وحدتي GPU:

> ملاحظة: ستحصل على بعض التحذيرات بشأن القيم التي يتم تخمينها بناءً على نظامك. لإزالة هذه التحذيرات، يمكنك تشغيل `accelerate config default` أو الانتقال عبر `accelerate config` لإنشاء ملف تكوين.

```bash
accelerate launch --num_processes 2 distributed_inference.py
```

لقد قمنا الآن بتخفيض رمز الحشو المطلوب لتقسيم هذه البيانات إلى بضع سطور من التعليمات البرمجية بسهولة.

ولكن ماذا لو كان لدينا توزيع غريب للموجهات إلى وحدات معالجة الرسومات؟ على سبيل المثال، ماذا لو كان لدينا 3 موجهات، ولكن فقط 2 GPU؟

في إطار مدير السياق، ستتلقى وحدة معالجة الرسومات الأولى الموجهين الأولين وستتلقى وحدة معالجة الرسومات الثانية الثالثة، مما يضمن تقسيم جميع الموجهات وعدم وجود نفقات عامة.

ومع ذلك، ماذا لو أردنا بعد ذلك القيام بشيء ما بنتائج *جميع وحدات معالجة الرسومات*؟ (قل جمعها جميعًا وأداء بعض أنواع ما بعد المعالجة)

يمكنك تمرير `apply_padding=True` لضمان أن قوائم الموجهات مبطنة إلى نفس الطول، مع أخذ البيانات الإضافية من العينة الأخيرة. بهذه الطريقة، سيكون لدى جميع وحدات معالجة الرسومات نفس عدد الموجهات، ويمكنك بعد ذلك جمع النتائج.

<Tip>
هذا مطلوب فقط عند محاولة إجراء إجراء مثل جمع النتائج، حيث تحتاج البيانات على كل جهاز إلى أن يكون لها نفس الطول. لا يتطلب الاستدلال الأساسي ذلك.
</Tip>

على سبيل المثال:

```python
from accelerate import PartialState  # Can also be Accelerator or AcceleratorState
from diffusers import DiffusionPipeline

pipe = DiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5", torch_dtype=torch.float16)
distributed_state = PartialState()
pipe.to(distributed_state.device)

# Assume two processes
with distributed_state.split_between_processes(["a dog", "a cat", "a chicken"], apply_padding=True) as prompt:
    result = pipe(prompt).images
```

على وحدة معالجة الرسومات الأولى، ستكون الموجهات `[“a dog”، “a cat”]`، وعلى وحدة معالجة الرسومات الثانية ستكون `[“a chicken”، “a chicken”]`.

تأكد من إسقاط العينة الأخيرة، لأنها ستكون نسخة مكررة من السابقة.

يمكنك العثور على أمثلة أكثر تعقيدًا [هنا](https://github.com/huggingface/accelerate/tree/main/examples/inference/distributed) مثل كيفية استخدامه مع LLMs.

## موازاة الأنابيب الموفرة للذاكرة (تجريبي)

سيناقش هذا الجزء التالي استخدام *موازاة الأنابيب*. هذا هو واجهة برمجة التطبيقات **التجريبية** التي تستخدم [مكتبة PiPPy من PyTorch](https://github.com/pytorch/PiPPy/) كحل أصلي.

الفكرة العامة لموازاة الأنابيب هي: لنفترض أن لديك 4 وحدات معالجة الرسومات ونموذجًا كبيرًا بما يكفي يمكن تقسيمه على أربع وحدات معالجة رسومات باستخدام `device_map="auto"`. باستخدام هذه الطريقة، يمكنك إرسال 4 مدخلات في وقت واحد (على سبيل المثال هنا، يعمل أي مبلغ) وسيعمل كل جزء من النموذج على إدخال، ثم يتلقى الإدخال التالي بمجرد انتهاء الجزء السابق، مما يجعله *أكثر* كفاءة **وأسرع** من الطريقة الموضحة سابقًا. إليك رسم توضيحي مأخوذ من مستودع PyTorch:

![مثال PiPPy](https://camo.githubusercontent.com/681d7f415d6142face9dd1b837bdb2e340e5e01a58c3a4b119dea6c0d99e2ce0/68747470733a2f2f692e696d6775722e636f6d2f65793555633934372e706e67)

لتوضيح كيفية استخدام هذا مع Accelerate، فقد أنشأنا [مثالاً للحيوانات الأليفة](https://github.com/huggingface/accelerate/tree/main/examples/inference) يُظهر عددًا من النماذج والمواقف المختلفة. في هذا البرنامج التعليمي، سنُظهر هذه الطريقة لـ GPT2 عبر وحدتي GPU.

قبل المتابعة، يرجى التأكد من تثبيت أحدث إصدار من PiPPy عن طريق تشغيل ما يلي:

```bash
pip install torchpippy
```

نحن نطلب الإصدار 0.2.0 على الأقل. للتأكد من أن لديك الإصدار الصحيح، قم بتشغيل `pip show torchpippy`.

ابدأ بإنشاء النموذج على وحدة المعالجة المركزية:

```{python}
from transformers import GPT2ForSequenceClassification, GPT2Config

config = GPT2Config()
model = GPT2ForSequenceClassification(config)
model.eval()
```

بعد ذلك، ستحتاج إلى إنشاء بعض إدخالات المثال لاستخدامها. تساعد هذه الإدخالات PiPPy في تتبع النموذج.

<Tip warning={true}>
ومع ذلك، فإن الطريقة التي تقوم بها بهذا المثال ستحدد حجم الدفعة النسبية التي سيتم استخدامها/تمريرها
من خلال النموذج في وقت معين، لذا تأكد من تذكر عدد العناصر الموجودة!
</Tip>

```{python}
input = torch.randint(
low=0،
high=config.vocab_size،
size=(2, 1024)، # bs x seq_len
device="cpu"،
dtype=torch.int64،
requires_grad=False،
)
```

بعد ذلك، نحتاج إلى إجراء التتبع بالفعل وإعداد النموذج. للقيام بذلك، استخدم دالة [`inference.prepare_pippy`] وسيتم لف النموذج بالكامل لموازاة الأنابيب تلقائيًا:

```{python}
from accelerate.inference import prepare_pippy
example_inputs = {"input_ids": input}
model = prepare_pippy(model, example_args=(input,))
```

<Tip>
هناك مجموعة متنوعة من المعلمات التي يمكنك تمريرها إلى `prepare_pippy`:
* يسمح `split_points` بتحديد الطبقات لتقسيم النموذج عندها. بشكل افتراضي، نستخدم المكان الذي يعلن فيه `device_map="auto"`، مثل `fc` أو `conv1`.
* يحدد `num_chunks` كيفية تقسيم الدفعة وإرسالها إلى النموذج نفسه (لذا فإن `num_chunks=1` مع أربع نقاط تقسيم/أربع وحدات معالجة رسومات سيكون لها MP ساذج حيث يتم تمرير إدخال واحد بين نقاط التقسيم الأربع للطبقة)
</Tip>

من هنا، كل ما تبقى هو إجراء الاستدلال الموزع بالفعل!

<Tip warning={true}>
عند تمرير الإدخالات، نوصي بشدة بتمريرها كمجموعة من الحجج. يتم دعم استخدام `kwargs`، ومع ذلك، فإن هذا النهج تجريبي.
</Tip>

```python
args = some_more_arguments
with torch.no_grad ():
    output = model (* args)
```

عندما تنتهي جميع البيانات ستكون على العملية الأخيرة فقط:

```python
from accelerate import PartialState
if PartialState().is_last_process:
    print(output)
```

<Tip>
إذا قمت بتمرير `gather_output=True` إلى [`inference.prepare_pippy`]، فسيتم إرسال الإخراج
عبر جميع وحدات معالجة الرسومات بعد ذلك دون الحاجة إلى التحقق من `is_last_process`. هذا هو
`False` بشكل افتراضي لأنه يتطلب مكالمة اتصال.
</Tip>

وهذا كل شيء! لمزيد من الاستكشاف، يرجى الاطلاع على أمثلة الاستدلال في [مستودع Accelerate](https://github.com/huggingface/accelerate/tree/main/examples/inference/pippy) ووثائقنا [](../package_reference/inference) أثناء عملنا على تحسين هذا التكامل.