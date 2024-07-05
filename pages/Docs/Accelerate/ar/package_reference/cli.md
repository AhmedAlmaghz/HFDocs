# سطر الأوامر
فيما يلي قائمة بجميع الأوامر المتاحة 🤗 Accelerate مع معلماتها

## accelerate config
**الأمر**:
`accelerate config` أو `accelerate-config`

يبدأ سلسلة من المطالبات لإنشاء وحفظ ملف تكوين `default_config.yml` لنظام التدريب الخاص بك. يجب تشغيله دائمًا أولاً على جهازك.

**الاستخدام**:
```bash
accelerate config [arguments]
```

**الحجج الاختيارية**:
* `--config_file CONFIG_FILE` (`str`) -- مسار ملف التكوين الذي سيتم استخدامه. سيتم حفظه بشكل افتراضي في ملف باسم default_config.yaml في موقع ذاكرة التخزين المؤقت، والذي يكون محتوى متغير البيئة `HF_HOME` مضافًا إليه 'accelerate'، أو إذا لم يكن لديك مثل هذا المتغير البيئي، دليل ذاكرة التخزين المؤقت الخاص بك (`~/.cache` أو محتوى `XDG_CACHE_HOME`) مضافًا إليه `huggingface`.
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج

## accelerate config default
**الأمر**:
`accelerate config default` أو `accelerate-config default`

قم بإنشاء ملف تكوين افتراضي لـ Accelerate مع عدد قليل فقط من العلامات المحددة.

**الاستخدام**:
```bash
accelerate config default [arguments]
```

**الحجج الاختيارية**:
* `--config_file CONFIG_FILE` (`str`) -- مسار ملف التكوين الذي سيتم استخدامه. سيتم حفظه بشكل افتراضي في ملف باسم default_config.yaml في موقع ذاكرة التخزين المؤقت، والذي يكون محتوى متغير البيئة `HF_HOME` مضافًا إليه 'accelerate'، أو إذا لم يكن لديك مثل هذا المتغير البيئي، دليل ذاكرة التخزين المؤقت الخاص بك (`~/.cache` أو محتوى `XDG_CACHE_HOME`) مضافًا إليه `huggingface`.
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج
* `--mixed_precision {no,fp16,bf16}` (`str`) -- ما إذا كان سيتم استخدام التدريب على الدقة المختلطة أم لا. اختر بين FP16 وBF16 (bfloat16) للتدريب. يدعم التدريب BF16 فقط معالجات Nvidia Ampere GPUs وإصدار PyTorch 1.10 أو أحدث.

## accelerate config update
**الأمر**:
`accelerate config update` أو `accelerate-config update`

قم بتحديث ملف تكوين موجود بأحدث الإعدادات الافتراضية مع الحفاظ على التكوين القديم.

**الاستخدام**:
```bash
accelerate config update [arguments]
```

**الحجج الاختيارية**:
* `--config_file CONFIG_FILE` (`str`) -- مسار ملف التكوين الذي سيتم تحديثه. سيتم حفظه بشكل افتراضي في ملف باسم default_config.yaml في موقع ذاكرة التخزين المؤقت، والذي يكون محتوى متغير البيئة `HF_HOME` مضافًا إليه 'accelerate'، أو إذا لم يكن لديك مثل هذا المتغير البيئي، دليل ذاكرة التخزين المؤقت الخاص بك (`~/.cache` أو محتوى `XDG_CACHE_HOME`) مضافًا إليه `huggingface`.
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج

## accelerate env
**الأمر**:
`accelerate env` أو `accelerate-env` أو `python -m accelerate.commands.env`

