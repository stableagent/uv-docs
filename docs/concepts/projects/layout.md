# ساختار و فایل‌های project {#project-structure-and-files}

## فایل `pyproject.toml` {#the-pyprojecttoml}

metadata مربوط به project Python در فایل
[`pyproject.toml`](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) تعریف می‌شود. uv
برای شناسایی پوشهٔ ریشهٔ project به این فایل نیاز دارد.

!!! tip

    می‌توان از `uv init` برای ساخت project جدید استفاده کرد. برای
    جزئیات بیشتر، به [ساخت projectها](./init.md) مراجعه کنید.

تعریف حداقلی project شامل نام و نسخه است:

```toml title="pyproject.toml"
[project]
name = "example"
version = "0.1.0"
```

metadata و پیکربندی بیشتر project شامل موارد زیر است:

- [نیازمندی Python version](./config.md#python-version-requirement)
- [dependencyها](./dependencies.md)
- [build system](./config.md#build-systems)
- [entry pointها (دستورها)](./config.md#entry-points)

## project environment {#the-project-environment}

هنگام کار روی project با uv، uv در صورت نیاز یک virtual environment می‌سازد. در حالی که برخی دستورهای uv یک environment موقت می‌سازند (مثلاً `uv run --isolated`)، uv همچنین یک environment پایدار را با project و dependencyهای آن در پوشهٔ `.venv` در کنار فایل `pyproject.toml` مدیریت می‌کند. به‌طور پیش‌فرض، این environment داخل project ذخیره می‌شود تا ویرایشگرها به‌راحتی آن را پیدا کنند — آن‌ها برای ارائهٔ تکمیل کد و راهنمایی نوع (type hint) به environment نیاز دارند. توصیه نمی‌شود پوشهٔ `.venv` را در کنترل نسخه (version control) قرار دهید؛ این پوشه به‌طور خودکار با یک فایل `.gitignore` داخلی از `git` مستثنا شده است.

برای اجرای یک دستور در project environment، از `uv run` استفاده کنید. همچنین می‌توان project environment را مانند یک virtual environment معمولی فعال کرد.

هنگامی که `uv run` فراخوانی می‌شود، اگر project environment هنوز وجود نداشته باشد آن را می‌سازد یا اگر وجود داشته باشد، از به‌روز بودن آن اطمینان حاصل می‌کند. project environment را می‌توان به‌صورت صریح با `uv sync` نیز ساخت. برای جزئیات بیشتر، به مستندات [قفل‌کردن و همگام‌سازی](./sync.md) مراجعه کنید.

تغییر project environment به‌صورت دستی، مثلاً با `uv pip install`، _توصیه نمی‌شود_. برای dependencyهای project، از `uv add` استفاده کنید تا package به environment اضافه شود. برای نیازمندی‌های یک‌باره، از [`uvx`](../../guides/tools.md) یا
[`uv run --with`](./run.md#requesting-additional-dependencies) استفاده کنید.

!!! tip

    اگر نمی‌خواهید uv، project environment را مدیریت کند، [`managed = false`](../../reference/settings.md#managed) را تنظیم کنید
    تا قفل‌کردن و همگام‌سازی خودکار project غیرفعال شود. برای مثال:

    ```toml title="pyproject.toml"
    [tool.uv]
    managed = false
    ```

### environmentهای متمرکز project {#centralized-project-environments}

با قابلیت پیش‌نمایش [`centralized-project-envs`](../preview.md)، uv محیط پیش‌فرض project environment را در cache خود ذخیره می‌کند. uv تلاش می‌کند یک پیوند (link) پوشهٔ `.venv` به environment ذخیره‌شده در cache را حفظ کند تا گردش‌کارهای فعال‌سازی و ویرایشگر موجود بتوانند به استفاده از مسیر معمول ادامه دهند. اگر ساخت پیوند ناموفق باشد، uv تلاش می‌کند مسیر environment ذخیره‌شده در cache را به‌جای آن در `.venv` بنویسد. اگر هر دو تلاش ناموفق باشند، uv مستقیماً به استفاده از environment ذخیره‌شده در cache ادامه می‌دهد، اما ابزارهایی که به `.venv` وابسته‌اند ممکن است آن را پیدا نکنند. تغییر interpreter، environmentهای جداگانهٔ ذخیره‌شده در cache را انتخاب می‌کند و می‌توان بعداً دوباره از آن‌ها استفاده کرد.

مسیرهای صریح project environment، از جمله `UV_PROJECT_ENVIRONMENT` و environmentهایی که با `--active` انتخاب شده‌اند، متمرکز نمی‌شوند. این قابلیت وقتی `--no-cache` فعال باشد، اثری ندارد.

این قابلیت همچنین برای فراخوانی‌های بدون مسیر `uv venv` از ریشهٔ project یا workspace اعمال می‌شود.

## lockfile {#the-lockfile}

uv یک فایل `uv.lock` در کنار فایل `pyproject.toml` می‌سازد.

`uv.lock` یک lockfile _جهانی_ یا _چندسکویی_ (cross-platform) است که packageهایی را که در همهٔ markerهای ممکن Python، مانند سیستم‌عامل، معماری و Python version، نصب می‌شدند، ثبت می‌کند.

برخلاف `pyproject.toml` که برای مشخص‌کردن نیازمندی‌های کلی project شما استفاده می‌شود، lockfile شامل نسخه‌های دقیق حل‌شده‌ای است که در project environment نصب شده‌اند. این فایل باید در کنترل نسخه ثبت شود تا نصب‌های سازگار و قابل‌تکرار در ماشین‌های مختلف امکان‌پذیر باشد.

lockfile تضمین می‌کند که توسعه‌دهندگانی که روی project کار می‌کنند، از مجموعهٔ سازگاری از نسخه‌های package استفاده می‌کنند. علاوه بر این، هنگام استقرار project به‌عنوان یک برنامه، تضمین می‌کند که مجموعهٔ دقیق نسخه‌های package استفاده‌شده مشخص است.

lockfile در طول فراخوانی‌های uv که از project environment استفاده می‌کنند، یعنی `uv sync` و `uv run`، [به‌طور خودکار ساخته و به‌روز می‌شود](./sync.md#automatic-lock-and-sync). lockfile را می‌توان با استفاده از `uv lock` نیز به‌صورت صریح به‌روز کرد.

`uv.lock` یک فایل TOML خوانا برای انسان است، اما توسط uv مدیریت می‌شود و نباید به‌صورت دستی ویرایش شود. قالب `uv.lock` مختص uv است و با ابزارهای دیگر قابل‌استفاده نیست.

### رابطه با `pylock.toml` {#relationship-to-pylocktoml}

در [PEP 751](https://peps.python.org/pep-0751/)، Python یک قالب فایل resolution جدید به نام `pylock.toml` را استاندارد کرد.

`pylock.toml` یک قالب خروجی resolution است که قرار است جایگزین `requirements.txt` شود (مثلاً در زمینهٔ `uv pip compile`، که در آن یک فایل `requirements.txt` «قفل‌شده» از مجموعه‌ای از نیازمندی‌های ورودی ساخته می‌شود). `pylock.toml` استانداردشده و مستقل از ابزار است، به‌طوری که در آینده، فایل‌های `pylock.toml` ساخته‌شده توسط uv می‌توانند توسط ابزارهای دیگر نصب شوند و برعکس.

برخی از قابلیت‌های uv را نمی‌توان در قالب `pylock.toml` بیان کرد؛ به همین دلیل، uv همچنان در رابط project از قالب `uv.lock` استفاده خواهد کرد.

با این حال، uv از `pylock.toml` به‌عنوان هدف خروجی (export) و در CLI مربوط به `uv pip` پشتیبانی می‌کند. برای مثال:

- برای خروجی‌گرفتن از `uv.lock` به قالب `pylock.toml`، این دستور را اجرا کنید: `uv export -o pylock.toml`
- برای ساخت فایل `pylock.toml` از مجموعه‌ای از نیازمندی‌ها، این دستور را اجرا کنید:
  `uv pip compile requirements.in -o pylock.toml`
- برای نصب از فایل `pylock.toml`، این دستور را اجرا کنید: `uv pip sync pylock.toml` یا
  `uv pip install -r pylock.toml`
