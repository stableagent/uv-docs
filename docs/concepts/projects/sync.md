# قفل کردن و همگام‌سازی {#locking-and-syncing}

قفل کردن فرایند [حل کردن](../resolution.md) dependencyهای project شما در یک
[lockfile](./layout.md#the-lockfile) است. همگام‌سازی فرایند نصب زیرمجموعه‌ای از packageها از
lockfile در [environment مربوط به project](./layout.md#the-project-environment) است.

## قفل و همگام‌سازی خودکار {#automatic-lock-and-sync}

قفل کردن و همگام‌سازی در uv _خودکار_ هستند. برای مثال، هنگامی که از `uv run` استفاده می‌شود، project پیش از فراخوانی دستور درخواستی قفل
و همگام می‌شود. این کار تضمین می‌کند که environment مربوط به project همیشه
به‌روز باشد. به‌طور مشابه، دستورهایی که lockfile را می‌خوانند، مانند `uv tree`، به‌طور خودکار
پیش از اجرا آن را به‌روز می‌کنند.

برای غیرفعال کردن قفل کردن خودکار، از گزینهٔ `--locked` استفاده کنید:

```console
$ uv run --locked ...
```

اگر lockfile به‌روز نباشد، uv به‌جای به‌روزرسانی lockfile، یک خطا ایجاد می‌کند.

برای استفاده از lockfile بدون بررسی به‌روز بودن آن، از گزینهٔ `--frozen` استفاده کنید:

```console
$ uv run --frozen ...
```

به‌طور مشابه، برای اجرای یک دستور بدون بررسی به‌روز بودن environment، از گزینهٔ `--no-sync`
استفاده کنید:

```console
$ uv run --no-sync ...
```

## بررسی lockfile {#checking-the-lockfile}

هنگام بررسی به‌روز بودن lockfile، uv بررسی می‌کند که آیا با metadata مربوط به project مطابقت دارد یا خیر.
برای مثال، اگر یک dependency به `pyproject.toml` خود اضافه کنید، lockfile
منسوخ در نظر گرفته می‌شود. به‌طور مشابه، اگر قیدهای نسخهٔ یک dependency را به‌گونه‌ای تغییر دهید که نسخهٔ قفل‌شده
حذف شود، lockfile منسوخ در نظر گرفته می‌شود. با این حال، اگر قیدهای نسخه را
به‌گونه‌ای تغییر دهید که نسخهٔ قفل‌شدهٔ موجود همچنان شامل شود، lockfile همچنان
به‌روز در نظر گرفته می‌شود.

می‌توانید با دادن فلگ `--check` به `uv lock` بررسی کنید که آیا lockfile به‌روز است یا خیر:

```console
$ uv lock --check
```

این معادل فلگ `--locked` برای دستورهای دیگر است.

!!! important

    uv وقتی نسخه‌های جدیدی از packageها منتشر می‌شوند، lockfileها را منسوخ در نظر نمی‌گیرد — اگر می‌خواهید dependencyها را ارتقا دهید، lockfile
    باید به‌طور صریح به‌روز شود. برای جزئیات، به مستندات مربوط به
    [ارتقای نسخه‌های قفل‌شدهٔ package](#upgrading-locked-package-versions) مراجعه کنید.

## ایجاد lockfile {#creating-the-lockfile}

در حالی که lockfile به‌صورت [خودکار](#automatic-lock-and-sync) ایجاد می‌شود، lockfile را می‌توان با استفاده از `uv lock` به‌طور صریح نیز
ایجاد یا به‌روز کرد:

```console
$ uv lock
```

## همگام‌سازی environment {#syncing-the-environment}

در حالی که environment به‌صورت [خودکار](#automatic-lock-and-sync) همگام می‌شود، می‌توان آن را با استفاده از `uv sync` به‌طور صریح نیز
همگام کرد:

```console
$ uv sync
```

همگام‌سازی دستی environment به‌ویژه برای اطمینان از اینکه ویرایشگر شما نسخه‌های درست
dependencyها را دارد، مفید است.

### نصب editable {#editable-installation}

هنگامی که environment همگام می‌شود، uv project (و دیگر memberهای workspace) را به‌صورت
packageهای _editable_ نصب می‌کند، به‌گونه‌ای که برای انعکاس تغییرات در
environment نیازی به همگام‌سازی مجدد نیست.

برای انصراف از این رفتار، از گزینهٔ `--no-editable` استفاده کنید.

!!! note

    اگر project یک build system تعریف نکند، نصب نخواهد شد.
    برای جزئیات، به مستندات [build systemها](./config.md#build-systems) مراجعه کنید.

### مدیریت packageهای اضافی {#handling-of-extraneous-packages}

`uv sync` به‌طور پیش‌فرض همگام‌سازی «دقیق» (exact) انجام می‌دهد، یعنی هر packageای را که برای نصب انتخاب نشده است، حذف می‌کند.

برای حفظ packageهای اضافی، از فلگ `--inexact` استفاده کنید:

```console
$ uv sync --inexact
```

در مقابل، `uv run` به‌طور پیش‌فرض از همگام‌سازی «غیردقیق» (inexact) استفاده می‌کند و تضمین می‌کند که همهٔ packageهای موردنیاز
نصب شده‌اند اما packageهای اضافی را حذف نمی‌کند. برای فعال کردن همگام‌سازی دقیق با `uv run`، از فلگ
`--exact` استفاده کنید:

```console
$ uv run --exact ...
```

### همگام‌سازی dependencyهای اختیاری {#syncing-optional-dependencies}

uv dependencyهای اختیاری را از جدول `[project.optional-dependencies]` می‌خواند. این‌ها
اغلب با نام «extra» شناخته می‌شوند.

uv به‌طور پیش‌فرض extraها را همگام نمی‌کند. برای گنجاندن یک extra، از گزینهٔ `--extra` استفاده کنید.

```console
$ uv sync --extra foo
```

برای فعال کردن سریع همهٔ extraها، از گزینهٔ `--all-extras` استفاده کنید.

برای جزئیات
نحوهٔ مدیریت dependencyهای اختیاری، به مستندات [dependencyهای اختیاری](./dependencies.md#optional-dependencies) مراجعه کنید.

### همگام‌سازی dependencyهای توسعه {#syncing-development-dependencies}

uv dependencyهای توسعه را از جدول `[dependency-groups]` می‌خواند (همان‌طور که در
[PEP 735](https://peps.python.org/pep-0735/) تعریف شده است).

group مربوط به `dev` به‌صورت خاص در نظر گرفته شده و به‌طور پیش‌فرض همگام می‌شود. برای جزئیات تغییر
پیش‌فرض‌ها، به مستندات
[groupهای پیش‌فرض](./dependencies.md#default-groups) مراجعه کنید.

می‌توان از فلگ `--no-dev` برای حذف group مربوط به `dev` استفاده کرد.

می‌توان از فلگ `--only-dev` برای نصب group مربوط به `dev` _بدون_ project و
dependencyهای آن استفاده کرد.

groupهای اضافی را می‌توان با گزینه‌های `--all-groups`، `--no-default-groups`،
`--group <name>`، `--only-group <name>` و `--no-group <name>` گنجاند یا حذف کرد. معنای
`--only-group` همان `--only-dev` است: project گنجانده نمی‌شود. با این حال،
`--only-group` groupهای پیش‌فرض را نیز حذف می‌کند.

حذف‌های group همیشه بر گنجاندن‌ها اولویت دارند، بنابراین با توجه به دستور:

```
$ uv sync --no-group foo --group foo
```

group مربوط به `foo` نصب نمی‌شود.

برای جزئیات
نحوهٔ مدیریت dependencyهای توسعه، به مستندات [dependencyهای توسعه](./dependencies.md#development-dependencies) مراجعه کنید.

## ارتقای نسخه‌های قفل‌شدهٔ package {#upgrading-locked-package-versions}

با وجود یک فایل `uv.lock`، uv هنگام اجرای `uv sync` و `uv lock` نسخه‌های قفل‌شدهٔ پیشین packageها را ترجیح می‌دهد. نسخه‌های package فقط در صورتی تغییر می‌کنند که قیدهای dependency مربوط به project نسخهٔ قفل‌شدهٔ پیشین را حذف کنند.

برای ارتقای همهٔ packageها:

```console
$ uv lock --upgrade
```

برای ارتقای یک package منفرد به آخرین نسخه، در حالی که نسخه‌های قفل‌شدهٔ همهٔ packageهای دیگر
حفظ می‌شوند:

```console
$ uv lock --upgrade-package <package>
```

برای ارتقای یک package منفرد به یک نسخهٔ مشخص:

```console
$ uv lock --upgrade-package <package>==<version>
```

در همهٔ موارد، ارتقاها به قیدهای dependency مربوط به project محدود می‌شوند. برای مثال، اگر
project یک حد بالایی برای یک package تعریف کند، ارتقا از آن نسخه فراتر نمی‌رود.

!!! note

    uv منطق مشابهی را برای dependencyهای Git اعمال می‌کند. برای مثال، اگر یک dependency مربوط به Git به
    شاخهٔ `main` اشاره کند، uv SHA مربوط به commit قفل‌شده در یک فایل `uv.lock` موجود را بر
    آخرین commit در شاخهٔ `main` ترجیح می‌دهد، مگر اینکه از فلگ‌های `--upgrade` یا `--upgrade-package`
    استفاده شود.

این فلگ‌ها را می‌توان به `uv sync` یا `uv run` نیز داد تا lockfile _و_ environment
به‌روز شوند.

## خروجی گرفتن از lockfile {#exporting-the-lockfile}

اگر نیاز دارید uv را با toolها یا workflowهای دیگر یکپارچه کنید، می‌توانید `uv.lock` را به قالب‌های مختلفی
از جمله `requirements.txt`، `pylock.toml` (PEP 751) و CycloneDX SBOM خروجی بگیرید.

```console
$ uv export --format requirements.txt
$ uv export --format pylock.toml
$ uv export --format cyclonedx1.5
```

برای مستندات جامع دربارهٔ همهٔ قالب‌های خروجی و موارد استفادهٔ آن‌ها، به [راهنمای خروجی گرفتن](./export.md) مراجعه کنید.

## نصب‌های جزئی {#partial-installations}

گاهی انجام نصب‌ها در چند مرحله مفید است، برای مثال برای بهینه‌سازی کش لایه‌ها
هنگام ساخت یک image مربوط به Docker. `uv sync` چندین فلگ برای این منظور دارد.

- `--no-install-project`: project فعلی را نصب نکنید
- `--no-install-workspace`: هیچ‌یک از memberهای workspace، از جمله project ریشه را نصب نکنید
- `--no-install-package <NO_INSTALL_PACKAGE>`: package(های) داده‌شده را نصب نکنید

هنگامی که از این گزینه‌ها استفاده می‌شود، همهٔ dependencyهای هدف همچنان نصب می‌شوند. برای مثال،
`--no-install-project` _project_ را حذف می‌کند اما هیچ‌یک از dependencyهای آن را حذف نمی‌کند.

اگر این فلگ‌ها به‌درستی استفاده نشوند، می‌توانند به یک environment خراب منجر شوند، زیرا ممکن است یک package
dependencyهای خود را نداشته باشد.

## بررسی‌های بدافزار {#malware-checks}

!!! important

    بررسی بدافزار هنگام sync در [پیش‌نمایش](../preview.md) است و تا زمان تثبیت، ممکن است تغییر کند.

هنگام همگام‌سازی، uv می‌تواند با بررسی lockfile شما در برابر [OSV](https://osv.dev)، یک اسکن سبک برای بدافزار شناخته‌شده انجام دهد. OSV به هشدارهای MAL از
[پایگاه دادهٔ packageهای مخرب](https://github.com/ossf/malicious-packages) مربوط به OpenSSF اشاره می‌کند.

اگر یک dependency قفل‌شده با یک هشدار بدافزار مطابقت داشته باشد، sync خاتمه می‌یابد.

برای فعال کردن بررسی‌های بدافزار، `audit.malware-check = true` را در تنظیمات uv خود تنظیم کنید یا
`UV_MALWARE_CHECK=1` را در environment خود تنظیم کنید.

برای استفاده از یک سرویس آسیب‌پذیری جایگزین، `audit.malware-check-url` را در تنظیمات uv خود تنظیم کنید یا
`UV_MALWARE_CHECK_URL` را در environment خود تنظیم کنید.
