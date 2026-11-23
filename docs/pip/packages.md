# مدیریت package‌ها {#managing-packages}

## نصب یک package {#installing-a-package}

برای نصب یک package در virtual environment، مثلاً Flask:

```console
$ uv pip install flask
```

برای نصب یک package با dependency‌های اختیاری فعال، مثلاً Flask با extra «dotenv»:

```console
$ uv pip install "flask[dotenv]"
```

برای نصب چند package، مثلاً Flask و Ruff:

```console
$ uv pip install flask ruff
```

برای نصب یک package با یک constraint، مثلاً Ruff نسخهٔ 0.2.0 یا جدیدتر:

```console
$ uv pip install 'ruff>=0.2.0'
```

برای نصب یک package در یک نسخهٔ مشخص، مثلاً Ruff نسخهٔ 0.3.0:

```console
$ uv pip install 'ruff==0.3.0'
```

برای نصب یک package از روی دیسک:

```console
$ uv pip install "ruff @ ./projects/ruff"
```

برای نصب یک package از GitHub:

```console
$ uv pip install "git+https://github.com/astral-sh/ruff"
```

برای نصب یک package از GitHub در یک مرجع مشخص:

```console
$ # Install a tag
$ uv pip install "git+https://github.com/astral-sh/ruff@v0.2.0"

$ # Install a commit
$ uv pip install "git+https://github.com/astral-sh/ruff@1fadefa67b26508cc59cf38e6130bde2243c929d"

$ # Install a branch
$ uv pip install "git+https://github.com/astral-sh/ruff@main"
```

برای نصب از یک repository خصوصی، به مستندات [احراز هویت Git](../concepts/authentication/git.md) مراجعه کنید.

## package‌های قابل ویرایش {#editable-packages}

package‌های قابل ویرایش برای اینکه تغییرات کد منبع آن‌ها اعمال شود، نیازی به نصب دوباره ندارند.

برای نصب project فعلی به‌عنوان یک package قابل ویرایش:

```console
$ uv pip install -e .
```

برای نصب یک project در پوشه‌ای دیگر به‌عنوان یک package قابل ویرایش:

```console
$ uv pip install -e "ruff @ ./project/ruff"
```

## نصب package‌ها از فایل‌ها {#installing-packages-from-files}

می‌توان چند package را هم‌زمان از قالب‌های استاندارد فایل نصب کرد.

نصب از یک فایل `requirements.txt`:

```console
$ uv pip install -r requirements.txt
```

برای اطلاعات بیشتر دربارهٔ فایل‌های `requirements.txt`، به مستندات [`uv pip compile`](./compile.md) مراجعه کنید.

نصب از یک فایل `pyproject.toml`:

```console
$ uv pip install -r pyproject.toml
```

نصب از یک فایل `pyproject.toml` با dependency‌های اختیاری فعال، مثلاً extra «foo»:

```console
$ uv pip install -r pyproject.toml --extra foo
```

نصب از یک فایل `pyproject.toml` با همهٔ dependency‌های اختیاری فعال:

```console
$ uv pip install -r pyproject.toml --all-extras
```

برای نصب group‌های dependency در فایل `pyproject.toml` پوشهٔ project فعلی، مثلاً group `foo`:

```console
$ uv pip install --group foo
```

برای مشخص کردن پوشهٔ project که group‌ها باید از آن خوانده شوند:

```console
$ uv pip install --project some/path/ --group foo --group bar
```

همچنین می‌توانید برای هر group یک مسیر به فایل `pyproject.toml` مشخص کنید:

```console
$ uv pip install --group some/path/pyproject.toml:foo --group other/pyproject.toml:bar
```

!!! note

    مانند pip، فلگ‌های `--group` برای source‌های دیگری که با فلگ‌هایی مانند `-r` یا `-e` مشخص شده‌اند اعمال نمی‌شوند. برای مثال، `uv pip install -r some/path/pyproject.toml --group foo` مقدار `foo` را از `./pyproject.toml` می‌خواند و **نه** از `some/path/pyproject.toml`.

## حذف نصب یک package {#uninstalling-a-package}

برای حذف نصب یک package، مثلاً Flask:

```console
$ uv pip uninstall flask
```

برای حذف نصب چند package، مثلاً Flask و Ruff:

```console
$ uv pip uninstall flask ruff
```
