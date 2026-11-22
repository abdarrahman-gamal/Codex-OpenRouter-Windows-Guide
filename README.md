# تشغيل نماذج OpenRouter المجانية داخل Codex Desktop على Windows

هذا الدليل يشرح إعداد **OpenRouter كموفّر نماذج مخصص داخل Codex Desktop GUI**، واستخدام واجهة **OpenAI Responses API**، وإظهار عدة نماذج مجانية في قائمة اختيار الموديل باستخدام ملف Model Catalog محلي.

> **نطاق الدليل:** الإعداد مبني على تجربة Windows مع Codex Desktop الذي يحتوي على `codex.exe` مضمّن. قد تختلف أسماء الإعدادات أو سلوك قائمة الموديلات بين إصدارات Codex. اختبر الاتصال في إصدارك قبل الاعتماد عليه.

## ما الذي سنحصل عليه؟

- تشغيل Codex Desktop عبر OpenRouter بدل تسجيل الدخول إلى OpenAI لاستخدام الموديل.
- استخدام `https://openrouter.ai/api/v1/responses`، وليس `/chat/completions`.
- وضع API key مباشرة في `config.toml`، كما في هذا المثال، من دون OAuth.
- تعريف `openrouter/free` ومجموعة من النماذج المجانية المحددة في قائمة Codex.
- الاحتفاظ بإعدادات Marketplace وPlugins وMCP الموجودة لديك دون الحاجة إلى استبدالها.

## متطلبات قبل البدء

