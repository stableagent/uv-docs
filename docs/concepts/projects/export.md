---
title: خروجی گرفتن از یک lockfile
description: خروجی گرفتن از یک lockfile به قالب‌های مختلف
---

# خروجی گرفتن از یک lockfile {#exporting-a-lockfile}

uv می‌تواند یک lockfile را برای یکپارچه‌سازی با toolها و workflowهای دیگر به قالب‌های مختلف خروجی بگیرد. دستور
`uv export` از چندین قالب خروجی پشتیبانی می‌کند که هر کدام برای موارد استفادهٔ متفاوتی مناسب‌اند.

برای جزئیات بیشتر دربارهٔ lockfileها و نحوهٔ ایجاد آن‌ها، به مستندات [چیدمان project](./layout.md) و
[قفل کردن و همگام‌سازی](./sync.md) مراجعه کنید.

## مروری بر قالب‌های خروجی {#overview-of-export-formats}

uv از سه قالب خروجی پشتیبانی می‌کند:

- `requirements.txt`: قالب سنتی و سازگار با pip برای
  [فایل requirement](https://pip.pypa.io/en/stable/reference/requirements-file-format/).
- `pylock.toml`: قالب استانداردشدهٔ lockfile Python که در
  [PEP 751](https://peps.python.org/pep-0751/) تعریف شده است.
- `CycloneDX`: یک قالب استاندارد صنعتی برای [Software Bill of Materials (SBOM)](https://cyclonedx.org/).

می‌توان قالب را با فلگ `--format` مشخص کرد:

```console
$ uv export --format requirements.txt
$ uv export --format pylock.toml
$ uv export --format cyclonedx1.5
```

!!! tip

    به‌طور پیش‌فرض، `uv export` در stdout چاپ می‌کند. برای نوشتن در یک فایل برای هر قالب، از `--output-file` استفاده کنید:

    ```console
    $ uv export --format requirements.txt --output-file requirements.txt
    $ uv export --format pylock.toml --output-file pylock.toml
    $ uv export --format cyclonedx1.5 --output-file sbom.json
    ```

## قالب `requirements.txt` {#requirements-txt-format}

قالب `requirements.txt` پرکاربردترین قالب برای dependencyهای Python است. می‌توان از آن
با `pip` و دیگر package managerهای Python استفاده کرد.

### استفادهٔ پایه {#basic-usage}

```console
$ uv export --format requirements.txt
```

سپس می‌توان فایل `requirements.txt` تولیدشده را از طریق `uv pip install` نصب کرد، یا با toolهای دیگری
مانند `pip`.

!!! note

    به‌طور کلی، توصیه می‌کنیم هم‌زمان از یک `uv.lock` و یک فایل `requirements.txt` استفاده نکنید.
    قالب `uv.lock` قدرتمندتر است و شامل ویژگی‌هایی است که نمی‌توان آن‌ها را در
    `requirements.txt` بیان کرد. اگر متوجه شدید که از یک فایل `uv.lock` خروجی می‌گیرید، در نظر بگیرید که یک issue باز کنید
    تا دربارهٔ مورد استفادهٔ خود صحبت کنید.

## قالب `pylock.toml` {#pylock-toml-format}

[PEP 751](https://peps.python.org/pep-0751/) یک قالب lockfile مبتنی بر TOML را برای dependencyهای
Python تعریف می‌کند. uv می‌تواند lockfile مربوط به dependency مربوط به project شما را به این قالب خروجی بگیرد.

### استفادهٔ پایه {#basic-usage-1}

```console
$ uv export --format pylock.toml
```

مسیرهای local نسبی نسبت به پوشهٔ فایل خروجی نوشته می‌شوند. هنگام نوشتن در stdout،
آن‌ها نسبت به پوشهٔ کاری فعلی هستند. هنگام نوشتن یک `pylock.toml`
در پوشه‌ای دیگر، از `--output-file` استفاده کنید.

## قالب CycloneDX SBOM {#cyclonedx-sbom-format}

uv می‌تواند lockfile مربوط به dependency مربوط به project شما را به‌عنوان یک Software Bill of Materials (SBOM) در قالب CycloneDX
خروجی بگیرد. SBOMها یک فهرست جامع از همهٔ اجزای نرم‌افزاری در application شما ارائه می‌دهند،
که برای ممیزی امنیتی، انطباق و شفافیت زنجیرهٔ تأمین مفید است.

!!! important

    پشتیبانی از خروجی گرفتن به CycloneDX در [پیش‌نمایش](../preview.md) است و ممکن است در هر نسخهٔ آینده تغییر کند.

### CycloneDX چیست؟ {#what-is-cyclonedx}

[CycloneDX](https://cyclonedx.org/) یک قالب استاندارد صنعتی برای ایجاد Software Bill of
Materials است. CycloneDX قابل خواندن توسط ماشین است و به‌طور گسترده توسط toolهای اسکن امنیتی،
پایگاه‌های دادهٔ آسیب‌پذیری و پلتفرم‌های Software Composition Analysis (SCA) پشتیبانی می‌شود.

### استفادهٔ پایه {#basic-usage-2}

برای خروجی گرفتن از lockfile مربوط به project خود به‌عنوان یک CycloneDX SBOM:

```console
$ uv export --format cyclonedx1.5
```

این کار یک سند CycloneDX نسخهٔ ۱٫۵ با کدگذاری JSON تولید می‌کند که شامل project شما و همهٔ
dependencyهای آن است.

### ساختار SBOM {#sbom-structure}

SBOM تولیدشده از
[مشخصات CycloneDX](https://cyclonedx.org/specification/overview/) پیروی می‌کند. uv همچنین ویژگی‌های
سفارشی زیر را روی componentها شامل می‌شود:

- `uv:package:marker`: Environment markerها (برای مثال `python_version >= "3.8"`)
- `uv:workspace:path`: مسیر نسبی برای memberهای workspace

## گام‌های بعدی {#next-steps}

برای یادگیری بیشتر دربارهٔ lockfileها و خروجی گرفتن، به مستندات [قفل کردن و همگام‌سازی](./sync.md)
و [مرجع دستور](../../reference/cli.md#uv-export) مراجعه کنید.

یا، برای یادگیری نحوهٔ
[ساخت و انتشار project خود در یک package index](../../guides/package.md) ادامهٔ مطلب را بخوانید.
