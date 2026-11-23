---
title: ساخت و انتشار یک package
description: راهنمایی برای استفاده از uv جهت ساخت و انتشار packageهای Python در یک package index، مانند PyPI.
---

# ساخت و انتشار یک package {#building-and-publishing-a-package}

uv از ساخت packageهای Python به‌صورت source distribution و binary distribution از طریق `uv build` و بارگذاری آن‌ها در یک registry با `uv publish` پشتیبانی می‌کند.

## آماده‌سازی پروژهٔ شما {#preparing-your-project}

پیش از تلاش برای انتشار پروژهٔ خود، باید مطمئن شوید که آمادهٔ بسته‌بندی برای distribution است.

اگر پروژهٔ شما در `pyproject.toml` تعریفی برای `[build-system]` نداشته باشد، uv آن را در جریان عملیات `uv sync` در پروژه نمی‌سازد، بلکه در جریان `uv build` به build system قدیمی setuptools بازمی‌گردد.

!!! note

    پروژه‌هایی که با `uv init` ایجاد می‌شوند، به‌صورت پیش‌فرض یک تعریف `[build-system]` دارند.

ما به‌شدت توصیه می‌کنیم یک build system پیکربندی کنید. دربارهٔ build systemها در مستندات [پیکربندی پروژه](../concepts/projects/config.md#build-systems) بیشتر بخوانید.

## ساخت package شما {#building-your-package}

package خود را با `uv build` بسازید:

```console
$ uv build
```

به‌صورت پیش‌فرض، `uv build` پروژه را در پوشهٔ جاری می‌سازد و artifactهای ساخته‌شده را در یک زیرپوشهٔ `dist/` قرار می‌دهد.

همچنین، `uv build <SRC>` package را در پوشهٔ مشخص‌شده می‌سازد، در حالی که `uv build --package <PACKAGE>` package مشخص‌شده را در workspace جاری می‌سازد.

!!! info

    به‌صورت پیش‌فرض، `uv build` هنگام حل build dependencyها از بخش `build-system.requires` در `pyproject.toml`، به `tool.uv.sources` احترام می‌گذارد. هنگام انتشار یک package، توصیه می‌کنیم `uv build --no-sources` را اجرا کنید تا مطمئن شوید package هنگامی که `tool.uv.sources` غیرفعال است، به‌درستی ساخته می‌شود؛ همان‌طور که هنگام استفاده از ابزارهای ساخت دیگر، مانند [`pypa/build`](https://github.com/pypa/build)، چنین است.

## به‌روزرسانی نسخهٔ شما {#updating-your-version}

دستور `uv version` امکاناتی برای به‌روزرسانی نسخهٔ package شما پیش از انتشار آن فراهم می‌کند.
[برای خواندن نسخهٔ package خود، مستندات پروژه را ببینید](./projects.md#viewing-your-version).

برای به‌روزرسانی به یک نسخهٔ دقیق، آن را به‌عنوان یک آرگومان موقعیتی بدهید:

```console
$ uv version 1.0.0
hello-world 0.7.0 => 1.0.0
```

برای پیش‌نمایش تغییر بدون به‌روزرسانی `pyproject.toml`، از فلگ `--dry-run` استفاده کنید:

```console
$ uv version 2.0.0 --dry-run
hello-world 1.0.0 => 2.0.0
$ uv version
hello-world 1.0.0
```

برای افزایش یکی از اجزای نسخهٔ package، از گزینهٔ `--bump` استفاده کنید:

```console
$ uv version --bump minor
hello-world 1.2.3 => 1.3.0
```

گزینهٔ `--bump` از اجزای رایج نسخهٔ زیر پشتیبانی می‌کند: `major`، `minor`، `patch`، `stable`، `alpha`، `beta`، `rc`، `post` و `dev`. وقتی بیش از یک بار داده شود، اجزا به ترتیب، از بزرگ‌ترین (`major`) تا کوچک‌ترین (`dev`)، اعمال می‌شوند.

به‌صورت اختیاری می‌توانید با `--bump <component>=<value>` یک مقدار عددی بدهید تا جزء حاصل را به‌صراحت تعیین کنید:

```console
$ uv version --bump patch --bump dev=66463664
hello-world 0.0.1 => 0.0.2.dev66463664
```

برای رفتن از یک نسخهٔ پایدار به یک نسخهٔ پیش‌انتشار، یکی از اجزای major یا minor یا patch را علاوه بر جزء پیش‌انتشار افزایش دهید:

```console
$ uv version --bump patch --bump beta
hello-world 1.3.0 => 1.3.1b1
$ uv version --bump major --bump alpha
hello-world 1.3.0 => 2.0.0a1
```

هنگام رفتن از یک نسخهٔ پیش‌انتشار به یک نسخهٔ پیش‌انتشار جدید، فقط جزء پیش‌انتشار مربوط را افزایش دهید:

```console
$ uv version --bump beta
hello-world 1.3.0b1 => 1.3.0b2
```

هنگام رفتن از یک نسخهٔ پیش‌انتشار به یک نسخهٔ پایدار، می‌توان از گزینهٔ `stable` برای پاک کردن جزء پیش‌انتشار استفاده کرد:

```console
$ uv version --bump stable
hello-world 1.3.1b2 => 1.3.1
```

!!! info

    به‌صورت پیش‌فرض، وقتی `uv version` پروژه را تغییر می‌دهد، یک lock و sync انجام می‌دهد. برای جلوگیری از lock و sync، از `--frozen` استفاده کنید. برای جلوگیری فقط از sync، از `--no-sync` استفاده کنید.

## انتشار package شما {#publishing-your-package}

!!! note

    یک راهنمای کامل برای انتشار از GitHub Actions به PyPI را می‌توانید در
    [راهنمای GitHub](integration/github.md#publishing-to-pypi) پیدا کنید.

package خود را با `uv publish` منتشر کنید:

```console
$ uv publish
```

یک توکن PyPI را با `--token` یا `UV_PUBLISH_TOKEN` تنظیم کنید، یا یک نام کاربری را با `--username` یا `UV_PUBLISH_USERNAME` و یک رمز عبور را با `--password` یا `UV_PUBLISH_PASSWORD` تنظیم کنید. برای انتشار در PyPI از GitHub Actions یا یک Trusted Publisher دیگر، نیازی به تنظیم هیچ credentialای نیست. در عوض،
[یک trusted publisher به پروژهٔ PyPI اضافه کنید](https://docs.pypi.org/trusted-publishers/adding-a-publisher/).

هنگام استفاده از trusted publishing، uv تلاش می‌کند توکن کوتاه‌مدت PyPI را پس از انتشار باطل کند، حتی اگر انتشار ناموفق باشد. این کار دورهٔ در معرض بودن توکن کوتاه‌مدت را، فراتر از عمر کوتاهی که از پیش دارد، باز هم کاهش می‌دهد.

اگر باطل‌سازی ناموفق باشد، uv یک هشدار صادر می‌کند بدون اینکه نتیجهٔ انتشار را تغییر دهد. توکن‌هایی که به‌صراحت با `--token` یا `UV_PUBLISH_TOKEN` داده شده‌اند، لغو نمی‌شوند.

!!! note

    PyPI دیگر از انتشار با نام کاربری و رمز عبور پشتیبانی نمی‌کند؛ در عوض باید یک توکن ایجاد کنید. استفاده از یک توکن معادل تنظیم `--username __token__` و استفاده از توکن به‌عنوان رمز عبور است.

اگر از یک index سفارشی از طریق `[[tool.uv.index]]` استفاده می‌کنید، `publish-url` را اضافه کنید و از `uv publish --index <name>` استفاده کنید. برای نمونه:

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true
```

!!! note

    هنگام استفاده از `uv publish --index <name>`، فایل `pyproject.toml` باید وجود داشته باشد؛ یعنی باید در یک job انتشار در CI یک مرحلهٔ checkout داشته باشید.

با اینکه `uv publish` بارگذاری‌های ناموفق را دوباره تلاش می‌کند، ممکن است انتشار در میانهٔ کار ناموفق شود، به‌طوری که برخی فایل‌ها بارگذاری شده باشند و برخی فایل‌ها هنوز جا مانده باشند. با PyPI می‌توانید دقیقاً همان دستور را دوباره اجرا کنید؛ فایل‌های یکسانِ موجود نادیده گرفته می‌شوند. با registryهای دیگر، از `--check-url <index url>` با URL مربوط به index (نه URL انتشار) که packageها به آن تعلق دارند استفاده کنید. هنگام استفاده از `--index`، URL مربوط به index به‌عنوان check URL استفاده می‌شود. uv از بارگذاری فایل‌هایی که با فایل‌های موجود در registry یکسان‌اند صرف‌نظر می‌کند، و همچنین بارگذاری‌های موازیِ هم‌زمان را نیز مدیریت می‌کند. توجه کنید که فایل‌های موجود باید دقیقاً با فایل‌هایی که پیش‌تر در registry بارگذاری شده‌اند مطابقت داشته باشند. این کار از انتشار تصادفی source distribution و wheelهایی با محتوای متفاوت برای نسخهٔ یکسان جلوگیری می‌کند.

### بارگذاری attestationها همراه با package شما {#uploading-attestations-with-your-package}

!!! note

    برخی از package indexهای شخص ثالث ممکن است از attestationها پشتیبانی نکنند و بارگذاری‌هایی را که شامل آن‌ها هستند رد کنند (به‌جای اینکه بی‌صدا نادیده‌شان بگیرند).
    اگر هنگام بارگذاری با مشکلی روبه‌رو شدید، می‌توانید از `--no-attestations` یا
    `UV_PUBLISH_NO_ATTESTATIONS` برای غیرفعال کردن رفتار پیش‌فرض uv استفاده کنید.

!!! tip

    `uv publish` در حال حاضر attestation تولید نمی‌کند؛ attestationها باید پیش از انتشار به‌صورت جداگانه ایجاد شوند.

`uv publish` از بارگذاری [attestation](https://peps.python.org/pep-0740/) در registryهایی که از آن‌ها پشتیبانی می‌کنند، مانند PyPI، پشتیبانی می‌کند.

uv به‌صورت خودکار attestationها را پیدا و تطبیق می‌دهد. برای نمونه، با توجه به پوشهٔ `dist/` زیر، `uv publish` attestationها را همراه با distributionهای متناظرشان بارگذاری می‌کند:

```console
$ ls dist/
hello_world-1.0.0-py3-none-any.whl
hello_world-1.0.0-py3-none-any.whl.publish.attestation
hello_world-1.0.0.tar.gz
hello_world-1.0.0.tar.gz.publish.attestation
```

## نصب package شما {#installing-your-package}

با `uv run` بررسی کنید که package قابل نصب و import شدن است:

```console
$ uv run --with <PACKAGE> --no-project -- python -c "import <PACKAGE>"
```

از فلگ `--no-project` برای جلوگیری از نصب package از پوشهٔ پروژهٔ محلی خود استفاده می‌شود.

!!! tip

    اگر به‌تازگی package را نصب کرده‌اید، ممکن است لازم باشد گزینهٔ
    `--refresh-package <PACKAGE>` را اضافه کنید تا از استفاده از یک نسخهٔ cacheشدهٔ package جلوگیری شود.

## گام‌های بعدی {#next-steps}

برای یادگیری بیشتر دربارهٔ انتشار packageها، [راهنماهای PyPA](https://packaging.python.org/en/latest/guides/section-build-and-publish/) دربارهٔ ساخت و انتشار را ببینید.

یا ادامه دهید و [راهنماها](./integration/index.md) را دربارهٔ یکپارچه‌سازی uv با نرم‌افزارهای دیگر بخوانید.
