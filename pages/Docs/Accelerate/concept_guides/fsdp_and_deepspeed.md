# الانتقال بين FSDP و DeepSpeed

يوفر 🤗 Accelerate مرونة في أطر التدريب، من خلال دمج أداتين قويتين للغاية للتدريب الموزع، وهما [Pytorch FSDP](../usage_guides/fsdp) و [Microsoft DeepSpeed](../usage_guides/deepspeed). ويهدف هذا البرنامج التعليمي إلى رسم أوجه التشابه، وكذلك توضيح الاختلافات المحتملة، لتمكين المستخدم من التبديل بسلاسة بين هذين الإطارين.

<Tip>

للتبديل بين الأطر، نوصي بتشغيل التعليمات البرمجية 🤗 `accelerate launch` مع تمرير ملف التكوين الصحيح باستخدام `--config_file`، أو تمرير الحجج المقابلة مباشرة لـ [FSDP و DeepSpeed](../package_reference/cli#accelerate-launch).

يمكن العثور على أمثلة لتكوينات 🤗 Accelerate هنا لـ [DeepSpeed](../usage_guides/deepspeed#accelerate-deepspeed-plugin) و [FSDP](../usage_guides/fsdp#how-it-works-out-of-the-box)، أو في [مثال Zoo تحت "تكوينات الإطلاق"](../usage_guides/explore)

</Tip>

<Tip warning={true}>

هذا البرنامج التعليمي مخصص لسيناريوهات العقدة الواحدة، متعددة وحدات معالجة الرسومات (GPU) فقط.

</Tip>

## تكوين الوظائف

يتم تقسيم تنسورات النموذج إلى وحدات معالجة رسومات (GPU) مختلفة في محاولة لزيادة أحجام النماذج؛ يُطلق على ذلك *sharding* في FSDP، و *partitioning* في DeepSpeed. يتم تكوين مراحل FSDP sharding و DeepSpeed ZeRO (التقسيم) بواسطة `--fsdp_sharding_strategy`، و `--zero_stage`، على التوالي. على وجه الخصوص، يتوافق FSDP `FULL_SHARD` مع DeepSpeed ZeRO stage `3`؛ راجع هذا [الرسم البياني الشامل بين استراتيجيات FSDP sharding وإعدادات DeepSpeed ZeRO](../usage_guides/fsdp#mapping-between-fsdp-sharding-strategies-and-deepspeed-zero-stages). يلخص الجدول أدناه ويصنف الإعدادات المتشابهة:

المجموعة | الإطار | التكوين | مثال | القيود (إن وجدت)
--|--|--|--|--
التقسيم / التجزئة | FSDP<br>DeepSpeed | `--fsdp_sharding_strategy`<br>`--zero_stage` | `1` (`FULL_SHARD`) <br>`3` |
التحميل من الذاكرة | FSDP<br>DeepSpeed | `--fsdp_offload_params`<br>`--offload_param_device`<br>`--offload_optimizer_device` | `true`<br>`cpu`<br>`cpu` | الكل أو لا شيء <br><br>
تحميل النموذج | FSDP<br>DeepSpeed | <span style="white-space:nowrap;">`--fsdp_cpu_ram_efficient_loading`</span><br>`--zero3_init_flag` | `true`<br>`true` | <br>Zero 3 فقط
النقطة المرجعية الفعالة | FSDP<br>DeepSpeed | `--fsdp_state_dict_type`<br>`--zero3_save_16bit_model` |  `SHARDED_STATE_DICT`<br>`true` |  <br>Zero 3 فقط
استرداد الأوزان مسبقًا | FSDP<br><br>DeepSpeed | `--fsdp_forward_prefetch`<br>`--fsdp_backward_prefetch`<br>None | `true`<br>`BACKWARD_PRE` | <br><br>
النموذج | FSDP<br><br>DeepSpeed |  `--fsdp_auto_wrap_policy`<br><span style="white-space:nowrap;">`--fsdp_transformer_layer_cls_to_wrap`</span><br>None | `TRANSFORMER_BASED_WRAP`<br><Layer Class> |<br>عادة ما لا تكون هناك حاجة <br>شفافة للمستخدم.
استدعاء المعلمات | FSDP<br>DeepSpeed | `--fsdp_use_orig_params`<br>None | `true` | مطلوب لـ `torch.compile`<br>شفافة للمستخدم
مزامنة المعلمات | FSDP<br>DeepSpeed | `--fsd-p_sync_module_states`<br>None | `true` |
التدريب | FSDP<br>DeepSpeed | None<br>`--gradient_accumulation_steps`<br>`--gradient_clipping` | <br>`auto`<br>`auto` | شفافة للمستخدم

للاطلاع على أوصاف مفصلة لما سبق، راجع [🤗 وثائق إطلاق `Accelerate`](../package_reference/cli#accelerate-launch).

<Tip>

للوصول إلى تكوينات DeepSpeed الأخرى، مثل إعدادات الدقة المختلطة، يجب تمرير ملف تكوين DeepSpeed باستخدام `--deepspeed_config_file`، راجع [الوثائق](../usage_guides/deepspeed#deepspeed-config-file).

يمكن أيضًا تكوين DeepSpeed عبر [`DeepSpeedPlugin`]، على سبيل المثال، `DeepSpeedPlugin.zero_stage` يعادل `--zero_stage`، ويمكن استخدام `DeepSpeedPlugin.hf_ds_config` لتمرير `--deepeed_config_file.`

</Tip>

<Tip>

يمكن أيضًا تكوين FSDP عبر [`FullyShardedDataParallelPlugin`]، على سبيل المثال، `FullyShardedDataParallelPlugin.sharding_strategy` يعادل `--fsdp_sharding_strategy`.

</Tip>

### نقاط المراقبة

لاحظ أنه في حين يمكن تكوين FSDP عبر `--fsdp_state_dict_type` لحفظ نقاط المراقبة الكاملة / المجزأة.

<Tip>

بالنسبة لـ DeepSpeed Zero3، يمكنك تمرير `--zero3_save_16bit_model true`، والذي يقوم بتوحيد النموذج بشكل مناسب في ترتيب واحد وحفظه؛ هذا يعادل FSDP من `fsdp_state_dict_type: FULL_STATE_DICT`.

</Tip>

<Tip warning={true}>

بالنسبة للنماذج الكبيرة، قد يكون توحيد النموذج إلى ترتيب واحد بطيئًا جدًا.

</Tip>

<Tip>

للحصول على نقاط مرجعية أسرع، استخدم FSDP `fsdp_state_dict_type: SHARDED_STATE_DICT`، وبالنسبة لـ DeepSpeed Zero3 [استخدم نص البرنامج النصي `zero_to_fp32.py` لتحويل نقاط المراقبة المجزأة](https://www.deepspeed.ai/tutorials/zero/#extracting-weights).

</Tip>

### التحميل من الذاكرة

يسمح FSDP فقط بالتحميل من الذاكرة *all-or-nothing* (أي إما تحميل المعلمات والمدرب، أو الاحتفاظ بها جميعًا في وحدة معالجة الرسومات (GPU))، ولكن يمكن لـ DeepSpeed تحميل المعلمات والمدرب بشكل مختلف. علاوة على ذلك، يدعم DeepSpeed أيضًا [التحميل من الذاكرة إلى NVME](https://www.deepspeed.ai/docs/config-json/#parameter-offloading).

### الاسترداد المسبق

يسمح FSDP بتكوين استرداد مسبق للاسترداد المسبق `--fsdp_forward_prefetch` و `--fsdp_backward_prefetch` لتحسين تداخل الاتصالات / الحسابات على حساب الذاكرة الإضافية، راجع [وثائق FSDP](https://pytorch.org/docs/stable/fsdp.html).

بالنسبة لـ DeepSpeed، سيتم تشغيل الاسترداد المسبق عند الحاجة، ويتم تشغيله بناءً على بعض المعلمات فائقة الدقة مثل `stage3_param_persistence_threshold`، `stage3_max_reuse_distance`، إلخ، [التي يمكن تكوينها لـ Zero3](https://www.deepspeed.ai/docs/config-json/#parameter-offloading)؛ قد يقوم 🤗 `accelerate` بتعيين هذه المعلمات فائقة الدقة تلقائيًا إذا لم تقم بتعيينها صراحةً في ملف تكوين DeepSpeed.

<Tip>

بالنسبة لـ FSDP، قم بتعيين `fsdp_backward_prefetch: BACKWARD_PRE` لتحسين الإنتاجية إذا سمحت الذاكرة.

</Tip>

### تحميل النموذج

في حين أن FSDP يتطلب `--fsdp_cpu_ram_efficient_loading true` لتنشيط تحميل النموذج الفعال، سيقوم 🤗 `transformers` بتنشيط ميزة مماثلة كلما تم استخدام DeepSpeed Zero3.

<Tip>

بالنسبة لـ FSDP، عندما تقوم بتعيين `--fsdp_cpu_ram_efficient_loading true`، سيقوم 🤗 `accelerate` تلقائيًا بتعيين `sync_module_states` إلى true.

بالنسبة لتحميل ذاكرة الوصول العشوائي (RAM) الفعال، سيتم تحميل الأوزان فقط في ترتيب واحد، وبالتالي يتطلب `sync_module_states` لبث الأوزان إلى الترتيبات الأخرى.

</Tip>

### النموذج

يتطلب FSDP `--fsdp_auto_wrap_policy` صريحًا حتى يتمكن الخوارزمية من تحديد كيفية جدولة عمليات all-gather و reduce-scatter. ولكن بالنسبة لـ DeepSpeed، يكون ذلك شفافًا للمستخدم.

<Tip>

بالنسبة لـ FSDP، قم ببساطة بتعيين `fsdp_auto_wrap_policy: TRANSFORMER_BASED_WRAP`. باستخدام أحدث إصدارات [`transformers`]، نبذل قصارى جهدنا لمعرفة فئة الطبقة المناسبة لـ HF transformers models. ومع ذلك، إذا تلقيت خطأً بخصوص ذلك، يرجى تحديده.

</Tip>

### استدعاء المعلمات

يتطلب FSDP علمًا صريحًا `--fsdp_use_orig_params` إذا كنت تستخدم `torch.compile`، راجع [وثائق PyTorch](https://pytorch.org/docs/stable/fsdp.html#module-torch.distributed.fsdp). بالنسبة لـ DeepSpeed، يكون ذلك شفافًا للمستخدم.

<Tip>

بالنسبة لـ FSDP، عند استخدام `torch.compile`، يرجى تعيين `fsdp_use_orig_params: True`.

</Tip>

## التدريب

يتطلب Deepspeed أعلام `--gradient_accumulation_steps` و `--gradient_clipping` صريحة. بالنسبة لـ FSDP، يكون ذلك شفافًا للمستخدم.

<Tip>

عند استخدام DeepSpeed، قم بتعيين `gradient_accumulation_steps: "auto"` و `gradient_clipping: "auto"` لاختيار القيم التي تم تعيينها تلقائيًا في [`Accelerator`] أو [`TrainingArguments`] (إذا كنت تستخدم `transformers`).

</Tip>

## الاختلافات في التعامل مع دقة البيانات

للتحدث عن كيفية التعامل مع دقة البيانات في كل من FSDP و Deepspeed، من المفيد أولاً إعطاء نظرة عامة حول كيفية التعامل مع معلمات النموذج في هذه الأطر. قبل توزيع معلمات النموذج / المدرب عبر وحدات معالجة الرسومات (GPU)، تكون هناك عملية إعداد المعلمات لإنشاء "معلمات مسطحة" أولاً في تنسور أحادي البعد [`torch.Tensor`](https://pytorch.org/docs/stable/tensors.html#torch-tensor). يختلف تنفيذ FSDP / DeepSpeed في ما يتعلق بـ `dtype` الذي يتم فيه تخزين هذه المعلمات "المسطحة"، وهناك عواقب فيما يتعلق بكيفية تخصيص [`torch.Optimizer`](https://pytorch.org/docs/stable/optim.html#module-torch.optim) `dtype`s الخاصة بهم. يلخص الجدول أدناه العمليات لكلا الإطارين؛ يشير عمود "المحلي" إلى العملية التي تحدث على مستوى وحدة معالجة الرسومات (GPU) الفردية، لذلك يجب فهم أي نفقات عامة للذاكرة بسبب upcasting من خلال عدد وحدات معالجة الرسومات (GPU) المستخدمة.

<Tip>

كقاعدة عامة، بالنسبة للتدريب المستقر باستخدام الدقة المختلطة التلقائية، يجب أن تكون جميع المعلمات القابلة للتدريب في `torch.float32`.

</Tip>

العملية | المحلي | الإطار | التفاصيل
--|--|--|--
التحميل، أي [`AutoModel.from_pretrained(..., torch_dtype=torch_dtype)`] |
الإعداد، أي إنشاء "معلمات مسطحة" | ✅ | FSDP<br>DeepSpeed | تم إنشاؤه في `torch_dtype`.<br> تجاهل `torch_dtype`، تم إنشاؤه في `float32`.
تهيئة المدرب | ✅ | FSDP<br>DeepSpeed  | إنشاء المعلمات في `torch_dtype`<br> إنشاء المعلمات في `float32`
خطوة التدريب، أي، إلى الأمام، إلى الخلف، التخفيض | | FSDP<br>DeepSpeed  | اتبع [`MixedPrecision`](https://pytorch.org/docs/stable/fsdp.html#torch.distributed.fsdp.MixedPrecision)<br> اتبع إعدادات الدقة المختلطة في ملف تكوين DeepSpeed.
المدرب (ما قبل الخطوة) | ✅ | FSDP<br>DeepSpeed | upcasting (إن وجد) إلى `torch_dtype`<br> تم upcast إلى `float32`
المدرب (الخطوة الفعلية) | ✅ | FSDP<br>DeepSpeed  | يحدث في `torch_dtype` <br> يحدث في `float32`.
<Tip warning={true}>

لذلك عند استخدام DeepSpeed بعدد صغير من وحدات معالجة الرسومات (GPU)، كن على دراية باحتمالية وجود نفقات عامة كبيرة للذاكرة بسبب upcasting أثناء الإعداد.

</Tip>

<Tip>

مع FSDP، في حالة عدم وجود دقة مختلطة، من الممكن تشغيل [`torch.Optimizer`](https://pytorch.org/docs/stable/optim.html#module-torch.optim) في دقة منخفضة `torch_dtype`، والتي قد تكون مفيدة عند استخدام عدد صغير من وحدات معالجة الرسومات (GPU).

</Tip>

<Tip warning={true}>

مع الدقة المختلطة، سيقوم كل من FSDP و DeepSpeed بالتحويل إلى دقة أعلى في خطوة إعداد النموذج (راجع الجدول أعلاه). ولكن لاحظ أن FSDP سيقوم بعد ذلك بحفظ نقاط المراقبة في الدقة الأعلى؛ قد يقوم Deepspeed أيضًا بحفظ نقاط المراقبة منخفضة الدقة إذا تم تحديد `--zero3_save_16bit_model`.

</Tip>

لتوضيح الجدول أعلاه، ضع في اعتبارك الأمثلة الملموسة أدناه؛ تم دمج خطوة المدرب الفعلية وخطوة ما قبل المدرب للاختصار. مع FSDP، من الممكن العمل في الوضعين الموضحين أدناه، ولكن يمكن لـ DeepSpeed العمل في وضع واحد فقط.

الإطار | تحميل النموذج (`torch_dtype`) | الدقة المختلطة | الإعداد (المحلي) | التدريب | المدرب (المحلي)
--|--|--|--|--|--
FSDP | bf16 | الافتراضي (لا شيء) | bf16 | bf16 | bf16
FSDP | bf16 | bf16 | fp32 | bf16 | fp32
DeepSpeed | bf16 | bf16 | fp32 | bf16 | fp32