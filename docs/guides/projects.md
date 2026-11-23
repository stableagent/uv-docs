---
title: کار کردن روی پروژه‌ها
description:
  راهنمایی برای استفاده از uv جهت ایجاد و مدیریت پروژه‌های Python، از جمله افزودن dependencyها، اجرای
  دستورها و ساخت distributionهای قابل انتشار.
---

# کار کردن روی پروژه‌ها {#working-on-projects}

uv از مدیریت پروژه‌های Python پشتیبانی می‌کند؛ پروژه‌هایی که dependencyهای خود را در یک فایل `pyproject.toml` تعریف می‌کنند.

## ایجاد یک پروژهٔ جدید {#creating-a-new-project}

می‌توانید با دستور `uv init` یک پروژهٔ Python جدید ایجاد کنید:

```console
$ uv init hello-world
$ cd hello-world
```

همچنین می‌توانید یک پروژه را در پوشهٔ کاری فعلی مقداردهی اولیه کنید:

```console
$ mkdir hello-world
$ cd hello-world
$ uv init
```

uv فایل‌ها و پوشه‌های زیر را ایجاد می‌کند:

```text
├── .git/
├── .gitignore
├── .python-version
├── pyproject.toml
├── README.md
└── src
    └── hello_world
        └── __init__.py
```

فایل `pyproject.toml` یک entry point با نام `hello-world` تعریف می‌کند که به یک برنامهٔ سادهٔ «Hello world» در `__init__.py` اشاره دارد. آن را با `uv run` امتحان کنید:

```console
$ uv run hello-world
Hello from hello-world!
```

## ساختار پروژه {#project-structure}

یک پروژه از چند بخش مهم تشکیل شده است که با هم کار می‌کنند و به uv امکان می‌دهند پروژهٔ شما را مدیریت کند. علاوه بر فایل‌هایی که `uv init` ایجاد می‌کند، uv در اولین باری که یک دستور پروژه را اجرا می‌کنید، یعنی `uv run`، `uv sync` یا `uv lock`، یک virtual environment و یک فایل `uv.lock` در ریشهٔ پروژهٔ شما ایجاد می‌کند.

فهرست کامل به این شکل خواهد بود:

```text
.
├── .git/
├── .venv/
│   ├── bin
│   ├── lib
│   └── pyvenv.cfg
├── .gitignore
├── .python-version
├── README.md
├── src
│   └── hello_world
│       └── __init__.py
├── pyproject.toml
└── uv.lock
```

### `pyproject.toml` {#pyprojecttoml}

فایل `pyproject.toml` حاوی metadata دربارهٔ پروژهٔ شما است:

```toml title="pyproject.toml"
[project]
name = "hello-world"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [
  { name = "ferris", email = "ferris@example.org" }
]
requires-python = ">=3.14"
dependencies = []

[project.scripts]
hello-world = "hello_world:main"

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

از این فایل برای مشخص کردن dependencyها و همچنین جزئیاتی دربارهٔ پروژه، مانند description یا license آن، استفاده می‌کنید. می‌توانید این فایل را به‌صورت دستی ویرایش کنید، یا از دستورهایی مانند `uv add` و `uv remove` برای مدیریت پروژهٔ خود از ترمینال استفاده کنید.

!!! tip

    برای جزئیات بیشتر دربارهٔ شروع کار با قالب `pyproject.toml`، راهنمای رسمی [`pyproject.toml`](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) را ببینید.

از این فایل همچنین برای مشخص کردن گزینه‌های [پیکربندی](../concepts/configuration-files.md) uv در یک بخش [`[tool.uv]`](../reference/settings.md) استفاده می‌کنید.

### `.python-version` {#python-version}

فایل `.python-version` حاوی Python version پیش‌فرض پروژه است. این فایل به uv می‌گوید که هنگام ایجاد virtual environment پروژه از کدام Python version استفاده کند.

### `.venv` {#venv}

پوشهٔ `.venv` حاوی virtual environment پروژهٔ شما است؛ یک Python environment که از بقیهٔ سیستم شما جدا است. uv dependencyهای پروژهٔ شما را در همین‌جا نصب می‌کند.

برای جزئیات بیشتر، مستندات [project environment](../concepts/projects/layout.md#the-project-environment) را ببینید.

### `uv.lock` {#uvlock}

`uv.lock` یک lockfile چندپلتفرمی است که اطلاعات دقیق دربارهٔ dependencyهای پروژهٔ شما را در بر دارد. برخلاف `pyproject.toml` که برای مشخص کردن نیازمندی‌های کلی پروژهٔ شما به کار می‌رود، lockfile نسخه‌های دقیقِ حل‌شده‌ای را در بر دارد که در project environment نصب شده‌اند. این فایل باید در کنترل نسخه ثبت شود تا نصب‌های سازگار و قابل تکرار در ماشین‌های مختلف امکان‌پذیر باشد.

`uv.lock` یک فایل TOML خوانا برای انسان است، اما توسط uv مدیریت می‌شود و نباید به‌صورت دستی ویرایش شود.

برای جزئیات بیشتر، مستندات [lockfile](../concepts/projects/layout.md#the-lockfile) را ببینید.

## مدیریت dependencies {#managing-dependencies}

می‌توانید با دستور `uv add` dependencyهایی را به `pyproject.toml` خود اضافه کنید. این کار lockfile و project environment را نیز به‌روز می‌کند:

```console
$ uv add requests
```

همچنین می‌توانید version constraintها یا sourceهای جایگزین را مشخص کنید:

```console
$ # Specify a version constraint
$ uv add 'requests==2.31.0'

