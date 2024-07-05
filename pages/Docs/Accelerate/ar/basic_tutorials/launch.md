# تشغيل نصوص 🤗 Accelerate الخاصة بك
في البرنامج التعليمي السابق، تم تعريفك على كيفية تعديل نص التدريب الحالي لاستخدام 🤗 Accelerate.
فيما يلي الإصدار النهائي من هذا الرمز:

```python
from accelerate import Accelerator

accelerator = Accelerator()

model, optimizer, training_dataloader, scheduler = accelerator.prepare(
    model, optimizer, training_dataloader, scheduler
)

for batch in training_dataloader:
    optimizer.zero_grad()
    inputs, targets = batch
    outputs = model(inputs)
    loss = loss_function(outputs, targets)
    accelerator.backward(loss)
    optimizer.step()
    scheduler.step()
```

ولكن كيف يمكنك تشغيل هذا الرمز واستخدام الأجهزة الخاصة المتاحة له؟

أولاً، يجب إعادة كتابة الرمز أعلاه في دالة، وجعله قابل للاستدعاء كنص برمجي. على سبيل المثال:

```diff
  from accelerate import Accelerator
  
+ def main():
      accelerator = Accelerator()

      model, optimizer, training_dataloader, scheduler = accelerator.prepare(
          model, optimizer, training_dataloader, scheduler
      )

      for batch in training_dataloader:
          optimizer.zero_grad()
          inputs, targets = batch
          outputs = model(inputs)
          loss = loss_function(outputs, targets)
          accelerator.backward(loss)
          optimizer.step()
          scheduler.step()

+ if __name__ == "__main__":
+     main()
```

بعد ذلك، تحتاج إلى تشغيله باستخدام `accelerate launch`.

<Tip warning={true}>
من المستحسن تشغيل `accelerate config` قبل استخدام `accelerate launch` لتهيئة بيئتك حسب تفضيلاتك.
وإلا، سيستخدم 🤗 Accelerate الافتراضيات الأساسية جدًا اعتمادًا على إعداد نظامك.
</Tip>

## استخدام الأمر accelerate launch
يحتوي 🤗 Accelerate على أمر CLI خاص لمساعدتك في تشغيل الرمز الخاص بك في نظامك من خلال `accelerate launch`.
يغلف هذا الأمر جميع الأوامر المختلفة اللازمة لتشغيل نصك البرمجي على منصات مختلفة، دون الحاجة إلى تذكر كل منها.

<Tip>
إذا كنت معتادًا على تشغيل النصوص البرمجية في PyTorch بنفسك، مثل `torchrun`، فيمكنك الاستمرار في القيام بذلك. ليس من الضروري استخدام `accelerate launch`.
</Tip>

يمكنك تشغيل نصك البرمجي بسرعة باستخدام ما يلي:

```bash
accelerate launch {script_name.py} --arg1 --arg2 ...
```

ضع ببساطة `accelerate launch` في بداية أمرك، ومرر الحجج والمعلمات الإضافية إلى نصك البرمجي لاحقًا كما هو معتاد!

نظرًا لأن هذا الأمر يشغل طرق الإطلاق المختلفة لـ torch، فيمكن تعديل جميع متغيرات البيئة المتوقعة هنا أيضًا.
على سبيل المثال، فيما يلي كيفية استخدام `accelerate launch` مع وحدة معالجة رسومات (GPU) واحدة:

```bash
CUDA_VISIBLE_DEVICES="0" accelerate launch {script_name.py} --arg1 --arg2 ...
```

يمكنك أيضًا استخدام `accelerate launch` دون تنفيذ `accelerate config` أولاً، ولكن قد تحتاج إلى تمرير معلمات التكوين الصحيحة يدويًا.
في هذه الحالة، سيقوم 🤗 Accelerate باتخاذ بعض القرارات حول المعلمات المتغيرة لك، على سبيل المثال، إذا كانت وحدات معالجة الرسومات (GPU) متاحة، فسيستخدمها جميعًا بشكل افتراضي دون الدقة المختلطة.
فيما يلي كيفية استخدام جميع وحدات معالجة الرسومات (GPU) والتدريب مع تعطيل الدقة المختلطة:

