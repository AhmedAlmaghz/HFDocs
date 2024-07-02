# تكامل PEFT 

تمتد الفوائد العملية لـ PEFT إلى مكتبات Hugging Face الأخرى مثل [Diffusers](https://hf.co/docs/diffusers) و [Transformers](https://hf.co/docs/transformers). تتمثل إحدى المزايا الرئيسية لـ PEFT في أن ملف المحول الذي تم إنشاؤه بواسطة طريقة PEFT أصغر بكثير من النموذج الأصلي، مما يجعله سهل الإدارة والاستخدام لعدة محولات. يمكنك استخدام نموذج أساسي واحد مُدرب مسبقًا لمهام متعددة ببساطة عن طريق تحميل محول جديد مُدرب مسبقًا للمهام التي تقوم بحلها. أو يمكنك الجمع بين محولات متعددة مع نموذج انتشار النص إلى الصورة لإنشاء تأثيرات جديدة.

سيوضح هذا البرنامج التعليمي كيف يمكن لـ PEFT مساعدتك في إدارة المحولات في Diffusers و Transformers.

## الناشرون

Diffusers هي مكتبة الذكاء الاصطناعي للصور ومقاطع الفيديو النصية أو الصور باستخدام نماذج الانتشار. LoRA هي طريقة تدريب شائعة بشكل خاص لنماذج الانتشار لأنك يمكن أن تدرب بسرعة ومشاركة نماذج الانتشار لتوليد الصور بأنماط جديدة. لتسهيل استخدام ومحاولة استخدام عدة نماذج LoRA، يستخدم الناشرون مكتبة PEFT للمساعدة في إدارة محولات مختلفة للاستنتاج.

على سبيل المثال، قم بتحميل نموذج أساسي ثم قم بتحميل محول [artificialguybr/3DRedmond-V1](https://huggingface.co/artificialguybr/3DRedmond-V1) للاستدلال باستخدام طريقة [`load_lora_weights`](https://huggingface.co/docs/diffusers/v0.24.0/en/api/loaders/lora#diffusers.loaders.LoraLoaderMixin.load_lora_weights). تم تمكين وسيط `adapter_name` في طريقة التحميل بواسطة PEFT ويسمح لك بتعيين اسم للمحول بحيث يكون من السهل الإشارة إليه.

```py
استيراد الشعلة
من الناشرين استيراد DiffusionPipeline

خط الأنابيب = DiffusionPipeline.from_pretrained (
"stabilityai/stable-diffusion-xl-base-1.0"، torch_dtype=torch.float16
).to("cuda")
خط الأنابيب. تحميل_لورا_الأوزان (
"peft-internal-testing/artificialguybr__3DRedmond-V1"،
اسم_الوزن="3DRedmond-3DRenderStyle-3DRenderAF.safetensors"،
محول_الاسم="3d"
)
الصورة = خط الأنابيب ("لفات السوشي على شكل وجوه قطط لطيف"). الصور [0]
الصورة
```

<div class="flex justify-center">
<img src="https://huggingface.co/datasets/ybelkada/documentation-images/resolve/main/test-lora-diffusers.png"/>
</div>

الآن دعونا نجرب نموذج LoRA رائع آخر، [ostris/super-cereal-sdxl-lora](https://huggingface.co/ostris/super-cereal-sdxl-lora). كل ما عليك فعله هو تحميل هذا المحول الجديد وتسميته باستخدام `adapter_name`، واستخدام طريقة [`set_adapters`](https://huggingface.co/docs/diffusers/api/loaders/unet#diffusers.loaders.UNet2DConditionLoadersMixin.set_adapters) لتعيينه كمحول نشط حالي.

```py
خط الأنابيب. تحميل_لورا_الأوزان (
"ostris/super-cereal-sdxl-lora"،
اسم_الوزن="cereal_box_sdxl_v1.safetensors"،
محول_الاسم="حبوب الإفطار"
)
خط الأنابيب. set_adapters ("حبوب الإفطار")
الصورة = خط الأنابيب ("لفات السوشي على شكل وجوه قطط لطيف"). الصور [0]
الصورة
```

<div class="flex justify-center">
<img src="https://huggingface.co/datasets/ybelkada/documentation-images/resolve/main/test-lora-diffusers-2.png"/>
</div>

أخيرًا، يمكنك استدعاء طريقة [`disable_lora`](https://huggingface.co/docs/diffusers/api/loaders/unet#diffusers.loaders.UNet2DConditionLoadersMixin.disable_lora) لاستعادة النموذج الأساسي.

```py
خط الأنابيب. تعطيل_لورا ()
```

تعرف على المزيد حول كيفية دعم PEFT لـ Diffusers في البرنامج التعليمي [الاستدلال باستخدام PEFT](https://huggingface.co/docs/diffusers/tutorials/using_peft_for_inference).

## المحولات

🤗 [المحولات](https://hf.co/docs/transformers) هي مجموعة من النماذج المُدربة مسبقًا لجميع أنواع المهام في جميع الطرائق. يمكنك تحميل هذه النماذج للتدريب أو الاستدلال. العديد من النماذج هي نماذج اللغة الكبيرة (LLMs)، لذلك من المنطقي دمج PEFT مع المحولات لإدارة وتدريب المحولات.

قم بتحميل نموذج أساسي مُدرب مسبقًا للتدريب.

```py
من المحولات استيراد AutoModelForCausalLM

النموذج = AutoModelForCausalLM.from_pretrained ("facebook/opt-350m")
```

بعد ذلك، أضف تكوين المحول لتحديد كيفية تكييف معلمات النموذج. استدعاء طريقة [`~PeftModel.add_adapter`] لإضافة التكوين إلى النموذج الأساسي.

```py
من peft استيراد LoraConfig

peft_config = LoraConfig (
لورا_ألفا = 16،
لورا_التخلي = 0.1،
ص = 64،
التحيز = "لا شيء"،
نوع_المهمة = "CAUSAL_LM"
)
النموذج. add_adapter (peft_config)
```

الآن يمكنك تدريب النموذج باستخدام فئة [`~transformers.Trainer`] من المحول أو أي إطار تدريب تفضله.

لاستخدام النموذج المدرب حديثًا للاستدلال، تستخدم فئة [`~transformers.AutoModel`] PEFT في الخلفية لتحميل أوزان المحول وملف التكوين في نموذج أساسي مُدرب مسبقًا.

```py
من المحولات استيراد AutoModelForCausalLM

النموذج = AutoModelForCausalLM.from_pretrained ("peft-internal-testing/opt-350m-lora")
```

بدلاً من ذلك، يمكنك استخدام أنابيب المحول [Pipelines](https://huggingface.co/docs/transformers/en/main_classes/pipelines) لتحميل النموذج لتشغيل الاستدلال بشكل مناسب:

```py
من المحولات استيراد الأنابيب

النموذج = الأنابيب ("توليد النص"، "peft-internal-testing/opt-350m-lora")
طباعة (النموذج ("مرحبا بالعالم"))
```

إذا كنت مهتمًا بمقارنة أو استخدام أكثر من محول واحد، فيمكنك استدعاء طريقة [`~PeftModel.add_adapter`] لإضافة تكوين المحول إلى النموذج الأساسي. الشرط الوحيد هو أن نوع المحول يجب أن يكون هو نفسه (لا يمكنك خلط محول LoRA و LoHa).

```py
من المحولات استيراد AutoModelForCausalLM
من peft استيراد LoraConfig

النموذج = AutoModelForCausalLM.from_pretrained ("facebook/opt-350m")
النموذج. add_adapter (lora_config_1، محول_الاسم="المحول_1")
```

استدعاء [`~PeftModel.add_adapter`] مرة أخرى لربط محول جديد بالنموذج الأساسي.

```py
النموذج. add_adapter (lora_config_2، محول_الاسم="المحول_2")
```

بعد ذلك، يمكنك استخدام [`~PeftModel.set_adapter`] لتعيين المحول النشط حاليًا.

```py
النموذج. set_adapter ("المحول_1")
الإخراج = النموذج. توليد (** المدخلات)
طباعة (فك تشفير المحلل اللغوي (الإخراج_معطل [0]، تخطي_الرموز الخاصة=صحيح))
```

لإلغاء تنشيط المحول، اتصل بطريقة [disable_adapters](https://github.com/huggingface/transformers/blob/4e3490f79b40248c53ee54365a9662611e880892/src/transformers/integrations/peft.py#L313).

```py
النموذج. تعطيل_المحولات ()
```

يمكن استخدام [enable_adapters](https://github.com/huggingface/transformers/blob/4e3490f79b40248c53ee54365a9662611e880892/src/transformers/integrations/peft.py#L336) لتمكين المحولات مرة أخرى.

إذا كنت فضوليًا، فتحقق من البرنامج التعليمي [تحميل وتدريب المحولات باستخدام PEFT](https://huggingface.co/docs/transformers/main/peft) لمعرفة المزيد.