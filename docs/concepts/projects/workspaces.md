# استفاده از workspaceها {#using-workspaces}

با الهام از مفهوم [Cargo](https://doc.rust-lang.org/cargo/reference/workspaces.html) به همین نام،
یک workspace «مجموعه‌ای از یک یا چند package است که با نام _اعضای workspace_ شناخته می‌شوند و
به‌صورت مشترک مدیریت می‌شوند.»

workspaceها با تقسیم کدبیس‌های بزرگ به چند package با dependencyهای مشترک، آن‌ها را سازمان‌دهی
می‌کنند. به این فکر کنید: یک وب‌اپلیکیشن مبتنی بر FastAPI، در کنار مجموعه‌ای از کتابخانه‌ها که
به‌عنوان packageهای جداگانهٔ Python نسخه‌بندی و نگهداری می‌شوند، همه در یک مخزن Git.

در یک workspace، هر package فایل `pyproject.toml` خودش را تعریف می‌کند، اما workspace یک
lockfile مشترک دارد و این تضمین می‌کند که workspace با مجموعه‌ای یکپارچه از dependencyها کار کند.

به همین دلیل، `uv lock` روی کل workspace به‌صورت یکجا عمل می‌کند، در حالی که `uv run` و `uv sync`
به‌صورت پیش‌فرض روی ریشهٔ workspace عمل می‌کنند، هرچند هر دو آرگومان `--package` را می‌پذیرند و
به شما اجازه می‌دهند دستوری را در یک عضو خاص workspace، از هر پوشهٔ workspace، اجرا کنید.

## شروع کار {#getting-started}

برای ساخت یک workspace، یک جدول `tool.uv.workspace` به `pyproject.toml` اضافه کنید؛ با این کار
به‌صورت ضمنی یک workspace ساخته می‌شود که ریشهٔ آن همان package است.

!!! tip

    به‌صورت پیش‌فرض، اجرای `uv init` درون یک package موجود، عضو تازه‌ساخته‌شده را به workspace اضافه می‌کند و اگر جدول `tool.uv.workspace` در ریشهٔ workspace وجود نداشته باشد، آن را ایجاد می‌کند.

در تعریف یک workspace، باید کلیدهای `members` (الزامی) و `exclude` (اختیاری) را مشخص کنید؛
این کلیدها به‌ترتیب به workspace می‌گویند پوشه‌های مشخصی را به‌عنوان عضو شامل یا حذف کند و
فهرست‌هایی از glob می‌پذیرند:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/seeds"]
```

هر پوشه‌ای که در globهای `members` گنجانده شده باشد (و در globهای `exclude` حذف نشده باشد) باید
یک فایل `pyproject.toml` داشته باشد. با این حال، اعضای workspace می‌توانند _یا_
[application](./init.md#applications) باشند _یا_ [library](./init.md#libraries)؛ هر دو در
بستر workspace پشتیبانی می‌شوند.

هر workspace به یک ریشه نیاز دارد که _خودش هم_ یک عضو workspace است. در مثال بالا، `albatross`
ریشهٔ workspace است و اعضای workspace شامل همهٔ projectهای زیر پوشهٔ `packages` هستند، به‌جز
`seeds`.

به‌صورت پیش‌فرض، `uv run` و `uv sync` روی ریشهٔ workspace عمل می‌کنند. در مثال بالا، `uv run` و
`uv run --package albatross` معادل یکدیگر بودند، در حالی که `uv run --package bird-feeder` دستور
را در packageی `bird-feeder` اجرا می‌کرد.

## منبع‌های workspace {#workspace-sources}

درون یک workspace، dependencyهای روی اعضای workspace از طریق
[`tool.uv.sources`](./dependencies.md) فراهم می‌شوند، مانند:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

در این مثال، projectی `albatross` به projectی `bird-feeder` وابسته است که عضوی از
workspace است. جفت کلید-مقدار `workspace = true` در جدول `tool.uv.sources` نشان می‌دهد که
dependencyی `bird-feeder` باید از workspace تأمین شود، نه اینکه از PyPI یا
registry دیگری دریافت شود. فیلد `workspace` را می‌توان روی یک رشتهٔ مسیر هم تنظیم کرد تا یک
dependency از workspace دیگری مشخص شود. مسیر نسبت به projectی که source را اعلام می‌کند
(یا نسبت به ریشهٔ workspace برای یک source در سطح workspace) مشخص می‌شود و باید به ریشهٔ
workspace خارجی اشاره کند. uv عضوی را انتخاب می‌کند که با نام dependency مطابقت داشته باشد.

!!! note

    dependencyهای میان اعضای workspace به‌صورت editable هستند.

هر تعریف `tool.uv.sources` در ریشهٔ workspace برای همهٔ اعضا اعمال می‌شود، مگر اینکه در
`tool.uv.sources` یک عضو خاص بازنویسی شده باشد. برای نمونه، با توجه به `pyproject.toml` زیر:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }
tqdm = { git = "https://github.com/tqdm/tqdm" }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

هر عضو workspace به‌صورت پیش‌فرض `tqdm` را از GitHub نصب می‌کرد، مگر اینکه یک عضو خاص
ورودی `tqdm` را در جدول `tool.uv.sources` خودش بازنویسی کند.

!!! note

    اگر یک عضو workspace برای dependencyای `tool.uv.sources` ارائه کند، هر
    `tool.uv.sources` مربوط به همان dependency در ریشهٔ workspace را نادیده می‌گیرد، حتی اگر source عضو
    با یک [marker](dependencies.md#platform-specific-sources) محدود شده باشد که با platform فعلی
    مطابقت ندارد.

## چیدمان‌های workspace {#workspace-layouts}

رایج‌ترین چیدمان workspace را می‌توان یک project ریشه با مجموعه‌ای از کتابخانه‌های همراه دانست.

برای نمونه، با ادامهٔ مثال بالا، این workspace یک ریشهٔ صریح در `albatross` دارد،
با دو کتابخانه (`bird-feeder` و `seeds`) در پوشهٔ `packages`:

```text
albatross
├── packages
│   ├── bird-feeder
│   │   ├── pyproject.toml
│   │   └── src
│   │       └── bird_feeder
│   │           ├── __init__.py
│   │           └── foo.py
│   └── seeds
│       ├── pyproject.toml
│       └── src
│           └── seeds
│               ├── __init__.py
│               └── bar.py
├── pyproject.toml
├── README.md
├── uv.lock
└── src
    └── albatross
        └── __init__.py
```

از آنجا که `seeds` در `pyproject.toml` حذف شده بود، workspace در مجموع دو عضو دارد: `albatross`
(ریشه) و `bird-feeder`.

## چه زمانی از workspaceها استفاده کنیم (و چه زمانی نه) {#when-not-to-use-workspaces}

workspaceها برای تسهیل توسعهٔ چند package به‌هم‌پیوسته در یک مخزن واحد در نظر گرفته شده‌اند. با
رشد پیچیدگی یک کدبیس، تقسیم آن به packageهای کوچک‌تر و ترکیب‌پذیر می‌تواند مفید باشد؛ هر یک با
dependencyها و قیدهای نسخهٔ خودشان.

workspaceها به اعمال ایزولاسیون و جداسازی دغدغه‌ها کمک می‌کنند. برای نمونه، در uv، ما packageهای
جداگانه‌ای برای کتابخانهٔ اصلی و رابط خط فرمان داریم که به ما امکان می‌دهد کتابخانهٔ اصلی را
مستقل از CLI آزمایش کنیم و برعکس.

دیگر موارد رایج استفاده از workspaceها عبارت‌اند از:

- یک کتابخانه با یک زیرروال حیاتی از نظر کارایی که در یک module توسعه‌ای (Rust ،C++ و
  غیره) پیاده‌سازی شده است.
- یک کتابخانه با یک سیستم پلاگین، که در آن هر پلاگین یک package جداگانهٔ workspace با یک
  dependency روی ریشه است.

workspaceها برای مواردی که اعضا requirementهای متناقض دارند یا یک virtual environment جداگانه برای
هر عضو می‌خواهند، مناسب _نیستند_. در این حالت، path dependencyها اغلب ترجیح داده می‌شوند.
برای نمونه، به‌جای گروه‌بندی `albatross` و اعضای آن در یک workspace، همیشه می‌توانید
هر package را به‌عنوان project مستقل خودش تعریف کنید و dependencyهای میان packageها را به‌عنوان path
dependency در `tool.uv.sources` تعریف کنید:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { path = "packages/bird-feeder" }

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

این رویکرد بسیاری از همان مزایا را ارائه می‌کند، اما امکان کنترل دقیق‌تر بر
مشخص‌کردن dependency و مدیریت virtual environment را فراهم می‌کند (با این نقطه‌ضعف که `uv run --package`
دیگر در دسترس نیست؛ در عوض، دستورها باید از پوشهٔ package مربوطه اجرا شوند).

در نهایت، workspaceهای uv یک `requires-python` واحد را برای کل workspace اعمال می‌کنند و
اشتراک همهٔ مقدارهای `requires-python` اعضا را می‌گیرند. اگر نیاز دارید آزمایش یک عضو مشخص را
روی یک Python version پشتیبانی کنید که بقیهٔ workspace از آن پشتیبانی نمی‌کند، ممکن است لازم باشد
از `uv pip` استفاده کنید تا آن عضو را در یک virtual environment جداگانه نصب کنید.

!!! note

    از آنجا که Python ایزولاسیون dependency ارائه نمی‌دهد، uv نمی‌تواند تضمین کند که یک package فقط از dependencyهای اعلام‌شدهٔ خودش استفاده می‌کند و نه چیز دیگری. به‌طور خاص برای workspaceها، uv نمی‌تواند تضمین کند که packageها dependencyهای اعلام‌شده توسط عضو دیگری از workspace را import نمی‌کنند.
