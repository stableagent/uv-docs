# ساخت distributionها {#building-distributions}

برای توزیع project خود به دیگران (برای مثال، برای بارگذاری آن در یک index مانند PyPI)، باید آن را
به یک قالب قابل‌توزیع بسازید.

projectهای Python معمولاً هم به‌صورت source distributionها (sdist) و هم به‌صورت binary
distributionها (wheel) توزیع می‌شوند. مورد اول معمولاً یک فایل `.tar.gz` یا `.zip` است که شامل کد منبع project
همراه با مقداری metadata اضافی است، در حالی که مورد دوم یک فایل `.whl` است که شامل
artifactهای از پیش ساخته‌شده‌ای است که می‌توان آن‌ها را مستقیماً نصب کرد.

!!! important

    هنگام استفاده از `uv build`، uv به‌عنوان یک [build frontend](https://peps.python.org/pep-0517/#terminology-and-goals)
    عمل می‌کند و فقط نسخهٔ Python مورد استفاده را تعیین می‌کند و build backend را فراخوانی می‌کند. جزئیات
    buildها، مانند فایل‌های گنجانده‌شده و نام فایل‌های distribution، توسط build
    backend تعیین می‌شوند، همان‌طور که در [`[build-system]`](./config.md#build-systems) تعریف شده است. اطلاعات مربوط به پیکربندی build
    را می‌توان در مستندات tool مربوطه یافت.

## استفاده از `uv build` {#using-uv-build}

می‌توان از `uv build` برای ساخت هم source distributionها و هم binary distributionها برای project شما استفاده کرد.
به‌طور پیش‌فرض، `uv build` project را در پوشهٔ فعلی می‌سازد و artifactهای ساخته‌شده را در یک زیرپوشهٔ `dist/` قرار می‌دهد:

```console
$ uv build
$ ls dist/
example-0.1.0-py3-none-any.whl
example-0.1.0.tar.gz
```

می‌توانید project را در یک پوشهٔ متفاوت با دادن یک مسیر به `uv build` بسازید، برای مثال
`uv build path/to/project`.

`uv build` ابتدا یک source distribution می‌سازد و سپس یک binary distribution (wheel) را از
آن source distribution می‌سازد.

می‌توانید `uv build` را با `uv build --sdist` به ساخت یک source distribution، با `uv build --wheel` به ساخت یک binary
distribution، یا با
`uv build --sdist --wheel` به ساخت هر دو distribution از منبع محدود کنید.

## قیدهای build {#build-constraints}

`uv build` گزینهٔ `--build-constraint` را می‌پذیرد که می‌توان از آن برای محدود کردن نسخه‌های هر requirement مربوط به build
در طول فرایند build استفاده کرد. هنگامی که با `--require-hashes` همراه شود، uv اجبار می‌کند که
requirementهای مورد استفاده برای ساخت project با hashهای مشخص و شناخته‌شده مطابقت داشته باشند، برای تکرارپذیری.

برای مثال، با داشتن فایل `constraints.txt` زیر:

```text
setuptools==68.2.2 --hash=sha256:b454a35605876da60632df1a60f736524eb73cc47bbc9f3f1ef1b644de74fd2a
```

اجرای دستور زیر project را با نسخهٔ مشخص‌شدهٔ `setuptools` می‌سازد و بررسی
می‌کند که distribution دانلودشدهٔ `setuptools` با hash مشخص‌شده مطابقت دارد:

```console
$ uv build --build-constraint constraints.txt --require-hashes
```

برای الزام hash برای هر dependency مربوط به build، از جمله dependencyهای transitive، به‌جای آن از
`--require-build-hashes` استفاده کنید.

### hashهای dependency مربوط به build در project {#project-build-dependency-hashes}

برای مثال، برای بررسی dependencyهای build در طول resolution و نصب project، hashها را به
[`build-constraint-dependencies`](../../reference/settings.md#build-constraint-dependencies) در
`pyproject.toml` مربوط به workspace خود اضافه کنید:

```toml
[tool.uv]
build-constraint-dependencies = [
    { requirement = "setuptools==68.2.2", hashes = ["sha256:b454a35605876da60632df1a60f736524eb73cc47bbc9f3f1ef1b644de74fd2a"] },
    "wheel<1",
]
```

uv هنگام دانلود dependencyهای build که pin شده‌اند، hashهای ارائه‌شده را بررسی می‌کند. قیدهای بدون hash،
مانند `wheel<1`، نیز مجاز هستند. برای الزام hash برای هر dependency مربوط به build، از جمله
dependencyهای transitive،
[`require-build-hashes = true`](../../reference/settings.md#require-build-hashes) را زیر `[tool.uv]`
تنظیم کنید یا `--require-build-hashes` را بدهید.

باید هر dependency مربوط به build را به یک نسخهٔ دقیق ثابت کنید (یا از یک URL مستقیم استفاده کنید) و یک
hash ارائه دهید. hashها همچنین می‌توانند از fragmentهای URL در `build-system.requires` بیایند، اما نه از requirementهایی که
توسط یک build backend برگردانده می‌شوند.

هنگامی که build isolation غیرفعال است، uv از dependencyهای build نصب‌شده بدون بررسی hashهای آن‌ها استفاده می‌کند. uv همچنین packageهای از قبل نصب‌شده یا wheelهای ساخته‌شدهٔ پیشین را دوباره بررسی نمی‌کند. backend همراه `uv_build` به hash نیاز ندارد زیرا بخشی از فایل اجرایی uv است.

!!! note

    `--require-build-hashes` آزمایشی است. برای سرکوب هشدار، از `--preview-features build-dependency-hashes` استفاده کنید.

## جلوگیری از publish در PyPI {#preventing-publish-to-pypi}

اگر packageهای داخلی دارید که نمی‌خواهید منتشر شوند، می‌توانید آن‌ها را به‌عنوان خصوصی علامت‌گذاری کنید:

```toml
[project]
classifiers = ["Private :: Do Not Upload"]
```

این تنظیم باعث می‌شود PyPI package بارگذاری‌شدهٔ شما را از انتشار رد کند. این تنظیم بر تنظیمات امنیتی یا
حریم خصوصی در registryهای جایگزین تأثیری ندارد.

همچنین توصیه می‌کنیم فقط [توکن‌های API مربوط به PyPI برای هر project](https://pypi.org/help/#apitoken) را تولید کنید:
بدون یک توکن PyPI مطابق با project، نمی‌توان آن را به‌اشتباه منتشر کرد.
