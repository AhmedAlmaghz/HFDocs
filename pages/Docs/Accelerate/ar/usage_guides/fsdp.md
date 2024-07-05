# النموذج المتوازي للبيانات المجزأة بالكامل

لتسريع تدريب النماذج الضخمة باستخدام أحجام دفعات أكبر، يمكننا استخدام نموذج متوازي للبيانات مجزأ بالكامل. يمكّن هذا النوع من النماذج المتوازية للبيانات من ملاءمة المزيد من البيانات والنماذج الأكبر عن طريق تجزئة حالات المحسن والتدرجات والوسائط.

لمعرفة المزيد حول ذلك والمزايا، تحقق من [مدونة النموذج المتوازي للبيانات المجزأة بالكامل](https://pytorch.org/blog/introducing-pytorch-fully-sharded-data-parallel-api/).

لقد قمنا بتكامل أحدث ميزة تدريب PyTorch's Fully Sharded Data Parallel (FSDP). كل ما عليك فعله هو تمكينه من خلال config.

## كيف يعمل خارج الصندوق

قم ببساطة بتشغيل ما يلي على آلتك (آلاتك):

```bash
accelerate config
```

والإجابة عن الأسئلة المطروحة. سيؤدي هذا إلى إنشاء ملف تكوين سيتم استخدامه تلقائيًا لتعيين الخيارات الافتراضية بشكل صحيح عند القيام بما يلي:

```bash
accelerate launch my_script.py --args_to_my_script
```

على سبيل المثال، فيما يلي كيفية تشغيل `examples/nlp_example.py` (من الجذر الخاص بالمستودع) مع تمكين FSDP:

```bash
compute_environment: LOCAL_MACHINE
debug: false
distributed_type: FSDP
downcast_bf16: 'no'
fsdp_config:
  fsdp_auto_wrap_policy: TRANSFORMER_BASED_WRAP
  fsdp_backward_prefetch_policy: BACKWARD_PRE
  fsdp_forward_prefetch: false
  fsdp_cpu_ram_efficient_loading: true
  fsdp_offload_params: false
  fsdp_sharding_strategy: FULL_SHARD
  fsdp_state_dict_type: SHARDED_STATE_DICT
  fsdp_sync_module_states: true
  fsdp_transformer_layer_cls_to_wrap: BertLayer
  fsdp_use_orig_params: true
machine_rank: 0
main_training_function: main
mixed_precision: bf16
num_machines: 1
num_processes: 2
rdzv_backend: static
same_network: true
tpu_env: []
tpu_use_cluster: false
tpu_use_sudo: false
use_cpu: false
```

```bash
accelerate launch examples/nlp_example.py
```

يدعم `Accelerate` حاليًا التكوين التالي عبر CLI:

`fsdp_sharding_strategy`: [1] FULL_SHARD (تجزئة حالات المحسن والتدرجات والوسائط)، [2] SHARD_GRAD_OP (تجزئة حالات المحسن والتدرجات)، [3] NO_SHARD (DDP)، [4] HYBRID_SHARD (تجزئة حالات المحسن والتدرجات والوسائط داخل كل عقدة في حين أن لكل عقدة نسخة كاملة)، [5] HYBRID_SHARD_ZERO2 (تجزئة حالات المحسن والتدرجات داخل كل عقدة في حين أن لكل عقدة نسخة كاملة). لمزيد من المعلومات، يرجى الرجوع إلى وثائق PyTorch الرسمية [هنا](https://pytorch.org/docs/stable/fsdp.html#torch.distributed.fsdp.ShardingStrategy).

`fsdp_offload_params` : يقرر ما إذا كان سيتم نقل المعلمات والتدرجات إلى وحدة المعالجة المركزية

`fsdp_auto_wrap_policy`: [1] TRANSFORMER_BASED_WRAP، [2] SIZE_BASED_WRAP، [3] NO_WRAP

`fsdp_transformer_layer_cls_to_wrap`: ينطبق فقط على 🤗 Transformers. عندما يكون `fsdp_auto_wrap_policy=TRANSFORMER_BASED_WRAP`، يمكن للمستخدم توفير سلسلة مفصولة بفواصل لعناوين فئات طبقة المحول (مع مراعاة حالة الأحرف)، مثل `BertLayer`، `GPTJBlock`، `T5Block`، `BertLayer،BertEmbeddings،BertSelfOutput`. هذا مهم لأن الوحدات الفرعية التي تشترك في الأوزان (مثل طبقات التعلم) يجب ألا تنتهي في وحدات FSDP مختلفة. باستخدام هذه السياسة، يحدث التغليف لكل كتلة تحتوي على Attention متعدد الرؤوس تليها طبقات MLP. يتم تغليف الطبقات المتبقية بما في ذلك التعلم المشترك بشكل مناسب في نفس وحدة FSDP الخارجية. لذلك، استخدم هذا للنماذج المستندة إلى المحول. يمكنك استخدام `model._no_split_modules` لـ 🤗 Transformer models من خلال الإجابة بـ "نعم" على "هل تريد استخدام` _no_split_modules` للطراز؟" سيحاول استخدام `model._no_split_modules` عند الإمكان.

`fsdp_min_num_params`: الحد الأدنى لعدد المعلمات عند استخدام `fsdp_auto_wrap_policy=SIZE_BASED_WRAP`.

`fsdp_backward_prefetch_policy`: [1] BACKWARD_PRE، [2] BACKWARD_POST، [3] NO_PREFETCH

`fsdp_forward_prefetch`: إذا كان صحيحًا، فسيقوم FSDP بشكل صريح باستباق التجميع التالي أثناء التنفيذ في تمرير الإرسال. يجب استخدامه فقط لنماذج الرسوم البيانية الثابتة نظرًا لأن الاستباق يتبع ترتيب التنفيذ الخاص بالتكرار الأول. أي إذا تم تغيير ترتيب الوحدات الفرعية ديناميكيًا أثناء تنفيذ النموذج، فلا تقم بتمكين هذه الميزة.

`fsdp_state_dict_type`: [1] FULL_STATE_DICT، [2] LOCAL_STATE_DICT، [3] SHARDED_STATE_DICT

`fsdp_use_orig_params`: إذا كان صحيحًا، فيسمح بـ `requires_grad` غير الموحد أثناء التهيئة، مما يعني دعمًا للمعلمات المجمدة والقابلة للتدريب المتناثرة. هذا الإعداد مفيد في حالات مثل الضبط الدقيق الفعال للمعلمات كما هو موضح في [هذه المشاركة](https://dev-discuss.pytorch.org/t/rethinking-pytorch-fully-sharded-data-parallel-fsdp-from-first-principles/1019). يسمح هذا الخيار أيضًا بوجود عدة مجموعات من المعلمات المحسنة. يجب أن يكون هذا `True` عند إنشاء محسن قبل إعداد/تغليف النموذج باستخدام FSDP.

`fsdp_cpu_ram_efficient_loading`: ينطبق فقط على نماذج 🤗 Transformers. إذا كان صحيحًا، فإن العملية الأولى فقط تقوم بتحميل نقطة تفتيش النموذج المسبق التدريب في حين أن جميع العمليات الأخرى لها أوزان فارغة. يجب تعيين هذا على false إذا واجهت أخطاء عند تحميل نموذج 🤗 Transformers المسبق التدريب عبر طريقة `from_pretrained`. عندما يكون هذا الإعداد صحيحًا، يجب أيضًا أن يكون `fsdp_sync_module_states` صحيحًا، وإلا فستكون جميع العمليات باستثناء العملية الرئيسية ذات أوزان عشوائية تؤدي إلى سلوك غير متوقع أثناء التدريب. لكي يعمل ذلك، تأكد من تهيئة مجموعة العمليات الموزعة قبل استدعاء طريقة `from_pretrained` الخاصة بـ Transformers. عند استخدام واجهة برمجة تطبيقات مدرب 🤗، يتم تهيئة مجموعة العمليات الموزعة عند إنشاء مثيل لفئة `TrainingArguments`.

`fsdp_sync_module_states`: إذا كان صحيحًا، فستقوم كل وحدة FSDP ملفوفة بشكل فردي ببث معلمات الوحدة النمطية من الرتبة 0.

للحصول على تحكم إضافي وأكثر دقة، يمكنك تحديد معلمات FSDP الأخرى عبر `FullyShardedDataParallelPlugin`.

عند إنشاء كائن `FullyShardedDataParallelPlugin`، قم بتمريره إلى المعلمات التي لم تكن جزءًا من تكوين التسريع أو إذا كنت تريد تجاوزها.

سيتم تحديد معلمات FSDP بناءً على ملف تكوين التسريع أو حجج أمر الإطلاق، وستقوم المعلمات الأخرى التي تقوم بتمريرها مباشرةً من خلال كائن `FullyShardedDataParallelPlugin` بتعيين/تجاوز ذلك.

فيما يلي مثال:

```py
from accelerate import FullyShardedDataParallelPlugin
from torch.distributed.fsdp.fully_sharded_data_parallel import FullOptimStateDictConfig, FullStateDictConfig

fsdp_plugin = FullyShardedDataParallelPlugin(
    state_dict_config=FullStateDictConfig(offload_to_cpu=False, rank0_only=False),
    optim_state_dict_config=FullOptimStateDictConfig(offload_to_cpu=False, rank0_only=False),
)

accelerator = Accelerator(fsdp_plugin=fsdp_plugin)
```

## الحفظ والتحميل

الطريقة الجديدة الموصى بها لإنشاء نقطة تفتيش عند استخدام نماذج FSDP هي استخدام `SHARDED_STATE_DICT` كـ `StateDictType` عند إعداد تكوين التسريع.

فيما يلي مقتطف من التعليمات البرمجية لحفظ باستخدام برنامج المساعدة `save_state` في Accelerate:

```py
accelerator.save_state("ckpt")
```

تفقد مجلد نقطة التفتيش لرؤية النموذج والمحسن كشظايا لكل عملية:

```
ls ckpt
# optimizer_0  pytorch_model_0  random_states_0.pkl  random_states_1.pkl  scheduler.bin

cd ckpt

ls optimizer_0
# __0_0.distcp  __1_0.distcp

ls pytorch_model_0
# __0_0.distcp  __1_0.distcp
```

لتحميلها مرة أخرى لاستئناف التدريب، استخدم برنامج المساعدة `load_state` في Accelerate:

```py
accelerator.load_state("ckpt")
```

عند استخدام `save_pretrained` من Transformers، قم بتمرير `state_dict=accelerator.get_state_dict(model)` لحفظ حالة قاموس النموذج.

فيما يلي مثال:

```diff
  unwrapped_model.save_pretrained(
      args.output_dir,
      is_main_process=accelerator.is_main_process,
      save_function=accelerator.save,
+     state_dict=accelerator.get_state_dict(model),
)
```

### قاموس الحالة

سيقوم `accelerator.get_state_dict` باستدعاء تنفيذ `model.state_dict` الأساسي باستخدام `FullStateDictConfig(offload_to_cpu=True، rank0_only=True)` كمدير سياق للحصول على قاموس الحالة للرتبة 0 فقط وسيتم نقله إلى وحدة المعالجة المركزية.

يمكنك بعد ذلك تمرير `state` إلى طريقة `save_pretrained`. هناك عدة أوضاع لـ `StateDictType` و`FullStateDictConfig` يمكنك استخدامها للتحكم في سلوك `state_dict`. لمزيد من المعلومات، راجع [وثائق PyTorch](https://pytorch.org/docs/stable/fsdp.html).

إذا اخترت استخدام `StateDictType.SHARDED_STATE_DICT`، فسيتم تقسيم أوزان النموذج أثناء `Accelerator.save_state` إلى `n` ملفات لكل قسم فرعي في النموذج. لدمجها مرة أخرى في قاموس واحد لتحميلها مرة أخرى في النموذج لاحقًا بعد التدريب، يمكنك استخدام برنامج المساعدة `merge_weights`:

```py
from accelerate.utils import merge_fsdp_weights

# يتم حفظ أوزاننا عادةً في مجلد `pytorch_model_fsdp_{model_number}`
merge_fsdp_weights("pytorch_model_fsdp_0", "output_path", safe_serialization=True)
```

ستكون النتيجة النهائية محفوظة إما في `model.safetensors` أو `pytorch_model.bin` (إذا تم تمرير `safe_serialization=False`).

يمكن استدعاء هذا أيضًا باستخدام CLI:

```bash
accelerate merge-weights pytorch_model_fsdp_0/ output_path
```

## التطابق بين استراتيجيات تجزئة FSDP ومراحل DeepSpeed ZeRO

* `FULL_SHARD` يتوافق مع DeepSpeed `ZeRO Stage-3`. تجزئة حالات المحسن والتدرجات والوسائط.
* `SHARD_GRAD_OP` يتوافق مع DeepSpeed `ZeRO Stage-2`. تجزئة حالات المحسن والتدرجات.
* `NO_SHARD` يتوافق مع `ZeRO Stage-0`. لا توجد تجزئة حيث تحتوي كل وحدة معالجة رسومية على نسخة كاملة من النموذج وحالات المحسن والتدرجات.
* `HYBRID_SHARD` يتوافق مع `ZeRO++ Stage-3` حيث `zero_hpz_partition_size=<num_gpus_per_node>`. هنا، سيتم تجزئة حالات المحسن والتدرجات والوسائط داخل كل عقدة في حين أن لكل عقدة نسخة كاملة.

## بعض التحذيرات التي يجب مراعاتها

- في حالة وجود عدة نماذج، قم بتمرير المحسنات إلى مكالمة الإعداد بنفس الترتيب الخاص بالنماذج المقابلة، وإلا فإن `accelerator.save_state()` و`accelerator.load_state()` ستؤدي إلى سلوك غير صحيح/غير متوقع.
- هذه الميزة غير متوافقة مع `--predict_with_generate` في نص `run_translation.py` في مكتبة 🤗 `Transformers`.

لمزيد من التحكم، يمكن للمستخدمين الاستفادة من `FullyShardedDataParallelPlugin`. بعد إنشاء مثيل لهذه الفئة، يمكن للمستخدمين تمريره إلى مثيل فئة Accelerator.

لمزيد من المعلومات حول هذه الخيارات، يرجى الرجوع إلى رمز PyTorch [FullyShardedDataParallel](https://github.com/pytorch/pytorch/blob/0df2e863fbd5993a7b9e652910792bd21a516ff3/torch/distributed/fsdp/fully_sharded_data_parallel.py#L236).

<Tip>

بالنسبة لأولئك المهتمين بأوجه التشابه والاختلاف بين FSDP وDeepSpeed، يرجى الاطلاع على [دليل المفاهيم هنا](../concept_guides/fsdp_and_deepspeed.md)!

</Tip>