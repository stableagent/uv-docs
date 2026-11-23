# استفاده از environment‌های Python {#using-python-environments}

هر نصب Python یک environment دارد که هنگام استفاده از Python فعال است. می‌توان package‌ها را در یک environment نصب کرد تا module‌های آن‌ها از اسکریپت‌های Python شما در دسترس باشند. به‌طور کلی، بهترین روش این است که environment نصب Python را تغییر ندهید. این موضوع به‌ویژه برای نصب‌های Python که همراه سیستم‌عامل عرضه می‌شوند اهمیت دارد، زیرا آن‌ها اغلب خودشان package‌ها را مدیریت می‌کنند. یک virtual environment روشی سبک برای جداسازی package‌ها از environment نصب Python است. برخلاف `pip`، uv به‌صورت پیش‌فرض نیازمند استفاده از یک virtual environment است.

## ساخت یک virtual environment {#creating-a-virtual-environment}

uv از ساخت virtual environment‌ها پشتیبانی می‌کند؛ مثلاً برای ساخت یک virtual environment در مسیر `.venv`:

```console
$ uv venv
```

می‌توان یک نام یا مسیر مشخص را تعیین کرد؛ مثلاً برای ساخت یک virtual environment در مسیر `my-name`:

```console
$ uv venv my-name
```

می‌توان یک Python version را درخواست کرد؛ مثلاً برای ساخت یک virtual environment با Python 3.11:

```console
$ uv venv --python 3.11
```

توجه کنید که این کار نیازمند در دسترس بودن Python version درخواستی در سیستم است. با این حال، اگر در دسترس نباشد، uv برای شما Python را دانلود می‌کند. برای جزئیات بیشتر، به مستندات [Python version](../concepts/python-versions.md) مراجعه کنید.

## استفاده از یک virtual environment {#using-a-virtual-environment}

هنگام استفاده از نام پیش‌فرض virtual environment، uv در فراخوانی‌های بعدی به‌صورت خودکار virtual environment را پیدا و از آن استفاده می‌کند.

```console
$ uv venv

$ # Install a package in the new virtual environment
$ uv pip install ruff
```

می‌توان virtual environment را «فعال» کرد تا package‌های آن در دسترس قرار گیرند:

=== "macOS and Linux"

    ```console
    $ source .venv/bin/activate
    ```

=== "Windows"

    ```pwsh-session
    PS> .venv\Scripts\activate
    ```

!!! note

    اسکریپت فعال‌سازی پیش‌فرض در Unix برای shell‌های سازگار با POSIX مانند `sh`، `bash` یا `zsh` است. اسکریپت‌های فعال‌سازی دیگری نیز برای shell‌های جایگزین رایج وجود دارد.

    === "fish"

        ```console
        $ source .venv/bin/activate.fish
        ```

    === "csh / tcsh"


        ```console
        $ source .venv/bin/activate.csh
        ```

    === "Nushell"

        ```console
        $ use .venv\Scripts\activate.nu
        ```

## غیرفعال کردن یک environment {#deactivating-an-environment}

برای خروج از یک virtual environment، از دستور `deactivate` استفاده کنید:

```console
$ deactivate
```

## استفاده از environment‌های دلخواه Python {#using-arbitrary-python-environments}