```bash
accelerate launch --multi_gpu {script_name.py} {--arg1} {--arg2} ...
```

أو عن طريق تحديد عدد وحدات معالجة الرسومات (GPU) التي سيتم استخدامها:

```bash
accelerate launch --num_processes=2 {script_name.py} {--arg1} {--arg2} ...
```

ولمزيد من التحديد، يجب عليك تمرير المعلمات المطلوبة بنفسك. على سبيل المثال، فيما يلي كيفية
يمكنك أيضًا تشغيل هذا النص البرمجي نفسه على وحدتي معالجة رسومات (GPU) باستخدام الدقة المختلطة مع تجنب جميع التحذيرات:

```bash
accelerate launch --multi_gpu --mixed_precision=fp16 --num_processes=2 {script_name.py} {--arg1} {--arg2} ...
```

لعرض قائمة كاملة من المعلمات التي يمكنك تمريرها، قم بتشغيل:

```bash
accelerate launch -h
```

<Tip>
حتى إذا لم تكن تستخدم 🤗 Accelerate في رمزك، فيمكنك لا تزال استخدام أداة الإطلاق لبدء نصوصك البرمجية!
</Tip>

للحصول على توضيح مرئي لهذا الاختلاف، فإن أمر `accelerate launch` السابق على وحدات معالجة الرسومات (GPU) المتعددة سيبدو شيئًا مثل ما يلي مع `torchrun`:

```bash
MIXED_PRECISION="fp16" torchrun --nproc_per_node=2 --num_machines=1 {script_name.py} {--arg1} {--arg2} ...
```

يمكنك أيضًا تشغيل نصك البرمجي باستخدام واجهة سطر الأوامر (CLI) للإطلاق كنص برمجي Python نفسه، مما يمكّن القدرة على تمرير سلوكيات الإطلاق الأخرى الخاصة بـ Python. للقيام بذلك، استخدم `accelerate.commands.launch` بدلاً من `accelerate launch`:

```bash
python -m accelerate.commands.launch --num_processes=2 {script_name.py} {--arg1} {--arg2}
```

إذا كنت تريد تنفيذ النص البرمجي باستخدام أي أعلام Python أخرى، فيمكنك تمريرها أيضًا بشكل مشابه لـ `-m`، مثل
في المثال التالي الذي يمكّن stdout و stderr غير المؤلف:

```bash
python -u -m accelerate.commands.launch --num_processes=2 {script_name.py} {--arg1} {--arg2}
```

<Tip>
يمكنك تشغيل رمزك على وحدة المعالجة المركزية (CPU) أيضًا! هذا مفيد لأغراض التصحيح والاختبار على نماذج وبيانات الألعاب.
```bash
accelerate launch --cpu {script_name.py} {--arg1} {--arg2}
```
</Tip>

## لماذا يجب عليك دائمًا استخدام `accelerate config`
لماذا من المفيد أن تقوم دائمًا بتشغيل `accelerate config`؟

تذكر مكالمة `accelerate launch` السابقة بالإضافة إلى `torchrun`؟
بعد التهيئة، لتشغيل هذا النص البرمجي مع الأجزاء المطلوبة، فأنت بحاجة فقط إلى استخدام `accelerate launch` بشكل صريح، دون تمرير أي شيء آخر:

```bash
accelerate launch {script_name.py} {--arg1} {--arg2} ...
```

## التهيئات المخصصة
كما ذكرنا سابقًا، يجب استخدام `accelerate launch` بشكل أساسي من خلال الجمع بين التهيئات المحددة
تم إنشاؤها باستخدام أمر `accelerate config`. يتم حفظ هذه التهيئات إلى ملف باسم `default_config.yaml` في مجلد ذاكرة التخزين المؤقت لـ 🤗 Accelerate.
يقع مجلد ذاكرة التخزين المؤقت هذا في (بترتيب تنازلي للأولوية):

