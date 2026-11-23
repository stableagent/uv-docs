---
title: استفاده از toolها
description:
  راهنمای استفاده از uv برای اجرای toolهای منتشرشده به‌صورت packageهای Python، شامل فراخوانی‌های
  یک‌باره با uvx، درخواست نسخه‌های خاص tool، نصب toolها، ارتقای toolها و موارد دیگر.
---

# استفاده از toolها {#using-tools}

بسیاری از packageهای Python برنامه‌هایی ارائه می‌دهند که می‌توان از آن‌ها به‌عنوان tool استفاده کرد.
uv پشتیبانی تخصصی برای فراخوانی و نصب آسان toolها دارد.

## اجرای toolها {#running-tools}

دستور `uvx` یک tool را بدون نصب آن فراخوانی می‌کند.

برای مثال، برای اجرای `ruff`:

```console
$ uvx ruff
```

!!! note

    این دقیقاً معادل این است:

    ```console
    $ uv tool run ruff
    ```

    `uvx` به‌عنوان یک alias برای راحتی ارائه شده است.

می‌توان آرگومان‌ها را پس از نام tool ارائه داد:

```console
$ uvx pycowsay hello from uv

  -------------
< hello from uv >
  -------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||

```

هنگام استفاده از `uvx`، toolها در environmentهای موقت و ایزوله نصب می‌شوند.

