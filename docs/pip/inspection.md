# بازرسی environment‌ها {#inspecting-environments}

## فهرست کردن package‌های نصب‌شده {#listing-installed-packages}

برای فهرست کردن همهٔ package‌های موجود در environment:

```console
$ uv pip list
```

برای فهرست کردن package‌ها در قالب JSON:

```console
$ uv pip list --format json
```

برای فهرست کردن همهٔ package‌های موجود در environment در قالب `requirements.txt`:

```console
$ uv pip freeze
```

## بازرسی یک package {#inspecting-a-package}

برای نمایش اطلاعات دربارهٔ یک package نصب‌شده، مثلاً `numpy`:

```console
$ uv pip show numpy
```

می‌توان چند package را هم‌زمان بازرسی کرد.

## صحت‌سنجی یک environment {#verifying-an-environment}

ممکن است package‌هایی با requirement‌های متضاد در یک environment نصب شوند، اگر در چند مرحله نصب شوند.

برای بررسی تداخل‌ها یا dependency‌های گمشده در environment:

```console
$ uv pip check
```