1. Windows 10 أو 11.
2. تثبيت Codex Desktop.
3. حساب ومفتاح API من [OpenRouter](https://openrouter.ai/settings/keys).
4. اتصال إنترنت.
5. يفضّل وجود Python Launcher (`py`) لاختبارات اختيارية لقراءة قاعدة بيانات Codex.

### افهم الفرق بين OAuth وAPI key

- **OAuth**: تسجيل الدخول إلى ChatGPT/OpenAI عبر المتصفح. هذا الدليل لا يعتمد عليه لمصادقة موفّر OpenRouter.
- **API key**: مفتاح OpenRouter الذي يعرّف حسابك ويُرسل مع الطلبات. ما زلت تحتاج إليه؛ عدم استخدام OAuth لا يعني أن الـ API مفتوح بلا مصادقة.
- `requires_openai_auth = false` يمنع مطالبة Codex بمصادقة OpenAI لهذا الموفّر، لكنه لا يلغي مصادقة OpenRouter.

## 1. أنشئ مفتاح OpenRouter

1. افتح [OpenRouter API Keys](https://openrouter.ai/settings/keys).
2. أنشئ مفتاحًا جديدًا.
3. لا ترسل المفتاح لأحد، ولا تضعه في GitHub أو لقطات الشاشة أو ملفات README.

**ملاحظة أمان:** المثال التالي يضع المفتاح داخل `config.toml` لأن هذا هو الأسلوب المباشر المطلوب هنا. Codex يصف `experimental_bearer_token` بأنه خيار تجريبي، ويوصي باستخدام `env_key` بدلًا منه لأسباب أمنية. إذا كان الجهاز مشتركًا أو المجلد متزامنًا أو المشروع عامًا، استخدم متغير بيئة بدل تخزين المفتاح في الملف.

## 2. افتح ملف إعدادات Codex

المسار الافتراضي:

```text
%USERPROFILE%\.codex\config.toml
```

يمكن فتحه من Codex Desktop عبر **Settings → Configuration → Open config.toml** إذا كان الخيار متاحًا في إصدارك.

إذا كان لديك `CODEX_HOME` مضبوطًا، فسيستخدم Codex مجلد الإعدادات المحدد به بدل `%USERPROFILE%\.codex`.

**قبل التعديل:** انسخ `config.toml` إلى ملف احتياطي مثل `config.toml.bak`.

## 3. أضف إعداد OpenRouter إلى config.toml

ضع الإعدادات العامة في بداية الملف، قبل أول قسم TOML يبدأ بأقواس مربعة. لا تضع `model` أو `model_provider` أو `model_catalog_json` داخل `[model_providers.openrouter]`؛ وإلا ستصبح مفاتيح تابعة لذلك القسم بدل الإعدادات العامة.

```toml
# اختيار الموديل الافتراضي
model = "openrouter/free"
model_provider = "openrouter"
model_reasoning_effort = "low"

# ملف قائمة الموديلات المحلية
model_catalog_json = 'C:\Users\YOUR_WINDOWS_USER\.codex\openrouter-models.json'

[model_providers.openrouter]
name = "OpenRouter"
base_url = "https://openrouter.ai/api/v1"
wire_api = "responses"

# ضع مفتاح OpenRouter الجديد هنا. لا تشاركه أو ترفعه إلى Git.
experimental_bearer_token = "sk-or-v1-REPLACE-WITH-YOUR-KEY"

# لا تطلب تسجيل الدخول عبر OpenAI لهذا الموفّر.
requires_openai_auth = false
```

استبدل `YOUR_WINDOWS_USER` باسم مجلد المستخدم الفعلي على Windows، واستبدل قيمة المفتاح بمفتاحك الجديد. مثال المسار للمستخدم `Alex`:

```toml
model_catalog_json = 'C:\Users\Alex\.codex\openrouter-models.json'
```

**إذا كنت تفضّل تخزين المفتاح بأمان أكبر:** احذف سطر `experimental_bearer_token`، وضع `env_key = "OPENROUTER_API_KEY"` داخل قسم الموفّر، ثم أنشئ متغير البيئة `OPENROUTER_API_KEY` في Windows. هذه الطريقة لا تستخدم OAuth أيضًا.

## 4. أنشئ ملف Model Catalog

أنشئ ملفًا جديدًا باسم:

```text
%USERPROFILE%\.codex\openrouter-models.json
```

ضع فيه JSON التالي. يتضمن `openrouter/free` بالإضافة إلى النماذج المحددة التي أُضيفت إلى القائمة في هذه التجربة.

```json
{
  "models": [
    {
      "slug": "openrouter/free",
      "display_name": "OpenRouter Free Router",
      "base_instructions": "Help the user with coding tasks, understanding code, editing files, and testing software.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 0,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "inclusionai/ling-3.2-flash:free",
      "display_name": "InclusionAI - Ling 3.2 Flash (Free)",
      "base_instructions": "Help with coding, reasoning, image understanding, and software development tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 1,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "baidu/ernie-5.5-flash-preview:free",
      "display_name": "Baidu - ERNIE 5.5 Flash Preview (Free)",
      "base_instructions": "Help with coding, programming, reasoning, research, and software development tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 2,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "liquid/lfm-3-1.7b:free",
      "display_name": "LiquidAI - LFM3 1.7B (Free)",
      "base_instructions": "Help with coding, lightweight reasoning, and software development tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 3,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "particle-ai/particle-1:free",
      "display_name": "Particle AI - Particle 1 (Free)",
      "base_instructions": "Help with agentic coding, terminal workflows, codebase changes, debugging, and data engineering.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 4,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "recursive/flint-v1.0:free",
      "display_name": "Recursive - Flint V1.0 (Free)",
      "base_instructions": "Help investigate codebases, fix bugs, implement features, write tests, and perform software engineering tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 5,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "nvidia/nemotron-3.5-super-vl:free",
      "display_name": "NVIDIA - Nemotron 3.5 Super VL (Free)",
      "base_instructions": "Help with coding, visual understanding, document analysis, GUI reasoning, and software development.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 6,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "nvidia/nemotron-3.5-lightning:free",
      "display_name": "NVIDIA - Nemotron 3.5 Lightning (Free)",
      "base_instructions": "Help with coding, reasoning, agentic workflows, debugging, and software engineering tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 7,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "nvidia/nemotron-3-ultra-550b-a55b:free",
      "display_name": "NVIDIA - Nemotron 3 Ultra (Free)",
      "base_instructions": "Help with complex reasoning, planning, coding, research, and multi-step agentic workflows.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 8,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free",
      "display_name": "NVIDIA - Nemotron 3 Nano Omni (Free)",
      "base_instructions": "Help with multimodal understanding, reasoning, coding assistance, and agentic workflows.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 9,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "google/gemma-4-26b-a4b-it:free",
      "display_name": "Google - Gemma 4 26B A4B (Free)",
      "base_instructions": "Help with coding, reasoning, image understanding, and software development tasks.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 10,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    },
    {
      "slug": "google/gemma-4-31b-it:free",
      "display_name": "Google - Gemma 4 31B (Free)",
      "base_instructions": "Help with coding, reasoning, multilingual tasks, and software development.",
      "supported_reasoning_levels": [],
      "shell_type": "shell_command",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 11,
      "support_verbosity": false,
      "truncation_policy": { "mode": "tokens", "limit": 10000 },
      "experimental_supported_tools": []
    }
  ]
}
```

> هذه القائمة تصف أسماء الموديلات التي تظهر في Codex. لا تعني أن كل موديل يدعم كل ميزات Codex أو أن كل موديل متاح دائمًا. افحص صفحة كل موديل واختبر `/responses` وTool Calling قبل الاعتماد عليه.

## 5. تحقق من صحة JSON

افتح PowerShell وشغّل:

```powershell
$path = "$env:USERPROFILE\.codex\openrouter-models.json"

try {
    $catalog = Get-Content $path -Raw | ConvertFrom-Json -ErrorAction Stop
    Write-Host "JSON: VALID" -ForegroundColor Green
    Write-Host "Models: $($catalog.models.Count)"
    $catalog.models | Select-Object slug, display_name | Format-Table
}
catch {
    Write-Host "JSON: INVALID" -ForegroundColor Red
    Write-Host $_
}
```

المتوقع أن تكون النتيجة:

```text
JSON: VALID
Models: 12
```

## 6. اختبر OpenRouter Responses API قبل فتح Codex

هذا الاختبار يرسل طلبًا مباشرًا إلى `/v1/responses` باستخدام `openrouter/free`. سيطلب PowerShell المفتاح دون إظهاره أثناء الكتابة:

```powershell
$secureKey = Read-Host "OpenRouter API key" -AsSecureString
$ptr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($secureKey)

try {
    $key = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($ptr)
}
finally {
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ptr)
}

$headers = @{ Authorization = "Bearer $key" }
$body = @{
    model = "openrouter/free"
    input = "Reply with exactly: OPENROUTER RESPONSES OK"
} | ConvertTo-Json -Depth 10

$response = Invoke-RestMethod `
    -Uri "https://openrouter.ai/api/v1/responses" `
    -Method POST `
    -Headers $headers `
    -ContentType "application/json" `
    -Body $body

$response | Select-Object object, status, model

Remove-Variable key, secureKey, headers, body -ErrorAction SilentlyContinue
```

النجاح يعني أن المفتاح والاتصال الأساسي وResponses endpoint يعملون. لا يثبت ذلك وحده أن Codex GUI أو أدوات Codex تعمل مع كل موديل.

## 7. شغّل Codex Desktop GUI

1. احفظ `config.toml` و`openrouter-models.json`.
2. أغلق Codex Desktop بالكامل، بما في ذلك عملياته في Task Manager عند الحاجة.
3. افتح Codex Desktop مجددًا.
4. ابدأ محادثة جديدة؛ لا تعتمد على محادثة قديمة قد تكون حفظت موديلًا آخر.
5. افتح Model Picker وابحث عن `OpenRouter Free Router` وبقية الأسماء المعرفة في JSON.
6. اختر موديلًا واحدًا وجرّب طلبًا نصيًا بسيطًا.
7. افحص [OpenRouter Activity](https://openrouter.ai/activity) للتأكد من اسم الموديل الذي استقبل الطلب.

## 8. اختبر عمل أدوات Codex

ظهور الموديل في القائمة والنجاح في إرسال رسالة نصية لا يثبتان أن Tool Calling متوافق بالكامل. جرّب في مشروع اختباري غير حساس:

```text
Create a file named codex-openrouter-test.txt containing:
Codex OpenRouter tool test passed.
Then read the file and show its contents.
```

تحقق من أن Codex نفّذ عملية إنشاء الملف وقراءته بالفعل. كرر الاختبار لكل موديل تريد الاعتماد عليه؛ لا تفترض أن النتائج واحدة لجميع النماذج.

## 9. تحقق من الموديل الذي اختارته GUI

في هذا الإعداد، يمكن فحص قاعدة بيانات جلسات Codex محليًا للقراءة فقط، إذا كانت موجودة:

```text
%USERPROFILE%\.codex\state_5.sqlite
```

مع Python، نفّذ:

```powershell
py -c "import sqlite3,pathlib; p=pathlib.Path.home()/'.codex'/'state_5.sqlite'; c=sqlite3.connect('file:'+str(p)+'?mode=ro',uri=True); print(c.execute('SELECT title, source, model_provider, model FROM threads ORDER BY COALESCE(created_at_ms, created_at*1000) DESC LIMIT 10').fetchall()); c.close()"
```

ابحث عن آخر محادثة GUI وتأكد من:

- `model_provider` يساوي `openrouter`.
- `model` يساوي الـ `slug` الذي اخترته.

قد تختلف قيمة `source` بين إصدارات Codex؛ لا تعتمد عليها وحدها لتحديد الواجهة.

## 10. فهم OpenRouter Free Router وحدود المجاني

`openrouter/free` ليس موديلًا ثابتًا؛ إنه Router يختار موديلًا مجانيًا متاحًا يناسب خصائص الطلب. إذا أردت التحكم في الموديل المحدد، اختر أحد المعرّفات المنتهية بـ `:free` بدل Router العام.

حدود OpenRouter المجانية قابلة للتغيير. توضح صفحة Free Models الحالية أن نماذج `:free` تخضع لقيود طلبات يومية ودقيقة، وأن الحد اليومي قد يرتفع بعد شراء رصيد. راجع صفحة الأسعار والحدود قبل الاعتماد على الأرقام في هذا الدليل:

- [OpenRouter Free Models](https://openrouter.ai/collections/free-models)
- [OpenRouter Free Router](https://openrouter.ai/openrouter/free)
- [OpenRouter API Keys](https://openrouter.ai/settings/keys)
- [OpenRouter Activity](https://openrouter.ai/activity)

النماذج المجانية قد تكون مزدحمة أو غير متاحة مؤقتًا، ويمكن أن تتغير قائمة النماذج المجانية ومعرّفاتها وسياسات استخدامها.

## 11. حل المشكلات الشائعة

### الموديل لا يظهر في GUI

- تأكد من وجود ملف JSON في المسار نفسه المكتوب في `model_catalog_json`.
- تأكد من صحة JSON وأن كل `slug` فريد.
- تأكد أن `model_catalog_json` في المستوى الأعلى قبل أول TOML table.
- أغلق Codex Desktop بالكامل ثم افتحه من جديد؛ الكتالوج المحلي يُقرأ عند بدء التشغيل.
- تأكد أنك تبدأ محادثة جديدة وتختار الموديل من Model Picker.
- بعض إصدارات Codex أو إعدادات المصادقة قد تعرض سلوكًا مختلفًا لقائمة النماذج المخصصة. افحص سجل الطلبات بدل الاعتماد على الاسم الظاهر وحده.

### يظهر موديل GPT بدل موديل OpenRouter

- راجع `model` و`model_provider` في أول `config.toml`.
- تأكد أن لا توجد إعدادات مشروع أو Profile أو إعدادات أعلى أولوية تغيّر الاختيار.
- أنشئ محادثة جديدة؛ قد تحفظ المحادثات القديمة الموديل الذي بدأت به.
- افحص عمودي `model_provider` و`model` في قاعدة بيانات الجلسات، ثم راجع OpenRouter Activity.

### خطأ 401 Unauthorized

- المفتاح غير صحيح أو أُلغي.
- تأكد أن `experimental_bearer_token` يحتوي مفتاح OpenRouter وليس مفتاح مزود آخر.
- لا تضع `Bearer ` داخل قيمة المفتاح؛ Codex يضيفها عند إرسال الطلب.

### خطأ 402 Payment Required

- افحص اسم الموديل الذي وصل إلى OpenRouter Activity. قد تكون GUI أرسلت موديلًا مدفوعًا بدل موديل `:free`.
- راجع الرصيد وحدود المفتاح وحالة الحساب.
- لا تفترض أن كل موديل سعره ظاهر `$0` يستخدم نفس شروط حد الاستخدام؛ راجع صفحة OpenRouter الخاصة به.
- لا تشترِ رصيدًا قبل التأكد من الموديل الذي أُرسل فعليًا.

### الطلب النصي يعمل لكن إنشاء الملفات أو الأدوات يفشل

- نجاح `/responses` لا يثبت دعم كل وظائف Codex.
- تحقق من دعم النموذج لـ Function/Tool Calling.
- نفّذ اختبار إنشاء ملف في مجلد تجريبي.
- اختبر موديلًا آخر؛ دعم الأدوات وسلوكها يختلفان من موديل لآخر.

## 12. ملاحظات مهمة حول Model Catalog

- `model_catalog_json` يحمّل كتالوجًا محليًا متوافقًا مع Codex. وفق وثائق Codex، الكتالوج المحلي **يستبدل الكتالوج المضمّن** بدل أن يضيف إليه؛ قد تختفي موديلات Codex الافتراضية من القائمة أثناء تفعيله.
- إذا أردت موديلات إضافية مستقبلًا، أضف عناصر جديدة داخل مصفوفة `models`، مع `slug` فريد وصحيح.
- `supported_in_api: true` و`visibility: "list"` هما بيانات وصفية للكتالوج؛ لا يختبران مزود الخدمة ولا يضمنان أن الموديل يدعم Responses أو أدوات Codex.
- `experimental_supported_tools: []` لا يعلن عن أدوات تجريبية إضافية لهذا الموديل. دعم أدوات Codex الفعلي يجب اختباره مع الموديل والمزوّد.
- لا تنسخ مسارات Windows الخاصة بمستخدم آخر أو إعداداته المحلية مثل `notify` و`node_repl` وMarketplace؛ هذه قد تحتوي مسارات خاصة بتثبيت Windows أو إصدار Codex عند ذلك المستخدم. احتفظ بإعداداتك المحلية وأضف فقط إعدادات الموفّر والكتالوج.

## 13. روابط مرجعية

- [Codex Advanced Configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)
- [Codex: Roll out a gateway and local model catalog](https://learn.chatgpt.com/docs/enterprise/roll-out-a-gateway)
- [Codex model catalog source](https://github.com/openai/codex/blob/main/codex-rs/models-manager/models.json)
- [OpenRouter Responses API / Free Router](https://openrouter.ai/openrouter/free)
- [OpenRouter Free Models collection and limits](https://openrouter.ai/collections/free-models)
- [OpenRouter API Keys](https://openrouter.ai/settings/keys)

---

## تنبيه أخير

هذا الدليل يصف إعدادًا تجريبيًا باستخدام مزوّد خارجي ونماذج مجانية. لا تعتبر ظهور الموديل في القائمة أو نجاح رد نصي ضمانًا لتوافقه الكامل مع Codex Agent، ولا ترسل أسرارًا أو شيفرات خاصة إلى مزود لا تثق بسياسة الاحتفاظ بالبيانات لديه. راجع سياسات الخصوصية لكل نموذج ومزوّد قبل استخدامه.
