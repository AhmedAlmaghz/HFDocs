# مثال Zoo

فيما يلي قائمة غير شاملة بالبرامج التعليمية والنصوص البرمجية التي تعرض 🤗 Accelerate

## أمثلة Accelerate الرسمية:

### الأمثلة الأساسية:

تعرض هذه الأمثلة الميزات الأساسية لـ Accelerate وهي نقطة انطلاق رائعة.

- [مثال NLP الأساسي](https://github.com/huggingface/accelerate/blob/main/examples/nlp_example.py)
- [مثال NLP الموزع الأساسي في Jupyter Notebook](https://github.com/huggingface/notebooks/blob/main/examples/accelerate_examples/simple_nlp_example.ipynb)
- [مثال رؤية حاسوبية أساسي](https://github.com/huggingface/accelerate/blob/main/examples/cv_example.py)
- [مثال رؤية حاسوبية موزع أساسي في Jupyter Notebook](https://github.com/huggingface/notebooks/blob/main/examples/accelerate_examples/simple_cv_example.ipynb)
- [استخدام Accelerate في Kaggle](https://www.kaggle.com/code/muellerzr/multi-gpu-and-accelerate)

### أمثلة خاصة بالميزات:

تعرض هذه الأمثلة ميزات محددة يقدمها إطار عمل Accelerate.

- [التجميع التلقائي للذاكرة](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/automatic_gradient_accumulation.py)
- [حالات التثبيت](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/checkpointing.py)
- [التحقق من صحة التعابر](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/cross_validation.py)
- [DeepSpeed](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/deepspeed_with_config_support.py)
- [Fully Sharded Data Parallelism](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/fsdp_with_peak_mem_tracking.py)
- [تجميع التدرجات](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/gradient_accumulation.py)
- [محدد حجم الدفعة الواعية بالذاكرة](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/memory.py)
- [حساب القياس](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/multi_process_metrics.py)
- [استخدام أجهزة التتبع](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/tracking.py)
- [استخدام Megatron-LM](https://github.com/huggingface/accelerate/blob/main/examples/by_feature/megatron_lm_gpt_pretraining.py)

### الأمثلة الكاملة:

تعرض هذه الأمثلة كل ميزة في Accelerate في نفس الوقت الذي تم عرضه في "أمثلة خاصة بالميزات".

- [مثال NLP الكامل](https://github.com/huggingface/accelerate/blob/main/examples/complete_nlp_example.py)
- [مثال الرؤية الحاسوبية الكامل](https://github.com/huggingface/accelerate/blob/main/examples/complete_cv_example.py)
- [مثال رؤية شامل وقابل للتوسيع للغاية يُظهر SLURM و hydra واستخدام إطار العمل القابل للتوسيع للغاية](https://github.com/yuvalkirstain/PickScore)
- [مثال ضبط نموذج اللغة السببية](https://github.com/huggingface/transformers/blob/main/examples/pytorch/language-modeling/run_clm_no_trainer.py)
- [مثال ضبط نموذج اللغة المُقنعة](https://github.com/huggingface/transformers/blob/main/examples/pytorch/language-modeling/run_mlm_no_trainer.py)
- [مثال الضبط المسبق للكلام](https://github.com/huggingface/transformers/blob/main/examples/pytorch/speech-pretraining/run_wav2vec2_pretraining_no_trainer.py)
- [مثال الضبط الدقيق للترجمة](https://github.com/huggingface/transformers/blob/main/examples/pytorch/translation/run_translation_no_trainer.py)
- [مثال الضبط الدقيق لتصنيف النصوص](https://github.com/huggingface/transformers/blob/main/examples/pytorch/text-classification/run_glue_no_trainer.py)
- [مثال الضبط الدقيق لتجزئة الصور](https://github.com/huggingface/transformers/blob/main/examples/pytorch/semantic-segmentation/run_semantic_segmentation_no_trainer.py)
- [مثال الضبط الدقيق للإجابة على الأسئلة](https://github.com/huggingface/transformers/blob/main/examples/pytorch/question-answering/run_qa_no_trainer.py)
- [مثال الضبط الدقيق للإجابة على الأسئلة باستخدام البحث الشعاعي](https://github.com/huggingface/transformers/blob/main/examples/pytorch/question-answering/run_qa_beam_search_no_trainer.py)
- [مثال الضبط الدقيق للإجابة على أسئلة الاختيار المتعدد](https://github.com/huggingface/transformers/blob/main/examples/pytorch/multiple-choice/run_swag_no_trainer.py)
- [مثال الضبط الدقيق للتعرف على الكيانات المسماة](https://github.com/huggingface/transformers/blob/main/examples/pytorch/token-classification/run_ner_no_trainer.py)
- [مثال الضبط الدقيق لتصنيف الصور](https://github.com/huggingface/transformers/blob/main/examples/pytorch/image-classification/run_image_classification_no_trainer.py)
- [مثال الضبط الدقيق للتلخيص](https://github.com/huggingface/transformers/blob/main/examples/pytorch/summarization/run_summarization_no_trainer.py)
- [أمثلة شاملة حول كيفية استخدام تكامل AWS SageMaker لـ Accelerate](https://github.com/huggingface/notebooks/blob/main/sagemaker/22_accelerate_sagemaker_examples/README.md)
- [أمثلة Megatron-LM لمهام NLP المختلفة](https://github.com/pacman100/accelerate-megatron-test)

## أمثلة التكامل:

هذه هي البرامج التعليمية من المكتبات التي تدمج مع 🤗 Accelerate:

> هل لا تجد تكامل هنا؟ قم بإنشاء طلب سحب لإضافته!

### Amphion

- [تدريب نماذج Text-to-Speech باستخدام Amphion](https://github.com/open-mmlab/Amphion/blob/main/egs/tts/README.md)
- [تدريب نماذج تحويل صوت الغناء باستخدام Amphion](https://github.com/open-mmlab/Amphion/blob/main/egs/svc/README.md)
- [تدريب Vocoders باستخدام Amphion](https://github.com/open-mmlab/Amphion/blob/main/egs/vocoder/README.md)

### المحفز

- [برنامج تعليمي للتدريب الموزع باستخدام المحفز](https://catalyst-team.github.io/catalyst/tutorials/ddp.html)

### DALLE2-pytorch

- [ضبط دقيق لـ DALLE2](https://github.com/lucidrains/DALLE2-pytorch#usage)

### 🤗 الناشرون

- [أداء الانعكاس النصي باستخدام الناشرون](https://github.com/huggingface/diffusers/tree/main/examples/textual_inversion)
- [تدريب DreamBooth باستخدام الناشرون](https://github.com/huggingface/diffusers/tree/main/examples/dreambooth)

### fastai

- [التدريب الموزع من Jupyter Notebooks باستخدام fastai](https://docs.fast.ai/tutorial.distributed.html)
- [أمثلة التدريب الموزع الأساسية باستخدام fastai](https://docs.fast.ai/examples/distributed_app_examples.html)

### GradsFlow

- [التصنيف التلقائي للصور باستخدام GradsFlow](https://docs.gradsflow.com/en/latest/examples/nbs/01-ImageClassification/)

### imagen-pytorch

- [ضبط دقيق لـ Imagen](https://github.com/lucidrains/imagen-pytorch#usage)

### Kornia

- [ضبط دقيق لنماذج الرؤية باستخدام مدرب Kornia](https://kornia.readthedocs.io/en/latest/get-started/training.html)

### PyTorch Accelerated

- [برنامج تعليمي للبدء السريع للتدريب الموزع باستخدام PyTorch Accelerated](https://pytorch-accelerated.readthedocs.io/en/latest/quickstart.html)

### PyTorch3D

- [أداء التعلم العميق مع بيانات ثلاثية الأبعاد](https://pytorch3d.org/tutorials/)

### Stable-Dreamfusion

- [التدريب باستخدام Stable-Dreamfusion لتحويل النص إلى نموذج ثلاثي الأبعاد](https://colab.research.google.com/drive/1MXT3yfOFvO0ooKEfiUUvTKwUkrrlCHpF?usp=sharing)

### Tez

- [كشف أمراض الأوراق باستخدام Tez و Accelerate](https://www.kaggle.com/code/abhishek/tez-faster-and-easier-training-for-leaf-detection/notebook)

### trlx

- [كيفية تنفيذ مهمة تعلم المشاعر باستخدام trlx](https://github.com/CarperAI/trlx#example-how-to-add-a-task)

### Comfy-UI

- [تمكين استخدام نماذج Stable Diffusion الكبيرة في إعدادات vram المنخفضة باستخدام Accelerate](https://github.com/comfyanonymous/ComfyUI/blob/master/comfy/model_management.py#L291-L296)
## في العلوم

فيما يلي قائمة غير شاملة بالأوراق البحثية التي تستخدم Hugging Face Accelerate.

> إذا لم تجد ورقتك هنا، قم بإنشاء طلب سحب (Pull Request) لإضافتها!

* يوافال كريستين، وآدم بولياك، وأورييل سينجر، وشاهبولاند ماتيانا، وجو بينا، وعمر ليفي: "Pick-a-Pic: قاعدة بيانات مفتوحة لتفضيلات المستخدم في النص إلى توليد الصور"، 2023؛ [arXiv:2305.01569](http://arxiv.org/abs/2305.01569).
* لي وانج، ووانيو شو، وييهواي لان، وزيقيانج هو، ويونشي لان، وروي كا-وي لي، وإي-بينغ ليم: "تخطيط وإثارة الفكر: تحسين الاستدلال التسلسلي بدون سلسلة الصفر من خلال نماذج اللغة الضخمة"، 2023؛ [arXiv:2305.04091](http://arxiv.org/abs/2305.04091).
* أرثر كامارا، وكلوديا هوف: "نقل الأشياء: دراسة حول كفاءة نقل المستندات إلى الذاكرة لنماذج الاسترجاع العصبية"، 2022؛ [arXiv:2205.08343](http://arxiv.org/abs/2205.08343).
* ينج شينغ، ويانمين تشنغ، وبيهانغ يوان، وزوهوهان لي، وماكس ريبانين، ودانيال ي. فو، وزي تشيانغ شي، وبيدي تشن، وكلارك باريت، وجوزيف إي. غونزاليس، وبيرسي ليانغ، وكريستوفر ريه، وآيون ستوايكا، وتشانغ: "الاستدلال التنموي عالي الإنتاجية لنماذج اللغة الضخمة باستخدام وحدة معالجة الرسومات (GPU) واحدة"، 2023؛ [arXiv:2303.06865](http://arxiv.org/abs/2303.06865).
* بيتر ميلكيور، ويان ليانغ، وشانغهون هان، وأندي جولدينج: "ترميز الطيف المجري الأول: الهندسة المعمارية"، 2022؛ [arXiv:2211.07890](http://arxiv.org/abs/2211.07890).
* جياو تشن، وأستون تشانغ، ومو لي، وأليكس سمولا، ودييي يانغ: "نموذج انتشار أرخص وأفضل للغة مع ضوضاء مقنعة ناعمة"، 2023؛ [arXiv:2304.04746](http://arxiv.org/abs/2304.04746).
* أيان حق، وماثيو تانكيك، وأليكسي أي. إيفروس، وأليكساندر هولينسكي، وأنغجو كانازاوا: "Instruct-NeRF2NeRF: تحرير المشاهد ثلاثية الأبعاد بالتعليمات"، 2023؛ [arXiv:2303.12789](http://arxiv.org/abs/2303.12789).
* لوك ميلاس-كيريازي، وكريستيان روببريشت، وإيرو لاينا، وأندريا فيدالدي: "RealFusion: إعادة بناء 360 درجة لأي كائن من صورة واحدة"، 2023؛ [arXiv:2302.10663](http://arxiv.org/abs/2302.10663).
* شياوشي وو، وكيكيانج صن، وفنج تشو، وروي تشاو، وهونجشنج لي: "تحسين محاذاة نماذج النص إلى الصورة مع التفضيل البشري"، 2023؛ [arXiv:2303.14420](http://arxiv.org/abs/2303.14420).
* يونجليانج شين، وكايتاو سونج، وشي تان، ودونجشنج لي، وويمنج لو، ويويتينغ تشوانج: "HuggingGPT: حل مهام الذكاء الاصطناعي مع ChatGPT وأصدقائه في Hugging Face"، 2023؛ [arXiv:2303.17580](http://arxiv.org/abs/2303.17580).
* يو يانج، وونلين يايو، وهونجمينج تشانج، وشياويانج وانج، ودونج يو، وجيانشو تشن: "Z-LaVI: محدد الصيغة الصفرية المدعوم بالخيال المرئي"، 2022؛ [arXiv:2210.12261](http://arxiv.org/abs/2210.12261).
* شينج-ين تشو، و بين-يو تشن، وتسونج-يي هو: "كيفية اختراق نماذج الانتشار؟"، 2022؛ [arXiv:2212.05400](http://arxiv.org/abs/2212.05400).
* جونيونج سيو، ووسوك جانج، ومين-سوب كواك، وجايهون كو، وهايونسو كيم، وجونهو كيم، وجين-هوا كيم، وجيونج لي، وسيونجريونج كيم: "دع نموذج الانتشار ثنائي الأبعاد يعرف الاتساق ثلاثي الأبعاد للجيل القوي للنص إلى 3D"، 2023؛ [arXiv:2303.07937](http://arxiv.org/abs/2303.07937).
* أور باتاشنيك، ودانيال جاربي، وإيدان أزوري، وهادار أفيربوش-إلور، ودانيال كوهين-أور: "تحديد موقع تباين شكل الكائن باستخدام نماذج الانتشار من النص إلى الصورة"، 2023؛ [arXiv:2303.11306](http://arxiv.org/abs/2303.11306).
* ديداك سوريس، وساشيت مينون، وكارل فونديريك: "ViperGPT: الاستدلال المرئي عبر تنفيذ بايثون للاستدلال"، 2023؛ [arXiv:2303.08128](http://arxiv.org/abs/2303.08128).
* تشنيانج تشي، وشيادونج كون، ويونج تشانج، وتشنيانج لي، وشينتاو وانج، ويينج شان، وكيفن تشين: "FateZero: دمج الانتباه لتحرير الفيديو القائم على النص بدون تصوير"، 2023؛ [arXiv:2303.09535](http://arxiv.org/abs/2303.09535).
* شون ويليك، وجياشنج ليو، وشيمينج لو، وهانانه حاجيشيرزي، ويجين تشوي: "NaturalProver: إنشاء برهان رياضي مدعوم باللغة باستخدام نماذج اللغة"، 2022؛ [arXiv:2205.12910](http://arxiv.org/abs/2205.12910).
* إلاد ريتشاردسون، وجال ميتزر، ويواف ألالوف، وراجا جريس، ودانيال كوهين-أور: "نص الملمس: توجيه النص لتشكيل أشكال 3D"، 2023؛ [arXiv:2302.01721](http://arxiv.org/abs/2302.01721).
* بويجين تشنغ، ولي لين، ويوي جين هوانج، وهوا تشينغ هو، وونهان لوه، وشياوينج تانج: "تعلم التعزيز من التدهور: نموذج انتشار لتعزيز صورة القاع"، 2023؛ [arXiv:2303.04603](http://arxiv.org/abs/2303.04603).
* شون شاو، ويافتاه زيسر، وشاي كوهين: "محو الصفات غير المحاذاة من التمثيلات العصبية"، 2023؛ [arXiv:2302.02997](http://arxiv.org/abs/2302.02997).
* سيونجهيون يي، ويونجبين هوانج، وسوهي يانج، وهيونجو يون، وييرون كيم، ومينجون سيو: "التعلم التعليمي في السياق"، 2023؛ [arXiv:2302.14691](http://arxiv.org/abs/2302.14691).
* شيكون ليو، ولينكسي فان، وإدوارد جونز، وزهيدينج يو، وتشاووي شياو، وأنيميا أناندكومار: "Prismer: نموذج اللغة والرؤية مع مجموعة من الخبراء"، 2023؛ [arXiv:2303.02506](http://arxiv.org/abs/2303.02506).
* هايو تشين، وزهيهوا وانج، ويانج يانج، و تشيلين صن، وكيدي ما: "تعلم مقياس اختلاف الألوان العميق للصور الفوتوغرافية"، 2023؛ [arXiv:2303.14964](http://arxiv.org/abs/2303.14964).
* فان-هوانج لي، وهونجيو تشانج: "تحليل السجلات باستخدام التعلم القائم على التوجيه القليل التصوير"، 2023؛ [arXiv:2302.07435](http://arxiv.org/abs/2302.07435).
* كيتو كودو، ويويتشي أوكي، وتاتسوكي كوريباياشي، وآنا براسارد، وماساشي يوشيكاوا، وكيسوكي ساكاجوتشي، وكينتارو إينوي: "هل تلتقط الشبكات العصبية العميقة التكوين في الاستدلال الحسابي؟"، 2023؛ [arXiv:2302.07866](http://arxiv.org/abs/2302.07866).
* روياو وانج، وبيتر جانسن، ومارك-ألكسندر كوت، وبريثفيراج أمانابرو: "المحولات المستنسخة للسلوك هي أجهزة استدلال رمزية عصبية"، 2022؛ [arXiv:2210.07382](http://arxiv.org/abs/2210.07382).
* مارتن ويسل، وتوماش هوريتش، وتيري رواس، وأكيكو أيزاوا، وبيلا جيب، وتيمو سبندي: "تقديم MBIB - أول مجموعة بيانات وتعيين مهام لتحديد الانحياز الإعلامي"، 2023؛ [arXiv:2304.13148](http://arxiv.org/abs/2304.13148). DOI: [https://dx.doi.org/10.1145/3539618.3591882 10.1145/3539618.3591882].
* هيلا شيفر، ويواف ألالوف، ويائيل فينكر، وليور وولف، ودانيال كوهين-أور: "Attend-and-Excite: التوجيه الدلالي القائم على الاهتمام لنماذج الانتشار من النص إلى الصورة"، 2023؛ [arXiv:2301.13826](http://arxiv.org/abs/2301.13826).
* مارسيو فونسيكا، ويافتاه زيسر، وشاي ب. كوهين: "عامل المحتوى وقرارات الميزانية في الملخص الاستخلاصي للوثائق الطويلة"، 2022؛ [arXiv:2205.12486](http://arxiv.org/abs/2205.12486).
* إلاد ريتشاردسون، وجال ميتزر، ويواف ألالوف، وراجا جريس، ودانيال كوهين-أور: "نص الملمس: توجيه النص لتشكيل أشكال 3D"، 2023؛ [arXiv:2302.01721](http://arxiv.org/abs/2302.01721).
* تيانكسينج هي، وجينغيو تشانج، وتيانلي وانج، وساتشين كومار، وكيونجهيون تشو، وجيمس جلاس، ويوليا تسفيتكوف: "حول البقع العمياء لتقييم المهام المستندة إلى النماذج لتوليد النصوص"، 2022؛ [arXiv:2212.10020](http://arxiv.org/abs/2212.10020).
* أوري رام، ويواف ليفين، وإيتاي دالميديجوس، ودور موهليجاي، وأمنون شاشوا، وكيفن ليتون-براون، ويواف شوهام: "نماذج اللغة المعززة بالاسترجاع في السياق"، 2023؛ [arXiv:2302.00083](http://arxiv.org/abs/2302.00083).
* داتشنج لي، ورولين شاو، وهونجي وانج، وهان جو، وإريك ب. إكسينج، وهاو تشانج: "MPCFormer: تحويل سريع وفعال وآمن للاستدلال باستخدام MPC"، 2022؛ [arXiv:2211.01452](http://arxiv.org/abs/2211.01452).
* باولين بينج، وميشيل جالي، وبيشنغ هي، وكريس بروكيت، ولارس ليدن، وإلناز نوري، وزو يو، وبيل دولان، وجيانفنغ جاو: "GODEL: التدريب المسبق واسع النطاق للحوار الموجه نحو الهدف"، 2022؛ [arXiv:2206.11309](http://arxiv.org/abs/2206.11309).
* إجيل رونينجستاد، وإريك فيلدال، وليليا أوفيرليد: "تحليل المشاعر على مستوى الكيان (ELSA): دراسة استقصائية حول المهام"، 2023، وقائع المؤتمر الدولي التاسع والعشرين حول اللغويات الحاسوبية، 2022، الصفحات 6773-6783؛ [arXiv:2304.14241](http://arxiv.org/abs/2304.14241).
* تشارلي سنيل، وإيليا كوستريوف، ووي سو، ومينجياو يانج، وسيرجي ليفين: "تعزيز التعلم خارج الاتصال لتوليد اللغة الطبيعية مع التعلم الضمني Q"، 2022؛ [arXiv:2206.11871](http://arxiv.org/abs/2206.11871).
* زيروو وانج، وشويان تشو، ودانيال فرايد، وغراهام نيوبيج: "التقييم القائم على التنفيذ لتوليد التعليمات البرمجية مفتوحة المجال"، 2022؛ [arXiv:2212.10481](http://arxiv.org/abs/2212.10481).
* مينه-لونج لوو، وزيي هوانج، وإريك ب. إكسينج، ويونج جاي لي، وهاوهان وانج: "المزج السريع الموجه بالبروز من خلال عتبات التدرجات العشوائية"، 2022؛ [arXiv:2212.04875](http://arxiv.org/abs/2212.04875).
* جون هاو ليو، وهانشو يان، وداقوان تشو، وجياشي فينج: "MagicMix: المزج الدلالي مع نماذج الانتشار"، 2022؛ [arXiv:2210