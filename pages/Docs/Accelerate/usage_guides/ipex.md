# Intel® Extension for PyTorch

[IPEX](https://github.com/intel/intel-extension-for-pytorch) تم تحسينه للمعالجات التي تستخدم AVX-512 أو أعلى، ويعمل بشكل وظيفي للمعالجات التي تستخدم AVX2 فقط. لذلك، من المتوقع أن يحقق فائدة في الأداء لأجيال معالجات Intel مع AVX-512 أو أعلى، في حين أن المعالجات التي تستخدم AVX2 فقط (مثل معالجات AMD أو معالجات Intel الأقدم) قد تؤدي إلى أداء أفضل مع IPEX، ولكن ذلك غير مضمون. يوفر IPEX تحسينات أداء لتدريب المعالج باستخدام كل من Float32 و BFloat16. ويعد استخدام BFloat16 هو محور التركيز الرئيسي للفروع التالية.

تم دعم نوع البيانات منخفض الدقة BFloat16 بشكل أصلي على معالجات Xeon® Scalable من الجيل الثالث (المعروفة باسم Cooper Lake) مع مجموعة تعليمات AVX512، وسيتم دعمها في الجيل التالي من معالجات Intel® Xeon® Scalable مع مجموعة تعليمات Intel® Advanced Matrix Extensions (Intel® AMX) مع تعزيز الأداء بشكل أكبر. تم تمكين الدقة المختلطة التلقائية لخلفية المعالج منذ PyTorch-1.10. وفي الوقت نفسه، تم تمكين دعم الدقة المختلطة التلقائية مع BFloat16 للمعالج وBFloat16 لتحسين المشغلين بشكل كبير في Intel® Extension for PyTorch، وتم إرسالها جزئيًا إلى فرع PyTorch الرئيسي. يمكن للمستخدمين الحصول على أداء أفضل وتجربة مستخدم محسنة مع الدقة المختلطة التلقائية IPEX.

## تثبيت IPEX:

يتبع إصدار IPEX إصدار PyTorch، لتثبيته عبر pip:

| إصدار PyTorch | إصدار IPEX |
| :---------------: | :----------: |
| 2.0               |  2.0.0         |
| 1.13              |  1.13.0        |
| 1.12              |  1.12.300      |
| 1.11              |  1.11.200      |
| 1.10              |  1.10.100      |

```
pip install intel_extension_for_pytorch==<version_name> -f https://developer.intel.com/ipex-whl-stable-cpu
```

تحقق من المزيد من الأساليب لتثبيت [IPEX](https://intel.github.io/intel-extension-for-pytorch/cpu/latest/tutorials/installation.html).

## كيف يعمل لتحسين التدريب على المعالج:

قامت 🤗 Accelerate بتكامل [IPEX](https://github.com/intel/intel-extension-for-pytorch)، كل ما عليك فعله هو تمكينه من خلال التكوين.

**السيناريو 1**: تسريع التدريب على معالج CPU غير الموزع

قم بتشغيل <u>accelerate config</u> على جهازك:

```bash
$ accelerate config
-----------------------------------------------------------------------------------------------------------------------------------------------------------
في أي بيئة حوسبة تعمل؟
هذه الآلة
-----------------------------------------------------------------------------------------------------------------------------------------------------------
ما نوع الجهاز الذي تستخدمه؟
لا يوجد تدريب موزع
هل تريد تشغيل التدريب الخاص بك على CPU فقط (حتى إذا كان GPU / جهاز Apple Silicon متاحًا)؟ [yes/NO]:yes
هل تريد استخدام Intel PyTorch Extension (IPEX) لتسريع التدريب على CPU؟ [yes/NO]:yes
هل تريد تحسين نصك البرمجي باستخدام torch dynamo؟ [yes/NO]:NO
هل تريد استخدام DeepSpeed؟ [yes/NO]: NO
-----------------------------------------------------------------------------------------------------------------------------------------------------------
هل تريد استخدام FP16 أو BF16 (الدقة المختلطة)؟
bf16
```

سيؤدي هذا إلى إنشاء ملف تكوين سيتم استخدامه تلقائيًا لتعيين الخيارات الافتراضية بشكل صحيح عند القيام بما يلي:

```bash
accelerate launch my_script.py --args_to_my_script
```

على سبيل المثال، إليك كيفية تشغيل مثال NLP `examples/nlp_example.py` (من الجذر الخاص بالمستودع) مع تمكين IPEX.

default_config.yaml الذي يتم إنشاؤه بعد `accelerate config`

```bash
compute_environment: LOCAL_MACHINE
distributed_type: 'NO'
downcast_bf16: 'no'
ipex_config:
ipex: true
machine_rank: 0
main_training_function: main
mixed_precision: bf16
num_machines: 1
num_processes: 1
rdzv_backend: static
same_network: true
tpu_env: []
tpu_use_cluster: false
tpu_use_sudo: false
use_cpu: true
```

```bash
accelerate launch examples/nlp_example.py
```

**السيناريو 2**: تسريع التدريب على معالج CPU الموزع

نستخدم Intel oneCCL للاتصال، مقترنًا بمكتبة Intel® MPI لتقديم رسائل مجموعات مرنة وفعالة وقابلة للتطوير على بنية Intel®. يمكنك الرجوع إلى [هنا](https://huggingface.co/docs/transformers/perf_train_cpu_many) للحصول على دليل التثبيت.

قم بتشغيل <u>accelerate config</u> على جهازك (node0):

```bash
$ accelerate config
-----------------------------------------------------------------------------------------------------------------------------------------------------------
في أي بيئة حوسبة تعمل؟
هذه الآلة
-----------------------------------------------------------------------------------------------------------------------------------------------------------
ما نوع الجهاز الذي تستخدمه؟
متعدد المعالجات
كم عدد الآلات المختلفة التي ستستخدمها (استخدم أكثر من 1 للتدريب متعدد العقد)؟ [1]: 4
-----------------------------------------------------------------------------------------------------------------------------------------------------------
ما هو ترتيب هذه الآلة؟
0
ما هو عنوان IP للآلة التي ستستضيف العملية الرئيسية؟ 36.112.23.24
ما هو المنفذ الذي ستستخدمه للتواصل مع العملية الرئيسية؟ 29500
هل جميع الآلات على نفس الشبكة المحلية؟ أجب بـ "no" إذا كانت العقد موجودة في السحابة و/أو على مضيفي الشبكة المختلفة [YES/no]: yes
هل تريد استخدام Intel PyTorch Extension (IPEX) لتسريع التدريب على CPU؟ [yes/NO]:yes
هل تريد أن يقوم Accelerate بتشغيل mpirun؟ [yes/NO]: yes
يرجى إدخال مسار ملف المضيف الذي سيتم استخدامه مع mpirun [~/hostfile]: ~/hostfile
أدخل عدد مؤشرات ترابط عامل oneCCL [1]: 1
هل تريد تحسين نصك البرمجي باستخدام torch dynamo؟ [yes/NO]:NO
كم عدد العمليات التي يجب استخدامها للتدريب الموزع؟ [1]:16
-----------------------------------------------------------------------------------------------------------------------------------------------------------
هل تريد استخدام FP16 أو BF16 (الدقة المختلطة)؟
bf16
```

على سبيل المثال، إليك كيفية تشغيل مثال NLP `examples/nlp_example.py` (من الجذر الخاص بالمستودع) مع تمكين IPEX للتدريب على معالج CPU الموزع.

default_config.yaml الذي يتم إنشاؤه بعد `accelerate config`

```bash
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_CPU
downcast_bf16: 'no'
ipex_config:
ipex: true
machine_rank: 0
main_process_ip: 36.112.23.24
main_process_port: 29500
main_training_function: main
mixed_precision: bf16
mpirun_config:
mpirun_ccl: '1'
mpirun_hostfile: /home/user/hostfile
num_machines: 4
num_processes: 16
rdzv_backend: static
same_network: true
tpu_env: []
tpu_use_cluster: false
tpu_use_sudo: false
use_cpu: true
```

قم بتعيين ما يلي واستخدام intel MPI لبدء التدريب:

في node0، تحتاج إلى إنشاء ملف تكوين يحتوي على عناوين IP لكل عقدة (على سبيل المثال hostfile) ومرر مسار ملف التكوين كحجة.

إذا اخترت أن يقوم Accelerate بتشغيل `mpirun`، فتأكد من مطابقة موقع ملف المضيف للمسار الموجود في التكوين.

```bash
$ cat hostfile
xxx.xxx.xxx.xxx #node0 ip
xxx.xxx.xxx.xxx #node1 ip
xxx.xxx.xxx.xxx #node2 ip
xxx.xxx.xxx.xxx #node3 ip
```

عندما يقوم Accelerate بتشغيل `mpirun`، قم بتنفيذ ارتباطات oneCCL setvars.sh للحصول على بيئة Intel MPI الخاصة بك، ثم قم بتشغيل نصك البرمجي باستخدام `accelerate launch`. لاحظ أن النص البرمجي لـ Python والبيئة يجب أن يكونا موجودين على جميع الآلات المستخدمة للتدريب على معالجات CPU متعددة.

```bash
oneccl_bindings_for_pytorch_path=$(python -c "from oneccl_bindings_for_pytorch import cwd; print(cwd)")
source $oneccl_bindings_for_pytorch_path/env/setvars.sh

accelerate launch examples/nlp_example.py
```

من ناحية أخرى، إذا اخترت عدم قيام Accelerate بتشغيل `mpirun`، فقم بتشغيل الأمر التالي في node0 وسيتم
تمكين **16DDP** في node0 وnode1 وnode2 وnode3 مع الدقة المختلطة BF16. عند استخدام هذه الطريقة، يجب أن يكون نص Python البرمجي وبيئة Python وملف تكوين Accelerate موجودين على جميع الآلات المستخدمة للتدريب على معالجات CPU متعددة.

```bash
oneccl_bindings_for_pytorch_path=$(python -c "from oneccl_bindings_for_pytorch import cwd; print(cwd)")
source $oneccl_bindings_for_pytorch_path/env/setvars.sh
export CCL_WORKER_COUNT=1
export MASTER_ADDR=xxx.xxx.xxx.xxx #node0 ip
export CCL_ATL_TRANSPORT=ofi
mpirun -f hostfile -n 16 -ppn 4 accelerate launch examples/nlp_example.py
```

## الموارد ذات الصلة:

- [مشروع على GitHub](https://github.com/intel/intel-extension-for-pytorch)
- [وثائق واجهة برمجة التطبيقات](https://intel.github.io/intel-extension-for-pytorch/cpu/latest/tutorials/api_doc.html)
- [دليل الضبط الدقيق](https://intel.github.io/intel-extension-for-pytorch/cpu/latest/tutorials/performance_tuning/tuning_guide.html)
- [المدونات والمنشورات](https://intel.github.io/intel-extension-for-pytorch/cpu/latest/tutorials/blogs_publications.html)