# نسخه‌های Python {#python-versions}

یک Python version از یک Python interpreter (یعنی فایل اجرایی `python`)، کتابخانهٔ استاندارد و فایل‌های پشتیبان دیگر تشکیل شده است.

## نصب‌های managed و system Python {#managed-and-system-python-installations}

از آنجا که معمولاً در یک سیستم یک نصب Python وجود دارد، uv از [کشف](#discovery-of-python-versions) نسخه‌های Python پشتیبانی می‌کند. با این حال، uv از [نصب نسخه‌های Python](#installing-a-python-version) توسط خودش هم پشتیبانی می‌کند. برای تمایز بین این دو نوع نصب Python، uv به نسخه‌های Pythonی که خودش نصب می‌کند _managed_ Python installations و به همهٔ نصب‌های دیگر Python _system_ Python installations می‌گوید.

!!! note

    uv بین نسخه‌های Python نصب‌شده توسط سیستم‌عامل و نسخه‌هایی که توسط ابزارهای دیگر نصب و مدیریت می‌شوند تمایزی قائل نمی‌شود. برای مثال، اگر یک نصب Python با `pyenv` مدیریت شود، در uv همچنان یک _system_ Python version محسوب می‌شود.

## درخواست یک نسخه {#requesting-a-version}

می‌توان یک Python version مشخص را با فلگ `--python` در بیشتر دستورهای uv درخواست کرد. برای مثال، هنگام ساخت یک virtual environment:

```console
$ uv venv --python 3.11.6
```

uv مطمئن می‌شود که Python 3.11.6 در دسترس است — و در صورت نیاز آن را دانلود و نصب می‌کند — و سپس virtual environment را با آن می‌سازد.

قالب‌های زیر برای درخواست Python version پشتیبانی می‌شوند:

- `<version>` (e.g., `3`, `3.12`, `3.12.3`)
- `<version-specifier>` (e.g., `>=3.12,<3.13`)
- `<version><short-variant>` (e.g., `3.13t`, `3.12.0d`)
- `<version>+<variant>` (e.g., `3.13+freethreaded`, `3.12.0+debug`, `3.14+gil`)
- `<implementation>` (e.g., `cpython` or `cp`)
- `<implementation>@<version>` (e.g., `cpython@3.12`)
- `<implementation><version>` (e.g., `cpython3.12` or `cp312`)
- `<implementation><version-specifier>` (e.g., `cpython>=3.12,<3.13`)
- `<implementation>-<version>-<os>-<arch>-<libc>` (e.g., `cpython-3.12.3-macos-aarch64-none`)

علاوه بر این، می‌توان یک system Python interpreter مشخص را با موارد زیر درخواست کرد:

- `<executable-path>` (e.g., `/opt/homebrew/bin/python3`)
- `<executable-name>` (e.g., `mypython3`)
- `<install-dir>` (e.g., `/some/environment/`)

از [`UV_PYTHON_ARCH`](../reference/environment.md#uv_python_arch) برای انتخاب یک معماری مستقل از Python version استفاده کنید. برای مثال، تنظیم `UV_PYTHON_ARCH=x86_64` باعث می‌شود `uv sync` از یک interpreter با معماری `x86_64` استفاده کند و در عین حال درخواست Python version پروژه را رعایت کند. درخواستی که شامل یک معماری باشد یا نام یک فایل اجرایی interpreter را ببرد، اولویت دارد.

به‌طور پیش‌فرض، اگر نسخه‌های Python در سیستم پیدا نشوند، uv آن‌ها را به‌صورت خودکار دانلود می‌کند. می‌توان این رفتار را [با گزینهٔ `python-downloads` غیرفعال کرد](#disabling-automatic-python-downloads).

### فایل‌های Python version {#python-version-files}

از فایل `.python-version` می‌توان برای ایجاد یک درخواست پیش‌فرض Python version استفاده کرد. uv در پوشهٔ کاری و هر یک از پوشه‌های والد آن به دنبال فایل `.python-version` می‌گردد. اگر فایلی پیدا نشود، uv پوشهٔ پیکربندی سطح کاربر را بررسی می‌کند. می‌توان از هر یک از قالب‌های درخواست که در بالا توضیح داده شد استفاده کرد، هرچند برای سازگاری با ابزارهای دیگر استفاده از شمارهٔ نسخه توصیه می‌شود.

می‌توان با دستور [`uv python pin`](../reference/cli.md#uv-python-pin) یک فایل `.python-version` در پوشهٔ جاری ساخت.

می‌توان با دستور [`uv python pin --global`](../reference/cli.md#uv-python-pin) یک فایل `.python-version` سراسری در پوشهٔ پیکربندی کاربر ساخت.

می‌توان کشف فایل‌های `.python-version` را با `--no-config` غیرفعال کرد.

uv فراتر از مرزهای project یا workspace به دنبال فایل‌های `.python-version` نمی‌گردد (به‌جز پوشهٔ پیکربندی کاربر).

## نصب یک Python version {#installing-a-python-version}

uv فهرستی از distributionهای قابل دانلود CPython و PyPy را برای macOS و Linux و Windows در خود دارد.

!!! tip

    به‌طور پیش‌فرض، نسخه‌های Python در صورت نیاز به‌صورت خودکار دانلود می‌شوند، بدون استفاده از `uv python install`.

برای نصب یک Python version در یک نسخهٔ مشخص:

```console
$ uv python install 3.12.3
```

برای نصب آخرین نسخهٔ patch:

```console
$ uv python install 3.12
```

برای نصب نسخه‌ای که constraintها را برآورده کند:

```console
$ uv python install '>=3.8,<3.10'
```

برای نصب چند نسخه:

```console
$ uv python install 3.9 3.10 3.11
```

برای نصب یک پیاده‌سازی مشخص:

```console
$ uv python install pypy
```

همهٔ قالب‌های [درخواست Python version](#requesting-a-version) پشتیبانی می‌شوند، به‌جز قالب‌هایی که برای درخواست interpreterهای محلی استفاده می‌شوند، مانند مسیر فایل.

به‌طور پیش‌فرض، `uv python install` بررسی می‌کند که یک managed Python version نصب شده باشد یا آخرین نسخه را نصب می‌کند. اگر فایل `.python-version` وجود داشته باشد، uv نسخهٔ Python فهرست‌شده در فایل را نصب می‌کند. پروژه‌ای که به چند Python version نیاز دارد می‌تواند یک فایل `.python-versions` تعریف کند. اگر این فایل وجود داشته باشد، uv همهٔ نسخه‌های Python فهرست‌شده در فایل را نصب می‌کند.

!!! important

    نسخه‌های Python موجود برای هر انتشار uv ثابت هستند. برای نصب نسخه‌های جدید Python، ممکن است لازم باشد uv را ارتقا دهید.

برای جزئیات دربارهٔ محل ذخیرهٔ نسخه‌های نصب‌شدهٔ Python، به [مستندات storage](../reference/storage.md#python-versions) مراجعه کنید.

### نصب فایل‌های اجرایی Python {#installing-python-executables}

uv به‌طور پیش‌فرض فایل‌های اجرایی Python را در `PATH` شما نصب می‌کند؛ برای مثال، در Unix دستور `uv python install 3.12` یک فایل اجرایی Python را در `~/.local/bin` نصب می‌کند، مثلاً با نام `python3.12`. برای جزئیات بیشتر دربارهٔ پوشهٔ مقصد، به [مستندات storage](../reference/storage.md#python-executables) مراجعه کنید.

!!! tip

    اگر `~/.local/bin` در `PATH` شما نیست، می‌توانید آن را با `uv python update-shell` اضافه کنید.

برای نصب فایل‌های اجرایی `python` و `python3`، گزینهٔ آزمایشی `--default` را اضافه کنید:

```console
$ uv python install 3.12 --default
```

هنگام نصب فایل‌های اجرایی Python، uv یک فایل اجرایی موجود را فقط در صورتی بازنویسی می‌کند که توسط uv مدیریت شود — برای مثال، اگر `~/.local/bin/python3.12` از قبل وجود داشته باشد، uv بدون فلگ `--force` آن را بازنویسی نمی‌کند.

uv فایل‌های اجرایی‌ای را که مدیریت می‌کند به‌روزرسانی می‌کند. با این حال، به‌طور پیش‌فرض آخرین نسخهٔ patch هر نسخهٔ فرعی Python را ترجیح می‌دهد. برای مثال:

```console
$ uv python install 3.12.7  # Adds `python3.12` to `~/.local/bin`
$ uv python install 3.12.6  # Does not update `python3.12`
$ uv python install 3.12.8  # Updates `python3.12` to point to 3.12.8
```

## ارتقای نسخه‌های Python {#upgrading-python-versions}

!!! important

    ارتقاها فقط برای نسخه‌های Python مدیریت‌شده توسط uv پشتیبانی می‌شوند.

    ارتقاها در حال حاضر برای PyPy و GraalPy و Pyodide پشتیبانی نمی‌شوند.

uv امکان ارتقای شفاف نسخه‌های Python به آخرین نسخهٔ patch را فراهم می‌کند، مثلاً از 3.13.4 به 3.13.5. uv ارتقای شفاف بین نسخه‌های فرعی Python، مثلاً از 3.12 به 3.13، را مجاز نمی‌داند، زیرا تغییر نسخهٔ فرعی می‌تواند بر resolution وابستگی‌ها تأثیر بگذارد.

نسخه‌های Python مدیریت‌شده توسط uv را می‌توان با دستور `python upgrade` به آخرین نسخهٔ patch پشتیبانی‌شده ارتقا داد:

برای ارتقای یک Python version به آخرین نسخهٔ patch پشتیبانی‌شده:

```console
$ uv python upgrade 3.12
```

برای ارتقای همهٔ نسخه‌های نصب‌شدهٔ Python:

```console
$ uv python upgrade
```

پس از ارتقا، uv نسخهٔ جدید را ترجیح می‌دهد، اما نسخهٔ موجود را حفظ می‌کند، زیرا ممکن است همچنان توسط virtual environmentها استفاده شود.

virtual environmentهایی که از آن Python version استفاده می‌کنند، به‌صورت خودکار به نسخهٔ patch جدید ارتقا پیدا می‌کنند.

اگر یک virtual environment با یک نسخهٔ patch که صراحتاً درخواست شده ساخته شده باشد، مثلاً `uv venv -p 3.10.8`، به‌صورت شفاف به نسخهٔ جدید ارتقا پیدا نمی‌کند.

### پوشه‌های نسخهٔ فرعی {#minor-version-directories}

ارتقاهای خودکار برای virtual environmentها با استفاده از یک پوشه با نسخهٔ فرعی Python پیاده‌سازی شده‌اند، مثلاً:

```
~/.local/share/uv/python/cpython-3.12-macos-aarch64-none
```

که یک symbolic link (در Unix) یا junction (در Windows) است و به یک نسخهٔ patch مشخص اشاره می‌کند:

```console
$ readlink ~/.local/share/uv/python/cpython-3.12-macos-aarch64-none
~/.local/share/uv/python/cpython-3.12.11-macos-aarch64-none
```

اگر این لینک توسط ابزار دیگری resolve شود، مثلاً با canonical کردن مسیر Python interpreter، و برای ساخت یک virtual environment استفاده شود، به‌صورت خودکار ارتقا پیدا نمی‌کند.

## نسخه‌های Python پروژه {#project-python-versions}

uv هنگام اجرای دستورهای project به نیازمندی‌های Python تعریف‌شده در `requires-python` در فایل `pyproject.toml` احترام می‌گذارد. اولین Python version سازگار با آن نیازمندی استفاده می‌شود، مگر اینکه نسخه‌ای به شکل دیگری درخواست شده باشد، مثلاً از طریق فایل `.python-version` یا فلگ `--python`.

## مشاهدهٔ نسخه‌های Python موجود {#viewing-available-python-versions}

برای فهرست کردن نسخه‌های Python نصب‌شده و موجود:

```console
$ uv python list
```

برای فیلتر کردن نسخه‌های Python، یک درخواست بدهید؛ مثلاً برای نمایش همهٔ interpreterهای Python 3.13:

```console
$ uv python list 3.13
```

یا برای نمایش همهٔ interpreterهای PyPy:

```console
$ uv python list pypy
```

به‌طور پیش‌فرض، دانلودها برای platformهای دیگر و نسخه‌های patch قدیمی پنهان هستند.

برای مشاهدهٔ همهٔ نسخه‌ها:

```console
$ uv python list --all-versions
```

برای مشاهدهٔ نسخه‌های Python برای platformهای دیگر:

```console
$ uv python list --all-platforms
```

برای حذف دانلودها و نمایش فقط نسخه‌های نصب‌شدهٔ Python:

```console
$ uv python list --only-installed
```

برای جزئیات بیشتر به مرجع [`uv python list`](../reference/cli.md#uv-python-list) مراجعه کنید.

## یافتن یک فایل اجرایی Python {#finding-a-python-executable}

برای یافتن یک فایل اجرایی Python، از دستور `uv python find` استفاده کنید:

```console
$ uv python find
```

به‌طور پیش‌فرض، این دستور مسیر اولین فایل اجرایی Python موجود را نمایش می‌دهد. برای جزئیات دربارهٔ نحوهٔ کشف فایل‌های اجرایی، به [قواعد کشف](#discovery-of-python-versions) مراجعه کنید.

این رابط همچنین از بسیاری از [قالب‌های درخواست](#requesting-a-version) پشتیبانی می‌کند؛ مثلاً برای یافتن یک فایل اجرایی Python با نسخهٔ 3.11 یا جدیدتر:

```console
$ uv python find '>=3.11'
```

به‌طور پیش‌فرض، `uv python find` نسخه‌های Python از virtual environmentها را هم شامل می‌شود. اگر یک پوشهٔ `.venv` در پوشهٔ کاری یا هر یک از پوشه‌های والد آن پیدا شود یا environment variable مربوط به `VIRTUAL_ENV` تنظیم شده باشد، نسبت به هر فایل اجرایی Python در `PATH` اولویت دارد.

برای نادیده گرفتن virtual environmentها، از فلگ `--system` استفاده کنید:

```console
$ uv python find --system
```

## کشف نسخه‌های Python {#discovery-of-python-versions}

هنگام جستجو برای یک Python version، مکان‌های زیر بررسی می‌شوند:

- نصب‌های managed Python در `UV_PYTHON_INSTALL_DIR`.
- یک Python interpreter در `PATH` با نام `python`، `python3` یا `python3.x` در macOS و Linux، یا `python.exe` در Windows.
- در Windows، interpreterهای Python در رجیستری Windows و interpreterهای Python فروشگاه Microsoft (به `py --list-paths` مراجعه کنید) که با نسخهٔ درخواستی مطابقت دارند.

در برخی موارد، uv اجازهٔ استفاده از یک Python version از یک virtual environment را می‌دهد. در این حالت، interpreter مربوط به virtual environment پیش از جستجو برای یک نصب، مطابق توضیحات بالا، از نظر سازگاری با درخواست بررسی می‌شود. برای جزئیات به مستندات [کشف virtual environment سازگار با pip](../pip/environments.md#discovery-of-python-environments) مراجعه کنید.

هنگام کشف، فایل‌های غیرقابل اجرا نادیده گرفته می‌شوند. از هر فایل اجرایی کشف‌شده metadata پرسیده می‌شود تا مطمئن شوند که [Python version درخواستی](#requesting-a-version) را برآورده می‌کند. اگر پرس‌وجو ناموفق باشد، آن فایل اجرایی رد می‌شود. اگر فایل اجرایی درخواست را برآورده کند، بدون بررسی فایل‌های اجرایی دیگر استفاده می‌شود.

هنگام جستجو برای یک managed Python version، uv ابتدا نسخه‌های جدیدتر را ترجیح می‌دهد. هنگام جستجو برای یک system Python version، uv از اولین نسخهٔ سازگار استفاده می‌کند — نه جدیدترین نسخه.

اگر یک Python version در سیستم پیدا نشود، uv دانلود یک managed Python version سازگار را بررسی می‌کند.

## پیش‌انتشارهای Python {#python-pre-releases}

پیش‌انتشارهای Python به‌طور پیش‌فرض انتخاب نمی‌شوند. پیش‌انتشارهای Python در صورتی استفاده می‌شوند که نصب دیگری مطابق با درخواست در دسترس نباشد. برای مثال، اگر فقط یک نسخهٔ پیش‌انتشار در دسترس باشد، همان استفاده می‌شود، اما در غیر این صورت یک نسخهٔ انتشار پایدار استفاده می‌شود. به همین ترتیب، اگر مسیر یک فایل اجرایی Python پیش‌انتشار داده شود، هیچ Python version دیگری با درخواست مطابقت ندارد و نسخهٔ پیش‌انتشار استفاده می‌شود.

اگر یک Python version پیش‌انتشار در دسترس باشد و با درخواست مطابقت داشته باشد، uv به‌جای آن یک Python version پایدار دانلود نمی‌کند.

## Python free-threaded {#free-threaded-python}

uv از کشف و نصب گونه‌های [free-threaded](https://docs.python.org/3.14/glossary.html#term-free-threading) Python در CPython 3.13+ پشتیبانی می‌کند.

برای Python 3.13، نسخه‌های free-threaded Python به‌طور پیش‌فرض انتخاب نمی‌شوند. نسخه‌های free-threaded Python فقط زمانی انتخاب می‌شوند که صراحتاً درخواست شده باشند، مثلاً با `3.13t` یا `3.13+freethreaded`.

برای Python 3.14+، uv امکان استفاده از interpreterهای free-threaded Python 3.14+ را بدون انتخاب صریح فراهم می‌کند. بیلد Python با GIL فعال همچنان ترجیح داده می‌شود، مثلاً هنگام نصب با `uv python install 3.14`. با این حال، مثلاً اگر یک interpreter free-threaded در `PATH` پیش از یک بیلد با GIL فعال قرار گیرد، همان استفاده می‌شود.

اگر هم نسخه‌های free-threaded و هم نسخه‌های GIL-enabled Python در سیستم در دسترس باشند، می‌توانید از variant specifier مربوط به `+gil` برای الزام گونهٔ GIL-enabled در یک project استفاده کنید.

## گونه‌های دیباگ Python {#debug-python-variants}

uv از کشف و نصب [بیلدهای دیباگ](https://docs.python.org/3.14/using/configure.html#debug-build) Python پشتیبانی می‌کند، یعنی بیلدهایی که assertionهای دیباگ در آن‌ها فعال است.

!!! important

    بیلدهای دیباگ Python کندتر هستند و برای استفادهٔ عمومی مناسب نیستند.

بیلدهای دیباگ در صورتی استفاده می‌شوند که نصب دیگری مطابق با درخواست در دسترس نباشد. برای مثال، اگر فقط یک نسخهٔ دیباگ در دسترس باشد، همان استفاده می‌شود، اما در غیر این صورت یک نسخهٔ انتشار پایدار استفاده می‌شود. به همین ترتیب، اگر مسیر یک فایل اجرایی دیباگ Python داده شود، هیچ Python version دیگری با درخواست مطابقت ندارد و نسخهٔ دیباگ استفاده می‌شود.

بیلدهای دیباگ Python را می‌توان صراحتاً درخواست کرد، مثلاً با `3.13d` یا `3.13+debug`.

!!! note

    نسخه‌های CPython نصب‌شده توسط uv معمولاً برای کاهش اندازهٔ distribution، نمادهای دیباگ را حذف کرده‌اند. این بیلدهای دیباگ نمادهای دیباگ را حذف نکرده‌اند، که می‌تواند هنگام دیباگ کردن فرایندهای Python با یک دیباگر سطح C مفید باشد.

## غیرفعال کردن دانلود خودکار Python {#disabling-automatic-python-downloads}

به‌طور پیش‌فرض، uv در صورت نیاز نسخه‌های Python را به‌صورت خودکار دانلود می‌کند.

از گزینهٔ [`python-downloads`](../reference/settings.md#python-downloads) می‌توان برای غیرفعال کردن این رفتار استفاده کرد. به‌طور پیش‌فرض، مقدار آن `automatic` است؛ آن را روی `manual` تنظیم کنید تا دانلودهای Python فقط هنگام `uv python install` مجاز باشند.

!!! tip

    می‌توان تنظیم `python-downloads` را در یک [فایل پیکربندی ماندگار](./configuration-files.md) تنظیم کرد تا رفتار پیش‌فرض تغییر کند، یا فلگ `--no-python-downloads` را به هر دستور uv پاس داد.

## الزام یا غیرفعال کردن نسخه‌های managed Python {#requiring-or-disabling-managed-python-versions}

به‌طور پیش‌فرض، uv تلاش می‌کند از نسخه‌های Python موجود در سیستم استفاده کند و فقط در صورت نیاز نسخه‌های managed Python را دانلود کند. برای نادیده گرفتن نسخه‌های system Python و استفادهٔ فقط از نسخه‌های managed Python، از فلگ `--managed-python` استفاده کنید:

```console
$ uv python list --managed-python
```

به همین ترتیب، برای نادیده گرفتن نسخه‌های managed Python و استفادهٔ فقط از نسخه‌های system Python، از فلگ `--no-managed-python` استفاده کنید:

```console
$ uv python list --no-managed-python
```

برای تغییر رفتار پیش‌فرض uv در یک فایل پیکربندی، از [تنظیم `python-preference`](#adjusting-python-version-preferences) استفاده کنید.

## تنظیم ترجیحات Python version {#adjusting-python-version-preferences}

تنظیم [`python-preference`](../reference/settings.md#python-preference) تعیین می‌کند که آیا نصب‌های Python موجود در سیستم ترجیح داده شوند یا نصب‌هایی که توسط uv دانلود و نصب می‌شوند.

به‌طور پیش‌فرض، `python-preference` روی `managed` تنظیم شده است که نصب‌های managed Python را بر نصب‌های system Python ترجیح می‌دهد. با این حال، نصب‌های system Python همچنان بر دانلود یک managed Python version ترجیح داده می‌شوند.

گزینه‌های جایگزین زیر در دسترس هستند:

- `only-managed`: فقط از نصب‌های managed Python استفاده کنید؛ هرگز از نصب‌های system Python استفاده نکنید. معادل `--managed-python`.
- `system`: نصب‌های system Python را بر نصب‌های managed Python ترجیح دهید.
- `only-system`: فقط از نصب‌های system Python استفاده کنید؛ هرگز از نصب‌های managed Python استفاده نکنید. معادل `--no-managed-python`.

!!! note

    دانلودهای خودکار Python version را می‌توان [غیرفعال کرد](#disabling-automatic-python-downloads) بدون اینکه ترجیح تغییر کند.

## پشتیبانی از پیاده‌سازی‌های Python {#python-implementation-support}

uv از پیاده‌سازی‌های CPython و PyPy و Pyodide و GraalPy Python پشتیبانی می‌کند. اگر یک پیاده‌سازی Python پشتیبانی نشود، uv در کشف interpreter آن ناموفق خواهد بود.

می‌توان پیاده‌سازی‌ها را با نام بلند یا کوتاه درخواست کرد:

- CPython: `cpython`, `cp`
- PyPy: `pypy`, `pp`
- GraalPy: `graalpy`, `gp`
- Pyodide: `pyodide`

درخواست‌های نام پیاده‌سازی به بزرگی و کوچکی حروف حساس نیستند.

برای جزئیات بیشتر دربارهٔ قالب‌های پشتیبانی‌شده، به مستندات [درخواست Python version](#requesting-a-version) مراجعه کنید.

## توزیع‌های managed Python {#managed-python-distributions}

uv از دانلود و نصب distributionهای CPython و PyPy و GraalPy و Pyodide پشتیبانی می‌کند.

### توزیع‌های CPython {#cpython-distributions}

از آنجا که Python باینری‌های CPython قابل توزیع رسمی منتشر نمی‌کند، uv در عوض از distributionهای از پیش ساخته‌شدهٔ پروژهٔ [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) مربوط به Astral استفاده می‌کند. `python-build-standalone` در بسیاری از پروژه‌های دیگر Python هم استفاده می‌شود، مانند [Mise](https://mise.jdx.dev/lang/python.html) و [bazelbuild/rules_python](https://github.com/bazelbuild/rules_python).

توزیع‌های Python مربوط به uv خودکفا، بسیار قابل‌حمل و کارآمد هستند. هرچند Python را می‌توان از source ساخت، مانند ابزارهایی چون `pyenv`، انجام این کار به dependencyهای سیستمی از پیش نصب‌شده نیاز دارد و ساخت بیلدهای بهینه و کارآمد (مثلاً با فعال بودن PGO و LTO) بسیار کند است.

این distributionها برخی رفتارهای خاص دارند، که عموماً نتیجهٔ قابل‌حمل بودن آن‌هاست؛ برای جزئیات به مستندات [ویژگی‌های خاص `python-build-standalone`](https://gregoryszorc.com/docs/python-build-standalone/main/quirks.html) مراجعه کنید.

### توزیع‌های PyPy {#pypy-distributions}

!!! note

    نسخه‌های PyPy از CPython عقب هستند و در حال حاضر فقط از نسخه‌های Python تا 3.11 پشتیبانی می‌کنند.

توزیع‌های PyPy توسط [پروژهٔ PyPy](https://pypy.org) ارائه می‌شوند.

### توزیع‌های Pyodide {#pyodide-distributions}

توزیع‌های Pyodide توسط [پروژهٔ Pyodide](https://github.com/pyodide/pyodide) ارائه می‌شوند.

Pyodide یک پورت از CPython برای پلتفرم WebAssembly / Emscripten است.

## شبیه‌سازی شفاف x86_64 روی aarch64 {#transparent-x8664-emulation-on-aarch64}

هم macOS و هم Windows اجرای باینری‌های x86_64 روی aarch64 را از طریق شبیه‌سازی شفاف پشتیبانی می‌کنند. این قابلیت [Rosetta 2](https://support.apple.com/en-gb/102527) یا [شبیه‌سازی Windows on ARM (WoA)](https://learn.microsoft.com/en-us/windows/arm/apps-on-arm-x86-emulation) نامیده می‌شود. می‌توان از uv با معماری x86_64 روی aarch64 استفاده کرد و همچنین می‌توان از یک Python interpreter با معماری x86_64 روی aarch64 استفاده کرد. هر باینری uv می‌تواند از هر Python interpreter استفاده کند، اما یک Python interpreter به packageهایی برای معماری خودش نیاز دارد، یا همه x86_64 یا همه aarch64.

برای اجبار استفاده از Python شبیه‌سازی‌شدهٔ `x86_64`، [`UV_PYTHON_ARCH`](../reference/environment.md#uv_python_arch) را روی `x86_64` تنظیم کنید یا [یک معماری صریح درخواست کنید](#requesting-a-version)، مثلاً `cpython-3.14-windows-x86_64`.

## ثبت در رجیستری Windows {#registration-in-the-windows-registry}

در Windows، نصب نسخه‌های managed Python آن‌ها را طبق [PEP 514](https://peps.python.org/pep-0514/) در رجیستری Windows ثبت می‌کند.

پس از نصب، می‌توان نسخه‌های Python را با launcher مربوط به `py` انتخاب کرد، مثلاً:

```console
$ uv python install 3.13.1
$ py -V:Astral/CPython3.13.1
```

هنگام حذف نصب، uv ورودی رجیستری برای نسخهٔ هدف و همچنین هر ورودی رجیستری خراب را حذف می‌کند.