از آنجا که uv هیچ dependencyای به Python ندارد، می‌تواند در virtual environment‌هایی غیر از virtual environment خودش نصب کند. برای مثال، تنظیم `VIRTUAL_ENV=/path/to/venv` باعث می‌شود uv در مسیر `/path/to/venv` نصب کند، صرف‌نظر از اینکه uv کجا نصب شده است. توجه کنید که اگر `VIRTUAL_ENV` به پوشه‌ای تنظیم شود که یک virtual environment **مطابق** [PEP 405](https://peps.python.org/pep-0405/#specification) **نیست**، نادیده گرفته می‌شود.

uv همچنین می‌تواند با گزینهٔ `--python` در environment‌های دلخواه، حتی environment‌های غیرمجازی، نصب کند. برای مثال، `uv pip install --python /path/to/python` در environment مرتبط با interpreter مسیر `/path/to/python` نصب می‌کند، صرف‌نظر از اینکه virtual environment باشد یا نه. گزینهٔ `--python` یک مسیر به پوشهٔ ریشهٔ یک virtual environment را نیز می‌پذیرد.

برای راحتی، `uv pip install --system` در environment سیستم Python نصب می‌کند. استفاده از `--system` تقریباً معادل `uv pip install --python $(which python)` است، اما توجه کنید که فایل‌های اجرایی مرتبط با virtual environment‌ها نادیده گرفته می‌شوند. اگرچه معمولاً توصیه می‌کنیم برای مدیریت dependency‌ها از virtual environment‌ها استفاده کنید، `--system` در environment‌های یکپارچه‌سازی مداوم و کانتینری مناسب است.

از فلگ `--system` همچنین برای اعلام موافقت با تغییر environment‌های سیستم استفاده می‌شود. برای مثال، می‌توان از آرگومان `--python` برای درخواست یک Python version استفاده کرد (مثلاً `--python 3.12`) و uv به دنبال interpreterای می‌گردد که با درخواست مطابقت داشته باشد. اگر uv یک interpreter سیستمی پیدا کند (مثلاً `/usr/lib/python3.12`)، آنگاه فلگ `--system` برای اجازهٔ تغییر این environment غیرمجازی Python لازم است. بدون فلگ `--system`، uv همهٔ interpreter‌هایی را که در virtual environment‌ها نیستند نادیده می‌گیرد. برعکس، وقتی فلگ `--system` داده شود، uv همهٔ interpreter‌هایی را که _در_ virtual environment‌ها هستند نادیده می‌گیرد.

نصب در Python سیستم در platform‌ها و distribution‌های مختلف به‌طور مشهوری دشوار است. uv از موارد رایج پشتیبانی می‌کند، اما در همهٔ موارد کار نمی‌کند. برای مثال، نصب در Python سیستم در Debian پیش از Python 3.10 به دلیل [وصلهٔ distribution بر `distutils` (اما نه `sysconfig`)](https://ffy00.github.io/blog/02-python-debian-and-the-install-locations/) پشتیبانی نمی‌شود. اگرچه ما همیشه استفاده از virtual environment‌ها را توصیه می‌کنیم، uv در این environment‌های غیراستاندارد آن‌ها را الزامی می‌داند.

اگر uv در یک environment Python نصب شده باشد، مثلاً با `pip`، باز هم می‌توان از آن برای تغییر environment‌های دیگر استفاده کرد. با این حال، هنگام فراخوانی با `python -m uv`، uv به‌صورت پیش‌فرض از environment interpreter والد استفاده می‌کند. فراخوانی uv از طریق Python سربار راه‌اندازی اضافه می‌کند و برای استفادهٔ عمومی توصیه نمی‌شود.

خود uv به Python وابسته نیست، اما برای (۱) نصب dependency‌ها در environment و (۲) build کردن distribution‌های منبع، نیاز دارد یک environment Python را پیدا کند.

## کشف environment‌های Python {#discovery-of-python-environments}

هنگام اجرای دستوری که یک environment را تغییر می‌دهد، مانند `uv pip sync` یا `uv pip install`، uv به ترتیب زیر به دنبال یک virtual environment می‌گردد:

- یک virtual environment فعال‌شده بر اساس environment variable به نام `VIRTUAL_ENV`.
- یک environment فعال‌شدهٔ Conda بر اساس environment variable به نام `CONDA_PREFIX`.
- یک virtual environment در مسیر `.venv` در پوشهٔ فعلی یا در نزدیک‌ترین پوشهٔ والد.

اگر هیچ virtual environmentای پیدا نشد، uv از کاربر می‌خواهد که یکی را در پوشهٔ فعلی با `uv venv` بسازد.

اگر فلگ `--system` داده شود، uv virtual environment‌ها را نادیده می‌گیرد و به دنبال یک Python version نصب‌شده می‌گردد. به همین ترتیب، هنگام اجرای دستوری که environment را تغییر نمی‌دهد، مانند `uv pip compile`، uv به virtual environment _نیاز ندارد_ — با این حال، هنوز به یک interpreter Python نیاز است. برای جزئیات دربارهٔ کشف Python version‌های نصب‌شده، به مستندات [کشف Python](../concepts/python-versions.md#discovery-of-python-versions) مراجعه کنید.