$ # Add a git dependency
$ uv add git+https://github.com/psf/requests
```

اگر از یک فایل `requirements.txt` مهاجرت می‌کنید، می‌توانید با فلگ `-r` در `uv add` همهٔ dependencyها را از آن فایل اضافه کنید:

```console
$ # Add all dependencies from `requirements.txt`.
$ uv add -r requirements.txt -c constraints.txt
```

برای حذف یک package، می‌توانید از `uv remove` استفاده کنید:

```console
$ uv remove requests
```

برای ارتقای یک package، دستور `uv lock` را با فلگ `--upgrade-package` اجرا کنید:

```console
$ uv lock --upgrade-package requests
```

فلگ `--upgrade-package` تلاش می‌کند package مشخص‌شده را به جدیدترین نسخهٔ سازگار به‌روز کند، در حالی که بقیهٔ lockfile دست‌نخورده باقی می‌ماند.

برای جزئیات بیشتر، مستندات [مدیریت dependencies](../concepts/projects/dependencies.md) را ببینید.

## مشاهدهٔ نسخهٔ خود {#viewing-your-version}

از دستور `uv version` می‌توان برای خواندن نسخهٔ package شما استفاده کرد.

برای گرفتن نسخهٔ package خود، دستور `uv version` را اجرا کنید:

```console
$ uv version
hello-world 0.7.0
```

برای گرفتن نسخه بدون نام package، از گزینهٔ `--short` استفاده کنید:

```console
$ uv version --short
0.7.0
```

برای گرفتن اطلاعات نسخه در قالب JSON، از گزینهٔ `--output-format json` استفاده کنید:

```console
$ uv version --output-format json
{
    "package_name": "hello-world",
    "version": "0.7.0",
    "commit_info": null
}
```

برای جزئیات مربوط به به‌روزرسانی نسخهٔ package خود، [راهنمای انتشار](./package.md#updating-your-version) را ببینید.

## اجرای دستورها {#running-commands}

از `uv run` می‌توان برای اجرای اسکریپت‌ها یا دستورهای دلخواه در project environment خود استفاده کرد.

پیش از هر فراخوانی `uv run`، uv بررسی می‌کند که lockfile با `pyproject.toml` به‌روز باشد و environment با lockfile به‌روز باشد، و پروژهٔ شما را بدون نیاز به دخالت دستی همگام نگه می‌دارد. `uv run` تضمین می‌کند که دستور شما در یک environment با همهٔ dependencyهای موردنیاز در نسخه‌های قفل‌شدهٔ آن‌ها اجرا شود.

!!! note

    `uv run` به‌صورت پیش‌فرض packageهای اضافی (آن‌هایی که در lockfile نیستند) را از environment حذف نمی‌کند. برای جزئیات، [برخورد با packageهای اضافی](../concepts/projects/sync.md#handling-of-extraneous-packages) را ببینید.

برای نمونه، برای استفاده از `flask`:

```console
$ uv add flask
$ uv run -- flask run -p 3000
```

یا برای اجرای یک script:

```python title="example.py"
# Require a project dependency
import flask

print("hello world")
```

```console
$ uv run example.py
```

همچنین می‌توانید از `uv sync` برای به‌روزرسانی دستی environment استفاده کنید و سپس پیش از اجرای یک دستور، آن را فعال کنید:

=== "macOS and Linux"

    ```console
    $ uv sync
    $ source .venv/bin/activate
    $ flask run -p 3000
    $ python example.py
    ```

=== "Windows"

    ```pwsh-session
    PS> uv sync
    PS> .venv\Scripts\activate
    PS> flask run -p 3000
    PS> python example.py
    ```

!!! note

    برای اجرای اسکریپت‌ها و دستورها در پروژه بدون `uv run`، باید virtual environment فعال باشد. فعال‌سازی virtual environment بسته به shell و platform متفاوت است.

برای جزئیات بیشتر، مستندات [اجرای دستورها و اسکریپت‌ها](../concepts/projects/run.md) در پروژه‌ها را ببینید.

## ساخت distributionها {#building-distributions}

از `uv build` می‌توان برای ساخت source distributionها و binary distributionها (wheel) برای پروژهٔ شما استفاده کرد.

به‌صورت پیش‌فرض، `uv build` پروژه را در پوشهٔ جاری می‌سازد و artifactهای ساخته‌شده را در یک زیرپوشهٔ `dist/` قرار می‌دهد:

```console
$ uv build
$ ls dist/
hello_world-0.1.0-py3-none-any.whl
hello_world-0.1.0.tar.gz
```

برای جزئیات بیشتر، مستندات [ساخت پروژه‌ها](../concepts/projects/build.md) را ببینید.

## گام‌های بعدی {#next-steps}

برای یادگیری بیشتر دربارهٔ کار کردن روی پروژه‌ها با uv، صفحهٔ [مفهوم پروژه‌ها](../concepts/projects/index.md) و [مرجع دستورها](../reference/cli.md#uv) را ببینید.

یا ادامه دهید تا یاد بگیرید چگونه یک [lockfile uv را به قالب‌های دیگر صادر کنید](../concepts/projects/export.md).