- محتوى متغير البيئة `HF_HOME` ملحق بـ `accelerate`.
- إذا لم يكن موجودًا، فسيتم استخدام محتوى متغير البيئة `XDG_CACHE_HOME` ملحقًا بـ
`huggingface/accelerate`.
- إذا لم يكن هذا موجودًا أيضًا، فسيتم استخدام المجلد `~/.cache/huggingface/accelerate`.

للحصول على تهيئات متعددة، يمكن تمرير علم `--config_file` إلى أمر `accelerate launch` مقترنًا
بموقع ملف yaml المخصص.

قد تبدو ملف yaml المخصص كما يلي لوحدتي معالجة رسومات (GPU) على جهاز واحد باستخدام `fp16` للدقة المختلطة:

```yaml
compute_environment: LOCAL_MACHINE
deepspeed_config: {}
distributed_type: MULTI_GPU
fsdp_config: {}
machine_rank: 0
main_process_ip: null
main_process_port: null
main_training_function: main
mixed_precision: fp16
num_machines: 1
num_processes: 2
use_cpu: false
```

يبدو تشغيل نص برمجي من موقع ملف yaml المخصص كما يلي:

```bash
accelerate launch --config_file {path/to/config/my_config_file.yaml} {script_name.py} {--arg1} {--arg2} ...
```

## التدريب متعدد العقد
التدريب متعدد العقد باستخدام 🤗 Accelerate مشابه لـ [التدريب متعدد العقد باستخدام torchrun](https://pytorch.org/tutorials/intermediate/ddp_series_multinode.html). أبسط طريقة لبدء تشغيل التدريب متعدد العقد هي القيام بما يلي:

- قم بنسخ رمزك الأساسي وبياناتك إلى جميع العقد. (أو ضعها في نظام ملفات مشترك)
- قم بتهيئة حزم Python الخاصة بك على جميع العقد.
- قم بتشغيل `accelerate config` على العقدة الفردية الرئيسية أولاً. بعد تحديد عدد العقد، سيُطلب منك تحديد ترتيب كل عقدة (سيكون هذا 0 للعقدة الرئيسية/الرئيسية)، إلى جانب عنوان IP ومنفذ العملية الرئيسية. هذا مطلوب لعقد العمال للتواصل مع العملية الرئيسية. بعد ذلك، يمكنك نسخ أو إرسال ملف التهيئة هذا عبر جميع العقد الخاصة بك، وتغيير `machine_rank` إلى 1 و2 و3، وما إلى ذلك لتجنب الاضطرار إلى تشغيل الأمر (أو اتباع توجيهاتهم مباشرةً للتشغيل باستخدام `torchrun`)
بمجرد قيامك بذلك، يمكنك بدء تشغيل التدريب متعدد العقد عن طريق تشغيل `accelerate launch` (أو `torchrun`) على جميع العقد.

<Tip>
من الضروري تشغيل الأمر على جميع العقد لبدء كل شيء، وليس فقط تشغيله من العقدة الرئيسية. يمكنك استخدام شيء مثل SLURM أو منفذ عملية مختلف للالتفاف حول هذا الشرط واستدعاء كل شيء من أمر واحد.
</Tip>

<Tip>
من المستحسن استخدام عنوان IP الداخلي لعقدتك الرئيسية بدلاً من عنوان IP العام لتحقيق أفضل حالة تأخير. هذا هو عنوان `192.168.x.x` أو `172.x.x.x` الذي تراه عند تشغيل `hostname -I` على العقدة الرئيسية.
</Tip>

للحصول على فكرة أفضل حول التدريب متعدد العقد، تحقق من مثالنا لـ [التدريب متعدد العقد باستخدام FSDP](https://huggingface.co/blog/ram-efficient-pytorch-fsdp).