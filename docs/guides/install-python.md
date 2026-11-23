---
title: نصب و مدیریت Python
description:
  راهنمایی برای استفاده از uv جهت نصب Python، شامل درخواست نسخه‌های مشخص، نصب
  خودکار، مشاهدهٔ نسخه‌های نصب‌شده و موارد دیگر.
---

# نصب Python {#installing-python}

اگر Python از قبل روی سیستم شما نصب شده باشد، uv بدون هیچ پیکربندی آن را
[تشخیص می‌دهد و استفاده می‌کند](#using-existing-python-versions). با این حال، uv می‌تواند
نسخه‌های Python را نیز نصب و مدیریت کند. uv نسخه‌های گمشدهٔ Python را در صورت نیاز به‌صورت
[خودکار نصب می‌کند](#automatic-python-downloads) — برای شروع نیازی نیست Python را نصب کنید.

## شروع کار {#getting-started}

برای نصب آخرین نسخهٔ Python:

```console
$ uv python install
```

!!! note

    Python باینری‌های قابل‌توزیع رسمی منتشر نمی‌کند. به همین دلیل، uv از distributionهای پروژهٔ [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) شرکت Astral استفاده می‌کند. برای جزئیات بیشتر، مستندات [distributionهای Python](../concepts/python-versions.md#managed-python-distributions) را ببینید.

پس از نصب Python، دستورهای `uv` به‌صورت خودکار از آن استفاده می‌کنند. uv همچنین نسخهٔ نصب‌شده
را به `PATH` شما اضافه می‌کند:

```console
$ python3.15
```

uv به‌صورت پیش‌فرض فقط یک فایل اجرایی _نسخه‌دار_ نصب می‌کند. برای نصب فایل‌های اجرایی `python` و `python3`،
گزینهٔ آزمایشی `--default` را اضافه کنید:

```console
$ uv python install --default
```

!!! tip

    برای جزئیات بیشتر، مستندات مربوط به [نصب فایل‌های اجرایی Python](../concepts/python-versions.md#installing-python-executables) را ببینید.

## نصب یک نسخهٔ مشخص {#installing-a-specific-version}

برای نصب یک نسخهٔ مشخص از Python:

```console
$ uv python install 3.12
```

برای نصب چند نسخهٔ Python:

```console
$ uv python install 3.11 3.12
```

برای نصب یک پیاده‌سازی جایگزین Python، مثلاً PyPy:

```console
$ uv python install pypy@3.10
```

برای جزئیات بیشتر، مستندات [`python install`](../concepts/python-versions.md#installing-a-python-version) را
ببینید.

## نصب مجدد Python {#reinstalling-python}

برای نصب مجدد نسخه‌های Python مدیریت‌شده با uv، از `--reinstall` استفاده کنید، مثلاً:

```console
$ uv python install --reinstall
```

با این کار، همهٔ نسخه‌های Python که قبلاً نصب شده‌اند دوباره نصب می‌شوند. بهبودها به‌طور مداوم
به distributionهای Python اضافه می‌شوند، پس نصب مجدد ممکن است حتی اگر نسخهٔ Python تغییر نکند،
باگ‌ها را برطرف کند.

## مشاهدهٔ نصب‌های Python {#viewing-python-installations}

برای مشاهدهٔ نسخه‌های Python موجود و نصب‌شده:

```console
$ uv python list
```

برای جزئیات بیشتر، مستندات [`python list`](../concepts/python-versions.md#viewing-available-python-versions)
را ببینید.

## دانلودهای خودکار Python {#automatic-python-downloads}

برای استفاده از uv نیازی نیست Python به‌صورت صریح نصب شده باشد. به‌صورت پیش‌فرض، uv هنگامی که
نسخه‌های Python مورد نیاز باشند، آن‌ها را به‌صورت خودکار دانلود می‌کند. برای مثال، دستور زیر در صورت
نصب نبودن، Python 3.12 را دانلود می‌کند:

```console
$ uvx python@3.12 -c "print('hello world')"
```

حتی اگر نسخهٔ مشخصی از Python درخواست نشده باشد، uv آخرین نسخه را در صورت تقاضا دانلود می‌کند.
برای مثال، اگر هیچ نسخه‌ای از Python روی سیستم شما نباشد، دستور زیر پیش از ایجاد یک virtual environment
جدید، Python را نصب می‌کند:

```console
$ uv venv
```

!!! tip

    اگر می‌خواهید کنترل بیشتری بر زمان دانلود Python داشته باشید، دانلودهای خودکار Python را می‌توان به‌راحتی [غیرفعال کرد](../concepts/python-versions.md#disabling-automatic-python-downloads).

<!-- TODO(zanieb): Restore when Python shim management is added
Note that when an automatic Python installation occurs, the `python` command will not be added to the shell. Use `uv python install-shim` to ensure the `python` shim is installed.
-->

## استفاده از نسخه‌های موجود Python {#using-existing-python-versions}

اگر نصب‌های موجود Python روی سیستم شما وجود داشته باشند، uv از آن‌ها استفاده می‌کند. برای این رفتار
به هیچ پیکربندی‌ای نیاز نیست: uv در صورتی از Python سیستم استفاده می‌کند که نیازمندی‌های فراخوانی
دستور را برآورده کند. برای جزئیات، مستندات
[کشف Python](../concepts/python-versions.md#discovery-of-python-versions) را ببینید.

برای اینکه uv را مجبور کنید از Python سیستم استفاده کند، فلگ `--no-managed-python` را بدهید. برای
جزئیات بیشتر، مستندات
[ترجیح نسخهٔ Python](../concepts/python-versions.md#requiring-or-disabling-managed-python-versions)
را ببینید.

## ارتقای نسخه‌های Python {#upgrading-python-versions}

برای ارتقای یک نسخهٔ Python به آخرین انتشار اصلاحی (patch) پشتیبانی‌شده:

```console
$ uv python upgrade 3.12
```

برای ارتقای همهٔ نسخه‌های Python مدیریت‌شده با uv:

```console
$ uv python upgrade
```

برای جزئیات بیشتر، مستندات [`python upgrade`](../concepts/python-versions.md#upgrading-python-versions)
را ببینید.

## گام‌های بعدی {#next-steps}

برای آشنایی بیشتر با `uv python`، صفحهٔ [مفهوم نسخهٔ Python](../concepts/python-versions.md)
و [مرجع دستور](../reference/cli.md#uv-python) را ببینید.

یا برای یادگیری [اجرای scriptها](./scripts.md) و فراخوانی Python با uv، ادامه دهید.
