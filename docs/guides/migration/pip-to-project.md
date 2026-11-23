# مهاجرت از pip به یک uv project {#migrating-from-pip-to-a-uv-project}

در این راهنما، تبدیل از یک workflow مبتنی بر `pip` و `pip-tools` که حول فایل‌های `requirements` می‌چرخد، به workflow پروژهٔ uv با استفاده از یک فایل `pyproject.toml` و یک فایل `uv.lock` بررسی می‌شود.

!!! note

    اگر به دنبال مهاجرت از `pip` و `pip-tools` به رابط جایگزین (drop-in) uv هستید یا از یک workflow موجود که در آن از پیش از یک `pyproject.toml` استفاده می‌کنید، آن راهنماها هنوز نوشته نشده‌اند. برای پیگیری پیشرفت، [#5200](https://github.com/astral-sh/uv/issues/5200) را ببینید.

با یک مرور کلی بر توسعه با `pip` شروع می‌کنیم، سپس به مهاجرت به uv می‌پردازیم.

!!! tip

    اگر با این اکوسیستم آشنا هستید، می‌توانید مستقیم به دستورالعمل‌های
    [وارد کردن فایل‌های requirements](#importing-requirements-files) بروید.

## درک workflowهای pip {#understanding-pip-workflows}

### dependencyهای پروژه {#project-dependencies}

وقتی می‌خواهید از یک package در پروژهٔ خود استفاده کنید، ابتدا باید آن را نصب کنید. `pip` از نصب دستوری (imperative) packageها پشتیبانی می‌کند، برای نمونه:

```console
$ pip install fastapi
```

این کار package را در environmentای نصب می‌کند که `pip` در آن نصب شده است. این environment ممکن است یک virtual environment باشد، یا environment سراسری نصب Python سیستم شما.

سپس می‌توانید یک اسکریپت Python را که به آن package نیاز دارد اجرا کنید:

```python title="example.py"
import fastapi
```

بهترین روش این است که برای هر پروژه یک virtual environment ایجاد کنید تا از قاطی شدن packageها بین آن‌ها جلوگیری شود. برای نمونه:

```console
$ python -m venv
$ source .venv/bin/activate
$ pip ...
```

ما در بخش [environmentهای پروژه](#project-environments) زیر دوباره به این موضوع بازمی‌گردیم.

### فایل‌های requirements {#requirements-files}

هنگام به اشتراک گذاشتن پروژه‌ها با دیگران، مفید است که همهٔ packageهایی را که نیاز دارید از پیش اعلام کنید. `pip` از نصب requirementها از یک فایل پشتیبانی می‌کند، برای نمونه:

```requirements title="requirements.txt"
fastapi
```

```console
$ pip install -r requirements.txt
```

توجه کنید که در بالا، `fastapi` به یک نسخهٔ مشخص «قفل» نشده است — هر کسی که روی پروژه کار می‌کند ممکن است نسخهٔ متفاوتی از `fastapi` را نصب کرده باشد. `pip-tools` برای بهبود این تجربه ساخته شد.

هنگام استفاده از `pip-tools`، فایل‌های requirements هم dependencyهای پروژهٔ شما را مشخص می‌کنند و هم dependencyها را به یک نسخهٔ مشخص قفل می‌کنند — پسوند فایل برای تمایز بین این دو به کار می‌رود. برای نمونه، اگر به `fastapi` و `pydantic` نیاز دارید، آن‌ها را در یک فایل `requirements.in` مشخص می‌کنید:

```requirements title="requirements.in"
fastapi
pydantic>2
```

توجه کنید که روی `pydantic` یک version constraint وجود دارد — این یعنی فقط نسخه‌های `pydantic` که بعد از `2.0.0` هستند می‌توانند استفاده شوند. در مقابل، `fastapi` هیچ version constraintای ندارد — هر نسخه‌ای می‌تواند استفاده شود.

این dependencyها را می‌توان در یک فایل `requirements.txt` کامپایل کرد:

```console
$ pip-compile requirements.in -o requirements.txt
```

```requirements title="requirements.txt"
annotated-types==0.7.0
    # via pydantic
anyio==4.8.0
    # via starlette
fastapi==0.115.11
    # via -r requirements.in
idna==3.10
    # via anyio
pydantic==2.10.6
    # via
    #   -r requirements.in
    #   fastapi
pydantic-core==2.27.2
    # via pydantic
sniffio==1.3.1
    # via anyio
starlette==0.46.1
    # via fastapi
typing-extensions==4.12.2
    # via
    #   fastapi
    #   pydantic
    #   pydantic-core
```

در اینجا همهٔ version constraintها _دقیق_ هستند. فقط یک نسخهٔ واحد از هر package می‌تواند استفاده شود. مثال بالا با `uv pip compile` ساخته شده است، اما می‌توانست با `pip-compile` از `pip-tools` نیز ساخته شود.

هرچند کمتر رایج است، فایل `requirements.txt` را می‌توان با استفاده از `pip freeze` نیز ساخت؛ به این صورت که ابتدا dependencyهای ورودی را در environment نصب می‌کنید و سپس نسخه‌های نصب‌شده را صادر می‌کنید:

```console
$ pip install -r requirements.in
$ pip freeze > requirements.txt
```

```requirements title="requirements.txt"
annotated-types==0.7.0
anyio==4.8.0
fastapi==0.115.11
idna==3.10
pydantic==2.10.6
pydantic-core==2.27.2
sniffio==1.3.1
starlette==0.46.1
typing-extensions==4.12.2
```

پس از کامپایل dependencyها به یک مجموعهٔ قفل‌شده از نسخه‌ها، این فایل‌ها در کنترل نسخه ثبت و همراه با پروژه توزیع می‌شوند.

سپس، وقتی کسی می‌خواهد از پروژه استفاده کند، از فایل requirements نصب می‌کند:

```console
$ pip install -r requirements.txt
```

<!--- TODO: Discuss equivalent commands for `uv pip compile` and `pip compile` -->

### dependencyهای توسعه {#development-dependencies}

قالب فایل requirements در هر بار فقط می‌تواند یک مجموعهٔ واحد از dependencyها را توصیف کند. این یعنی اگر _group_های دیگری از dependencyها دارید، مانند dependencyهای توسعه، به فایل‌های جداگانه نیاز دارند. برای نمونه، یک فایل dependency برای `-dev` ایجاد می‌کنیم:

```requirements title="requirements-dev.in"
-r requirements.in
-c requirements.txt

pytest
```

توجه کنید که requirementهای پایه با `-r requirements.in` گنجانده شده‌اند. این کار تضمین می‌کند که environment توسعه _همهٔ_ dependencyها را با هم در نظر می‌گیرد. `-c requirements.txt` نسخهٔ package را _محدود_ می‌کند تا مطمئن شود `requirements-dev.txt` از همان نسخه‌هایی استفاده می‌کند که `requirements.txt` استفاده می‌کند.

!!! note

    رایج است که به‌جای استفاده از هر دو مورد
    `-r requirements.in` و `-c requirements.txt`، مستقیم از `-r requirements.txt` استفاده شود. در نسخه‌های حاصلِ package هیچ تفاوتی نیست، اما استفاده از هر دو فایل حاشیه‌نویسی‌هایی (annotation) تولید می‌کند که به شما امکان می‌دهد تشخیص دهید کدام dependencyها _مستقیم_ هستند (با `-r requirements.in` حاشیه‌نویسی شده‌اند) و کدام‌ها _غیرمستقیم_ هستند (فقط با `-c requirements.txt` حاشیه‌نویسی شده‌اند).

dependencyهای توسعهٔ کامپایل‌شده به این شکل‌اند:

```requirements title="requirements-dev.txt"
annotated-types==0.7.0
    # via
    #   -c requirements.txt
    #   pydantic
anyio==4.8.0
    # via
    #   -c requirements.txt
    #   starlette
fastapi==0.115.11
    # via
    #   -c requirements.txt
    #   -r requirements.in
idna==3.10
    # via
    #   -c requirements.txt
    #   anyio
iniconfig==2.0.0
    # via pytest
packaging==24.2
    # via pytest
pluggy==1.5.0
    # via pytest
pydantic==2.10.6
    # via
    #   -c requirements.txt
    #   -r requirements.in
    #   fastapi
pydantic-core==2.27.2
    # via
    #   -c requirements.txt
    #   pydantic
pytest==8.3.5
    # via -r requirements-dev.in
sniffio==1.3.1
    # via
    #   -c requirements.txt
    #   anyio
starlette==0.46.1
    # via
    #   -c requirements.txt
    #   fastapi
typing-extensions==4.12.2
    # via
    #   -c requirements.txt
    #   fastapi
    #   pydantic
    #   pydantic-core
```

مانند فایل‌های dependency پایه، این‌ها نیز در کنترل نسخه ثبت و همراه با پروژه توزیع می‌شوند. وقتی کسی می‌خواهد روی پروژه کار کند، از فایل requirements نصب می‌کند:

```console
$ pip install -r requirements-dev.txt
```

### dependencyهای مخصوص platform {#platform-specific-dependencies}

هنگام کامپایل dependencyها با `pip` یا `pip-tools`، نتیجه فقط روی همان platformای قابل استفاده است که روی آن ساخته شده است. این موضوع برای پروژه‌هایی که باید روی چند platform قابل استفاده باشند، مانند Windows و macOS، مشکل ایجاد می‌کند.

برای نمونه، یک dependency ساده را در نظر بگیرید:

```requirements title="requirements.in"
tqdm
```

روی Linux، این به صورت زیر کامپایل می‌شود:

```requirements title="requirements-linux.txt"
tqdm==4.67.1
    # via -r requirements.in
```

در حالی که روی Windows، به صورت زیر کامپایل می‌شود:

```requirements title="requirements-win.txt"
colorama==0.4.6
    # via tqdm
tqdm==4.67.1
    # via -r requirements.in
```

`colorama` یک dependency مخصوص Windows برای `tqdm` است.

هنگام استفاده از `pip` و `pip-tools`، یک پروژه باید برای هر platform پشتیبانی‌شده یک فایل قفل requirements اعلام کند.

!!! note

    resolver مربوط به uv می‌تواند dependencyها را برای چند platform به‌صورت هم‌زمان کامپایل کند (ببینید ["universal resolution"](../../concepts/resolution.md#universal-resolution))،
    و به شما امکان می‌دهد از یک فایل `requirements.txt` واحد برای همهٔ platformها استفاده کنید:

    ```console
    $ uv pip compile --universal requirements.in
    ```

    ```requirements title="requirements.txt"
    colorama==0.4.6 ; sys_platform == 'win32'
        # via tqdm
    tqdm==4.67.1
        # via -r requirements.in
    ```

    این حالت resolution هنگام استفاده از یک `pyproject.toml` و `uv.lock` نیز به کار می‌رود.

## مهاجرت به یک uv project {#migrating-to-a-uv-project}

### فایل `pyproject.toml` {#the-pyprojecttoml}

فایل `pyproject.toml` یک فایل استاندارد برای metadata پروژهٔ Python است. این فایل جایگزین فایل‌های `requirements.in` می‌شود و به شما امکان می‌دهد groupهای دلخواه از dependencyهای پروژه را نمایش دهید. همچنین یک مکان متمرکز برای metadata دربارهٔ پروژهٔ شما، مانند build system یا تنظیمات tool، فراهم می‌کند.

<!-- TODO: Link to the official docs on this or write more -->

برای نمونه، فایل‌های `requirements.in` و `requirements-dev.in` بالا را می‌توان به‌صورت زیر به یک `pyproject.toml` ترجمه کرد:

```toml title="pyproject.toml"
[project]
name = "example"
version = "0.0.1"
dependencies = [
    "fastapi",
    "pydantic>2"
]

[dependency-groups]
dev = ["pytest"]
```

دستورهای لازم برای خودکارسازی این وارد کردن‌ها را در ادامه بررسی می‌کنیم.

### lockfile مربوط به uv {#the-uv-lockfile}

uv از یک lockfile (`uv.lock`) برای قفل کردن نسخه‌های package استفاده می‌کند. قالب این فایل مخصوص uv است و به uv امکان می‌دهد از قابلیت‌های پیشرفته پشتیبانی کند. این فایل جایگزین فایل‌های `requirements.txt` می‌شود.

lockfile هنگام افزودن dependencyها به‌صورت خودکار ایجاد و پر می‌شود، اما می‌توانید آن را به‌صراحت با `uv lock` ایجاد کنید.

برخلاف فایل‌های `requirements.txt`، فایل `uv.lock` می‌تواند groupهای دلخواه از dependencyها را نمایش دهد، پس برای قفل کردن dependencyهای توسعه به چند فایل نیاز نیست.

lockfile مربوط به uv همیشه [universal](../../concepts/resolution.md#universal-resolution) است، پس برای
[قفل کردن dependencyها برای هر platform](#platform-specific-dependencies) به چند فایل نیاز نیست. این کار تضمین می‌کند که همهٔ توسعه‌دهندگان، صرف‌نظر از ماشین خود، از نسخه‌های قفل‌شدهٔ سازگار dependencyها استفاده می‌کنند.

lockfile مربوط به uv همچنین از مفاهیمی مانند
[سنجاق کردن (pin) packageها به indexهای مشخص](../../concepts/indexes.md#pinning-a-package-to-an-index) پشتیبانی می‌کند،
که در فایل‌های `requirements.txt` قابل نمایش نیست.

!!! tip

    اگر فقط نیاز دارید برای زیرمجموعه‌ای از platformها قفل کنید، از تنظیم
    [`tool.uv.environments`](../../concepts/resolution.md#limited-resolution-environments) استفاده کنید
    تا resolution و lockfile را محدود کنید.

برای یادگیری بیشتر، مستندات [lockfile](../../concepts/projects/layout.md#the-lockfile) را ببینید.

### وارد کردن فایل‌های requirements {#importing-requirements-files}

ابتدا، اگر هنوز یک `pyproject.toml` ایجاد نکرده‌اید، ایجاد کنید:

```console
$ uv init
```

سپس، ساده‌ترین راه برای وارد کردن requirementها استفاده از `uv add` است:

```console
$ uv add -r requirements.in
```

با این حال، در این گذار ظرافت‌هایی وجود دارد. توجه کنید که ما از فایل `requirements.in` استفاده کردیم که نسخه‌های package را به نسخه‌های دقیق سنجاق نمی‌کند، پس uv برای این packageها نسخه‌های جدید را حل می‌کند. ممکن است بخواهید همچنان از نسخه‌های قفل‌شدهٔ پیشین خود از `requirements.txt` استفاده کنید تا هنگام جابه‌جایی به uv، هیچ‌یک از نسخه‌های dependency شما تغییر نکند.

راه‌حل این است که نسخه‌های قفل‌شدهٔ خود را به‌عنوان _constraint_ اضافه کنید. uv از استفاده از این‌ها در `add` برای حفظ نسخه‌های قفل‌شده پشتیبانی می‌کند:

```console
$ uv add -r requirements.in -c requirements.txt
```

نسخه‌های موجود شما هنگام تولید یک فایل `uv.lock` حفظ می‌شوند.

#### وارد کردن constraintهای مخصوص platform {#importing-platform-specific-constraints}

اگر dependencyهای مخصوص platform شما در فایل‌های جداگانه کامپایل شده‌اند، همچنان می‌توانید به یک lockfile universal گذار کنید. با این حال، نمی‌توانید فقط از `-c` برای مشخص کردن constraintها از فایل‌های `requirements.txt` مخصوص platform خود استفاده کنید، چون آن‌ها markerهایی را که environment را توصیف می‌کنند شامل نمی‌شوند و در نتیجه با هم در تضاد قرار می‌گیرند.

برای افزودن markerهای لازم، از `uv pip compile` برای تبدیل فایل‌های موجود خود استفاده کنید. برای نمونه، با توجه به مورد زیر:

```requirements title="requirements-win.txt"
colorama==0.4.6
    # via tqdm
tqdm==4.67.1
    # via -r requirements.in
```

markerها را می‌توان با این دستور اضافه کرد:

```console
$ uv pip compile requirements.in -o requirements-win.txt --python-platform windows --no-strip-markers
```

توجه کنید که خروجی حاصل شامل یک marker مربوط به Windows روی `colorama` است:

```requirements title="requirements-win.txt"
colorama==0.4.6 ; sys_platform == 'win32'
    # via tqdm
tqdm==4.67.1
    # via -r requirements.in
```

هنگام استفاده از `-o`، uv نسخه‌ها را به‌گونه‌ای محدود می‌کند که در صورت امکان با فایل خروجی موجود مطابقت داشته باشند.

markerها را می‌توان با تغییر مقدارهای `--python-platform` و `-o` برای هر فایل requirements که باید وارد کنید، برای platformهای دیگر نیز اضافه کرد؛ برای نمونه، به `linux` و `macos`.

وقتی هر فایل `requirements.txt` تبدیل شد، dependencyها را می‌توان با `uv add` به `pyproject.toml` و `uv.lock` وارد کرد:

```console
$ uv add -r requirements.in -c requirements-win.txt -c requirements-linux.txt
```

#### وارد کردن فایل‌های dependency توسعه {#importing-development-dependency-files}

همان‌طور که در بخش [dependencyهای توسعه](#development-dependencies) بحث شد، رایج است که groupهایی از dependencyها برای اهداف توسعه وجود داشته باشد.

برای وارد کردن dependencyهای توسعه، هنگام `uv add` از فلگ `--dev` استفاده کنید:

```console
$ uv add --dev -r requirements-dev.in -c requirements-dev.txt
```

اگر فایل `requirements-dev.in` فایل `requirements.in` والد را از طریق `-r` شامل می‌شود، باید آن را حذف کنید تا از افزودن requirementهای پایه به dependency group مربوط به `dev` جلوگیری شود. مثال زیر از `sed` برای حذف خط‌هایی که با `-r` شروع می‌شوند استفاده می‌کند و سپس نتیجه را به `uv add` می‌فرستد (pipe می‌کند):

```console
$ sed '/^-r /d' requirements-dev.in | uv add --dev -r - -c requirements-dev.txt
```

علاوه بر dependency group مربوط به `dev`، uv از نام‌های دلخواه group نیز پشتیبانی می‌کند. برای نمونه، اگر یک مجموعهٔ اختصاصی از dependencyها برای ساخت مستندات خود نیز دارید، می‌توان آن‌ها را به یک group با نام `docs` وارد کرد:

```console
$ uv add -r requirements-docs.in -c requirements-docs.txt --group docs
```

#### وارد کردن sourceهای dependency {#importing-dependency-sources}

هنگام وارد کردن requirementها روی مسیرهای محلی یا مخزن‌های Git، برای نمونه:

```requirements title="requirements.in"
./path-dep
-e ./editable-path-dep
git-dep @ git+https://github.com/astral-sh/git-dep
```

uv آن‌ها را به [dependency sourceها](../../concepts/projects/dependencies.md#dependency-sources) در جدول `[tool.uv.sources]` در `pyproject.toml` نگاشت می‌کند:

```toml title="pyproject.toml"
[project]
dependencies = [
    "path-dep",
    "editable-path-dep",
    "git-dep",
]

[tool.uv.sources]
path-dep = { path = "./path-dep" }
editable-path-dep = { path = "./editable-path-dep", editable = true }
git-dep = { git = "https://github.com/astral-sh/git-dep" }
```

### environmentهای پروژه {#project-environments}

برخلاف `pip`، uv حول مفهوم یک virtual environment «فعال» متمرکز نیست. در عوض، uv برای هر پروژه از یک virtual environment اختصاصی در یک پوشهٔ `.venv` استفاده می‌کند. این environment به‌صورت خودکار مدیریت می‌شود، پس وقتی یک دستور را اجرا می‌کنید، مانند `uv add`، environment با dependencyهای پروژه همگام می‌شود.

روش ترجیحی برای اجرای دستورها در environment استفاده از `uv run` است، برای نمونه:

```console
$ uv run pytest
```

پیش از هر فراخوانی `uv run`، uv بررسی می‌کند که lockfile با `pyproject.toml` به‌روز باشد و environment با lockfile به‌روز باشد، و پروژهٔ شما را بدون نیاز به دخالت دستی همگام نگه می‌دارد. `uv run` تضمین می‌کند که دستور شما در یک environment سازگار و قفل‌شده اجرا شود.

project environment را می‌توان به‌صراحت با `uv sync` نیز ایجاد کرد؛ برای نمونه، برای استفاده با ویرایشگرها.

!!! note

    وقتی در پروژه‌ها هستید، uv به‌صورت پیش‌فرض یک `.venv` را در پوشهٔ پروژه ترجیح می‌دهد و environment فعال را که با متغیر `VIRTUAL_ENV` اعلام شده است نادیده می‌گیرد. می‌توانید با فلگ `--active` استفاده از environment فعال را انتخاب کنید.

برای یادگیری بیشتر، مستندات
[project environment](../../concepts/projects/layout.md#the-project-environment) را ببینید.

## گام‌های بعدی {#next-steps}

حالا که به uv مهاجرت کرده‌اید، صفحهٔ
[مفهوم پروژه](../../concepts/projects/index.md) را برای جزئیات بیشتر دربارهٔ پروژه‌های uv ببینید.
