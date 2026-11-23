---
title: استفاده از uv با Jupyter
description:
  راهنمای کامل استفاده از uv با نوت‌بوک‌های Jupyter برای محاسبات تعاملی، تحلیل داده و
  مصورسازی، شامل مدیریت kernel و یکپارچه‌سازی virtual environment.
---

# استفاده از uv با Jupyter {#using-uv-with-jupyter}

نوت‌بوک [Jupyter](https://jupyter.org/) یک tool محبوب برای محاسبات تعاملی، تحلیل داده و مصورسازی
است. می‌توانید Jupyter را با uv به چند روش مختلف استفاده کنید، چه برای تعامل با یک project و چه
به‌عنوان یک tool مستقل.

## استفاده از Jupyter در یک project {#using-jupyter-within-a-project}

اگر در یک [project](../../concepts/projects/index.md) کار می‌کنید، می‌توانید یک سرور Jupyter را با
دسترسی به virtual environment مربوط به project، از طریق دستور زیر شروع کنید:

```console
$ uv run --with jupyter jupyter lab
```

به‌طور پیش‌فرض، `jupyter lab` سرور را در آدرس
[http://localhost:8888/lab](http://localhost:8888/lab) شروع می‌کند.

در یک نوت‌بوک، می‌توانید moduleهای project خود را مانند هر فایل دیگری در project، import کنید. برای
مثال، اگر project شما به `requests` وابسته است، دستور `import requests`، `requests` را از virtual
environment مربوط به project، import می‌کند.

اگر به دنبال دسترسی فقط-خواندنی به virtual environment مربوط به project هستید، همین کافی است. اما
اگر نیاز دارید packageهای بیشتری را از داخل نوت‌بوک نصب کنید، چند نکتهٔ دیگر هم باید در نظر گرفته
شود.

### ساختن یک kernel {#creating-a-kernel}

اگر نیاز دارید packageها را از داخل نوت‌بوک نصب کنید، توصیه می‌کنیم یک kernel اختصاصی برای project
خود بسازید. kernelها به سرور Jupyter امکان می‌دهند در یک environment اجرا شود، در حالی که
نوت‌بوک‌های جداگانه در environmentهای جداگانهٔ خودشان اجرا می‌شوند.

در زمینهٔ uv، می‌توانیم یک kernel برای یک project بسازیم، در حالی که خود Jupyter را در
یک environment ایزوله نصب می‌کنیم، مانند `uv run --with jupyter jupyter lab`. ساختن یک kernel برای
project تضمین می‌کند که نوت‌بوک به environment درست متصل است و هر packageای که از داخل نوت‌بوک نصب
شود، در virtual environment مربوط به project نصب می‌شود.

برای ساختن یک kernel، باید `ipykernel` را به‌عنوان یک development dependency نصب کنید:

```console
$ uv add --dev ipykernel
```

سپس، می‌توانید kernel را برای `project` با این دستور بسازید:

```console
$ uv run ipython kernel install --user --env VIRTUAL_ENV $(pwd)/.venv --name=project
```

از آنجا، سرور را با این دستور شروع کنید:

```console
$ uv run --with jupyter jupyter lab
```

هنگام ساختن یک نوت‌بوک، kernel مربوط به `project` را از منوی کشویی انتخاب کنید. سپس از دستور
`!uv add pydantic` برای افزودن `pydantic` به dependencyهای project، یا از دستور
`!uv pip install pydantic` برای نصب `pydantic` در virtual environment مربوط به project، بدون اینکه
تغییر در فایل‌های `pyproject.toml` یا `uv.lock` مربوط به project ماندگار شود، استفاده کنید. هر یک از
این دو دستور، کارکردن `import pydantic` را در نوت‌بوک ممکن می‌کند.

### نصب packageها بدون kernel {#installing-packages-without-a-kernel}

اگر نمی‌خواهید kernel بسازید، باز هم می‌توانید packageها را از داخل نوت‌بوک نصب کنید. با این حال،
چند نکتهٔ احتیاطی وجود دارد که باید در نظر گرفته شود.

اگرچه `uv run --with jupyter` در یک environment ایزوله اجرا می‌شود، اما در خود نوت‌بوک، دستورهای
`!uv add` و دستورهای مرتبط، environment مربوط به _project_ را تغییر می‌دهند، حتی بدون kernel.

برای مثال، اجرای `!uv add pydantic` از داخل یک نوت‌بوک، `pydantic` را به dependencyهای project و
virtual environment اضافه می‌کند، به‌طوری که `import pydantic` بلافاصله و بدون پیکربندی بیشتر یا
راه‌اندازی مجدد سرور، کار می‌کند.

با این حال، از آنجا که سرور Jupyter همان environment «فعال» است، دستور `!uv pip install`،
packageها را در environment مربوط به _Jupyter_ نصب می‌کند، نه environment مربوط به project. چنین
dependencyهایی در طول عمر سرور Jupyter باقی می‌مانند، اما ممکن است در فراخوانی‌های بعدی `jupyter`
ناپدید شوند.

اگر با نوت‌بوکی کار می‌کنید که به pip متکی است (مثلاً از طریق magic مربوط به `%pip`)، می‌توانید با
اجرای `uv venv --seed` پیش از شروع سرور Jupyter، pip را در virtual environment مربوط به project
بگنجانید. برای مثال، با فرض اینکه:

```console
$ uv venv --seed
$ uv run --with jupyter jupyter lab
```

فراخوانی‌های بعدی `%pip install` در نوت‌بوک، packageها را در virtual environment مربوط به project
نصب می‌کنند. با این حال، چنین تغییراتی در فایل‌های `pyproject.toml` یا `uv.lock` مربوط به project
منعکس _نمی‌شود_.

## استفاده از Jupyter به‌عنوان یک tool مستقل {#using-jupyter-as-a-standalone-tool}

اگر زمانی به دسترسی موردی به یک نوت‌بوک نیاز دارید (یعنی برای اجرای تعاملی یک قطعه‌کد
Python)، می‌توانید در هر زمان یک سرور Jupyter را با `uv tool run jupyter lab` شروع کنید. این کار یک
سرور Jupyter را در یک environment ایزوله اجرا می‌کند.

## استفاده از Jupyter با یک environment غیر project {#using-jupyter-with-a-non-project-environment}

اگر نیاز دارید Jupyter را در یک virtual environment اجرا کنید که به یک
[project](../../concepts/projects/index.md) مرتبط نیست (مثلاً فایل `pyproject.toml` یا `uv.lock`
ندارد)، می‌توانید این کار را با افزودن مستقیم Jupyter به environment انجام دهید. برای مثال:

=== "macOS and Linux"

    ```console
    $ uv venv --seed
    $ uv pip install pydantic
    $ uv pip install jupyterlab
    $ .venv/bin/jupyter lab
    ```

=== "Windows"

    ```pwsh-session
    PS> uv venv --seed
    PS> uv pip install pydantic
    PS> uv pip install jupyterlab
    PS> .venv\Scripts\jupyter lab
    ```

از اینجا، دستور `import pydantic` در نوت‌بوک کار می‌کند و می‌توانید packageهای بیشتری را از طریق
`!uv pip install` یا حتی `!pip install` نصب کنید.

## استفاده از Jupyter از VS Code {#using-jupyter-from-vs-code}

همچنین می‌توانید با نوت‌بوک‌های Jupyter از داخل یک ویرایشگر مانند VS Code کار کنید. برای اتصال یک
project مدیریت‌شده با uv به یک نوت‌بوک Jupyter در VS Code، توصیه می‌کنیم یک kernel برای project
بسازید، به این شکل:

```console
# Create a project.
$ uv init project

# Move into the project directory.
$ cd project

# Add ipykernel as a dev dependency.
$ uv add --dev ipykernel

# Open the project in VS Code.
$ code .
```

وقتی پوشهٔ project در VS Code باز شد، می‌توانید با انتخاب «Create: New Jupyter Notebook» از command
palette، یک نوت‌بوک Jupyter جدید بسازید. وقتی از شما خواسته شد یک kernel انتخاب کنید، گزینهٔ «Python
Environments» را انتخاب کنید و virtual environmentای را که پیش‌تر ساختید انتخاب کنید (مثلاً
`.venv/bin/python` در macOS و Linux، یا `.venv\Scripts\python` در Windows).

!!! note

    VS Code به وجود `ipykernel` در environment مربوط به project نیاز دارد. اگر ترجیح می‌دهید
    `ipykernel` را به‌عنوان یک dev dependency اضافه نکنید، می‌توانید آن را مستقیماً با دستور
    `uv pip install ipykernel` در environment مربوط به project نصب کنید.

اگر نیاز دارید environment مربوط به project را از داخل نوت‌بوک دست‌کاری کنید، ممکن است لازم باشد
`uv` را به‌عنوان یک development dependency صریح اضافه کنید:

```console
$ uv add --dev uv
```

از آنجا می‌توانید از دستور `!uv add pydantic` برای افزودن `pydantic` به dependencyهای project، یا
از دستور `!uv pip install pydantic` برای نصب `pydantic` در virtual environment مربوط به project،
بدون به‌روزرسانی فایل‌های `pyproject.toml` یا `uv.lock` مربوط به project، استفاده کنید.
