# پیکربندی projectها {#configuring-projects}

## نیازمندی Python version {#python-version-requirement}

projectها می‌توانند Python versionهای پشتیبانی‌شده توسط project را در فیلد `project.requires-python`
فایل `pyproject.toml` اعلام کنند.

توصیه می‌شود یک مقدار `requires-python` تنظیم کنید:

```toml title="pyproject.toml" hl_lines="4"
[project]
name = "example"
version = "0.1.0"
requires-python = ">=3.12"
```

نیازمندی Python version، نحو (syntax) Python مجاز در project را تعیین می‌کند و
بر انتخاب نسخه‌های dependency تأثیر می‌گذارد (آن‌ها باید از همان بازهٔ Python version پشتیبانی کنند).

## entry pointها {#entry-points}

[entry pointها](https://packaging.python.org/en/latest/specifications/entry-points/#entry-points) عبارت رسمی برای این است که یک package نصب‌شده، رابط‌ها را معرفی کند. این موارد شامل موارد زیر است:

- [رابط‌های خط فرمان](#command-line-interfaces)
- [رابط‌های کاربری گرافیکی](#graphical-user-interfaces)
- [entry pointهای پلاگین](#plugin-entry-points)

!!! important

    استفاده از جدول‌های entry point به تعریف [build system](#build-systems) نیاز دارد.

### رابط‌های خط فرمان {#command-line-interfaces}

projectها می‌توانند رابط‌های خط فرمان (CLI) را برای project در جدول `[project.scripts]`
فایل `pyproject.toml` تعریف کنند.

برای مثال، برای اعلام دستوری به نام `hello` که تابع `hello` را در module مربوط به `example` فراخوانی می‌کند:

```toml title="pyproject.toml"
[project.scripts]
hello = "example:hello"
```

سپس، می‌توان دستور را از کنسول اجرا کرد:

```console
$ uv run hello
```

### رابط‌های کاربری گرافیکی {#graphical-user-interfaces}

projectها می‌توانند رابط‌های کاربری گرافیکی (GUI) را برای project در جدول `[project.gui-scripts]`
فایل `pyproject.toml` تعریف کنند.

!!! important

    این‌ها فقط در Windows با [رابط‌های خط فرمان](#command-line-interfaces) متفاوت‌اند، جایی که
    با یک فایل اجرایی GUI پیچیده می‌شوند تا بتوان آن‌ها را بدون کنسول اجرا کرد. در سکوهای دیگر،
    رفتار یکسانی دارند.

برای مثال، برای اعلام دستوری به نام `hello` که تابع `app` را در module مربوط به `example` فراخوانی می‌کند:

```toml title="pyproject.toml"
[project.gui-scripts]
hello = "example:app"
```

### entry pointهای پلاگین {#plugin-entry-points}

projectها می‌توانند entry pointها را برای کشف پلاگین در جدول
[`[project.entry-points]`](https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/#using-package-metadata)
فایل `pyproject.toml` تعریف کنند.

برای مثال، برای ثبت package مربوط به `example-plugin-a` به‌عنوان پلاگینی برای `example`:

```toml title="pyproject.toml"
[project.entry-points.'example.plugins']
a = "example_plugin_a"
```

سپس، در `example`، پلاگین‌ها به این صورت بارگذاری می‌شدند:

```python title="example/__init__.py"
from importlib.metadata import entry_points

for plugin in entry_points(group="example.plugins"):
    plugin.load()
```

!!! note

    کلید `group` می‌تواند مقدار دلخواهی باشد؛ لازم نیست نام package یا
    «plugins» را شامل شود. با این حال، توصیه می‌شود کلید را با نام package فضای نام (namespace) بدهید تا از
    تداخل با packageهای دیگر جلوگیری شود.

## build systemها {#build-systems}

build system تعیین می‌کند که project چگونه باید بسته‌بندی و نصب شود. projectها می‌توانند یک build system را در جدول `[build-system]` فایل `pyproject.toml` اعلام و پیکربندی کنند.

uv از وجود build system استفاده می‌کند تا تعیین کند آیا project شامل packageای است که باید در virtual environment مربوط به project نصب شود یا نه. اگر build system تعریف نشده باشد، uv تلاش نمی‌کند خود project را build یا نصب کند، فقط dependencyهای آن را نصب می‌کند. اگر build system تعریف شده باشد، uv، project را build و در project environment نصب می‌کند.

به‌طور پیش‌فرض، `uv init` یک project بسته‌بندی‌شده را با استفاده از [uv build backend](../build-backend.md) می‌سازد.
گزینهٔ `--build-backend` را می‌توان برای انتخاب یک build backend جایگزین ارائه داد و
`--no-package` را می‌توان برای ساخت یک project تخت و بدون بسته‌بندی ارائه داد.

!!! note

    در حالی که uv بدون تعریف build system، project جاری را build و نصب نمی‌کند،
    وجود جدول `[build-system]` در packageهای دیگر الزامی نیست. به دلایل قدیمی،
    اگر build system تعریف نشده باشد، از `setuptools.build_meta:__legacy__` برای build کردن
    package استفاده می‌شود. packageهایی که به آن‌ها وابسته‌اید ممکن است build system خود را به‌صورت صریح اعلام نکرده باشند، اما همچنان
    قابل‌نصب‌اند. به همین ترتیب، اگر [dependencyای به project محلی اضافه کنید](./dependencies.md#path)
    یا آن را با `uv pip` نصب کنید، uv تلاش می‌کند آن را build و نصب کند، صرف‌نظر از وجود
    جدول `[build-system]`.

build systemها برای پشتیبانی از قابلیت‌های زیر استفاده می‌شوند:

- شامل‌کردن یا مستثناکردن فایل‌ها از distributionها
- رفتار نصب editable
- metadata پویای project
- کامپایل کد بومی (native)
- vendoring کردن کتابخانه‌های مشترک

برای پیکربندی این قابلیت‌ها، به مستندات build system انتخابی خود مراجعه کنید.

## بسته‌بندی project {#project-packaging}

همان‌طور که در [build systemها](#build-systems) بحث شد، یک project Python باید build شود تا نصب شود.
این فرایند معمولاً «بسته‌بندی» (packaging) نامیده می‌شود.

احتمالاً به package نیاز دارید اگر می‌خواهید:

- دستورهایی به project اضافه کنید
- project را بین دیگران توزیع کنید
- از چیدمان `src` و `test` استفاده کنید
- کتابخانه بنویسید

احتمالاً به package نیاز _ندارید_ اگر:

- script می‌نویسید
- یک برنامهٔ ساده می‌سازید
- از چیدمان تخت استفاده می‌کنید

در حالی که uv معمولاً از اعلام [build system](#build-systems) استفاده می‌کند تا تعیین کند آیا project
باید بسته‌بندی شود یا نه، uv همچنین اجازه می‌دهد این رفتار با تنظیم
[`tool.uv.package`](../../reference/settings.md#package) بازنویسی شود.

تنظیم `tool.uv.package = true`، project را مجبور می‌کند که build و در project
environment نصب شود. اگر build system تعریف نشده باشد، uv از backend قدیمی setuptools استفاده می‌کند.

تنظیم `tool.uv.package = false`، مجبور می‌کند که package مربوط به project _ساخته و نصب_ نشود و در
project environment قرار نگیرد. uv هنگام تعامل با project، build system اعلام‌شده را نادیده می‌گیرد؛
با این حال، uv همچنان به تلاش‌های صریح برای build کردن project، مانند فراخوانی `uv build`، احترام می‌گذارد.

## مسیر project environment {#project-environment-path}

می‌توان از environment variable مربوط به `UV_PROJECT_ENVIRONMENT` برای پیکربندی مسیر virtual environment مربوط به project استفاده کرد (به‌طور پیش‌فرض `.venv`).

اگر مسیر نسبی ارائه شود، نسبت به ریشهٔ workspace حل می‌شود. اگر مسیر مطلق ارائه شود، همان‌طور که هست استفاده می‌شود، یعنی پوشهٔ فرزندی برای environment ساخته نمی‌شود. اگر environmentای در مسیر ارائه‌شده وجود نداشته باشد، uv آن را می‌سازد.

می‌توان از این گزینه برای نوشتن در environment سیستم Python استفاده کرد، هرچند توصیه نمی‌شود.
`uv sync` به‌طور پیش‌فرض packageهای اضافی را از environment حذف می‌کند و به همین دلیل، ممکن است
سیستم را در وضعیت خراب رها کند.

برای هدف‌گرفتن environment سیستم، `UV_PROJECT_ENVIRONMENT` را روی پیشوند (prefix) نصب Python تنظیم کنید. برای مثال، در سیستم‌های مبتنی بر Debian، این معمولاً `/usr/local` است:

```console
$ python -c "import sysconfig; print(sysconfig.get_config_var('prefix'))"
/usr/local
```

برای هدف‌گرفتن این environment، باید `UV_PROJECT_ENVIRONMENT=/usr/local` را export کنید.

!!! important

    اگر مسیر مطلق ارائه شود و این تنظیم در چند project استفاده شود،
    environment با فراخوانی‌ها در هر project بازنویسی می‌شود. این تنظیم فقط برای استفاده
    برای یک project در CI یا imageهای Docker توصیه می‌شود.

!!! note

    به‌طور پیش‌فرض، uv در طول عملیات‌های project، environment variable مربوط به `VIRTUAL_ENV` را نمی‌خواند.
    اگر `VIRTUAL_ENV` روی مسیری متفاوت از environment مربوط به project تنظیم شده باشد، هشداری نمایش داده می‌شود. می‌توان از فلگ `--active` برای انتخابِ رعایت‌کردن `VIRTUAL_ENV` استفاده کرد. می‌توان از فلگ
    `--no-active` برای بی‌صدا کردن هشدار استفاده کرد.

## ایزوله‌سازی build {#build-isolation}

به‌طور پیش‌فرض، uv همهٔ packageها را در virtual environmentهای ایزوله در کنار dependencyهای build اعلام‌شدهٔ آن‌ها build می‌کند، مطابق [PEP 517](https://peps.python.org/pep-0517/).

برخی packageها با این رویکرد به ایزوله‌سازی build ناسازگارند، چه عمدی و چه غیرعمدی.

برای مثال، packageهایی مانند [`flash-attn`](https://pypi.org/project/flash-attn/) و
[`deepspeed`](https://pypi.org/project/deepspeed/) باید در برابر همان نسخهٔ PyTorch build شوند
که در project environment نصب شده است؛ با build کردن آن‌ها در environment ایزوله، ممکن است
به‌طور ناخواسته در برابر نسخهٔ متفاوتی از PyTorch ساخته شوند و به خطاهای زمان اجرا منجر شوند.

در موارد دیگر، packageها ممکن است به‌طور تصادفی dependencyهای لازم را در فهرست dependencyهای build اعلام‌شدهٔ خود حذف کنند. برای مثال، [`cchardet`](https://pypi.org/project/cchardet/) نیاز دارد که `cython` پیش از نصب `cchardet` در project environment نصب شده باشد، اما آن را به‌عنوان dependency مربوط به build اعلام نمی‌کند.

برای رفع این مشکلات، uv از دو رویکرد جداگانه برای تغییر رفتار ایزوله‌سازی build پشتیبانی می‌کند:

1. **افزودن dependencyهای build**: این کار به شما اجازه می‌دهد package را در یک
   environment ایزوله نصب کنید، اما با dependencyهای build اضافی که خود package
   از طریق تنظیم [`extra-build-dependencies`](../../reference/settings.md#extra-build-dependencies)
   اعلام نکرده است. برای packageهایی مانند `flash-attn`، حتی می‌توانید اجبار کنید که آن dependencyهای build (مانند
   `torch`) با نسخهٔ packageای که در project environment نصب شده است یا نصب خواهد شد، مطابقت داشته باشند.

1. **غیرفعال‌کردن ایزوله‌سازی build برای packageهای مشخص**: این کار به شما اجازه می‌دهد package را بدون
   build کردن آن در environment ایزوله نصب کنید.

در صورت امکان، توصیه می‌کنیم dependencyهای build را افزوده کنید تا اینکه ایزوله‌سازی build را به‌طور کامل غیرفعال کنید،
زیرا رویکرد دوم نیاز دارد که dependencyهای build پیش از نصب خود package در project environment نصب شده باشند، که می‌تواند به مراحل نصب پیچیده‌تر، شامل‌شدن packageهای اضافی در project environment و دشواری در
بازتولید project environment در زمینه‌های دیگر منجر شود.

### افزودن dependencyهای build {#augmenting-build-dependencies}

برای افزودن فهرست dependencyهای build برای یک package مشخص، آن را به فهرست
[`extra-build-dependencies`](../../reference/settings.md#extra-build-dependencies) در فایل
`pyproject.toml` خود اضافه کنید.

برای مثال، برای build کردن `cchardet` با `cython` به‌عنوان dependency اضافی build، موارد زیر را در فایل `pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["cchardet"]

[tool.uv.extra-build-dependencies]
cchardet = ["cython"]
```

برای اطمینان از اینکه dependency مربوط به build با نسخهٔ packageای که در project environment نصب شده است یا نصب خواهد شد مطابقت دارد، `match-runtime = true` را در جدول `extra-build-dependencies` تنظیم کنید. برای
مثال، برای build کردن `deepspeed` با `torch` به‌عنوان dependency اضافی build، موارد زیر را در فایل `pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["deepspeed", "torch"]

[tool.uv.extra-build-dependencies]
deepspeed = [{ requirement = "torch", match-runtime = true }]
```

این کار تضمین می‌کند که `deepspeed` با همان نسخهٔ `torch` ساخته می‌شود که در
project environment نصب شده است.

!!! tip

    wheelهای از پیش ساخته‌شدهٔ `deepspeed` نیز از
    [indexهای GPU مربوط به Astral](../../guides/integration/pytorch.md#installing-gpu-enabled-pytorch-extensions) در دسترس‌اند.

به همین ترتیب، برای build کردن `flash-attn` با `torch` به‌عنوان dependency اضافی build، موارد زیر را در فایل `pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["flash-attn", "torch"]

[tool.uv.extra-build-dependencies]
flash-attn = [{ requirement = "torch", match-runtime = true }]

[tool.uv.extra-build-variables]
flash-attn = { FLASH_ATTENTION_SKIP_CUDA_BUILD = "TRUE" }
```

!!! note

    environment variable مربوط به `FLASH_ATTENTION_SKIP_CUDA_BUILD` به `flash-attn` اجازه می‌دهد که از
    یک wheel از پیش ساخته‌شده حل شود، به‌جای اینکه تلاش کند از source ساخته شود، که
    به دسترسی به ابزار توسعهٔ CUDA نیاز دارد.

    اگر ابزار CUDA در طول resolution در دسترس باشد، توصیه می‌کنیم متغیر
    `FLASH_ATTENTION_SKIP_CUDA_BUILD` را حذف کنید، زیرا تنظیم `FLASH_ATTENTION_SKIP_CUDA_BUILD`
    روی `TRUE` می‌تواند به نصب ناسازگار منجر شود اگر wheel از پیش ساخته‌شدهٔ سازگاری برای
    نسخهٔ هدف PyTorch، نسخهٔ GPU و platform در دسترس نباشد.

!!! tip

    wheelهای از پیش ساخته‌شدهٔ `flash-attn` نیز از
    [indexهای GPU مربوط به Astral](../../guides/integration/pytorch.md#installing-gpu-enabled-pytorch-extensions) در دسترس‌اند.

به همین ترتیب، [`deep_gemm`](https://github.com/deepseek-ai/DeepGEMM) از همان الگو پیروی می‌کند:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["deep_gemm", "torch"]

[tool.uv.sources]
deep_gemm = { git = "https://github.com/deepseek-ai/DeepGEMM" }

[tool.uv.extra-build-dependencies]
deep_gemm = [{ requirement = "torch", match-runtime = true }]
```

!!! tip

    wheelهای از پیش ساخته‌شدهٔ `deep_gemm` نیز از
    [indexهای GPU مربوط به Astral](../../guides/integration/pytorch.md#installing-gpu-enabled-pytorch-extensions) در دسترس‌اند.

استفاده از `extra-build-dependencies` و `extra-build-variables` در cache مربوط به uv ردیابی می‌شود، به‌طوری
که تغییر در این تنظیم‌ها باعث نصب مجدد و بازسازی packageهای تحت‌تأثیر می‌شود. برای
مثال، در مورد `flash-attn`، ارتقای نسخهٔ `torch` استفاده‌شده در project شما
پس از آن باعث بازسازی `flash-attn` با نسخهٔ جدید `torch` می‌شود.

#### metadata پویا {#dynamic-metadata}

استفاده از `match-runtime = true` فقط برای packageهایی مانند `flash-attn` در دسترس است که
metadata ایستا اعلام می‌کنند. اگر metadata ایستا در دسترس نباشد، uv ملزم است package را در طول
فاز resolution مربوط به dependency بسازد؛ به همین دلیل، uv نمی‌تواند نسخهٔ dependency مربوط به build را تعیین کند که
در نهایت در project environment نصب می‌شود.

به عبارت دیگر، اگر `flash-attn` metadata ایستا اعلام نمی‌کرد، uv نمی‌توانست نسخهٔ
`torch` را تعیین کند که در project environment نصب می‌شود، زیرا باید
`flash-attn` را پیش از حل‌کردن نسخهٔ `torch` می‌ساخت.

به‌عنوان مثال مشخص، [`axolotl`](https://pypi.org/project/axolotl/) یک package محبوب است که
به dependencyهای build افزوده‌شده نیاز دارد، اما metadata ایستا اعلام نمی‌کند، زیرا dependencyهای package
بر اساس نسخهٔ `torch` نصب‌شده در project environment متفاوت‌اند. در
این مورد، کاربران باید به‌جای آن نسخهٔ دقیق `torch` را که قصد دارند در
project خود استفاده کنند مشخص کنند و سپس dependencyهای build را با آن نسخه افزوده کنند.

برای مثال، برای build کردن `axolotl` در برابر `torch==2.6.0`، موارد زیر را در فایل
`pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["axolotl[deepspeed, flash-attn]", "torch==2.6.0"]

[tool.uv.extra-build-dependencies]
axolotl = ["torch==2.6.0"]
deepspeed = ["torch==2.6.0"]
flash-attn = ["torch==2.6.0"]
```

به همین ترتیب، نسخه‌های قدیمی‌تر `flash-attn` metadata ایستا اعلام نمی‌کردند و بنابراین از
`match-runtime = true` به‌صورت پیش‌فرض پشتیبانی نمی‌کردند. برخلاف `axolotl`، با این حال، `flash-attn` dependencyهای خود را بر اساس ویژگی‌های پویای environment مربوط به build تغییر نمی‌داد. به همین دلیل، کاربران می‌توانستند به‌جای آن
metadata مربوط به `flash-attn` را از پیش از طریق تنظیم
[`dependency-metadata`](../../reference/settings.md#dependency-metadata) ارائه دهند و به این ترتیب از
نیاز به build کردن package در طول فاز resolution مربوط به dependency صرف‌نظر کنند. برای مثال، برای ارائهٔ
metadata مربوط به `flash-attn` از پیش:

```toml title="pyproject.toml"
[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

!!! tip

    برای تعیین metadata مربوط به package برای packageای مانند `flash-attn`، به مخزن Git مناسب بروید،
    یا آن را در [PyPI](https://pypi.org/project/flash-attn) جستجو کنید و distribution مربوط به source مربوط به package را دانلود کنید.
    نیازمندی‌های package معمولاً در فایل `setup.py` یا `setup.cfg` یافت می‌شوند.

    (اگر package شامل یک distribution ساخته‌شده باشد، می‌توانید آن را از حالت فشرده خارج کنید تا فایل `METADATA` را پیدا کنید؛ با این حال، وجود
    یک distribution ساخته‌شده نیاز به ارائهٔ metadata از پیش را از بین می‌برد، زیرا آن metadata از قبل در دسترس
    uv خواهد بود.)

    فیلد `version` در `tool.uv.dependency-metadata` برای dependencyهای مبتنی بر registry اختیاری است
    (وقتی حذف شود، uv فرض می‌کند metadata برای همهٔ نسخه‌های package اعمال می‌شود)،
    اما برای dependencyهای URL مستقیم (مانند dependencyهای Git) _الزامی_ است.

### غیرفعال‌کردن ایزوله‌سازی build {#disabling-build-isolation}

نصب packageها بدون ایزوله‌سازی build نیاز دارد که dependencyهای build مربوط به package پیش از build کردن خود package در project environment نصب شده باشند.

برای مثال، در گذشته، برای نصب `cchardet` بدون ایزوله‌سازی build، ابتدا باید
packageهای `cython` و `setuptools` را در project environment نصب می‌کردید و پس از آن فراخوانی جداگانه‌ای برای نصب `cchardet` بدون ایزوله‌سازی build انجام می‌دادید:

```console
$ uv venv
$ uv pip install cython setuptools
$ uv pip install cchardet --no-build-isolation
```

uv این فرایند را با اجازه‌دادن به شما برای مشخص‌کردن packageهایی که نباید در ایزوله ساخته شوند، ساده می‌کند
از طریق تنظیم `no-build-isolation-package` در فایل `pyproject.toml` شما و فلگ
`--no-build-isolation-package` در خط فرمان. علاوه بر این، وقتی package برای
غیرفعال‌کردن ایزوله‌سازی build علامت‌گذاری می‌شود، uv یک نصب دومرحله‌ای انجام می‌دهد؛ ابتدا packageهایی را نصب می‌کند که
از ایزوله‌سازی build پشتیبانی می‌کنند و پس از آن packageهایی را که پشتیبانی نمی‌کنند. در نتیجه، اگر dependencyهای build مربوط به project به‌عنوان dependencyهای project گنجانده شده باشند، uv به‌طور خودکار آن‌ها را پیش از
نصب packageای که نیاز دارد ایزوله‌سازی build غیرفعال شود، نصب می‌کند.

برای مثال، برای نصب `cchardet` بدون ایزوله‌سازی build، موارد زیر را در فایل
`pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["cchardet", "cython", "setuptools"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

هنگام اجرای `uv sync`، uv ابتدا `cython` و `setuptools` را در project environment نصب می‌کند و
پس از آن `cchardet` را (بدون ایزوله‌سازی build) نصب می‌کند:

```console
$ uv sync --extra build
 + cchardet==2.1.7
 + cython==3.1.3
 + setuptools==80.9.0
```

به همین ترتیب، برای نصب `flash-attn` بدون ایزوله‌سازی build، موارد زیر را در فایل
`pyproject.toml` خود قرار دهید:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["flash-attn", "torch"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]
```

هنگام اجرای `uv sync`، uv ابتدا `torch` را در project environment نصب می‌کند و پس از آن
`flash-attn` را (بدون ایزوله‌سازی build) نصب می‌کند. از آنجا که `torch` هم dependency مربوط به project و هم dependency مربوط به build است، نسخهٔ
`torch` تضمین می‌شود که بین environmentهای build و runtime سازگار باشد.

نقطه‌ضعف رویکرد بالا این است که نیاز دارد dependencyهای build در
project environment نصب شوند، که برای `flash-attn` مناسب است (که به `torch` هم در
زمان build و هم در زمان اجرا نیاز دارد)، اما برای `cchardet` مناسب نیست (که فقط در زمان build به `cython` نیاز دارد).

برای جلوگیری از شامل‌کردن dependencyهای build در project environment، uv از فرایند نصب دومرحله‌ای پشتیبانی می‌کند که به شما اجازه می‌دهد dependencyهای build را از packageهایی که به آن‌ها نیاز دارند جدا کنید.

برای مثال، dependencyهای build برای `cchardet` را می‌توان به group اختیاری `build` محدود کرد، مانند:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["cchardet"]

[project.optional-dependencies]
build = ["setuptools", "cython"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

با توجه به موارد بالا، کاربر ابتدا با group اختیاری `build` همگام‌سازی می‌کند و سپس بدون آن تا
dependencyهای build حذف شوند:

```console
$ uv sync --extra build
 + cchardet==2.1.7
 + cython==3.1.3
 + setuptools==80.9.0
$ uv sync
 - cython==3.1.3
 - setuptools==80.9.0
```

برخی packageها، مانند `cchardet`، فقط در _فاز نصب_ مربوط به
`uv sync` به dependencyهای build نیاز دارند. برخی دیگر حتی فقط برای حل‌کردن dependencyهای project در طول _فاز resolution_ به حضور dependencyهای build خود نیاز دارند.

در چنین مواردی، dependencyهای build را می‌توان پیش از اجرای هر دستور `uv lock` یا `uv sync` نصب کرد،
با استفاده از API سطح پایین‌تر `uv pip`. برای مثال، با توجه به:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["flash-attn"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]
```

می‌توانید دنبالهٔ دستورهای زیر را برای همگام‌سازی `flash-attn` اجرا کنید:

```console
$ uv venv
$ uv pip install torch setuptools
$ uv sync
```

همچنین، کاربران می‌توانند به‌جای آن metadata مربوط به `flash-attn` را از پیش از طریق تنظیم
[`dependency-metadata`](../../reference/settings.md#dependency-metadata) ارائه دهند و به این ترتیب از
نیاز به build کردن package در طول فاز resolution مربوط به dependency صرف‌نظر کنند. برای مثال، برای ارائهٔ
metadata مربوط به `flash-attn` از پیش:

```toml title="pyproject.toml"
[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

## حالت editable {#editable-mode}

به‌طور پیش‌فرض، project در حالت editable نصب می‌شود، به‌طوری که تغییرها در کد منبع بلافاصله در environment منعکس می‌شوند. `uv sync` و `uv run` هر دو فلگ `--no-editable` را می‌پذیرند،
که به uv دستور می‌دهد project را در حالت غیر-editable نصب کند. `--no-editable` برای
موارد استفادهٔ استقرار در نظر گرفته شده است، مانند ساخت container مربوط به Docker، که در آن project باید در
environment مستقرشده گنجانده شود، بدون وابستگی به کد منبع اصلی.

## dependencyهای متعارض {#conflicting-dependencies}

uv همهٔ dependencyهای project را با هم حل می‌کند، از جمله dependencyهای اختیاری («extras») و
groupهای dependency. اگر dependencyهای اعلام‌شده در یک بخش با dependencyهای بخش دیگر سازگار نباشند،
uv در حل‌کردن نیازمندی‌های project با خطا مواجه می‌شود.

uv از اعلام صریح groupهای dependency متعارض پشتیبانی می‌کند. برای مثال، برای اعلام اینکه
groupهای `optional-dependency` مربوط به `extra1` و `extra2` ناسازگارند:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { extra = "extra1" },
      { extra = "extra2" },
    ],
]
```

یا، برای اعلام اینکه groupهای dependency توسعهٔ `group1` و `group2` ناسازگارند:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { group = "group1" },
      { group = "group2" },
    ],
]
```

برای اطلاعات بیشتر، به [مستندات resolution](../resolution.md#conflicting-dependencies) مراجعه کنید.

## environmentهای resolution محدود {#limited-resolution-environments}

اگر project شما از مجموعهٔ محدودتری از platformها یا Python versionها پشتیبانی می‌کند، می‌توانید مجموعهٔ
platformهای حل‌شده را از طریق تنظیم `environments` محدود کنید، که فهرستی از markerهای environment مربوط به PEP 508 را می‌پذیرد. برای مثال، برای محدودکردن lockfile به macOS و Linux و حذف Windows:

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "sys_platform == 'darwin'",
    "sys_platform == 'linux'",
]
```

برای اطلاعات بیشتر، به [مستندات resolution](../resolution.md#limited-resolution-environments) مراجعه کنید.

## environmentهای الزامی {#required-environments}

اگر project شما _باید_ از platform یا Python version مشخصی پشتیبانی کند، می‌توانید آن platform را از طریق تنظیم `required-environments` به‌عنوان الزامی علامت‌گذاری کنید. برای مثال، برای الزامی‌کردن پشتیبانی project از macOS مبتنی بر Intel:

```toml title="pyproject.toml"
[tool.uv]
required-environments = [
    "sys_platform == 'darwin' and platform_machine == 'x86_64'",
]
```

تنظیم `required-environments` فقط برای packageهایی مرتبط است که distribution مربوط به source را منتشر نمی‌کنند (مانند PyTorch)، زیرا چنین packageهایی _فقط_ می‌توانند در environmentهایی نصب شوند که تحت پوشش مجموعهٔ
distributionهای باینری از پیش ساخته‌شده (wheel) منتشرشده توسط آن package قرار دارند.

برای اطلاعات بیشتر، به [مستندات resolution](../resolution.md#required-environments) مراجعه کنید.

برای Linux، از [`minimum-libc-version`](../resolution.md#minimum-libc-version) در کنار
`required-environments` استفاده کنید تا پیاده‌سازی‌های libc و حداقل نسخه‌های مورد پشتیبانی را انتخاب کنید.
