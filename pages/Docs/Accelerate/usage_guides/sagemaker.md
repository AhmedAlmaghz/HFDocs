# Amazon SageMaker

قدمت Hugging Face و Amazon حاويات جديدة لـ [Hugging Face Deep Learning Containers (DLCs)](https://github.com/aws/deep-learning-containers/blob/master/available_images.md#huggingface-training-containers) لتسهيل تدريب نماذج Hugging Face Transformer في [Amazon SageMaker](https://aws.amazon.com/sagemaker/).

## البدء

### الإعداد والتثبيت

قبل أن تتمكن من تشغيل نصوص 🤗 Accelerate على Amazon SageMaker، يجب عليك الاشتراك في حساب AWS. إذا لم يكن لديك حساب AWS بعد، تعرف على المزيد [هنا](https://docs.aws.amazon.com/sagemaker/latest/dg/gs-set-up.html).

بعد حصولك على حساب AWS، يجب تثبيت حزمة برامج sagemaker لـ 🤗 Accelerate باستخدام ما يلي:

```bash
pip install "accelerate[sagemaker]" --upgrade
```

🤗 Accelerate يستخدم حاليًا حاويات 🤗 DLC، مع `transformers` و`datasets` و`tokenizers` مثبتة مسبقًا. 🤗 Accelerate غير متوفرة في DLC بعد (سيتم إضافتها قريبًا!) لذلك لاستخدامها داخل Amazon SageMaker، يجب عليك إنشاء ملف `requirements.txt` في نفس الدليل حيث يوجد نص التدريب الخاص بك وإضافته كاعتماد:

```
accelerate
```

يجب أيضًا إضافة أي تبعيات أخرى لديك إلى هذا الملف `requirements.txt`.

### تكوين 🤗 Accelerate

يمكنك تكوين تكوين التشغيل لـ Amazon SageMaker بنفس الطريقة التي تقوم بها لوظائف التدريب غير SageMaker باستخدام واجهة سطر الأوامر 🤗 Accelerate:

```bash
accelerate config
# في أي بيئة حوسبة تعمل؟ ([0] هذه الآلة، [1] AWS (Amazon SageMaker)): 1
```

🤗 Accelerate ستمر عبر استبيان حول إعداد Amazon SageMaker وإنشاء ملف تكوين يمكنك تحريره.

<Tip>

🤗 Accelerate لا يحفظ أي من بيانات اعتمادك.

</Tip>

### إعداد نص ضبط الدقة لـ 🤗 Accelerate

نص التدريب مشابه جدًا لنص التدريب الذي قد تقوم بتشغيله خارج SageMaker، ولكن لحفظ نموذجك بعد التدريب، يجب عليك تحديد إما `/opt/ml/model` أو استخدام `os.environ ["SM_MODEL_DIR"]` كدليل حفظ الخاص بك. بعد التدريب، يتم تحميل البرامج النصية في هذا الدليل إلى S3:

```diff
- torch.save('/opt/ml/model`)
+ accelerator.save('/opt/ml/model')
```

<Tip warning={true}>

لا تدعم SageMaker إجراءات argparse. إذا كنت تريد استخدام، على سبيل المثال، الوسيطات المنطقية، فيجب عليك تحديد النوع كقيمة منطقية في نصك وتوفير قيمة صريحة صحيحة أو خاطئة لهذا الوسيط. [[REF]](https://sagemaker.readthedocs.io/en/stable/frameworks/pytorch/using_pytorch.html#prepare-a-pytorch-training-script).

</Tip>

### إطلاق التدريب

يمكنك إطلاق التدريب باستخدام واجهة سطر الأوامر 🤗 Accelerate باستخدام ما يلي:

```
accelerate launch path_to_script.py --args_to_the_script
```

سيؤدي هذا إلى تشغيل نص التدريب باستخدام تكوينك. كل ما عليك فعله هو توفير جميع الحجج التي يحتاجها نص التدريب الخاص بك كحجج مسماة.

**أمثلة**

<Tip>

إذا كنت تشغل أحد النصوص المثالية، فلا تنس إضافة `accelerator.save('/opt/ml/model')` إليه.

</Tip>

```bash
accelerate launch ./examples/sagemaker_example.py
```

الإخراج:

```
Configuring Amazon SageMaker environment
Converting Arguments to Hyperparameters
Creating Estimator
2021-04-08 11:56:50 Starting - Starting the training job...
2021-04-08 11:57:13 Starting - Launching requested ML instancesProfilerReport-1617883008: InProgress
.........
2021-04-08 11:58:54 Starting - Preparing the instances for training.........
2021-04-08 12:00:24 Downloading - Downloading input data
2021-04-08 12:00:24 Training - Downloading the training image..................
2021-04-08 12:03:39 Training - Training image download completed. Training in progress..
........
epoch 0: {'accuracy': 0.7598039215686274, 'f1': 0.8178438661710037}
epoch 1: {'accuracy': 0.8357843137254902, 'f1': 0.882249560632689}
epoch 2: {'accuracy': 0.8406862745098039, 'f1': 0.8869565217391304}
........
2021-04-08 12:05:40 Uploading - Uploading generated training model
2021-04-08 12:05:40 Completed - Training job completed
Training seconds: 331
Billable seconds: 331
You can find your model data at: s3://your-bucket/accelerate-sagemaker-1-2021-04-08-11-56-47-108/output/model.tar.gz
```

## الميزات المتقدمة

### التدريب الموزع: الموازاة بالبيانات

قم بإعداد تكوين التعجيل عن طريق تشغيل `accelerate config` والإجابة على أسئلة SageMaker وإعداده. لاستخدام SageMaker DDP، حدد ذلك عند السؤال

`ما هو الوضع الموزع؟ ([0] لا يوجد تدريب موزع، [1] الموازاة بالبيانات):`.

مثال التكوين أدناه:

```yaml
base_job_name: accelerate-sagemaker-1
compute_environment: AMAZON_SAGEMAKER
distributed_type: DATA_PARALLEL
ec2_instance_type: ml.p3.16xlarge
iam_role_name: xxxxx
image_uri: null
mixed_precision: fp16
num_machines: 1
profile: xxxxx
py_version: py38
pytorch_version: 1.10.2
region: us-east-1
transformers_version: 4.17.0
use_cpu: false
```

### التدريب الموزع: الموازاة بالنماذج

*قيد التطوير حاليًا، وسيتم دعمه قريبًا.*

### حزم Python والاعتمادات

🤗 Accelerate يستخدم حاليًا حاويات 🤗 DLC، مع `transformers` و`datasets` و`tokenizers` مثبتة مسبقًا. إذا كنت تريد استخدام حزم Python مختلفة/أخرى، فيمكنك القيام بذلك عن طريق إضافتها إلى ملف `requirements.txt`. سيتم تثبيت هذه الحزم قبل بدء نص التدريب الخاص بك.

### التدريب المحلي: وضع SageMaker المحلي

يسمح الوضع المحلي في حزمة برامج SageMaker بتشغيل نص التدريب الخاص بك محليًا داخل حاوية HuggingFace DLC (حاوية التعلم العميق) أو باستخدام صورة حاوية مخصصة. هذا مفيد لتصحيح أخطاء نص التدريب الخاص بك واختباره داخل بيئة الحاوية النهائية.

يستخدم الوضع المحلي Docker compose (*ملاحظة: Docker Compose V2 غير مدعوم بعد*). ستتعامل حزمة البرامج مع المصادقة مقابل ECR لسحب DLC إلى بيئتك المحلية. يمكنك محاكاة وظائف التدريب CPU (وحدة المعالجة المركزية) (وحدة معالجة مركزية واحدة ومتعددة) وGPU (وحدة معالجة الرسومات) (وحدة معالجة رسومات واحدة) في SageMaker.

لاستخدام الوضع المحلي، يجب عليك تعيين نوع مثيل EC2 الخاص بك إلى `local`.

```yaml
ec2_instance_type: local
```

### التكوين المتقدم

يسمح التكوين بتجاوز المعلمات لـ [المقدر](https://sagemaker.readthedocs.io/en/stable/api/training/estimators.html). يجب تطبيق هذه الإعدادات في ملف التكوين وهي ليست جزءًا من `accelerate config`. يمكنك التحكم في العديد من الجوانب الإضافية لوظيفة التدريب، على سبيل المثال، استخدام مثيلات Spot، وتمكين العزل الشبكي والمزيد.

```yaml
additional_args:
# تمكين العزل الشبكي لتقييد الوصول إلى الإنترنت للحاويات
enable_network_isolation: True
```

يمكنك العثور على جميع التكوينات المتاحة [هنا](https://sagemaker.readthedocs.io/en/stable/api/training/estimators.html).

### استخدام مثيلات Spot

يمكنك استخدام مثيلات Spot، على سبيل المثال، باستخدام (راجع [التكوين المتقدم](#التكوين المتقدم)):

```yaml
additional_args:
use_spot_instances: True
max_wait: 86400
```

*ملاحظة: تخضع مثيلات Spot للإنهاء واستمرار التدريب من نقطة تفتيش. لا يتم التعامل مع هذا في 🤗 Accelerate خارج الصندوق. اتصل بنا إذا كنت تريد هذه الميزة.*

### النصوص البعيدة: استخدام النصوص الموجودة على GitHub

*لم يتم تحديد ما إذا كانت الميزة مطلوبة. اتصل بنا إذا كنت تريد هذه الميزة.*