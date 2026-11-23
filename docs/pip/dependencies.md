# اعلام dependency‌ها {#declaring-dependencies}

بهترین روش این است که dependency‌ها را در یک فایل ایستا اعلام کنید، به‌جای اینکه environment‌ها را با نصب‌های موردی تغییر دهید. وقتی dependency‌ها تعریف شدند، می‌توان آن‌ها را [قفل](./compile.md) کرد تا یک environment سازگار و قابل بازتولید ساخته شود.

## استفاده از `pyproject.toml` {#using-pyprojecttoml}

فایل `pyproject.toml` استاندارد Python برای تعریف پیکربندی یک project است.

برای تعریف dependency‌های project در یک فایل `pyproject.toml`:

```toml title="pyproject.toml"
[project]
dependencies = [
  "httpx",
  "ruff>=0.3.0"
]
```

برای تعریف dependency‌های اختیاری در یک فایل `pyproject.toml`:

```toml title="pyproject.toml"
[project.optional-dependencies]
cli = [
  "rich",
  "click",
]
```

هر یک از کلیدها یک «extra» تعریف می‌کند که می‌توان آن را با فلگ‌های `--extra` و `--all-extras` یا نحو `package[<extra>]` نصب کرد. برای جزئیات بیشتر، به مستندات [نصب package‌ها](./packages.md#installing-packages-from-files) مراجعه کنید.

برای جزئیات بیشتر دربارهٔ شروع کار با `pyproject.toml`، به [راهنمای رسمی `pyproject.toml`](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) مراجعه کنید.

## استفاده از `requirements.in` {#using-requirementsin}

همچنین رایج است که از یک [قالب فایل requirements](https://pip.pypa.io/en/stable/reference/requirements-file-format/) سبک برای اعلام dependency‌های project استفاده شود. هر requirement در یک خط جداگانه تعریف می‌شود. معمولاً این فایل `requirements.in` نامیده می‌شود تا از `requirements.txt` که برای dependency‌های قفل‌شده استفاده می‌شود، متمایز شود.

برای تعریف dependency‌ها در یک فایل `requirements.in`:

```requirements title="requirements.in"
httpx
ruff>=0.3.0
```

group‌های dependency اختیاری در این قالب پشتیبانی نمی‌شوند.
