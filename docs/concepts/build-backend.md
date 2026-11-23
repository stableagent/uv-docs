# بک‌اند بیلد uv {#the-uv-build-backend}

یک build backend یک source tree (یعنی یک پوشه) را به یک source distribution یا یک wheel تبدیل می‌کند.

uv از همهٔ build backendها پشتیبانی می‌کند (مطابق [PEP 517](https://peps.python.org/pep-0517/))، اما یک build backend بومی (`uv_build`) هم ارائه می‌دهد که به‌صورت تنگاتنگ با uv یکپارچه می‌شود تا عملکرد و تجربهٔ کاربری را بهبود دهد.

## انتخاب یک build backend {#choosing-a-build-backend}

build backend مربوط به uv برای بیشتر پروژه‌های Python انتخاب بسیار خوبی است. پیش‌فرض‌های معقولی دارد، با این هدف که برای بیشتر کاربران به هیچ پیکربندی‌ای نیاز نباشد، اما پیکربندی انعطاف‌پذیری ارائه می‌دهد تا با بیشتر ساختارهای پروژهٔ Python سازگار شود. به‌صورت تنگاتنگ با uv یکپارچه می‌شود تا پیام‌رسانی و تجربهٔ کاربری را بهبود دهد. metadata و ساختارهای پروژه را اعتبارسنجی می‌کند و از اشتباه‌های رایج جلوگیری می‌کند. و در نهایت، بسیار سریع است.

build backend مربوط به uv در حال حاضر **فقط از کد خالص Python پشتیبانی می‌کند**. برای ساخت یک [کتابخانه با ماژول‌های extension](../concepts/projects/init.md#projects-with-extension-modules) به یک backend جایگزین نیاز است.

!!! tip

    با اینکه این backend از تعدادی گزینه برای پیکربندی ساختار پروژهٔ شما پشتیبانی می‌کند، وقتی به اسکریپت‌های بیلد یا یک چیدمان انعطاف‌پذیرتر برای پروژه نیاز است، به‌جای آن استفاده از build backend مربوط به [hatchling](https://hatch.pypa.io/latest/config/build/#build-system) را در نظر بگیرید.

## استفاده از build backend مربوط به uv {#using-the-uv-build-backend}

برای استفاده از uv به‌عنوان یک build backend در یک پروژهٔ موجود، `uv_build` را به بخش [`[build-system]`](../concepts/projects/config.md#build-systems) در فایل `pyproject.toml` خود اضافه کنید:

```toml title="pyproject.toml"
[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

!!! note

    build backend مربوط به uv از همان [سیاست نسخه‌گذاری](../reference/policies/versioning.md) uv پیروی می‌کند. گنجاندن یک حد بالا برای نسخهٔ `uv_build` تضمین می‌کند که package شما با انتشار نسخه‌های جدید همچنان به‌درستی بیلد می‌شود.

برای ساخت یک پروژهٔ جدید که از build backend مربوط به uv استفاده می‌کند، از `uv init` استفاده کنید:

```console
$ uv init
```

هنگام بیلد پروژه، مثلاً با [`uv build`](../guides/package.md)، از build backend مربوط به uv برای ساخت source distribution و wheel استفاده می‌شود.

## build backend همراه {#bundled-build-backend}

build backend به‌عنوان یک package جداگانه (`uv_build`) منتشر می‌شود که برای قابل‌حمل بودن و اندازهٔ کوچک باینری بهینه شده است. با این حال، فایل اجرایی `uv` هم یک نسخه از build backend را در خود دارد که در طول بیلدهایی که توسط uv انجام می‌شوند، مثلاً هنگام `uv build`، استفاده می‌شود، به شرطی که نسخهٔ آن با نیازمندی `uv_build` سازگار باشد. اگر سازگار نباشد، یک نسخهٔ سازگار از package مربوط به `uv_build` استفاده می‌شود. frontendهای بیلد دیگر، مانند `python -m build`، همیشه از package مربوط به `uv_build` استفاده می‌کنند و معمولاً آخرین نسخهٔ سازگار را انتخاب می‌کنند.

## ماژول‌ها {#modules}

انتظار می‌رود packageهای Python یک یا چند module Python داشته باشند، که پوشه‌هایی حاوی یک فایل `__init__.py` هستند. به‌طور پیش‌فرض، انتظار می‌رود یک module ریشهٔ واحد در `src/<package_name>/__init__.py` باشد.

برای مثال، ساختار یک پروژه با نام `foo` چنین خواهد بود:

```text
pyproject.toml
src
└── foo
    └── __init__.py
```

uv نام package را normalize می‌کند تا نام پیش‌فرض module را تعیین کند: نام package به حروف کوچک تبدیل می‌شود و نقطه‌ها و خط‌تیره‌ها با زیرخط جایگزین می‌شوند؛ مثلاً `Foo-Bar` به `foo_bar` تبدیل می‌شود.

پوشهٔ `src/` پوشهٔ پیش‌فرض برای کشف module است.

می‌توان این پیش‌فرض‌ها را با تنظیمات `module-name` و `module-root` تغییر داد. برای مثال، برای استفاده از یک module با نام `FOO` در پوشهٔ ریشه، مطابق ساختار پروژه:

```text
pyproject.toml
FOO
└── __init__.py
```

پیکربندی صحیح بیلد چنین خواهد بود:

```toml title="pyproject.toml"
[tool.uv.build-backend]
module-name = "FOO"
module-root = ""
```

## پکیج‌های namespace {#namespace-packages}

پکیج‌های namespace برای موارد استفاده‌ای در نظر گرفته شده‌اند که در آن‌ها چند package ماژول‌هایی را در یک namespace مشترک می‌نویسند.

ماژول‌های پکیج namespace با یک `.` در `module-name` مشخص می‌شوند. برای مثال، برای بسته‌بندی module مربوط به `bar` در namespace مشترک `foo`، ساختار پروژه چنین خواهد بود:

```text
pyproject.toml
src
└── foo
    └── bar
        └── __init__.py
```

و پیکربندی `module-name` چنین خواهد بود:

```toml title="pyproject.toml"
[tool.uv.build-backend]
module-name = "foo.bar"
```

!!! important

    فایل `__init__.py` در `foo` گنجانده نمی‌شود، زیرا آن module مربوط به namespace مشترک است.

همچنین می‌توان یک پکیج namespace پیچیده با بیش از یک module ریشه داشت، مثلاً با ساختار پروژه:

```text
pyproject.toml
src
├── foo
│   └── __init__.py
└── bar
    └── __init__.py
```

با اینکه ما این ساختار را توصیه نمی‌کنیم (یعنی به‌جای آن باید از یک workspace با چند package استفاده کنید)، با تنظیم `module-name` روی فهرستی از نام‌ها پشتیبانی می‌شود:

```toml title="pyproject.toml"
[tool.uv.build-backend]
module-name = ["foo", "bar"]
```

برای packageهایی با ماژول‌های زیاد یا namespaceهای پیچیده، می‌توان از گزینهٔ `namespace = true` برای جلوگیری از اعلام صریح هر نام module استفاده کرد، مثلاً:

```toml title="pyproject.toml"
[tool.uv.build-backend]
namespace = true
```

!!! warning

    استفاده از `namespace = true` بررسی‌های ایمنی را غیرفعال می‌کند. استفاده از یک فهرست صریح از نام‌های module خارج از پروژه‌های قدیمی به‌شدت توصیه می‌شود.

گزینهٔ `namespace` را می‌توان با `module-name` هم استفاده کرد تا ریشه را صراحتاً اعلام کند، مثلاً برای ساختار پروژه:

```text
pyproject.toml
src
└── foo
    ├── bar
    │   └── __init__.py
    └── baz
        └── __init__.py
```

پیکربندی توصیه‌شده چنین خواهد بود:

```toml title="pyproject.toml"
[tool.uv.build-backend]
module-name = "foo"
namespace = true
```

## پکیج‌های stub {#stub-packages}

build backend همچنین از ساخت پکیج‌های stub مربوط به نوع پشتیبانی می‌کند که با پسوند `-stubs` در نام package یا module مشخص می‌شوند، مثلاً `foo-stubs`. نام module برای پکیج‌های stub مربوط به نوع باید به `-stubs` ختم شود، بنابراین uv علامت `-` را به زیرخط normalize نمی‌کند. علاوه بر این، uv به دنبال یک فایل `__init__.pyi` می‌گردد. برای مثال، ساختار پروژه چنین خواهد بود:

```text
pyproject.toml
src
└── foo-stubs
    └── __init__.pyi
```

ماژول‌های stub مربوط به نوع برای [پکیج‌های namespace](#namespace-packages) هم پشتیبانی می‌شوند.

## گنجاندن و حذف فایل {#file-inclusion-and-exclusion}

build backend مسئول تعیین این است که کدام فایل‌ها در یک source tree باید در distributionها بسته‌بندی شوند.

برای تعیین اینکه کدام فایل‌ها در یک source distribution گنجانده شوند، uv ابتدا فایل‌ها و پوشه‌های گنجانده‌شده را اضافه می‌کند و سپس فایل‌ها و پوشه‌های حذف‌شده را برمی‌دارد. این یعنی حذف‌ها همیشه بر گنجاندن‌ها اولویت دارند.

به‌طور پیش‌فرض، uv موارد `__pycache__` و `*.pyc` و `*.pyo` را حذف می‌کند.

هنگام ساخت یک source distribution، فایل‌ها و پوشه‌های زیر گنجانده می‌شوند:

- فایل `pyproject.toml`، که با سینتکس سازگار با TOML 1.0 بازنویسی شده است. فایل اصلی هم با نام `pyproject.toml.orig` گنجانده می‌شود.
- [module](#modules) زیر
  [`tool.uv.build-backend.module-root`](../reference/settings.md#build-backend_module-root).
- فایل‌هایی که توسط `project.license-files` و `project.readme` ارجاع داده شده‌اند.
- همهٔ پوشه‌های زیر [`tool.uv.build-backend.data`](../reference/settings.md#build-backend_data).
- همهٔ فایل‌هایی که با الگوهای
  [`tool.uv.build-backend.source-include`](../reference/settings.md#build-backend_source-include) مطابقت دارند.

از میان این موارد، مواردی که با
[`tool.uv.build-backend.source-exclude`](../reference/settings.md#build-backend_source-exclude) و
[حذف‌های پیش‌فرض](../reference/settings.md#build-backend_default-excludes) مطابقت دارند حذف می‌شوند.

هنگام ساخت یک wheel، فایل‌ها و پوشه‌های زیر گنجانده می‌شوند:

- [module](#modules) زیر
  [`tool.uv.build-backend.module-root`](../reference/settings.md#build-backend_module-root)
- فایل‌هایی که توسط `project.license-files` ارجاع داده شده‌اند و در پوشهٔ `.dist-info` کپی می‌شوند.
- فایل `project.readme` که در metadata پروژه کپی می‌شود.
- همهٔ پوشه‌های زیر [`tool.uv.build-backend.data`](../reference/settings.md#build-backend_data)،
  که در پوشهٔ `.data` کپی می‌شوند.

از میان این موارد،
[`tool.uv.build-backend.source-exclude`](../reference/settings.md#build-backend_source-exclude)،
[`tool.uv.build-backend.wheel-exclude`](../reference/settings.md#build-backend_wheel-exclude) و
حذف‌های پیش‌فرض برداشته می‌شوند. حذف‌های مربوط به source distribution اعمال می‌شوند تا یک wheel
که مستقیماً از source tree ساخته شده است، فایل‌های بیشتری نسبت به یک wheel ساخته‌شده از یک source
distribution نداشته باشد.

هیچ include خاصی برای wheel وجود ندارد. فقط باید یک module سطح بالا وجود داشته باشد و همهٔ فایل‌های داده یا باید زیر ریشهٔ module باشند یا در [پوشهٔ دادهٔ مناسب](../reference/settings.md#build-backend_data). بیشتر packageها داده‌های کوچک را در ریشهٔ module در کنار کد منبع ذخیره می‌کنند.

!!! tip

    هنگام استفاده از build backend مربوط به uv از طریق یک frontend که uv نیست، مانند pip یا
    `python -m build`، می‌توان ثبت لاگ دیباگ را از طریق environment variableها با
    `RUST_LOG=uv=debug` یا `RUST_LOG=uv=verbose` فعال کرد. هنگام استفاده از طریق uv، build backend
    مربوط به uv سطح جزئیات (verbosity) uv را به اشتراک می‌گذارد.

### سینتکس include و exclude {#include-and-exclude-syntax}

includeها anchored هستند، یعنی `pyproject.toml` فقط `<root>/pyproject.toml` را شامل می‌شود و نه `<root>/bar/pyproject.toml`. برای گنجاندن بازگشتی همهٔ فایل‌های زیر یک پوشه، از پسوند `/**` استفاده کنید، مثلاً `src/**`. گنجاندن‌های بازگشتی هم anchored هستند؛ مثلاً `assets/**/sample.csv` همهٔ فایل‌های `sample.csv` را در `<root>/assets` یا هر یک از فرزندان آن شامل می‌شود.

!!! note

    برای عملکرد و تکرارپذیری، از الگوهای بدون anchor مانند `**/sample.csv` خودداری کنید.

excludeها anchored نیستند، یعنی `__pycache__` همهٔ پوشه‌های با نام `__pycache__` را حذف می‌کند، صرف‌نظر از پوشهٔ والد آن. همهٔ فرزندان یک حذف هم حذف می‌شوند. برای anchor کردن یک پوشه، از پیشوند `/` استفاده کنید؛ مثلاً `/dist` فقط `<root>/dist` را حذف می‌کند.

همهٔ فیلدهایی که الگو می‌پذیرند از سینتکس glob قابل‌حملِ کاهش‌یافتهٔ [PEP 639](https://peps.python.org/pep-0639/#add-license-FILES-key) استفاده می‌کنند، با این اضافه که می‌توان کاراکترها را با یک backslash escape کرد.
