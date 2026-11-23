# toolها {#tools}

toolها packageهای Python هستند که رابط‌های خط دستور ارائه می‌دهند.

!!! note

    برای آشنایی با کار با رابط toolها، [راهنمای toolها](../guides/tools.md) را ببینید — این سند جزئیات مدیریت tool را بررسی می‌کند.

## رابط `uv tool` {#the-uv-tool-interface}

uv یک رابط اختصاصی برای تعامل با toolها دارد. می‌توان toolها را بدون نصب با `uv tool run` فراخواند؛ در این صورت dependencyهای آن‌ها در یک virtual environment موقت نصب می‌شوند که از project جاری جدا است.

از آنجا که اجرای toolها بدون نصب آن‌ها بسیار رایج است، یک نام مستعار `uvx` برای `uv tool run` ارائه شده است — این دو دستور دقیقاً معادل هم هستند. برای اختصار، مستندات بیشتر به `uvx` اشاره می‌کند تا `uv tool run`.

toolها را می‌توان با `uv tool install` نیز نصب کرد؛ در این صورت فایل‌های اجرایی آن‌ها [در `PATH` در دسترس‌اند](#tool-executables) — همچنان از یک virtual environment جدا استفاده می‌شود، اما وقتی دستور کامل شد، حذف نمی‌شود.

## اجرا در برابر نصب {#execution-vs-installation}

در بیشتر موارد، اجرای یک tool با `uvx` مناسب‌تر از نصب tool است. نصب tool زمانی مفید است که نیاز دارید tool برای برنامه‌های دیگر سیستم شما در دسترس باشد؛ مثلاً اگر scriptای که کنترل آن را ندارید به tool نیاز دارد، یا اگر در یک image Docker هستید و می‌خواهید tool را برای کاربران در دسترس کنید.

## environmentهای tool {#tool-environments}

هنگام اجرای یک tool با `uvx`، یک virtual environment در پوشهٔ cache uv ذخیره می‌شود و یک‌بارمصرف تلقی می‌شود؛ یعنی اگر `uv cache clean` را اجرا کنید، environment حذف می‌شود. environment فقط برای کاهش سربار فراخوانی‌های مکرر cache می‌شود. اگر environment حذف شود، یک environment جدید به‌صورت خودکار ساخته می‌شود.

هنگام نصب یک tool با `uv tool install`، یک virtual environment در [پوشهٔ toolهای uv](../reference/storage.md#tools) ساخته می‌شود. environment حذف نمی‌شود مگر اینکه tool حذف نصب شود. اگر environment به‌صورت دستی حذف شود، tool اجرا نمی‌شود.

!!! important

    environmentهای tool _قرار نیست_ مستقیماً تغییر داده شوند. به‌شدت توصیه می‌شود هرگز یک environment tool را به‌صورت دستی تغییر ندهید، مثلاً با یک عملیات `pip`.

## نسخه‌های tool {#tool-versions}

مگر اینکه نسخهٔ خاصی درخواست شود، `uv tool install` جدیدترین نسخهٔ موجود از tool درخواستی را نصب می‌کند. `uvx` در _اولین فراخوانی_ از جدیدترین نسخهٔ موجود از tool درخواستی استفاده می‌کند. پس از آن، `uvx` از نسخهٔ cacheشدهٔ tool استفاده می‌کند، مگر اینکه نسخهٔ دیگری درخواست شود، cache هرس شود یا cache تازه‌سازی شود.

برای مثال، برای اجرای یک نسخهٔ خاص از Ruff:

```console
$ uvx ruff@0.6.0 --version
ruff 0.6.0
```

فراخوانی بعدی `uvx` از جدیدترین نسخه استفاده می‌کند، نه نسخهٔ cacheشده.

```console
$ uvx ruff --version
ruff 0.6.2
```

اما اگر نسخهٔ جدیدی از Ruff منتشر شده باشد، استفاده نمی‌شود مگر اینکه cache تازه‌سازی شده باشد.

برای درخواست جدیدترین نسخهٔ Ruff و تازه‌سازی cache، از پسوند `@latest` استفاده کنید:

```console
$ uvx ruff@latest --version
0.6.2
```

وقتی یک tool با `uv tool install` نصب شد، `uvx` به‌طور پیش‌فرض از نسخهٔ نصب‌شده استفاده می‌کند.

برای مثال، پس از نصب نسخهٔ قدیمی‌تر Ruff:

```console
$ uv tool install ruff==0.5.0
```

نسخهٔ `ruff` و `uvx ruff` یکسان است:

```console
$ ruff --version
ruff 0.5.0
$ uvx ruff --version
ruff 0.5.0
```

با این حال، می‌توانید با درخواست صریح جدیدترین نسخه، نسخهٔ نصب‌شده را نادیده بگیرید، مثلاً:

```console
$ uvx ruff@latest --version
0.6.2
```

یا با استفاده از فلگ `--isolated` که از تازه‌سازی cache جلوگیری می‌کند اما نسخهٔ نصب‌شده را نادیده می‌گیرد:

```console
$ uvx --isolated ruff --version
0.6.2
```

`uv tool install` همچنین به specifierهای `{package}@{version}` و `{package}@latest` احترام می‌گذارد، مانند:

```console
$ uv tool install ruff@latest
$ uv tool install ruff@0.6.0
```

## ارتقای toolها {#upgrading-tools}

environmentهای tool را می‌توان با `uv tool upgrade` ارتقا داد یا با عملیات‌های بعدی `uv tool install` کاملاً از نو ساخت.

برای ارتقای همهٔ packageهای یک environment tool:

```console
$ uv tool upgrade black
```

برای ارتقای یک package تکی در یک environment tool:

```console
$ uv tool upgrade black --upgrade-package click
```

ارتقای toolها به constraintهای نسخهٔ ارائه‌شده هنگام نصب tool احترام می‌گذارد. برای مثال، `uv tool install black >=23,<24` و پس از آن `uv tool upgrade black`، Black را به جدیدترین نسخه در بازهٔ `>=23,<24` ارتقا می‌دهد.

برای جایگزین‌کردن constraintهای نسخه، tool را با `uv tool install` دوباره نصب کنید:

```console
$ uv tool install black>=24
```

به‌همین ترتیب، ارتقای toolها تنظیمات ارائه‌شده هنگام نصب tool را حفظ می‌کند. برای مثال، `uv tool install black --prerelease allow` و پس از آن `uv tool upgrade black`، تنظیم `--prerelease allow` را حفظ می‌کند.

!!! note

    ارتقای toolها فایل‌های اجرایی tool را دوباره نصب می‌کند، حتی اگر تغییر نکرده باشند.

برای نصب دوبارهٔ packageها هنگام ارتقا، از گزینه‌های `--reinstall` و `--reinstall-package` استفاده کنید.

برای نصب دوبارهٔ همهٔ packageهای یک environment tool:

```console
$ uv tool upgrade black --reinstall
```

برای نصب دوبارهٔ یک package تکی در یک environment tool:

```console
$ uv tool upgrade black --reinstall-package click
```

## گنجاندن dependencyهای اضافه {#including-additional-dependencies}

می‌توان packageهای اضافه را هنگام اجرای tool گنجاند:

```console
$ uvx --with <extra-package> <tool>
```

و هنگام نصب tool:

```console
$ uv tool install --with <extra-package> <tool-package>
```

گزینهٔ `--with` را می‌توان چند بار ارائه کرد تا packageهای اضافه گنجانده شوند.

گزینهٔ `--with` از مشخصات package پشتیبانی می‌کند، پس می‌توان نسخهٔ خاصی را درخواست کرد:

```console
$ uvx --with <extra-package>==<version> <tool-package>
```

می‌توان از مخفف `-w` به‌جای گزینهٔ `--with` استفاده کرد:

```console
$ uvx -w <extra-package> <tool-package>
```

اگر نسخهٔ درخواستی با نیازمندی‌های package tool در تضاد باشد، حل package با شکست مواجه می‌شود و دستور خطا می‌دهد.

## نصب فایل‌های اجرایی از packageهای اضافه {#installing-executables-from-additional-packages}

هنگام نصب یک tool، ممکن است بخواهید فایل‌های اجرایی packageهای اضافه را در همان environment tool بگنجانید. این کار زمانی مفید است که toolهای مرتبطی دارید که با هم کار می‌کنند یا زمانی که می‌خواهید چند فایل اجرایی نصب کنید که dependencyهای مشترک دارند.

گزینهٔ `--with-executables-from` به شما اجازه می‌دهد packageهای اضافه‌ای را مشخص کنید که فایل‌های اجرایی آن‌ها باید در کنار tool اصلی نصب شوند:

```console
$ uv tool install --with-executables-from <package1>,<package2> <tool-package>
```

برای مثال، برای نصب Ansible همراه با فایل‌های اجرایی `ansible-core` و `ansible-lint`:

```console
$ uv tool install --with-executables-from ansible-core,ansible-lint ansible
```

این کار همهٔ فایل‌های اجرایی packageهای `ansible`، `ansible-core` و `ansible-lint` را در همان environment tool نصب می‌کند و همهٔ آن‌ها را در `PATH` در دسترس قرار می‌دهد.

گزینهٔ `--with-executables-from` را می‌توان با گزینه‌های نصب دیگر ترکیب کرد:

```console
$ uv tool install --with-executables-from ansible-core --with mkdocs-material ansible
```

توجه کنید که `--with-executables-from` با `--with` در این موارد فرق می‌کند:

- `--with` packageهای اضافه را به‌عنوان dependency می‌گنجاند اما فایل‌های اجرایی آن‌ها را نصب نمی‌کند
- `--with-executables-from` هم packageها را به‌عنوان dependency می‌گنجاند و هم فایل‌های اجرایی آن‌ها را نصب می‌کند

## Python versionها {#python-versions}

هر environment tool به یک Python version خاص پیوند خورده است. این کار از همان [منطق کشف Python version](./python-versions.md#discovery-of-python-versions) استفاده می‌کند که دیگر virtual environmentهای ساخته‌شده توسط uv استفاده می‌کنند، اما درخواست‌های Python version غیرسراسری، مانند فایل‌های `.python-version` و مقدار `requires-python` از یک `pyproject.toml`، را نادیده می‌گیرد.

می‌توان از گزینهٔ `--python` برای درخواست یک نسخهٔ خاص استفاده کرد. برای جزئیات بیشتر، مستندات [Python version](./python-versions.md) را ببینید.

اگر Python version استفاده‌شده توسط یک tool _حذف نصب_ شود، environment آن tool خراب می‌شود و ممکن است tool قابل‌استفاده نباشد.

## فایل‌های اجرایی tool {#tool-executables}

فایل‌های اجرایی tool شامل همهٔ entry pointهای کنسول، entry pointهای script و scriptهای باینری ارائه‌شده توسط یک package Python است. فایل‌های اجرایی tool در Unix به‌صورت symlink در [پوشهٔ اجرایی](../reference/storage.md#tool-executables) قرار می‌گیرند و در Windows کپی می‌شوند.

!!! note

    فایل‌های اجرایی ارائه‌شده توسط dependencyهای packageهای tool نصب نمی‌شوند.

[پوشهٔ اجرایی](../reference/storage.md#executable-directory) باید در متغیر `PATH` باشد تا فایل‌های اجرایی tool از shell در دسترس باشند. اگر در `PATH` نباشد، یک هشدار نمایش داده می‌شود. می‌توان از دستور `uv tool update-shell` برای اضافه‌کردن پوشهٔ اجرایی به `PATH` در فایل‌های پیکربندی رایج shell استفاده کرد.

### بازنویسی فایل‌های اجرایی {#overwriting-executables}

نصب toolها فایل‌های اجرایی موجود در پوشهٔ اجرایی را که قبلاً توسط uv نصب نشده‌اند، بازنویسی نمی‌کند. برای مثال، اگر از `pipx` برای نصب یک tool استفاده شده باشد، `uv tool install` با شکست مواجه می‌شود. می‌توان از فلگ `--force` برای نادیده‌گرفتن این رفتار استفاده کرد.

## ارتباط با `uv run` {#relationship-to-uv-run}

فراخوانی `uv tool run <name>` (یا `uvx <name>`) تقریباً معادل این است:

```console
$ uv run --no-project --with <name> -- <name>
```

با این حال، هنگام استفاده از رابط tool uv چند تفاوت قابل‌توجه وجود دارد:

- به گزینهٔ `--with` نیازی نیست — package موردنیاز از نام دستور استنتاج می‌شود.
- environment موقت در یک مکان اختصاصی cache می‌شود.
- به فلگ `--no-project` نیازی نیست — toolها همیشه جدا از project اجرا می‌شوند.
- اگر یک tool از قبل نصب شده باشد، `uv tool run` از نسخهٔ نصب‌شده استفاده می‌کند اما `uv run` استفاده نمی‌کند.

اگر tool نباید از project جدا باشد، مثلاً هنگام اجرای `pytest` یا `mypy`، باید به‌جای `uv tool run` از `uv run` استفاده کرد.
