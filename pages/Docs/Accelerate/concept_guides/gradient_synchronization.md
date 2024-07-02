# مزامنة التدرج 

تعمل الوحدة النمطية الموزعة في PyTorch من خلال التواصل ذهابًا وإيابًا بين جميع وحدات معالجة الرسوميات (GPU) في نظامك. يستغرق هذا التواصل وقتًا، ويحدث التأكد من معرفة جميع العمليات لحالات بعضها البعض في نقاط تشغيل معينة عند استخدام وحدة `ddp`.

تضاف نقاط التشغيل هذه إلى نموذج PyTorch، وتحديدًا إلى طريقتي `forward()` و`backward()` الخاصتين بها. يحدث هذا عندما يتم لف النموذج باستخدام `DistributedDataParallel`:

```python
import torch.nn as nn
from torch.nn.parallel import DistributedDataParallel

model = nn.Linear(10, 10)
ddp_model = DistributedDataParallel(model)
```

في 🤗 Accelerate، يحدث هذا التحويل تلقائيًا عند استدعاء [`~Accelerator.prepare`] وإدخال نموذجك.

```diff
+ from accelerate import Accelerator
+ accelerator = Accelerator()
  import torch.nn as nn
- from torch.nn.parallel import DistributedDataParallel

  model = nn.Linear(10,10)
+ model = accelerator.prepare(model)
```

## التباطؤ في تراكم التدرجات

الآن، أنت تفهم أن PyTorch تضيف خطافات إلى طريقة `forward` و`backward` لنموذج PyTorch الخاص بك عند التدريب في إعداد موزع. ولكن كيف يمكن أن يؤدي هذا إلى إبطاء الكود الخاص بك؟

في DDP (البيانات الموزعة المتوازية)، يُتوقع الترتيب المحدد الذي يتم فيه تنفيذ العمليات وتشغيلها في نقاط معينة، ويجب أيضًا أن تحدث هذه النقاط تقريبًا في نفس الوقت قبل المتابعة.

وأوضح مثال على ذلك هو عند تحديث معلمات النموذج من خلال `optimizer.step()`.

بدون تراكم التدرج، يجب أن يكون لدى جميع مثيلات النموذج تحديث تدرجاتها المحسوبة والمجمّعة والمحدثة قبل الانتقال إلى الدفعة التالية من البيانات.

عند تنفيذ تراكم التدرج، تقوم بتراكم تدرجات فقدان `n` وتخطي `optimizer.step()` حتى يتم الوصول إلى `n` دفعات. نظرًا لأن جميع عمليات التدريب تحتاج فقط إلى مزامنة الوقت الذي يتم فيه استدعاء `optimizer.step()`، دون أي تعديل على خطوة التدريب الخاصة بك، يمكن أن يتسبب هذا التواصل غير الضروري بين العمليات في حدوث تباطؤ كبير.

كيف يمكنك تجنب هذه النفقات العامة؟

## حل مشكلة التباطؤ

نظرًا لأنك تقوم بتخطي تحديثات معلمات النموذج عند التدريب على هذه الدفعات، فلا يلزم مزامنة تدرجاتها حتى النقطة التي يتم فيها استدعاء `optimizer.step()` بالفعل.

لا يمكن لـ PyTorch أن يخبرك تلقائيًا عندما تحتاج إلى القيام بذلك، ولكنه يوفر أداة مساعدة من خلال مدير السياق [`no_sync`](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel.no_sync)

الذي تمت إضافته إلى نموذجك بعد تحويله إلى DDP.

ضمن مدير السياق هذا، ستتخطى PyTorch مزامنة التدرجات عند استدعاء `.backward()`، وسيؤدي أول استدعاء لـ `.backward()` خارج مدير السياق هذا إلى تشغيل المزامنة. راجع المثال أدناه:

```python
ddp_model, dataloader, optimizer = accelerator.prepare(model, dataloader, optimizer)

for index, batch in enumerate(dataloader):
    inputs, targets = batch
    # Trigger gradient synchronization on the last batch
    if index != (len(dataloader) - 1):
        with ddp_model.no_sync():
            # Gradients only accumulate
            outputs = ddp_model(inputs)
            loss = loss_func(outputs)
            accelerator.backward(loss)
    else:
        # Gradients finally sync
        outputs = ddp_model(inputs)
        loss = loss_func(outputs)
        accelerator.backward(loss)
        optimizer.step()
```

في 🤗 Accelerate لجعل هذا واجهة برمجة تطبيقات يمكن استدعاؤها بغض النظر عن جهاز التدريب (على الرغم من أنه قد لا يفعل أي شيء إذا لم تكن في نظام موزع!)، يتم استبدال `ddp_model.no_sync` بـ [`~Accelerator.no_sync`] ويعمل بنفس الطريقة:

```diff
  ddp_model, dataloader, optimizer = accelerator.prepare(model, dataloader, optimizer)

  for index, batch in enumerate(dataloader):
      inputs, targets = batch
      # Trigger gradient synchronization on the last batch
      if index != (len(dataloader)-1):
-         with ddp_model.no_sync():
+         with accelerator.no_sync(model):
              # Gradients only accumulate
              outputs = ddp_model(inputs)
              loss = loss_func(outputs, targets)
              accelerator.backward(loss)
      else:
          # Gradients finally sync
          outputs = ddp_model(inputs)
          loss = loss_func(outputs)
          accelerator.backward(loss)
          optimizer.step()
          optimizer.zero_grad()
```

كما تتوقع، فإن دالة [`~Accelerator.accumulate`] تحيط بهذا الفحص الشرطي من خلال متابعة رقم الدفعة الحالي، مما يتركك بواجهة برمجة تطبيقات تراكم التدرج النهائية:

```python
ddp_model, dataloader, optimizer = accelerator.prepare(model, dataloader, optimizer)

for batch in dataloader:
    with accelerator.accumulate(model):
        optimizer.zero_grad()
        inputs, targets = batch
        outputs = model(inputs)
        loss = loss_function(outputs, targets)
        accelerator.backward(loss)
        optimizer.step()
        optimizer.zero_grad()
```

ونتيجة لذلك، يجب عليك إما استخدام *`accelerator.accumulate` أو `accelerator.no_sync`* عندما يتعلق الأمر بخيار واجهة برمجة التطبيقات.

## إلى أي مدى يحدث التباطؤ، والأخطاء الشائعة التي يمكنك ارتكابها

لإعداد مثال واقعي، ضع في اعتبارك الإعداد التالي:

- عقدتان من GPU واحدة ووحدة معالجة مركزية (CPU) واحدة وعقدة واحدة بها وحدتا GPU
- كل GPU عبارة عن T4، ويتم استضافتها على GCP
- النص البرمجي المستخدم هو تعديل لنص البرنامج النصي [NLP Example](https://github.com/muellerzr/timing_experiments/blob/main/baseline.py)
- حجم الدفعة لكل GPU هو 16، ويتم تراكم التدرجات كل 4 خطوات

تتوفر جميع النصوص البرمجية في [هذا المستودع](https://github.com/muellerzr/timing_experiments).

إذا لم تكن حذرًا بشأن مزامنة التدرج وتواصل وحدة معالجة الرسوميات (GPU)، فيمكن أن يضيع قدر *كبير* من الوقت عندما تتواصل وحدات معالجة الرسوميات هذه مع بعضها البعض خلال الفترات غير الضرورية.

بأي قدر؟

مراجع:

- Baseline: لا تستخدم ممارسات المزامنة التي تمت مناقشتها هنا
- `no_sync` بشكل غير صحيح: `no_sync` فقط حول استدعاء `backward`، وليس `forward`
- `no_sync`: استخدام نمط `no_sync` بشكل صحيح
- `accumulate`: استخدام [`~Accelerator.accumulate`] بشكل صحيح

فيما يلي متوسط عدد الثواني لكل دفعة أثناء التنقل خلال 29 دفعة من البيانات لكل إعداد لكل من الإعداد أحادي العقدة والإعداد ثنائي العقدة:

|             | Baseline  | `no_sync` بشكل غير صحيح | `no_sync` | `accumulate`|
| :---------: | :-------: | :------------------: | :-------: | :---------: |
| Multi-Node  | 2±0.01s    | 2.13±0.08s | **0.91±0.11s** | **0.91±0.11s** |
| Single Node | 0.50±0.01s | 0.50±0.01s | **0.41±0.015s** | **0.41±0.015s** |

كما ترون، إذا لم تكن حذرًا بشأن كيفية إعداد مزامنة التدرج الخاص بك، فيمكنك الحصول على تباطؤ يزيد عن ضعف السرعة أثناء التدريب!

إذا كنت قلقًا بشأن التأكد من القيام بكل شيء بشكل صحيح، فنحن نوصي بشدة باستخدام دالة [`~Accelerator.accumulate`] وإدخال
`gradient_accumulation_steps` أو `gradient_accumulation_plugin` في كائن [`Accelerator`] حتى يتمكن Accelerate من التعامل مع ذلك نيابة عنك.

### يتطلب `no_sync` ذاكرة GPU إضافية عند استخدام FSDP

كن على علم بأن عدم مزامنة التدرجات يمكن أن يكون له آثار ضارة أثناء التدريب باستخدام FSDP. كما هو محذر في `torch`، فإن [مدير سياق `no_sync` لـ FSDP](https://pytorch.org/docs/stable/fsdp.html#torch.distributed.fsdp.FullyShardedDataParallel.no_sync) سيتطلب ذاكرة إضافية.

لذلك، في المواقف كثيرة الاستخدام للذاكرة أثناء استخدام FSDP، نوصي بتعيين `sync_each_batch` إلى `True` في [`~utils.GradientAccumulationPlugin`] لتعطيل `no_sync`.

راجع المثال أدناه حيث نقوم بتدريب نموذج Mixtral (47 مليار معلمة) على 8 وحدات معالجة رسومات A100-80 جيجابايت. نلاحظ أنه حتى بالنسبة لـ `gradient_accumulation_steps=2` المتواضع، فإننا نتعرض بسرعة لتجاوز سعة الذاكرة (OOM) إذا تم تمكين `no_sync`. ويرجع ذلك مرة أخرى إلى النفقات العامة الإضافية للذاكرة بسبب `no_sync` لـ FSDP. ومع ذلك، إذا تم تعطيل `no_sync` عن طريق `sync_each_batch=True`، فإن استهلاك الذاكرة لـ `gradient_accumulation_steps=16` يعود إلى ما كان عليه في `gradient_accumulation_steps=1`.

| النموذج           | `no_sync` (accum=1) | `no_sync` (accum=2) | `no_sync` معطل (accum=16)
| :-------------: | :-----------------: | :-----------------: | :-----------------:
mixtral 8x7B      | 69 جيجا بايت                 | تجاوز سعة الذاكرة                 | 69 جيجا بايت
> [!تحذير]
> تعطيل `no_sync` يعني أنه _سيكون هناك تباطؤ_ بسبب عمليات مزامنة البيانات الإضافية، كما هو موضح في الأقسام السابقة من هذا الدليل.