!!! note

    اگر در حال اجرای یک tool در یک [_project_](../concepts/projects/index.md) هستید و tool نیاز دارد
    که project شما نصب شده باشد، مثلاً هنگام استفاده از `pytest` یا `mypy`، بهتر است به‌جای `uvx`
    از [`uv run`](./projects.md#running-commands) استفاده کنید. در غیر این صورت، tool در یک
    virtual environment اجرا می‌شود که از project شما ایزوله است.

    اگر project شما ساختار تخت دارد، مثلاً به‌جای استفاده از یک پوشهٔ `src` برای moduleها،
    خود project نیازی به نصب ندارد و `uvx` مناسب است. در این حالت، استفاده از `uv run` تنها زمانی
    مفید است که بخواهید نسخهٔ tool را در dependencyهای project پین کنید.

## دستورهایی با نام‌های package متفاوت {#commands-with-different-package-names}

وقتی `uvx ruff` فراخوانی می‌شود، uv package `ruff` را نصب می‌کند که دستور `ruff` را ارائه می‌دهد.
با این حال، گاهی نام package و دستور متفاوت است.

گزینهٔ `--from` می‌تواند برای فراخوانی یک دستور از یک package خاص استفاده شود، مثلاً `http` که
توسط `httpie` ارائه می‌شود:

```console
$ uvx --from httpie http
```

## درخواست نسخه‌های خاص {#requesting-specific-versions}

برای اجرای یک tool در یک نسخهٔ خاص، از `command@<version>` استفاده کنید:

```console
$ uvx ruff@0.3.0 check
```

برای اجرای یک tool در آخرین نسخه، از `command@latest` استفاده کنید:

```console
$ uvx ruff@latest check
```

گزینهٔ `--from` همچنین می‌تواند برای مشخص کردن نسخه‌های package استفاده شود، مانند بالا:

```console
$ uvx --from 'ruff==0.3.0' ruff check
```

یا، برای محدود کردن به یک بازه از نسخه‌ها:

```console
$ uvx --from 'ruff>0.2.0,<0.3.0' ruff check
```

توجه کنید که سینتکس `@` نمی‌تواند برای چیزی به‌جز یک نسخهٔ دقیق استفاده شود.

## درخواست extraها {#requesting-extras}

گزینهٔ `--from` می‌تواند برای اجرای یک tool با extraها استفاده شود:

```console
$ uvx --from 'mypy[faster-cache,reports]' mypy --xml-report mypy_report
```

این کار همچنین می‌تواند با انتخاب نسخه ترکیب شود:

```console
$ uvx --from 'mypy[faster-cache,reports]==1.13.0' mypy --xml-report mypy_report
```

## درخواست sourceهای مختلف {#requesting-different-sources}

گزینهٔ `--from` همچنین می‌تواند برای نصب از sourceهای جایگزین استفاده شود.

برای مثال، برای دریافت از git:

```console
$ uvx --from git+https://github.com/httpie/cli httpie
```

همچنین می‌توانید آخرین commit را از یک branch نام‌دار خاص دریافت کنید:

```console
$ uvx --from git+https://github.com/httpie/cli@master httpie
```

یا یک tag خاص را دریافت کنید:

```console
$ uvx --from git+https://github.com/httpie/cli@3.2.4 httpie
```

یا حتی یک commit خاص را:

```console
$ uvx --from git+https://github.com/httpie/cli@2843b87 httpie
```

یا با پشتیبانی [Git LFS](https://git-lfs.com):

```console
$ uvx --lfs --from git+https://github.com/astral-sh/lfs-cowsay lfs-cowsay
```

## دستورهایی با pluginها {#commands-with-plugins}

می‌توان dependencyهای اضافی را لحاظ کرد، مثلاً برای لحاظ کردن `mkdocs-material` هنگام اجرای
`mkdocs`:

```console
$ uvx --with mkdocs-material mkdocs --help
```

## نصب toolها {#installing-tools}

اگر یک tool زیاد استفاده می‌شود، مفید است که آن را در یک environment ماندگار نصب کنید و به `PATH`
اضافه کنید، به‌جای اینکه `uvx` را مکرراً فراخوانی کنید.

!!! tip

    `uvx` یک alias راحت برای `uv tool run` است. همهٔ دستورهای دیگر برای تعامل با toolها به پیشوند
    کامل `uv tool` نیاز دارند.

برای نصب `ruff`:

```console
$ uv tool install ruff
```

وقتی یک tool نصب می‌شود، فایل‌های اجرایی آن در یک پوشهٔ `bin` در `PATH` قرار می‌گیرند که امکان
اجرای tool را بدون uv فراهم می‌کند. اگر در `PATH` نباشد، یک هشدار نمایش داده می‌شود و می‌توان از
`uv tool update-shell` برای اضافه کردن آن به `PATH` استفاده کرد.

پس از نصب `ruff`، باید در دسترس باشد:

```console
$ ruff --version
```

برخلاف `uv pip install`، نصب یک tool، moduleهای آن را در environment فعلی در دسترس قرار نمی‌دهد.
برای مثال، دستور زیر با خطا مواجه می‌شود:

```console
$ python -c "import ruff"
```

این ایزوله‌سازی برای کاهش تعامل‌ها و تداخل‌ها میان dependencyهای toolها، scriptها و projectها مهم
است.

برخلاف `uvx`، دستور `uv tool install` روی یک _package_ عمل می‌کند و همهٔ فایل‌های اجرایی
ارائه‌شده توسط tool را نصب می‌کند.

برای مثال، مورد زیر فایل‌های اجرایی `http`، `https` و `httpie` را نصب می‌کند:

```console
$ uv tool install httpie
```

همچنین، می‌توان نسخه‌های package را بدون `--from` لحاظ کرد:

```console
$ uv tool install 'httpie>0.1.0'
```

و به همین ترتیب، برای sourceهای package:

```console
$ uv tool install git+https://github.com/httpie/cli
```

یا sourceهای package با [Git LFS](https://git-lfs.com):

```console
$ uv tool install --lfs git+https://github.com/astral-sh/lfs-cowsay
```

مانند `uvx`، نصب‌ها می‌توانند packageهای اضافی را نیز شامل شوند:

```console
$ uv tool install mkdocs --with mkdocs-material
```

می‌توان چند فایل اجرایی مرتبط را با هم در همان tool environment نصب کرد، با استفاده از فلگ
`--with-executables-from`. برای مثال، مورد زیر فایل‌های اجرایی `ansible` را به‌علاوهٔ مواردی که
توسط `ansible-core` و `ansible-lint` ارائه می‌شوند نصب می‌کند:

```console
$ uv tool install --with-executables-from ansible-core,ansible-lint ansible
```

## ارتقای toolها {#upgrading-tools}

برای ارتقای یک tool، از `uv tool upgrade` استفاده کنید:

```console
$ uv tool upgrade ruff
```

ارتقای tool به constraintهای نسخهٔ ارائه‌شده هنگام نصب tool احترام می‌گذارد. برای مثال،
`uv tool install ruff >=0.3,<0.4` و پس از آن `uv tool upgrade ruff`، Ruff را به آخرین نسخه در بازهٔ
`>=0.3,<0.4` ارتقا می‌دهد.

برای جایگزینی constraintهای نسخه، tool را با `uv tool install` دوباره نصب کنید:

```console
$ uv tool install ruff>=0.4
```

برای ارتقای همهٔ toolها:

```console
$ uv tool upgrade --all
```

## درخواست Python versionها {#requesting-python-versions}

به‌طور پیش‌فرض، uv هنگام اجرا، نصب یا ارتقای toolها از Python interpreter پیش‌فرض شما (اولین موردی
که پیدا می‌کند) استفاده می‌کند. می‌توانید Python interpreter مورداستفاده را با گزینهٔ `--python`
مشخص کنید.

برای مثال، برای درخواست یک Python version خاص هنگام اجرای یک tool:

```console
$ uvx --python 3.10 ruff
```

یا، هنگام نصب یک tool:

```console
$ uv tool install --python 3.10 ruff
```

یا، هنگام ارتقای یک tool:

```console
$ uv tool upgrade --python 3.10 ruff
```

برای جزئیات بیشتر دربارهٔ درخواست Python versionها، به صفحهٔ مفهومی
[Python version](../concepts/python-versions.md#requesting-a-version) مراجعه کنید.

## scriptهای قدیمی Windows {#legacy-windows-scripts}

toolها همچنین از اجرای
[scriptهای قدیمی setuptools](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#scripts)
پشتیبانی می‌کنند. این scriptها هنگام نصب از طریق `$(uv tool dir)\<tool-name>\Scripts` در دسترس
هستند.

در حال حاضر تنها scriptهای قدیمی با پسوندهای `.ps1`، `.cmd` و `.bat` پشتیبانی می‌شوند.

برای مثال، برای اجرای یک script در Command Prompt:

```console
$ uv tool run --from nuitka==2.6.7 nuitka.cmd --version
```

علاوه بر این، نیازی نیست پسوند را مشخص کنید. `uvx` به‌طور خودکار به‌جای شما فایل‌هایی را که به
`.ps1`، `.cmd` و `.bat` ختم می‌شوند، به همین ترتیب جستجو و اجرا می‌کند.

```console
$ uv tool run --from nuitka==2.6.7 nuitka --version
```

## گام‌های بعدی {#next-steps}

برای آشنایی بیشتر با مدیریت toolها با uv، به صفحهٔ مفهوم
[toolها](../concepts/tools.md) و [مرجع دستور](../reference/cli.md#uv-tool) مراجعه کنید.

یا، برای یادگیری [کار روی projectها](./projects.md) ادامه دهید.
