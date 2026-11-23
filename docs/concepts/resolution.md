# Resolution {#resolution}

Resolution فرایند تبدیل فهرستی از نیازمندی‌ها به فهرستی از نسخه‌های package است که آن نیازمندی‌ها را برآورده می‌کنند. Resolution مستلزم جست‌وجوی بازگشتی برای یافتن نسخه‌های سازگار packageهاست، به‌طوری که نیازمندی‌های درخواستی برآورده شوند و نیازمندی‌های packageهای درخواستی با یکدیگر سازگار باشند.

## Dependencies {#dependencies}

بیشتر projectها و packageها dependency دارند. Dependencyها packageهای دیگری هستند که برای کار کردن packageٔ جاری لازم‌اند. یک package، dependencyهای خود را به‌صورت _requirement_ تعریف می‌کند؛ تقریباً ترکیبی از نام یک package و نسخه‌های قابل‌قبول آن. به dependencyهایی که projectٔ جاری تعریف می‌کند، _dependencyهای مستقیم_ گفته می‌شود. به dependencyهایی که هر dependency از projectٔ جاری اضافه می‌کند، _dependencyهای غیرمستقیم_ یا _transitive_ گفته می‌شود.

!!! note

    برای جزئیات بیشتر دربارهٔ dependencyها، به صفحهٔ [dependency specifiers](https://packaging.python.org/en/latest/specifications/dependency-specifiers/) در مستندات Python Packaging مراجعه کنید.

## مثال‌های پایه {#basic-examples}

برای نمایش فرایند resolution، dependencyهای زیر را در نظر بگیرید:

<!-- prettier-ignore -->
- project به `foo` و `bar` وابسته است.
- `foo` یک نسخه دارد، 1.0.0:
    - `foo 1.0.0` به `lib>=1.0.0` وابسته است.
- `bar` یک نسخه دارد، 1.0.0:
    - `bar 1.0.0` به `lib>=2.0.0` وابسته است.
- `lib` دو نسخه دارد، 1.0.0 و 2.0.0. هر دو نسخه dependency ندارند.

در این مثال، resolver باید مجموعه‌ای از نسخه‌های package را پیدا کند که نیازمندی‌های project را برآورده کند. از آنجا که هم `foo` و هم `bar` فقط یک نسخه دارند، از همان نسخه‌ها استفاده می‌شود. resolution باید dependencyهای transitive را هم شامل شود، پس باید نسخه‌ای از `lib` انتخاب شود. `foo 1.0.0` همهٔ نسخه‌های موجود `lib` را مجاز می‌داند، اما `bar 1.0.0` به `lib>=2.0.0` نیاز دارد، پس باید از `lib 2.0.0` استفاده شود.

در برخی resolutionها، ممکن است بیش از یک راه‌حل معتبر وجود داشته باشد. dependencyهای زیر را در نظر بگیرید:

<!-- prettier-ignore -->
- project به `foo` و `bar` وابسته است.
- `foo` دو نسخه دارد، 1.0.0 و 2.0.0:
    - `foo 1.0.0` هیچ dependencyای ندارد.
    - `foo 2.0.0` به `lib==2.0.0` وابسته است.
- `bar` دو نسخه دارد، 1.0.0 و 2.0.0:
    - `bar 1.0.0` هیچ dependencyای ندارد.
    - `bar 2.0.0` به `lib==1.0.0` وابسته است.
- `lib` دو نسخه دارد، 1.0.0 و 2.0.0. هر دو نسخه dependency ندارند.

در این مثال، باید نسخه‌ای از هر دو `foo` و `bar` انتخاب شود؛ اما تعیین اینکه کدام نسخه، نیازمند در نظر گرفتن dependencyهای هر نسخه از `foo` و `bar` است. `foo 2.0.0` و `bar 2.0.0` را نمی‌توان با هم نصب کرد، چون در نسخهٔ موردنیاز `lib` با یکدیگر تعارض دارند؛ پس resolver باید یا `foo 1.0.0` (همراه با `bar 2.0.0`) یا `bar 1.0.0` (همراه با `foo 2.0.0`) را انتخاب کند. هر دو راه‌حل معتبرند و الگوریتم‌های مختلف resolution ممکن است هر یک از آن‌ها را بدهند.

## Platform markers {#platform-markers}

Markerها امکان می‌دهند عبارتی به requirementها ضمیمه شود که نشان می‌دهد dependency چه زمانی باید استفاده شود. برای مثال، `bar ; python_version < "3.9"` نشان می‌دهد که `bar` فقط باید روی Python 3.8 و نسخه‌های قدیمی‌تر نصب شود.

Markerها برای تنظیم dependencyهای یک package بر اساس environment یا platform جاری استفاده می‌شوند. برای مثال، می‌توان از markerها برای تغییر dependencyها بر اساس سیستم‌عامل، معماری CPU، نسخهٔ Python، پیاده‌سازی Python و موارد دیگر استفاده کرد.

!!! note

    برای جزئیات بیشتر دربارهٔ markerها، به بخش [environment markers](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#environment-markers) در مستندات Python Packaging مراجعه کنید.

Markerها برای resolution مهم‌اند، چون مقدار آن‌ها dependencyهای موردنیاز را تغییر می‌دهد. معمولاً resolverهای package در Python از markerهای platform _جاری_ برای تعیین dependencyهای مورداستفاده بهره می‌برند، چون package اغلب روی همان platform جاری _نصب_ می‌شود. اما برای _lock کردن_ dependencyها این روش مشکل‌ساز است — lockfile فقط برای توسعه‌دهندگانی کار می‌کند که از همان platform استفاده می‌کنند که lockfile روی آن ساخته شده است. برای حل این مشکل، resolverهای مستقل از platform یا «سراسری» (universal) وجود دارند.

uv هم از [resolution مخصوص platform](#platform-specific-resolution) و هم از [resolution سراسری](#universal-resolution) پشتیبانی می‌کند.

## resolution مخصوص platform {#platform-specific-resolution}

به‌طور پیش‌فرض، رابط pip در uv، یعنی [`uv pip compile`](../pip/compile.md)، resolutionای تولید می‌کند که مخصوص platform است، مانند `pip-tools`. در رابط project در uv هیچ راهی برای استفاده از resolution مخصوص platform وجود ندارد.

uv همچنین از resolution برای platformها و نسخه‌های Python مشخص و جایگزین، با گزینه‌های `--python-platform` و `--python-version` پشتیبانی می‌کند. برای مثال، اگر از Python 3.12 روی macOS استفاده می‌کنید، می‌توان از `uv pip compile --python-platform linux --python-version 3.10 requirements.in` برای تولید resolutionای برای Python 3.10 روی Linux استفاده کرد. برخلاف resolution سراسری، در resolution مخصوص platform، مقدار `--python-version` دقیقاً همان نسخهٔ Python مورداستفاده است، نه یک کران پایین.

!!! note

    markerهای environment در Python اطلاعات بسیار بیشتری دربارهٔ ماشین جاری در اختیار می‌گذارند تا آنچه بتوان با یک آرگومان سادهٔ `--python-platform` بیان کرد. برای مثال، marker `platform_version` روی macOS شامل زمانی است که کرنل ساخته شده است؛ چیزی که (در تئوری) می‌توان آن را در requirementهای package کدگذاری کرد. resolver در uv با بهترین تلاش، resolutionای تولید می‌کند که با هر ماشینی که روی platform هدفِ `--python-platform` کار می‌کند سازگار باشد؛ این برای بیشتر موارد استفاده کافی است، اما ممکن است برای ترکیب‌های پیچیدهٔ package و platform دقت را از دست بدهد.

## resolution سراسری {#universal-resolution}

lockfile در uv (`uv.lock`) با یک resolution سراسری ساخته می‌شود و در platformهای مختلف قابل‌حمل است. این تضمین می‌کند که dependencyها برای همهٔ کسانی که روی project کار می‌کنند lock شده‌اند، صرف‌نظر از سیستم‌عامل، معماری و نسخهٔ Python. lockfile در uv با دستورهای [project](../concepts/projects/index.md) مانند `uv lock`، `uv sync` و `uv add` ساخته و تغییر داده می‌شود.

resolution سراسری در رابط pip در uv نیز موجود است، یعنی [`uv pip compile`](../pip/compile.md)، با فلگ `--universal`. فایل requirementهای حاصل، markerهایی خواهد داشت که نشان می‌دهند هر dependency مربوط به کدام platform است.

در resolution سراسری، ممکن است یک package چند بار با نسخه‌ها یا URLهای متفاوت فهرست شود، اگر برای platformهای مختلف به نسخه‌های متفاوتی نیاز باشد — markerها تعیین می‌کنند کدام نسخه استفاده خواهد شد. resolution سراسری اغلب محدودتر از resolution مخصوص platform است، چون باید requirementهای مربوط به همهٔ markerها را در نظر گرفت.

در resolution سراسری، همهٔ packageهای موردنیاز باید با _کل_ بازهٔ `requires-python` اعلام‌شده در `pyproject.toml` سازگار باشند. برای مثال، اگر مقدار `requires-python` در یک project برابر `>=3.8` باشد، resolution در صورتی شکست می‌خورد که همهٔ نسخه‌های یک dependency به Python 3.9 یا جدیدتر نیاز داشته باشند، چون آن dependency نسخهٔ قابل‌استفاده‌ای برای (مثلاً) Python 3.8، یعنی کران پایین بازهٔ پشتیبانی‌شدهٔ project، ندارد. به عبارت دیگر، مقدار `requires-python` در project باید زیرمجموعهٔ مقدار `requires-python` در همهٔ dependencyهای آن باشد.

هنگام انتخاب نسخهٔ سازگار برای یک dependency مشخص، uv ([به‌طور پیش‌فرض](#multi-version-resolution)) تلاش می‌کند جدیدترین نسخهٔ سازگار را برای هر نسخهٔ پشتیبانی‌شدهٔ Python انتخاب کند. برای مثال، اگر مقدار `requires-python` در یک project برابر `>=3.8` باشد و جدیدترین نسخهٔ یک dependency به Python 3.9 یا جدیدتر نیاز داشته باشد، در حالی که همهٔ نسخه‌های قبلی از Python 3.8 پشتیبانی می‌کنند، resolver جدیدترین نسخه را برای کاربرانی که Python 3.9 یا جدیدتر اجرا می‌کنند و نسخه‌های قبلی را برای کاربرانی که Python 3.8 اجرا می‌کنند انتخاب می‌کند.

هنگام ارزیابی بازه‌های `requires-python` برای dependencyها، uv فقط کران‌های پایین را در نظر می‌گیرد و کران‌های بالا را کاملاً نادیده می‌گیرد. برای مثال، `>=3.8, <4` به‌صورت `>=3.8` در نظر گرفته می‌شود. رعایت کران‌های بالا روی `requires-python` اغلب به resolutionهایی منجر می‌شود که از نظر صوری درست‌اند اما در عمل نادرست‌اند؛ چون، برای مثال، resolverها به اولین نسخهٔ منتشرشده‌ای بازمی‌گردند که کران بالا را ندارد (نگاه کنید به: [`Requires-Python` upper limits](https://discuss.python.org/t/requires-python-upper-limits/12663)).

## environmentهای resolution محدود {#limited-resolution-environments}

به‌طور پیش‌فرض، resolver سراسری تلاش می‌کند برای همهٔ platformها و نسخه‌های Python حل کند.

اگر project شما فقط از مجموعهٔ محدودی از platformها یا نسخه‌های Python پشتیبانی می‌کند، می‌توانید مجموعهٔ platformهای حل‌شده را با تنظیم `environments` محدود کنید؛ این تنظیم فهرستی از [markerهای environment در PEP 508](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#environment-markers) را می‌پذیرد. به عبارت دیگر، می‌توانید از تنظیم `environments` برای _کاهش_ مجموعهٔ platformهای پشتیبانی‌شده استفاده کنید.

برای مثال، برای محدود کردن lockfile به macOS و Linux و حل نکردن برای Windows:

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "sys_platform == 'darwin'",
    "sys_platform == 'linux'",
]
```

یا، برای حل نکردن برای پیاده‌سازی‌های جایگزین Python:

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "implementation_name == 'cpython'"
]
```

ورودی‌های تنظیم `environments` باید جدا از هم باشند (یعنی نباید هم‌پوشانی داشته باشند). برای مثال، `sys_platform == 'darwin'` و `sys_platform == 'linux'` جدا از هم‌اند، اما `sys_platform == 'darwin'` و `python_version >= '3.9'` جدا نیستند، چون هر دو می‌توانند هم‌زمان درست باشند.

## environmentهای موردنیاز {#required-environments}

در اکوسیستم Python، packageها می‌توانند به‌صورت source distribution، built distribution (wheel) یا هر دو منتشر شوند؛ اما برای نصب یک package، به یک built distribution نیاز است. اگر packageای built distribution نداشته باشد، یا distributionای برای platform یا نسخهٔ Python جاری نداشته باشد (built distributionها اغلب مخصوص platform هستند)، uv تلاش می‌کند package را از source بسازد و سپس distribution ساخته‌شده را نصب کند.

برخی packageها (مانند PyTorch) built distribution منتشر می‌کنند، اما source distribution ندارند. چنین packageهایی _فقط_ روی platformهایی قابل‌نصب‌اند که built distribution برایشان موجود باشد. برای مثال، اگر packageای built distribution برای Linux منتشر کند اما برای macOS یا Windows نه، آن package _فقط_ روی Linux قابل‌نصب خواهد بود.

packageهایی که source distribution ندارند برای resolution سراسری مشکل‌ساز می‌شوند، چون معمولاً حداقل یک platform یا نسخهٔ Python وجود خواهد داشت که package روی آن قابل‌نصب نیست.

به‌طور پیش‌فرض، uv ایجاب می‌کند که هر یک از این packageها حداقل یک wheel سازگار با نسخهٔ هدف Python داشته باشند. می‌توان از تنظیم `required-environments` استفاده کرد تا مطمئن شد resolution حاصل برای platformهای مشخص wheel دارد، یا در صورت نبود چنین wheelهایی شکست بخورد. این تنظیم فهرستی از [markerهای environment در PEP 508](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#environment-markers) را می‌پذیرد.

در حالی که تنظیم `environments` مجموعهٔ environmentهایی را که uv هنگام resolution dependencyها در نظر می‌گیرد _محدود_ می‌کند، `required-environments` مجموعهٔ platformهایی را که uv هنگام resolution dependencyها _باید_ پشتیبانی کند _گسترش_ می‌دهد.

برای مثال، `environments = ["sys_platform == 'darwin'"]` باعث می‌شود uv فقط برای macOS حل کند (و Linux و Windows را نادیده بگیرد). در مقابل، `required-environments = ["sys_platform == 'darwin'"]` _ایجاب می‌کند_ که هر package بدون source distribution، برای قابل‌نصب بودن، یک wheel برای macOS داشته باشد (و اگر چنین wheelای موجود نباشد، شکست می‌خورد).

در عمل، `required-environments` می‌تواند برای اعلام صریح پشتیبانی از platformهای غیر از جدیدترین‌ها مفید باشد، چون این کار اغلب نیازمند بازگشت به نسخه‌های قدیمی‌تر منتشرشدهٔ آن packageهاست. برای مثال، برای تضمین اینکه packageهای فقط-built-distribution از macOS اینتل پشتیبانی کنند:

```toml title="pyproject.toml"
[tool.uv]
required-environments = [
    "sys_platform == 'darwin' and platform_machine == 'x86_64'"
]
```

برای الزام به یک نسخهٔ macOS، از [نسخهٔ کرنل Darwin](<https://en.wikipedia.org/wiki/Darwin_(operating_system)#Darwin_20_onwards>) در `platform_release` استفاده کنید. برای مثال، macOS 15 از Darwin 24 استفاده می‌کند:

```toml title="pyproject.toml"
[tool.uv]
required-environments = [
    "sys_platform == 'darwin' and platform_machine == 'arm64' and platform_release == '24.0.0'"
]
```

برای الزام به پوشش در خط پایه، از `==` استفاده کنید. بازه‌ای مانند `>= '24.0.0'` را می‌توان با wheelای برآورده کرد که فقط از نسخهٔ جدیدتری پشتیبانی می‌کند. wheelهایی که نسخه‌های جدیدتر را هدف می‌گیرند، همچنان در lockfile باقی می‌مانند.

### حداقل نسخهٔ libc {#minimum-libc-version}

!!! note

    `minimum-libc-version` در preview است. برای غیرفعال کردن هشدار، از `--preview-features minimum-libc-version` یا `preview-features = ["minimum-libc-version"]` استفاده کنید.

markerهای environment شامل پیاده‌سازی یا نسخهٔ libc نمی‌شوند. تنظیم `minimum-libc-version` قدیمی‌ترین نسخه‌های libc را مشخص می‌کند که باید در `required-environments` پشتیبانی شوند.

برای مثال، برای الزام به پشتیبانی از glibc 2.31 روی Linux با x86-64 و ARM64:

```toml title="pyproject.toml"
[tool.uv]
preview-features = ["minimum-libc-version"]
required-environments = [
    "sys_platform == 'linux' and platform_machine == 'x86_64'",
    "sys_platform == 'linux' and platform_machine == 'aarch64'",
]
minimum-libc-version = { glibc = "2.31" }
```

با این پیکربندی، یک wheel از نوع `manylinux_2_17` نیازمندی glibc را برآورده می‌کند، اما wheel از نوع `manylinux_2_34` این کار را نمی‌کند. هر دو در lockfile و hashهای exportشده باقی می‌مانند، پس ماشین‌هایی که نسخه‌های جدیدتر glibc دارند می‌توانند wheel جدیدتر را نصب کنند. wheelهای Musl نیز باقی می‌مانند، اما نیازمندی glibc را برآورده نمی‌کنند. حذف یک پیاده‌سازی libc به این معناست که الزامی نیست، نه اینکه حذف شده باشد.

برای الزام به پشتیبانی هم از glibc و هم از musl، برای هر کدام نسخه‌ای تعیین کنید:

```toml
minimum-libc-version = { glibc = "2.31", musl = "1.2" }
```

هر نسخهٔ libc پیکربندی‌شده به پوشش در `required-environments` نیاز دارد. اگر یک environment موردنیاز wheel سازگار یا source distribution قابل‌استفاده‌ای نداشته باشد، uv نسخهٔ دیگری از package را امتحان می‌کند. wheelهای عمومی Linux، مانند `linux_x86_64`، libc را محدود نمی‌کنند و می‌توانند هر یک از پیاده‌سازی‌ها را برآورده کنند.

## مقادیر رایج marker {#common-marker-values}

تنظیم‌های `environments` و `required-environments`، [markerهای environment در PEP 508](https://packaging.python.org/en/latest/specifications/dependency-specifiers/#environment-markers) را می‌پذیرند. مقدار این markerها از runtime در Python گرفته می‌شود (برای مثال، [`sys.platform`](https://docs.python.org/3/library/sys.html#sys.platform)، [`platform.machine()`](https://docs.python.org/3/library/platform.html#platform.machine)، [`platform.system()`](https://docs.python.org/3/library/platform.html#platform.system) و [`os.name`](https://docs.python.org/3/library/os.html#os.name)).

برای مرجع سریع، رایج‌ترین مقدارهای marker بر اساس platform عبارت‌اند از:

| marker | Linux | macOS | Windows |
| --------------------------- | ----------- | ---------- | ----------- |
| `sys_platform` | `'linux'` | `'darwin'` | `'win32'` |
| `platform_system` | `'Linux'` | `'Darwin'` | `'Windows'` |
| `platform_machine` (x86-64) | `'x86_64'` | `'x86_64'` | `'AMD64'` |
| `platform_machine` (ARM64) | `'aarch64'` | `'arm64'` | `'ARM64'` |
| `os_name` | `'posix'` | `'posix'` | `'nt'` |

!!! note

    روی Windows، مقدار `sys_platform` همیشه `'win32'` است، حتی روی سیستم‌های ۶۴ بیتی.

می‌توانید مقدارها را برای platform جاری خود با اجرای این دستور بررسی کنید:

```console
$ uvx python -c "import sysconfig; print(sysconfig.get_config_vars())"
```

## ترجیحات dependency {#dependency-preferences}

اگر فایل خروجی resolution وجود داشته باشد، یعنی یک lockfile در uv (`uv.lock`) یا یک فایل خروجی requirementها (`requirements.txt`)، uv نسخه‌های dependency فهرست‌شده در آن را _ترجیح_ می‌دهد. به همین ترتیب، اگر packageای در یک virtual environment نصب می‌شود، uv نسخهٔ از پیش نصب‌شده را در صورت وجود ترجیح می‌دهد. این یعنی نسخه‌های lockشده یا نصب‌شده تغییر نمی‌کنند، مگر اینکه نسخهٔ ناسازگاری درخواست شود یا ارتقا به‌صورت صریح با `--upgrade` درخواست شود.

## استراتژی resolution {#resolution-strategy}

به‌طور پیش‌فرض، uv تلاش می‌کند از جدیدترین نسخهٔ هر package استفاده کند. برای مثال، `uv pip install flask>=2.0.0` جدیدترین نسخهٔ Flask، مثلاً 3.0.0، را نصب می‌کند. اگر `flask>=2.0.0` یک dependency در project باشد، فقط از `flask` نسخهٔ 3.0.0 استفاده می‌شود. این موضوع مهم است، چون برای مثال، اجرای تست‌ها بررسی نمی‌کند که project واقعاً با کران پایین اعلام‌شدهٔ خود، یعنی `flask` نسخهٔ 2.0.0، سازگار است.

با `--resolution lowest`، uv کمترین نسخهٔ ممکن را برای همهٔ dependencyها، هم مستقیم و هم غیرمستقیم (transitive)، نصب می‌کند. به‌عنوان جایگزین، `--resolution lowest-direct` کمترین نسخه‌های سازگار را برای همهٔ dependencyهای مستقیم استفاده می‌کند، در حالی که برای بقیهٔ dependencyها از جدیدترین نسخه‌های سازگار استفاده می‌کند. uv همیشه برای dependencyهای build از جدیدترین نسخه‌ها استفاده می‌کند.

برای مثال، با فرض فایل `requirements.in` زیر:

```requirements title="requirements.in"
flask>=2.0.0
```

اجرای `uv pip compile requirements.in -o requirements.txt` فایل `requirements.txt` زیر را تولید می‌کند:

```requirements title="requirements.txt"
# This file was autogenerated by uv via the following command:
#    uv pip compile requirements.in -o requirements.txt
blinker==1.7.0
    # via flask
click==8.1.7
    # via flask
flask==3.0.0
itsdangerous==2.1.2
    # via flask
jinja2==3.1.2
    # via flask
markupsafe==2.1.3
    # via
    #   jinja2
    #   werkzeug
werkzeug==3.0.1
    # via flask
```

اما `uv pip compile --resolution lowest requirements.in -o requirements.txt` به‌جای آن این را تولید می‌کند:

```requirements title="requirements.txt"
# This file was autogenerated by uv via the following command:
#    uv pip compile --resolution lowest requirements.in -o requirements.txt
click==7.1.2
    # via flask
flask==2.0.0
itsdangerous==2.0.0
    # via flask
jinja2==3.0.0
    # via flask
markupsafe==2.0.0
    # via jinja2
werkzeug==2.0.0
    # via flask
```

هنگام انتشار libraryها، توصیه می‌شود تست‌ها را جداگانه با `--resolution lowest` یا `--resolution lowest-direct` در یکپارچه‌سازی مداوم اجرا کنید تا از سازگاری با کران‌های پایین اعلام‌شده مطمئن شوید.

## مدیریت pre-release {#pre-release-handling}

به‌طور پیش‌فرض (`if-necessary`)، uv نسخه‌های پایدار را به pre-releaseها ترجیح می‌دهد و فقط در صورتی به pre-releaseها برمی‌گردد که همهٔ کاندیداهای پایداری که constraintهای فعال را برآورده می‌کنند، در طول resolution رد شوند.

از `--prerelease allow` استفاده کنید تا pre-releaseها برای هر package بدون ترجیح دادن اول کاندیداهای پایدار در نظر گرفته شوند، یا از `--prerelease disallow` استفاده کنید تا کاملاً حذف شوند.

از `--prerelease-package foo=allow` استفاده کنید تا استراتژی سراسری pre-release را برای یک package مشخص بازنویسی کنید. استراتژی‌های خاص package را می‌توان در `[tool.uv]` نیز پیکربندی کرد:

```toml
[tool.uv]
prerelease = "disallow"
prerelease-package = { foo = "allow", bar = "if-necessary" }
```

حالت `explicit` pre-releaseها را فقط برای requirementهای first-party که شناسهٔ pre-release دارند در نظر می‌گیرد (نسخه‌های پایدار را ترجیح می‌دهد و فقط در صورت لزوم به pre-releaseها برمی‌گردد)، در حالی که pre-releaseها را برای بقیهٔ packageها مجاز نمی‌داند.

برای جزئیات بیشتر، به [سازگاری pre-release](../pip/compatibility.md#pre-release-compatibility) مراجعه کنید.

## resolution چندنسخه‌ای {#multi-version-resolution}

در طول resolution سراسری، ممکن است یک package چند بار با نسخه‌ها یا URLهای متفاوت در همان lockfile فهرست شود، چون ممکن است برای platformها یا نسخه‌های مختلف Python به نسخه‌های متفاوتی نیاز باشد.

می‌توان از تنظیم `--fork-strategy` برای کنترل اینکه uv چگونه بین (۱) کمینه کردن تعداد نسخه‌های انتخاب‌شده و (۲) انتخاب جدیدترین نسخهٔ ممکن برای هر platform، تعادل برقرار می‌کند، استفاده کرد. مورد اول به سازگاری بیشتر در platformها منجر می‌شود، در حالی که مورد دوم به استفاده از نسخه‌های جدیدتر package در جاهایی که ممکن است منجر می‌شود.

به‌طور پیش‌فرض (`--fork-strategy requires-python`)، uv برای انتخاب جدیدترین نسخهٔ هر package برای هر نسخهٔ پشتیبانی‌شدهٔ Python بهینه‌سازی می‌کند، در حالی که تعداد نسخه‌های انتخاب‌شده در platformها را کمینه می‌کند.

برای مثال، هنگام resolution برای `numpy` با نیازمندی Python برابر `>=3.8`، uv نسخه‌های زیر را انتخاب می‌کند:

```txt
numpy==1.24.4 ; python_version == "3.8"
numpy==2.0.2 ; python_version == "3.9"
numpy==2.2.0 ; python_version >= "3.10"
```

این resolution نشان می‌دهد که NumPy 2.2.0 و نسخه‌های بعدی حداقل به Python 3.10 نیاز دارند، در حالی که نسخه‌های قبلی با Python 3.8 و 3.9 سازگارند.

با `--fork-strategy fewest`، uv به‌جای آن تعداد نسخه‌های انتخاب‌شده برای هر package را کمینه می‌کند و نسخه‌های قدیمی‌تری را ترجیح می‌دهد که با بازهٔ وسیع‌تری از نسخه‌های پشتیبانی‌شدهٔ Python یا platformها سازگارند.

برای مثال، در سناریوی بالا، uv به‌جای ارتقا به `numpy==2.0.2` برای Python 3.9 و `numpy==2.2.0` برای Python 3.10 و نسخه‌های بعدی، `numpy==1.24.4` را برای همهٔ نسخه‌های Python انتخاب می‌کند.

## constraintهای dependency {#dependency-constraints}

مانند pip، uv از فایل‌های constraint (`--constraint constraints.txt`) پشتیبانی می‌کند که مجموعهٔ نسخه‌های قابل‌قبول برای packageهای داده‌شده را محدود می‌کنند. فایل‌های constraint شبیه فایل‌های requirement هستند، اما صرف فهرست شدن به‌عنوان constraint باعث نمی‌شود package در resolution گنجانده شود. در عوض، constraintها فقط زمانی اثر می‌گذارند که یک package درخواستی از قبل به‌عنوان dependency مستقیم یا transitive کشیده شده باشد. constraintها برای کاهش بازهٔ نسخه‌های موجود برای یک dependency از نوع transitive مفیدند. همچنین می‌توان از آن‌ها برای همگام نگه داشتن یک resolution با مجموعهٔ دیگری از نسخه‌های حل‌شده استفاده کرد، صرف‌نظر از اینکه کدام packageها بین این دو هم‌پوشانی دارند.

## overrideهای dependency {#dependency-overrides}

overrideهای dependency امکان دور زدن resolutionهای ناموفق یا نامطلوب را با بازنویسی dependencyهای اعلام‌شدهٔ یک package فراهم می‌کنند. overrideها آخرین راه‌حل مفیدی برای مواردی هستند که _می‌دانید_ یک dependency با نسخهٔ مشخصی از یک package سازگار است، با وجود اینکه metadata خلاف آن را نشان می‌دهد.

برای مثال، اگر یک dependency از نوع transitive، requirement با مقدار `pydantic>=1.0,<2.0` را اعلام کند، اما با `pydantic>=2.0` هم کار کند، کاربر می‌تواند dependency اعلام‌شده را با گنجاندن `pydantic>=1.0,<3` در overrideها بازنویسی کند و به این ترتیب به resolver اجازه دهد نسخهٔ جدیدتری از `pydantic` را انتخاب کند.

به‌طور مشخص، اگر `pydantic>=1.0,<3` به‌عنوان override گنجانده شود، uv همهٔ requirementهای اعلام‌شده روی `pydantic` را نادیده می‌گیرد و آن‌ها را با override جایگزین می‌کند. در مثال بالا، requirement با مقدار `pydantic>=1.0,<2.0` کاملاً نادیده گرفته می‌شود و به‌جای آن با `pydantic>=1.0,<3` جایگزین می‌شود.

در حالی که constraintها فقط می‌توانند مجموعهٔ نسخه‌های قابل‌قبول برای یک package را _کاهش_ دهند، overrideها می‌توانند مجموعهٔ نسخه‌های قابل‌قبول را _گسترش_ دهند و راه فراری برای کران‌های بالای نادرست نسخه فراهم کنند. مانند constraintها، overrideهای سراسری dependencyای روی package اضافه نمی‌کنند و فقط زمانی اثر می‌گذارند که package در یک dependency مستقیم یا transitive درخواست شده باشد.

در `pyproject.toml`، از `tool.uv.override-dependencies` برای تعریف فهرستی از overrideها استفاده کنید. در رابط سازگار با pip، می‌توان از گزینهٔ `--override` برای ارسال فایل‌هایی با همان قالب فایل‌های constraint استفاده کرد.

به‌طور پیش‌فرض، یک override روی همهٔ requirementها برای dependency نام‌برده اعمال می‌شود، از جمله requirementهای مستقیم. در عوض، می‌توان یک override را به dependencyهای یک نسخهٔ مشخص از package با استفاده از یک inline table محدود کرد:

```toml
[tool.uv]
override-dependencies = [
    "foo>1",
    { package = { name = "bar", version = "0.0.5" }, dependencies = ["foo>2"] },
]
```

در این مثال، `foo>1` override سراسری است، در حالی که `foo>2` جایگزین requirementهای `foo` اعلام‌شده توسط `bar==0.0.5` می‌شود. اگر `bar` dependencyای روی `foo` اعلام نکند، override محدودشده آن را اضافه می‌کند. بقیهٔ dependencyهای اعلام‌شده توسط `bar` بدون تغییر می‌مانند. فیلد `version` در `package` را می‌توان حذف کرد تا overrideهای محدودشده روی همهٔ نسخه‌های `bar` اعمال شوند. یک ورودی مخصوص نسخه بر ورودی همهٔ نسخه‌ها اولویت دارد و یک override محدودشده بر override سراسری برای همان dependency اولویت دارد.

overrideهای محدودشده در حال حاضر فقط از specifierهای نسخهٔ registry پشتیبانی می‌کنند. sourceهای URL مستقیم و path، از جمله sourceهای Git، و indexهای صریح پشتیبانی نمی‌شوند.

در حالت pre-release از نوع `explicit`، یک specifier صریح pre-release در هر override محدودشده، بازگشت به pre-release با اولویت نسخهٔ پایدار را برای آن package در کل resolution مجاز می‌کند. به همین ترتیب، یک pin دقیق نسخهٔ yanked در هر override محدودشده، آن package را برای انتخاب کاندیدای نسخهٔ yanked در کل resolution وارد می‌کند، حتی اگر scope انتخاب نشده باشد.

اگر چند override برای همان package ارائه شود، باید با [markerها](#platform-markers) از هم متمایز شوند. اگر packageای dependencyای با marker داشته باشد، هنگام استفاده از overrideها بدون قید و شرط جایگزین می‌شود — مهم نیست marker درست ارزیابی شود یا نادرست.

## exclusionهای dependency {#dependency-exclusions}

exclusionهای dependency، packageها را از گراف dependency حذف می‌کنند. به‌طور پیش‌فرض، یک exclusion روی همهٔ requirementها برای dependency نام‌برده اعمال می‌شود، از جمله requirementهای مستقیم:

```toml
[tool.uv]
exclude-dependencies = ["foo"]
```

در عوض، می‌توان یک exclusion را به dependencyهای یک نسخهٔ مشخص از package محدود کرد:

```toml
[tool.uv]
exclude-dependencies = [
    { package = { name = "bar", version = "0.0.5" }, dependencies = ["foo"] },
]
```

در این مثال، requirementهای `foo` اعلام‌شده توسط `bar==0.0.5` حذف می‌شوند، در حالی که requirementهای `foo` از packageهای دیگر بدون تغییر می‌مانند. فیلد `version` در `package` را می‌توان حذف کرد تا exclusionهای محدودشده روی همهٔ نسخه‌های `bar` اعمال شوند. یک ورودی مخصوص نسخه بر ورودی همهٔ نسخه‌ها اولویت دارد.

exclusionهای محدودشده را می‌توان با overrideهای محدودشده ترکیب کرد تا یک dependency با dependency دیگری جایگزین شود:

```toml
[tool.uv]
override-dependencies = [
    { package = { name = "bar", version = "0.0.5" }, dependencies = ["pytorch-lightning"] },
]
exclude-dependencies = [
    { package = { name = "bar", version = "0.0.5" }, dependencies = ["lightning"] },
]
```

اگر همان dependency در یک scope تطبیق‌یافته هم override شده و هم exclude شده باشد، exclusion اولویت دارد.

## metadata مربوط به dependency {#dependency-metadata}

در طول resolution، uv نیاز دارد metadata هر package را که با آن روبه‌رو می‌شود حل کند تا dependencyهای آن را تعیین کند. این metadata اغلب به‌صورت یک فایل ایستا در package index در دسترس است؛ اما برای packageهایی که فقط source distribution ارائه می‌دهند، ممکن است metadata از ابتدا در دسترس نباشد.

در چنین مواردی، uv باید package را بسازد تا metadata آن را تعیین کند (مثلاً با فراخوانی `setup.py`). این کار می‌تواند در طول resolution هزینهٔ عملکردی ایجاد کند. علاوه بر این، این الزام را تحمیل می‌کند که package روی همهٔ platformها قابل ساخت باشد، که ممکن است درست نباشد.

برای مثال، ممکن است packageای داشته باشید که فقط باید روی Linux ساخته و نصب شود، اما روی macOS یا Windows با موفقیت ساخته نمی‌شود. در حالی که uv می‌تواند یک lockfile کاملاً معتبر برای این سناریو بسازد، انجام این کار نیازمند ساخت package است که روی platformهای غیر Linux شکست می‌خورد.

می‌توان از جدول `tool.uv.dependency-metadata` برای ارائهٔ metadata ایستا برای چنین dependencyهایی از ابتدا استفاده کرد و به این ترتیب به uv اجازه داد مرحلهٔ ساخت را رد کند و به‌جای آن از metadata ارائه‌شده استفاده کند.

برای مثال، برای ارائهٔ metadata برای `chumpy` از ابتدا، `dependency-metadata` آن را در `pyproject.toml` بگنجانید:

```toml
[[tool.uv.dependency-metadata]]
name = "chumpy"
version = "0.70"
requires-dist = ["numpy>=1.8.1", "scipy>=0.13.0", "six>=1.11.0"]
```

این اعلام‌ها برای مواردی در نظر گرفته شده‌اند که packageای metadata ایستا را از ابتدا اعلام _نمی‌کند_، هرچند برای packageهایی که نیازمند [غیرفعال کردن build isolation](./projects/config.md#build-isolation) هستند نیز مفیدند. در چنین مواردی، ممکن است اعلام metadata مربوط به package از ابتدا آسان‌تر از ایجاد یک environment ساخت سفارشی پیش از resolution package باشد.

برای مثال، نسخه‌های قدیمی `flash-attn` metadata ایستا اعلام نمی‌کردند. با اعلام metadata برای `flash-attn` از ابتدا، uv می‌تواند `flash-attn` را بدون ساخت package از source حل کند (کاری که خود نیازمند نصب `torch` است):

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["flash-attn"]

[tool.uv.sources]
flash-attn = { git = "https://github.com/Dao-AILab/flash-attention", tag = "v2.6.3" }

[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

مانند overrideهای dependency، می‌توان از `tool.uv.dependency-metadata` برای مواردی استفاده کرد که metadata یک package نادرست یا ناقص است، یا زمانی که package در package index موجود نیست. در حالی که overrideهای dependency اجازهٔ بازنویسی نسخه‌های مجاز یک package را به‌صورت سراسری می‌دهند، overrideهای metadata اجازهٔ بازنویسی metadata اعلام‌شدهٔ یک _package مشخص_ را می‌دهند.

برای یک wheel نوع variant در یک URL مستقیم، `METADATA` مربوط به wheel باید همچنان قابل تجزیه باشد، حتی زمانی که `tool.uv.dependency-metadata` dependencyهای آن را تأمین می‌کند. برای مثال، یک override می‌تواند یک constraint نادرست نسخه را جایگزین کند، اما ورودی اصلی `Requires-Dist` باید از نحو معتبر استفاده کند.

!!! note

    فیلد `version` در `tool.uv.dependency-metadata` برای dependencyهای مبتنی بر registry اختیاری است (وقتی حذف شود، uv فرض می‌کند metadata روی همهٔ نسخه‌های package اعمال می‌شود)، اما برای dependencyهای URL مستقیم (مانند dependencyهای Git) _الزامی_ است.

ورودی‌های جدول `tool.uv.dependency-metadata` از مشخصات [Metadata 2.3](https://packaging.python.org/en/latest/specifications/core-metadata/) پیروی می‌کنند، هرچند uv فقط `name`، `version`، `requires-dist`، `requires-python` و `provides-extra` را می‌خواند. فیلد `version` نیز اختیاری در نظر گرفته می‌شود. اگر حذف شود، metadata برای همهٔ نسخه‌های package مشخص‌شده استفاده خواهد شد.

## dependencyهای متعارض {#conflicting-dependencies}

uv نیاز دارد که همهٔ dependencyهای اعلام‌شده توسط یک project با یکدیگر سازگار باشند و هنگام ساخت lockfile همهٔ dependencyها را با هم حل می‌کند. این شامل dependencyهای project، dependencyهای اختیاری («extra»ها) و groupهای dependency (dependencyهای توسعه) می‌شود.

اگر dependencyهای اعلام‌شده در یک extra با dependencyهای extra دیگر سازگار نباشند، uv در حل requirementهای project با خطا شکست می‌خورد. برای مثال، دو مجموعهٔ dependency اختیاری را در نظر بگیرید که با یکدیگر تعارض دارند:

```toml title="pyproject.toml"
[project.optional-dependencies]
extra1 = ["numpy==2.1.2"]
extra2 = ["numpy==2.0.0"]
```

اگر `uv lock` را با dependencyهای بالا اجرا کنید، resolution شکست می‌خورد:

```console
$ uv lock
  x No solution found when resolving dependencies:
  `-> Because myproject[extra2] depends on numpy==2.0.0 and myproject[extra1] depends on numpy==2.1.2, we can conclude that myproject[extra1] and
      myproject[extra2] are incompatible.
      And because your project requires myproject[extra1] and myproject[extra2], we can conclude that your projects's requirements are unsatisfiable.
```

برای دور زدن این مشکل، uv از اعلام صریح تعارض‌ها پشتیبانی می‌کند. اگر مشخص کنید که `extra1` و `extra2` متعارض‌اند، uv آن‌ها را جداگانه حل می‌کند. تعارض‌ها را در بخش `tool.uv` مشخص کنید:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { extra = "extra1" },
      { extra = "extra2" },
    ],
]
```

حالا اجرای `uv lock` موفق خواهد بود. اما حالا نمی‌توانید هم‌زمان هم `extra1` و هم `extra2` را نصب کنید:

```console
$ uv sync --extra extra1 --extra extra2
Resolved 3 packages in 14ms
error: extra `extra1`, extra `extra2` are incompatible with the declared conflicts: {`myproject[extra1]`, `myproject[extra2]`}
```

این خطا به این دلیل رخ می‌دهد که نصب هم‌زمان `extra1` و `extra2` به نصب دو نسخهٔ متفاوت از یک package در همان environment منجر می‌شود.

استراتژی بالا برای برخورد با dependencyهای اختیاری متعارض با groupهای dependency نیز کار می‌کند:

```toml title="pyproject.toml"
[dependency-groups]
group1 = ["numpy==2.1.2"]
group2 = ["numpy==2.0.0"]

[tool.uv]
conflicts = [
    [
      { group = "group1" },
      { group = "group2" },
    ],
]
```

تنها تفاوت با extraهای متعارض این است که باید به‌جای کلید `extra` از کلید `group` استفاده کنید.

هنگام استفاده از یک workspace با چند project، همان محدودیت‌ها اعمال می‌شوند — uv نیاز دارد همهٔ اعضای workspace با یکدیگر سازگار باشند. به همین ترتیب، می‌توان تعارض‌ها را در اعضای workspace اعلام کرد.

برای مثال، workspace زیر را در نظر بگیرید:

```toml title="member1/pyproject.toml"
[project]
name = "member1"

[project.optional-dependencies]
extra1 = ["numpy==2.1.2"]
```

```toml title="member2/pyproject.toml"
[project]
name = "member2"

[project.optional-dependencies]
extra2 = ["numpy==2.0.0"]
```

برای اعلام تعارض بین extraها در این اعضای مختلف workspace، از کلید `package` استفاده کنید:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { package = "member1", extra = "extra1" },
      { package = "member2", extra = "extra2" },
    ],
]
```

همچنین ممکن است dependencyهای project (یعنی `project.dependencies`) یک عضو workspace با extra عضو دیگر تعارض داشته باشند، برای مثال:

```toml title="member1/pyproject.toml"
[project]
name = "member1"
dependencies = ["numpy==2.1.2"]
```

```toml title="member2/pyproject.toml"
[project]
name = "member2"

[project.optional-dependencies]
extra2 = ["numpy==2.0.0"]
```

این تعارض را نیز می‌توان با استفاده از کلید `package` اعلام کرد:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { package = "member1" },
      { package = "member2", extra = "extra2" },
    ],
]
```

به همین ترتیب، ممکن است برخی اعضای workspace dependencyهای project متعارضی داشته باشند:

```toml title="member1/pyproject.toml"
[project]
name = "member1"
dependencies = ["numpy==2.1.2"]
```

```toml title="member2/pyproject.toml"
[project]
name = "member2"
dependencies = ["numpy==2.0.0"]
```

این تعارض را نیز می‌توان با استفاده از کلید `package` اعلام کرد:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { package = "member1" },
      { package = "member2" },
    ],
]
```

این اعضای workspace با هم قابل‌نصب نخواهند بود؛ برای مثال، ریشهٔ workspace نمی‌تواند این را تعریف کند:

```toml title="pyproject.toml"
[project]
name = "root"
dependencies = ["member1", "member2"]
```

## کران‌های پایین {#lower-bounds}

به‌طور پیش‌فرض، `uv add` کران‌های پایین به dependencyها اضافه می‌کند و هنگام استفاده از uv برای مدیریت projectها، uv هشدار می‌دهد اگر dependencyهای مستقیم کران پایین نداشته باشند.

کران‌های پایین در «مسیر خوشایند» حیاتی نیستند، اما برای مواردی که تعارض dependency وجود دارد مهم‌اند. برای مثال، projectای را در نظر بگیرید که به دو package نیاز دارد و آن packageها dependencyهای متعارضی دارند. resolver باید همهٔ ترکیب‌های همهٔ نسخه‌ها را در محدودهٔ constraintها برای این دو package بررسی کند — اگر همهٔ آن‌ها تعارض داشته باشند، خطایی گزارش می‌شود، چون dependencyها قابل برآورده شدن نیستند. اگر کران پایینی وجود نداشته باشد، resolver می‌تواند (و اغلب این کار را می‌کند) تا قدیمی‌ترین نسخهٔ یک package عقب‌گرد کند. این موضوع نه‌تنها مشکل‌ساز است چون کند است، بلکه نسخهٔ قدیمی package اغلب ساخته نمی‌شود، یا resolver ممکن است نسخه‌ای را انتخاب کند که آن‌قدر قدیمی است که به package متعارض وابسته نیست، اما با کد شما هم کار نمی‌کند.

کران‌های پایین هنگام نوشتن یک library به‌ویژه حیاتی‌اند. مهم است که کمترین نسخه را برای هر dependency که library شما با آن کار می‌کند اعلام کنید و صحت کران‌ها را اعتبارسنجی کنید — با تست کردن با [`--resolution lowest` یا `--resolution lowest-direct`](#resolution-strategy). در غیر این صورت، کاربر ممکن است نسخهٔ قدیمی و ناسازگاری از یکی از dependencyهای library شما را دریافت کند و library با خطای غیرمنتظره‌ای شکست بخورد.

## resolutionهای تکرارپذیر {#reproducible-resolutions}

uv از گزینهٔ `--exclude-newer` پشتیبانی می‌کند تا resolution را به distributionهای آپلودشده پیش از یک تاریخ مشخص محدود کند و به این ترتیب امکان بازتولید نصب‌ها را بدون توجه به انتشار packageهای جدید فراهم کند. تاریخ با زمان آپلود هر آرتیفکت distribution به‌صورت جداگانه مقایسه می‌شود (یعنی زمانی که هر فایل در package index آپلود شده است)، نه تاریخ انتشار نسخهٔ package. تاریخ را می‌توان به‌صورت یک timestamp از نوع [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html) (برای مثال، `2006-12-02T02:07:43Z`) یا یک تاریخ محلی با همان قالب (برای مثال، `2006-12-02`) در منطقهٔ زمانی پیکربندی‌شدهٔ سیستم خود مشخص کرد.

!!! important

    package index باید از فیلد `upload-time` مطابق مشخصات [`PEP 700`](https://peps.python.org/pep-0700/) پشتیبانی کند. اگر فیلد برای یک distribution مشخص وجود نداشته باشد، آن distribution به‌عنوان ناموجود در نظر گرفته می‌شود، مگر اینکه package از طریق `--exclude-newer-package <package>=false` از این قاعده مستثنا شده باشد، یا index با مقدار `exclude-newer` خودش پیکربندی شده باشد، یا index از طریق `[[tool.uv.index]] exclude-newer = false` مستثنا شده باشد. PyPI برای همهٔ packageها `upload-time` ارائه می‌دهد.

برای اطمینان از تکرارپذیری، پیام‌های مربوط به resolutionهای غیرقابل‌برآورده شدن، اشاره‌ای نمی‌کنند که distributionها به دلیل فلگ `--exclude-newer` حذف شده‌اند — distributionهای جدیدتر چنان در نظر گرفته می‌شوند که گویی وجود ندارند.

!!! note

    گزینهٔ `--exclude-newer` فقط روی packageهایی اعمال می‌شود که از یک registry خوانده می‌شوند (برخلاف، برای مثال، dependencyهای Git). علاوه بر این، هنگام استفاده از رابط `uv pip`، uv packageهای از پیش نصب‌شده را تنزل نمی‌دهد، مگر اینکه فلگ `--reinstall` ارائه شده باشد؛ در این صورت uv یک resolution جدید انجام می‌دهد.

این گزینه در `pyproject.toml` نیز پشتیبانی می‌شود، برای مثال:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer = "2006-12-02T02:07:43Z"
```

برای غیرفعال کردن یک cutoff سراسری از یک منبع پیکربندی با اولویت پایین‌تر، `--exclude-newer false` را ارسال کنید، `UV_EXCLUDE_NEWER=false` را تنظیم کنید، یا `exclude-newer = false` را در یک فایل پیکربندی با اولویت بالاتر تنظیم کنید.

هنگامی که در پیکربندی پایدار مشخص شده باشد، زمان‌های محلی مجاز نیستند.

مقادیر را می‌توان برای packageهای مشخص نیز تعیین کرد، برای مثال، `--exclude-newer-package setuptools=2006-12-02`، یا:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer-package = { setuptools = "2006-12-02T02:07:43Z" }
```

گزینهٔ package همچنین `<package>=false` را می‌پذیرد تا یک package را از این محدودیت مستثنا کند، برای مثال، `--exclude-newer-package setuptools=false`، یا:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer-package = { setuptools = false }
```

این کار برای استفادهٔ موقت از نسخهٔ جدیدتر یک package یا برای اجازهٔ resolution یک package از indexای که زمان‌های آپلود را منتشر نمی‌کند، مفید است.

مقادیر خاص package بر مقادیر سراسری و خاص index اولویت دارند.

به همین ترتیب، یک index جداگانه می‌تواند cutoff سراسری را بازنویسی کند:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer = "2006-12-02T02:07:43Z"

[[tool.uv.index]]
name = "internal"
url = "https://internal.example.com/simple"
exclude-newer = "7 days"
```

یا آن را برای آن index کاملاً غیرفعال کند:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "internal"
url = "https://internal.example.com/simple"
exclude-newer = false
```

این کار برای indexهای خصوصی که `upload-time` منتشر نمی‌کنند، یا برای اعمال یک پنجرهٔ تکرارپذیری متفاوت روی یک index مشخص در حالی که رفتار سراسری حفظ می‌شود، مفید است.

## cooldownهای dependency {#dependency-cooldowns}

uv همچنین از «cooldown»های dependency پشتیبانی می‌کند که در آن‌ها resolution packageهای جدیدتر از یک مدت را نادیده می‌گیرد. این روش خوبی برای بهبود وضعیت امنیتی با به تأخیر انداختن به‌روزرسانی packageهاست تا زمانی که جامعه فرصت بررسی نسخه‌های جدید packageها را داشته باشد.

این ویژگی از طریق گزینهٔ [`exclude-newer`](#reproducible-resolutions) در دسترس است و همان معنا را دارد.

یک cooldown برای dependency را با تعیین یک مدت به‌جای یک مقدار مطلق تعریف کنید. می‌توان از یک مدت «دوستانه» (برای مثال، `24 hours`، `1 week`، `30 days`) یا یک مدت ISO 8601 (برای مثال، `PT24H`، `P7D`، `P30D`) استفاده کرد.

!!! note

    مدت‌ها معنای منطقهٔ زمانی محلی را رعایت نمی‌کنند و همیشه به تعداد ثابتی از ثانیه‌ها تبدیل می‌شوند، با این فرض که یک روز ۲۴ ساعت است (برای مثال، تغییرهای DST نادیده گرفته می‌شوند). واحدهای تقویمی مانند ماه‌ها و سال‌ها مجاز نیستند، چون طول آن‌ها ذاتاً ناسازگار است.

وقتی یک مدت برای resolution استفاده می‌شود، یک timestamp نسبت به زمان جاری محاسبه می‌شود. هنگام استفاده از یک فایل `uv.lock`، timestamp در lockfile گنجانده می‌شود. uv وقتی زمان جاری تغییر می‌کند lockfile را به‌روز نمی‌کند؛ در عوض، uv زمانی timestamp را به‌روز می‌کند که یک resolution جدید انجام شود، برای مثال وقتی از `--upgrade` یا `--refresh` استفاده می‌شود.

این گزینه در `pyproject.toml` نیز پشتیبانی می‌شود، برای مثال:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer = "1 week"
```

مقادیر را می‌توان برای packageهای مشخص نیز تعیین کرد، برای مثال، `--exclude-newer-package "setuptools=30 days"`، یا:

```toml title="pyproject.toml"
[tool.uv]
exclude-newer = "1 week"
exclude-newer-package = { setuptools = "30 days" }
```

## Source distribution {#source-distribution}

[PEP 625](https://peps.python.org/pep-0625/) مشخص می‌کند که packageها باید source distributionها را به‌صورت آرشیوهای tarball فشرده‌شده با gzip (`.tar.gz`) توزیع کنند. پیش از این مشخصات، قالب‌های آرشیو دیگری نیز که برای سازگاری با نسخه‌های قبلی باید پشتیبانی شوند، مجاز بودند.

!!! important

    از نسخهٔ 0.12، uv source distributionهایی را که با الزامات پسوند [PEP 625] مطابقت ندارند، رد می‌کند، به‌جز آرشیوهای `.zip` که همچنان برای سازگاری با نسخه‌های قبلی پذیرفته می‌شوند.

## نسخه‌بندی lockfile {#lockfile-versioning}

فایل `uv.lock` از یک اسکیمای نسخه‌دار استفاده می‌کند. نسخهٔ اسکیما در فیلد `version` در lockfile گنجانده شده است.

هر نسخهٔ مشخص از uv می‌تواند lockfileهایی با همان نسخهٔ اسکیما را بخواند و بنویسد، اما lockfileهایی با نسخهٔ اسکیمای بالاتر را رد می‌کند. برای مثال، اگر نسخهٔ uv شما از اسکیمای v1 پشتیبانی کند، `uv lock` در صورت برخورد با یک lockfile موجود با اسکیمای v2 خطا می‌دهد.

نسخه‌های uv که از اسکیمای v2 پشتیبانی می‌کنند، _ممکن است_ بتوانند lockfileهایی با اسکیمای v1 را بخوانند، اگر به‌روزرسانی اسکیما با نسخه‌های قبلی سازگار بوده باشد. اما این تضمین‌شده نیست و uv ممکن است در صورت برخورد با lockfileای با نسخهٔ اسکیمای قدیمی، با خطا خارج شود.

نسخهٔ اسکیما بخشی از API عمومی محسوب می‌شود و بنابراین فقط در انتشارهای minor به‌عنوان یک تغییر ناسازگار افزایش می‌یابد (نگاه کنید به [نسخه‌بندی](../reference/policies/versioning.md)). به این ترتیب، تضمین می‌شود که همهٔ نسخه‌های patch در uv در یک انتشار minor مشخص از uv، سازگاری کامل lockfile را دارند. به عبارت دیگر، lockfileها فقط در انتشارهای minor ممکن است رد شوند.

فیلد `revision` در lockfile برای ردیابی تغییرهای سازگار با نسخه‌های قبلی در lockfile استفاده می‌شود. برای مثال، اضافه کردن یک فیلد جدید به distributionها. تغییرهای revision باعث نمی‌شوند نسخه‌های قدیمی‌تر uv خطا بدهند.

## بیشتر بدانید {#learn-more}

برای جزئیات بیشتر دربارهٔ سازوکار داخلی resolver، به مستندات [مرجع resolver](../reference/internals/resolver.md) مراجعه کنید.