يسرد محتويات ملف تكوين 🤗 Accelerate الممرر. يجب استخدامه دائمًا عند فتح مشكلة في [مستودع GitHub](https://github.com/huggingface/accelerate).

**الاستخدام**:
```bash
accelerate env [arguments]
```

**الحجج الاختيارية**:
* `--config_file CONFIG_FILE` (`str`) -- مسار ملف التكوين الذي سيتم استخدامه. سيتم حفظه بشكل افتراضي في ملف باسم default_config.yaml في موقع ذاكرة التخزين المؤقت، والذي يكون محتوى متغير البيئة `HF_HOME` مضافًا إليه 'accelerate'، أو إذا لم يكن لديك مثل هذا المتغير البيئي، دليل ذاكرة التخزين المؤقت الخاص بك (`~/.cache` أو محتوى `XDG_CACHE_HOME`) مضافًا إليه `huggingface`.
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج

## accelerate launch
**الأمر**:
`accelerate launch` أو `accelerate-launch` أو `python -m accelerate.commands.launch`

يبدأ النص البرمجي المحدد على نظام موزع بالمعلمات الصحيحة.

**الاستخدام**:
```bash
accelerate launch [arguments] {training_script} --{training_script-argument-1} --{training_script-argument-2} ...
```

**الحجج الموضعية**:
- `{training_script}` -- المسار الكامل للنص البرمجي الذي سيتم تشغيله بشكل متوازي
- `--{training_script-argument-1}` -- حجج النص البرمجي للتدريب

**الحجج الاختيارية**:
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج
* `--config_file CONFIG_FILE` (`str`) -- ملف التكوين الذي سيتم استخدامه للقيم الافتراضية في نص الإطلاق.
* `-m`، `--module` (`bool`) -- تغيير كل عملية لتفسير نص الإطلاق كنص برمجي Python، والتنفيذ بنفس سلوك 'python -m'.
* `--no_python` (`bool`) -- تخطي إضافة 'python' قبل النص البرمجي للتدريب - قم بتنفيذه مباشرة. مفيد عندما لا يكون النص البرمجي نصًا برمجيًا Python.
* `--debug` (`bool`) -- ما إذا كان سيتم طباعة مكدس تتبع torch.distributed عند حدوث خطأ ما.
* `-q`، `--quiet` (`bool`) -- إخفاء أخطاء البرنامج الفرعي من مكدس تتبع الإطلاق لعرض تعقب الأخطاء ذات الصلة فقط. (ينطبق فقط على DeepSpeed وتكوينات العملية الواحدة).

بقية هذه الحجج يتم تكوينها من خلال `accelerate config` ويتم قراءتها من ملف `--config_file` (أو التكوين الافتراضي) لقيمها. يمكن أيضًا تمريرها يدويًا.

**حجج اختيار الأجهزة**:
* `--cpu` (`bool`) -- ما إذا كان سيتم إجبار التدريب على وحدة المعالجة المركزية أم لا.
* `--multi_gpu` (`bool`) -- ما إذا كان سيتم إطلاق تدريب GPU الموزع أم لا.
* `--tpu` (`bool`) -- ما إذا كان سيتم إطلاق تدريب TPU أم لا.
* `--ipex` (`bool`) -- ما إذا كان سيتم إطلاق تدريب Intel Pytorch Extension (IPEX) أم لا.

**حجج اختيار الموارد**:
الحجج التالية مفيدة لضبط كيفية استخدام الأجهزة المتاحة
* `--mixed_precision {no,fp16,bf16}` (`str`) -- ما إذا كان سيتم استخدام التدريب على الدقة المختلطة أم لا. اختر بين FP16 وBF16 (bfloat16) للتدريب. يدعم التدريب BF16 فقط معالجات Nvidia Ampere GPUs وإصدار PyTorch 1.10 أو أحدث.
* `--num_processes NUM_PROCESSES` (`int`) -- العدد الإجمالي للعمليات التي سيتم إطلاقها بشكل متوازي.
* `--num_machines NUM_MACHINES` (`int`) -- العدد الإجمالي للآلات المستخدمة في هذا التدريب.
* `--num_cpu_threads_per_process NUM_CPU_THREADS_PER_PROCESS` (`int`) -- عدد خيوط وحدة المعالجة المركزية لكل عملية. يمكن ضبطه للحصول على الأداء الأمثل.

**حجج نموذج التدريب**:
الحجج التالية مفيدة لاختيار نموذج التدريب الذي سيتم استخدامه.
* `--use_deepspeed` (`bool`) -- ما إذا كان سيتم استخدام DeepSpeed للتدريب أم لا.
* `--use_fsdp` (`bool`) -- ما إذا كان سيتم استخدام FullyShardedDataParallel للتدريب أم لا.
* `--use_megatron_lm` (`bool`) -- ما إذا كان سيتم استخدام Megatron-LM للتدريب أم لا.
* `--use_xpu` (`bool`) -- ما إذا كان سيتم استخدام IPEX plugin لتسريع التدريب على XPU على وجه التحديد.

**حجج GPU الموزع**:
الحجج التالية مفيدة فقط عندما يتم تمرير `multi_gpu` أو يتم تكوين التدريب متعدد GPUs من خلال `accelerate config`:
* `--gpu_ids` (`str`) -- أي GPUs (عن طريق المعرف) يجب استخدامها للتدريب على هذه الآلة كقائمة مفصولة بفواصل
* `--same_network` (`bool`) -- ما إذا كانت جميع الآلات المستخدمة للتدريب متعدد العقد موجودة على نفس الشبكة المحلية.
* `--machine_rank MACHINE_RANK` (`int`) -- ترتيب الآلة التي تم إطلاق هذا النص البرمجي عليها.
* `--main_process_ip MAIN_PROCESS_IP` (`str`) -- عنوان IP لآلة الترتيب 0.
* `--main_process_port MAIN_PROCESS_PORT` (`int`) -- المنفذ الذي سيتم استخدامه للتواصل مع آلة الترتيب 0.
* `--rdzv_backend` (`str`) -- طريقة الالتقاء التي سيتم استخدامها، مثل "static" أو "c10d"
* `--rdzv_conf` (`str`) -- تكوين الالتقاء الإضافي (<key1>=<value1>,<key2>=<value2>,...).
* `--max_restarts` (`int`) -- الحد الأقصى لعدد عمليات إعادة تشغيل مجموعة العمال قبل الفشل.
* `--monitor_interval` (`float`) -- الفاصل الزمني، بالثواني، لمراقبة حالة العمال.

**حجج TPU**:
الحجج التالية مفيدة فقط عندما يتم تمرير `tpu` أو يتم تكوين تدريب TPU من خلال `accelerate config`:
* `--main_training_function MAIN_TRAINING_FUNCTION` (`str`) -- اسم الدالة الرئيسية التي سيتم تنفيذها في نصك البرمجي.
* `--downcast_bf16` (`bool`) -- ما إذا كان سيتم، عند استخدام الدقة bf16 على وحدات TPU، قصب كل من الفلاتر والتوترات المزدوجة إلى bfloat16 أو إذا كانت التوترات المزدوجة تظل كما هي float32.

**حجج DeepSpeed**:
الحجج التالية مفيدة فقط عندما يتم تمرير `use_deepspeed` أو يتم تكوين DeepSpeed من خلال `accelerate config`:
* `--deepspeed_config_file` (`str`) -- ملف تكوين DeepSpeed.
* `--zero_stage` (`int`) -- مرحلة DeepSpeed's ZeRO optimization.
* `--offload_optimizer_device` (`str`) -- يقرر أين (none|cpu|nvme) لتفريغ حالات المحسن.
* `--offload_param_device` (`str`) -- يقرر أين (none|cpu|nvme) لتفريغ المعلمات.
* `--gradient_accumulation_steps` (`int`) -- عدد خطوات تجميع التدرجات المستخدمة في نصك البرمجي للتدريب.
* `--gradient_clipping` (`float`) -- قيمة قص التدرج المستخدمة في نصك البرمجي للتدريب.
* `--zero3_init_flag` (`str`) -- يقرر ما إذا كان سيتم (true|false) تمكين `deepspeed.zero.Init` لبناء نماذج ضخمة. ينطبق فقط مع DeepSpeed ZeRO Stage-3.
* `--zero3_save_16bit_model` (`str`) -- يقرر ما إذا كان سيتم (true|false) حفظ أوزان النموذج 16 بت عند استخدام ZeRO Stage-3. ينطبق فقط مع DeepSpeed ZeRO Stage-3.
* `--deepspeed_hostfile` (`str`) -- ملف المضيف DeepSpeed لتكوين موارد الحوسبة متعددة العقد.
* `--deepspeed_exclusion_filter` (`str`) -- سلسلة عامل تصفية الاستبعاد DeepSpeed عند استخدام إعداد متعدد العقد.
* `--deepspeed_inclusion_filter` (`str`) -- سلسلة عامل تصفية الإدراج DeepSpeed عند استخدام إعداد متعدد العقد.
* `--deepspeed_multinode_launcher` (`str`) -- برنامج الإطلاق متعدد العقد DeepSpeed الذي سيتم استخدامه.

**حجج Fully Sharded Data Parallelism**:
الحجج التالية مفيدة فقط عندما يتم تمرير `use_fsdp` أو يتم تكوين Fully Sharded Data Parallelism من خلال `accelerate config`:
* `--fsdp_offload_params` (`str`) -- يقرر ما إذا كان سيتم (true|false) تفريغ المعلمات والتدرجات إلى وحدة المعالجة المركزية.
* `--fsdp_min_num_params` (`int`) -- الحد الأدنى لعدد المعلمات لالتفاف السيارات الافتراضي لـ FSDP.
* `--fsdp_sharding_strategy` (`int`) -- استراتيجية التجزئة FSDP.
* `--fsdp_auto_wrap_policy` (`str`) -- سياسة الالتفاف التلقائي FSDP.
* `--fsdp_transformer_layer_cls_to_wrap` (`str`) -- اسم فئة طبقة المحول (حساس لحالة الأحرف) لتغليفه، على سبيل المثال، `BertLayer`، `GPTJBlock`، `T5Block` ...
* `--fsdp_backward_prefetch_policy` (`str`) -- سياسة الاسترجاع المسبق للخلف لـ FSDP.
* `--fsdp_state_dict_type` (`str`) -- نوع القاموس الحالة FSDP.
* `--fsdp_forward_prefetch` (`str`) -- الاسترجاع المسبق للأمام FSDP.
* `--fsdp_use_orig_params` (`str`) -- إذا كان صحيحًا، فيسمح ذلك بخلط `requires_grad` غير الموحد في وحدة FSDP.
* `--fsdp_cpu_ram_efficient_loading` (`str`) - إذا كان صحيحًا، فإن العملية الأولى فقط تقوم بتحميل نقطة التحقق للنموذج المسبق التدريب في حين أن جميع العمليات الأخرى لها أوزان فارغة. عند استخدام هذا، يجب أن يكون `--fsdp_sync_module_states` صحيحًا.
* `--fsdp_sync_module_states` (`str`) - إذا كان صحيحًا، فإن كل وحدة FSDP ملفوفة بشكل فردي ستقوم ببث معلمات الوحدة من الترتيب 0.

**حجج Megatron-LM**:
الحجج التالية مفيدة فقط عندما يتم تمرير `use_megatron_lm` أو يتم تكوين Megatron-LM من خلال `accelerate config`:
* `--megatron_lm_tp_degree` (``) -- درجة موازاة التوتر لـ Megatron-LM.
* `--megatron_lm_pp_degree` (``) -- درجة موازاة الأنابيب لـ Megatron-LM.
* `--megatron_lm_num_micro_batches` (``) -- عدد الدفعات الصغيرة لـ Megatron-LM عندما تكون درجة PP > 1.
* `--megatron_lm_sequence_parallelism` (``) -- يقرر ما إذا كان سيتم (true|false) تمكين موازاة التسلسل عندما تكون درجة TP > 1.
* `--megatron_lm_recompute_activations` (``) -- يقرر ما إذا كان سيتم (true|false) تمكين إعادة حساب التنشيط الانتقائي.
* `--megatron_lm_use_distributed_optimizer` (``) -- يقرر ما إذا كان سيتم (true|false) استخدام المحسن الموزع الذي يقوم بتجزئة حالات المحسن والتدرجات عبر رتب DP.
* `--megatron_lm_gradient_clipping` (``) -- قيمة قص التدرج Megatron-LM بناءً على L2 Norm العالمي (0 لتعطيل).

**حجج AWS SageMaker**:
الحجج التالية مفيدة فقط عند التدريب في SageMaker
* `--aws_access_key_id AWS_ACCESS_KEY_ID` (`str`) -- AWS_ACCESS_KEY_ID المستخدم لإطلاق مهمة التدريب Amazon SageMaker
* `--aws_secret_access_key AWS_SECRET_ACCESS_KEY` (`str`) -- AWS_SECRET_ACCESS_KEY المستخدم لإطلاق مهمة التدريب Amazon SageMaker
## accelerate estimate-memory

**الأمر**:

`accelerate estimate-memory` أو `accelerate-estimate-memory` أو `python -m accelerate.commands.estimate`

يقدر إجمالي الذاكرة VRAM التي يحتاجها نموذج معين مستضاف على Hub ليتم تحميله مع تقدير للتدريب. يتطلب تثبيت `huggingface_hub`.

<Tip>

عند إجراء الاستدلال، عادةً ما تضيف ≤20% إلى النتيجة كمخصص عام [كما هو موضح هنا](https://blog.eleuther.ai/transformer-math/). سنقدم تقديرات أكثر شمولاً في المستقبل والتي ستدرج تلقائيًا في الحساب.

</Tip>

**الاستخدام**:

```bash
accelerate estimate-memory {MODEL_NAME} --library_name {LIBRARY_NAME} --dtypes {dtype_1} {dtype_2} ...
```

**الحجج المطلوبة**:

* `MODEL_NAME` (`str`)-- اسم النموذج على Hugging Face Hub

**الحجج الاختيارية**:

* `--library_name {timm,transformers}` (`str`) -- مكتبة التكامل مع النموذج، مثل `transformers`، مطلوبة فقط إذا كانت هذه المعلومات غير مخزنة على Hub
* `--dtypes {float32,float16,int8,int4}` (`[{float32,float16,int8,int4} ...]`) -- أنواع البيانات التي سيتم استخدامها للنموذج، يجب أن تكون واحدة (أو أكثر) من `float32` أو `float16` أو `int8` أو `int4`
* `--trust_remote_code` (`bool`) -- ما إذا كان سيتم السماح بنماذج مخصصة محددة على Hub في ملفات النمذجة الخاصة بها. يجب تمرير هذا الخيار فقط للمستودعات التي تثق بها والتي قرأت فيها الكود، حيث سيتم تنفيذ الكود الموجود على Hub على جهازك المحلي.

## accelerate tpu-config

`accelerate tpu-config`

**الاستخدام**:

```bash
accelerate tpu-config [arguments]
```

**الحجج الاختيارية**:

* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج

**حجج التكوين**:

الحجج التي يمكن تكوينها من خلال `accelerate config`.

* `--config_file` (`str`) -- مسار ملف التكوين الذي سيتم استخدامه لـ accelerate.
* `--tpu_name` (`str`) -- اسم TPU الذي سيتم استخدامه. إذا لم يتم تحديده، فسيتم استخدام TPU المحدد في ملف التكوين.
* `--tpu_zone` (`str`) -- منطقة TPU التي سيتم استخدامها. إذا لم يتم تحديدها، فسيتم استخدام المنطقة المحددة في ملف التكوين.

**حجج TPU**:

الحجج الخاصة بالخيارات التي يتم تشغيلها داخل TPU.

* `--command_file` (`str`) -- مسار الملف الذي يحتوي على الأوامر التي سيتم تشغيلها على pod عند بدء التشغيل.
* `--command` (`str`) -- أمر لتشغيله على pod. يمكن تمريره عدة مرات.
* `--install_accelerate` (`bool`) -- ما إذا كان سيتم تثبيت accelerate على pod. الافتراضي هو False.
* `--accelerate_version` (`str`) -- إصدار accelerate الذي سيتم تثبيته على pod. إذا لم يتم تحديده، فسيتم استخدام أحدث إصدار من Pypi. حدد "dev" لتثبيته من GitHub.
* `--debug` (`bool`) -- إذا تم تعيينه، فسيتم طباعة الأمر الذي سيتم تشغيله بدلاً من تشغيله.

## accelerate test

`accelerate test` أو `accelerate-test`

يقوم بتشغيل `accelerate/test_utils/test_script.py` للتحقق من أن 🤗 Accelerate قد تم تكوينه بشكل صحيح على نظامك وأنه يعمل.

**الاستخدام**:

```bash
accelerate test [arguments]
```

**الحجج الاختيارية**:

* `--config_file CONFIG_FILE` (`str`) -- مسار ملف التكوين الذي سيتم استخدامه. سيتم تعيينه بشكل افتراضي إلى ملف باسم default_config.yaml في موقع التخزين المؤقت، وهو محتوى
متغير البيئة `HF_HOME` ملحق بـ 'accelerate'، أو إذا لم يكن لديك مثل هذا المتغير البيئي، دليل ذاكرة التخزين المؤقت الافتراضي
(`~/.cache` أو محتوى `XDG_CACHE_HOME`) ملحق بـ `huggingface`.
* `-h`، `--help` (`bool`) -- عرض رسالة مساعدة والخروج