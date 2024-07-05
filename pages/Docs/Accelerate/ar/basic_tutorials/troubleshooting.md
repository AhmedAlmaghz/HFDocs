# استكشاف الأخطاء وإصلاحها

يوفر هذا الدليل حلولًا لبعض المشكلات التي قد تواجهها عند استخدام Accelerate. لا يتم تغطية جميع الأخطاء لأن Accelerate عبارة عن مكتبة نشطة تتطور باستمرار، وهناك العديد من حالات الاستخدام المختلفة وتكوينات التدريب الموزعة. إذا لم تساعد الحلول الموضحة هنا في حل خطأك المحدد، فيرجى الاطلاع على قسم [طلب المساعدة](#ask-for-help) لمعرفة مكان وكيفية الحصول على المساعدة.

## التسجيل

يمكن أن يساعدك التسجيل في تحديد مصدر الخطأ. في الإعداد الموزع مع عمليات متعددة، يمكن أن يكون التسجيل تحديًا، ولكن يوفر Accelerate أداة [`~accelerate.logging`] لضمان تزامن السجلات.

لحل مشكلة ما، استخدم [`~accelerate.logging`] بدلاً من وحدة [`logging`](https://docs.python.org/3/library/logging.html#module-logging) القياسية في Python. قم بتعيين مستوى التفاصيل (`INFO` أو `DEBUG` أو `WARNING` أو `ERROR` أو `CRITICAL`) باستخدام معلمة `log_level`، ثم يمكنك إما:

1. قم بتصدير `log_level` كمتغير بيئة `ACCELERATE_LOG_LEVEL`.
2. قم بتمرير `log_level` مباشرة إلى `get_logger`.

على سبيل المثال، لتعيين `log_level="INFO"`:

```py
from accelerate.logging import get_logger

logger = get_logger(__name__, log_level="DEBUG")
```

بشكل افتراضي، يتم استدعاء السجل على العمليات الرئيسية فقط. لاستدعائه على جميع العمليات، قم بتمرير `main_process_only=False`.

إذا كان يجب استدعاء سجل على جميع العمليات وبترتيب معين، فقم أيضًا بتمرير `in_order=True`.

```py
from accelerate.logging import get_logger

logger = get_logger(__name__, log_level="DEBUG")
# سجل جميع العمليات
logger.debug("thing_to_log", main_process_only=False)
# سجل جميع العمليات بالترتيب
logger.debug("thing_to_log", main_process_only=False, in_order=True)
```

## تعليق التعليمات البرمجية وأخطاء المهلة

قد يكون هناك العديد من الأسباب لتعليق التعليمات البرمجية الخاصة بك. دعونا نلقي نظرة على كيفية حل بعض المشكلات الأكثر شيوعًا التي يمكن أن تتسبب في تعليق التعليمات البرمجية الخاصة بك.

### عدم تطابق أشكال tensor

عدم تطابق أشكال tensor هي مشكلة شائعة يمكن أن تتسبب في تعليق التعليمات البرمجية الخاصة بك لفترة طويلة من الوقت في إعداد موزع.

عند تشغيل النصوص في إعداد موزع، تكون الوظائف مثل [`Accelerator.gather`] و [`Accelerator.reduce`] ضرورية لالتقاط tensors عبر الأجهزة لأداء العمليات عليها بشكل جماعي. تعتمد هذه الوظائف (وغيرها) على `torch.distributed` لأداء عملية `gather`، والتي تتطلب أن يكون لدى tensors **نفس الشكل الدقيق** عبر جميع العمليات. عندما لا تتطابق أشكال tensor، فإن التعليمات البرمجية الخاصة بك تتعطل وستحصل في النهاية على استثناء المهلة.

يمكنك استخدام وضع التصحيح التشغيلي لـ Accelerate للقبض على هذه المشكلة على الفور. نوصي بتمكين هذا الوضع أثناء إعداد `accelerate config`، ولكن يمكنك أيضًا تمكينه من CLI، أو كمتغير بيئي، أو عن طريق تحرير ملف `config.yaml` يدويًا.

<hfoptions id="mismatch">
<hfoption id="CLI">

```bash
accelerate launch --debug {my_script.py} --arg1 --arg2
```

</hfoption>
<hfoption id="environment variable">

إذا كنت تقوم بتمكين وضع التصحيح كمتغير بيئي، فلست بحاجة إلى استدعاء `accelerate launch`.

```bash
ACCELERATE_DEBUG_MODE="1" torchrun {my_script.py} --arg1 --arg2
```

</hfoption>
<hfoption id="config.yaml">

أضف `debug: true` إلى ملف `config.yaml` الخاص بك.

```yaml
compute_environment: LOCAL_MACHINE
debug: true
```

</hfoption>
</hfoptions>

بمجرد تمكين وضع التصحيح، يجب أن تحصل على تتبع المكدس الذي يشير إلى مشكلة عدم تطابق شكل tensor.

```py
Traceback (most recent call last):
  File "/home/zach_mueller_huggingface_co/test.py", line 18, in <module>
    main()
  File "/home/zach_mueller_huggingface_co/test.py", line 15, in main
    broadcast_tensor = broadcast(tensor)
  File "/home/zach_mueller_huggingface_co/accelerate/src/accelerate/utils/operations.py", line 303, in wrapper
accelerate.utils.operations.DistributedOperationException:

Cannot apply desired operation due to shape mismatches. All shapes across devices must be valid.

Operation: `accelerate.utils.operations.broadcast`
Input shapes:
  - Process 0: [1, 5]
  - Process 1: [1, 2, 5]
```

### التوقف المبكر

بالنسبة للتوقف المبكر في التدريب الموزع، إذا كان لكل عملية شرط توقف محدد (على سبيل المثال، خسارة التحقق)، فقد لا يتم مزامنته عبر جميع العمليات. ونتيجة لذلك، قد يحدث انقطاع في العملية 0 ولكن ليس في العملية 1، مما قد يتسبب في تعليق التعليمات البرمجية الخاصة بك إلى أجل غير مسمى حتى يحدث مهلة.

إذا كان لديك شروط توقف مبكر، فاستخدم طريقتي `set_breakpoint` و `check_breakpoint` للتأكد من انتهاء جميع العمليات بشكل صحيح.

```py
# افترض أن `should_do_breakpoint` هي دالة مخصصة تعيد شرطًا،
# وقد يكون هذا الشرط صحيحًا فقط في العملية 1
if should_do_breakpoint(loss):
    accelerator.set_breakpoint()

# لاحقًا في نص التدريب عند الحاجة إلى التحقق من نقطة التوقف
if accelerator.check_breakpoint():
    break
```

### إصدارات kernel المنخفضة على Linux

على نظام Linux مع إصدار kernel < 5.5، تم الإبلاغ عن عمليات معلقة. لتجنب هذه المشكلة، قم بترقية نظامك إلى إصدار kernel لاحق.

### MPI

إذا كانت مهمة التدريب الموزع CPU باستخدام MPI معلقة، فتأكد من إعداد [SSH بدون كلمة مرور](https://www.open-mpi.org/faq/?category=rsh#ssh-keys) (باستخدام المفاتيح) بين العقد. وهذا يعني أنه يجب عليك أن تكون قادرًا على SSH من عقدة إلى أخرى دون مطالبتك بكلمة مرور لجميع العقد في ملف المضيف الخاص بك.

بعد ذلك، جرب تشغيل أمر `mpirun` كفحص للجودة. على سبيل المثال، يجب أن تطبع الأوامر أدناه أسماء المضيف لكل من العقد.

```bash
mpirun -f hostfile -n {عدد العقد} -ppn 1 hostname
```

## CUDA خارج الذاكرة

أحد أكثر الأخطاء إحباطًا عند تشغيل نصوص التدريب هو مواجهة خطأ "CUDA خارج الذاكرة". يجب إعادة تشغيل النص بأكمله ويضيع أي تقدم.

لحل هذه المشكلة، توفر Accelerate أداة [`find_executable_batch_size`] التي تعتمد بشكل كبير على [toma](https://github.com/BlackHC/toma).

تعيد هذه الأداة تشغيل التعليمات البرمجية التي تفشل بسبب ظروف OOM (نفاد الذاكرة) وتقلل تلقائيًا من أحجام الدفعات. لكل حالة OOM، تقلل الخوارزمية حجم الدفعة إلى النصف وتعيد تشغيل التعليمات البرمجية حتى تنجح.

لاستخدام [`find_executable_batch_size`]، قم بإعادة هيكلة دالة التدريب الخاصة بك لتضمين دالة داخلية مع `find_executable_batch_size` وقم ببناء برامج تحميل البيانات الخاصة بك داخلها. كحد أدنى، يتطلب هذا فقط 4 أسطر جديدة من التعليمات البرمجية.

<Tip warning={true}>

يجب أن تأخذ الدالة الداخلية **حجم الدفعة** كأول معلمة، ولكننا لا نمرر واحدة إليها عند استدعائها. سيقوم المعالج بتولي ذلك من أجلك. يجب إعلان أي كائن (نماذج، أو محسنات) يستهلك ذاكرة CUDA ويمرر إلى [`Accelerator`] داخل الدالة الداخلية.

</Tip>

```diff
def training_function(args):
    accelerator = Accelerator()

+   @find_executable_batch_size(starting_batch_size=args.batch_size)
+   def inner_training_loop(batch_size):
+       nonlocal accelerator # Ensure they can be used in our context
+       accelerator.free_memory() # Free all lingering references
        model = get_model()
        model.to(accelerator.device)
        optimizer = get_optimizer()
        train_dataloader, eval_dataloader = get_dataloaders(accelerator, batch_size)
        lr_scheduler = get_scheduler(
            optimizer, 
            num_training_steps=len(train_dataloader)*num_epochs
        )
        model, optimizer, train_dataloader, eval_dataloader, lr_scheduler = accelerator.prepare(
            model, optimizer, train_dataloader, eval_dataloader, lr_scheduler
        )
        train(model, optimizer, train_dataloader, lr_scheduler)
        validate(model, eval_dataloader)
+   inner_training_loop()
```

## نتائج غير قابلة للتكرار بين إعدادات الأجهزة

إذا قمت بتغيير إعداد الجهاز ولاحظت اختلاف أداء النموذج، فمن المحتمل أنك لم تقم بتحديث نصك عند الانتقال من إعداد إلى آخر. حتى إذا كنت تستخدم نفس النص بنفس حجم الدفعة، ستظل النتائج مختلفة على TPU وmulti-GPU وsingle GPU.

على سبيل المثال، إذا كنت تتدرب على GPU واحد بحجم دفعة يبلغ 16 وتبدأ في استخدام إعداد GPU مزدوج، فيجب عليك تغيير حجم الدفعة إلى 8 للحصول على نفس حجم الدفعة الفعال. ويرجع ذلك إلى أنه عند التدريب باستخدام Accelerate، يكون حجم الدفعة الذي يتم تمريره إلى برنامج تحميل البيانات هو **حجم الدفعة لكل GPU**.

للتأكد من إمكانية إعادة إنتاج النتائج بين الإعدادات، تأكد من استخدام نفس البذرة، وقم بتعديل حجم الدفعة وفقًا لذلك، ويجب مراعاة تحجيم معدل التعلم.

لمزيد من التفاصيل ودليل سريع لأحجام الدفعات، راجع دليل [مقارنة الأداء بين إعدادات الأجهزة المختلفة](../concept_guides/performance).

## مشكلات الأداء على GPUs مختلفة

إذا كان إعداد GPU المتعدد الخاص بك يتكون من GPUs مختلفة، فقد تواجه بعض مشكلات الأداء:

- قد يكون هناك عدم توازن في ذاكرة GPU بين GPUs. في هذه الحالة، فإن GPU ذو الذاكرة الأصغر سيحد من حجم الدفعة أو حجم النموذج الذي يمكن تحميله على GPUs.
- إذا كنت تستخدم GPUs بمستوى أداء مختلف، فسيتم تشغيل الأداء بواسطة أبطأ GPU تستخدمه لأنه يتعين على GPUs الأخرى الانتظار حتى ينتهي من عبء العمل الخاص به.

يمكن أن يؤدي اختلاف GPUs اختلافًا كبيرًا داخل نفس الإعداد إلى اختناقات في الأداء.

## طلب المساعدة

إذا لم تساعد أي من الحلول والمشورة هنا في حل مشكلتك، فيمكنك دائمًا التواصل مع المجتمع وفريق Accelerate للحصول على المساعدة.

- اطلب المساعدة على منتديات Hugging Face من خلال نشر سؤالك في فئة [🤗 Accelerate](https://discuss.huggingface.co/c/accelerate/18). تأكد من كتابة مشاركة وصفية مع سياق ذي صلة حول إعدادك ورموز قابلة للتكرار لزيادة احتمالية حل مشكلتك!
- قم بنشر سؤال على [Discord](http://hf.co/join/discord)، ودع الفريق والمجتمع يساعدانك.
- قم بإنشاء مشكلة على مستودع 🤗 Accelerate [GitHub](https://github.com/huggingface/accelerate/issues) إذا كنت تعتقد أنك وجدت خطأً متعلقًا بالمكتبة. تضمن السياق فيما يتعلق بالخطأ وتفاصيل حول إعدادك الموزع لمساعدتنا بشكل أفضل في معرفة ما هو الخطأ وكيف يمكننا إصلاحه.