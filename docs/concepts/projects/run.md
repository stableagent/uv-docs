# اجرای دستورها در projectها {#running-commands-in-projects}

هنگام کار روی یک project، آن project در virtual environment واقع در `.venv` نصب می‌شود. این environment
به‌طور پیش‌فرض از shell فعلی جداست، بنابراین فراخوانی‌هایی که به project نیاز دارند، برای مثال
`python -c "import example"`، با شکست مواجه می‌شوند. در عوض، از `uv run` برای اجرای دستورها در environment
project استفاده کنید:

```console
$ uv run python -c "import example"
```

هنگام استفاده از `run`، uv پیش از اجرای دستور داده‌شده، اطمینان حاصل می‌کند که environment مربوط به project به‌روز است.

دستور داده‌شده می‌تواند توسط environment مربوط به project فراهم شود یا خارج از آن وجود داشته باشد، برای مثال:

```console
$ # Presuming the project provides `example-cli`
$ uv run example-cli foo

$ # Running a `bash` script that requires the project to be available
$ uv run bash scripts/foo.sh
```

## درخواست dependencyهای اضافی {#requesting-additional-dependencies}

می‌توان dependencyهای اضافی یا نسخه‌های متفاوتی از dependencyها را برای هر فراخوانی درخواست کرد.

گزینهٔ `--with` برای گنجاندن یک dependency در فراخوانی استفاده می‌شود، برای مثال برای درخواست یک نسخهٔ متفاوت
از `httpx`:

```console
$ uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.__version__)"
0.26.0
$ uv run --with httpx==0.25.0 python -c "import httpx; print(httpx.__version__)"
0.25.0
```

نسخهٔ درخواستی صرف‌نظر از requirementهای project رعایت خواهد شد. برای مثال، حتی
اگر project به `httpx==0.24.0` نیاز داشته باشد، خروجی بالا همان خواهد بود.

## اجرای scriptها {#running-scripts}

scriptهایی که metadata درون‌خطی اعلام می‌کنند، به‌طور خودکار در environmentهایی جدا از
project اجرا می‌شوند. برای جزئیات بیشتر، به [راهنمای scriptها](../../guides/scripts.md#declaring-script-dependencies) مراجعه کنید.

برای مثال، با داشتن یک script:

```python title="example.py"
# /// script
# dependencies = [
#   "httpx",
# ]
# ///

import httpx

resp = httpx.get("https://peps.python.org/api/peps.json")
data = resp.json()
print([(k, v["title"]) for k, v in data.items()][:10])
```

فراخوانی `uv run example.py` به‌صورت _جدا_ از project و تنها با dependencyهای
داده‌شده اجرا می‌شود.

## scriptهای قدیمی در Windows {#legacy-scripts-on-windows}

پشتیبانی از
[scriptهای قدیمی setuptools](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#scripts) ارائه شده است.
این نوع scriptها فایل‌های اضافی‌ای هستند که setuptools در `.venv\Scripts` نصب می‌کند.

در حال حاضر فقط scriptهای قدیمی با پسوندهای `.ps1`، `.cmd` و `.bat` پشتیبانی می‌شوند.

برای مثال، برای اجرای یک script مربوط به Command Prompt:

```console
$ uv run --with nuitka==2.6.7 -- nuitka.cmd --version
```

علاوه بر این، نیازی نیست پسوند را مشخص کنید. `uv` به‌طور خودکار به‌جای شما به‌ترتیب به‌دنبال فایل‌هایی با پایان
`.ps1`، `.cmd` و `.bat` می‌گردد.

```console
$ uv run --with nuitka==2.6.7 -- nuitka --version
```

## مدیریت سیگنال {#signal-handling}

uv برای ارائهٔ پیام‌های خطای بهتر در صورت شکست، کنترل فرایند را به دستور ایجادشده واگذار نمی‌کند. در نتیجه، uv مسئول ارسال برخی سیگنال‌ها به فرایند
فرزندی است که دستور درخواستی در آن اجرا می‌شود.

در سیستم‌های Unix، uv بیشتر سیگنال‌ها را (به‌استثنای SIGKILL، SIGCHLD، SIGIO و
SIGPOLL) به فرایند فرزند ارسال می‌کند. از آنجا که ترمینال‌ها با Ctrl-C سیگنال SIGINT را به گروه فرایند پیش‌زمینه ارسال می‌کنند، uv یک SIGINT را فقط در صورتی به فرایند فرزند ارسال می‌کند که بیش از یک بار ارسال شود یا گروه فرایند فرزند با گروه فرایند uv متفاوت باشد.

در Windows، این مفاهیم کاربرد ندارند و uv رویدادهای Ctrl-C را نادیده می‌گیرد و مدیریت آن را به
فرایند فرزند واگذار می‌کند تا بتواند به‌تمیزی خارج شود.
