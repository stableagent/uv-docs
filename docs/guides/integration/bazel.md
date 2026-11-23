---
title: استفاده از uv با Bazel
description: استفاده از uv برای تأمین package resolution با Bazel
---

# استفاده از uv با Bazel {#using-uv-with-bazel}

برای گردش‌کارهای گسترده‌تر Bazel با uv، به
[راهنمای uv در `rules_py`](https://github.com/aspect-build/rules_py#dependency-resolution-with-uv) یا
[راهنمای uv در `rules_python`](https://rules-python.readthedocs.io/en/latest/pypi/lock.html#uv-pip-compile-bzlmod-only) مراجعه کنید.

## احراز هویت {#authentication}

Bazel نسخهٔ ۷ و جدیدتر از credential helperها با گزینهٔ `--credential_helper` پشتیبانی می‌کند. برای
استفاده از credentialهایی که uv برای fetchهای Bazel ذخیره کرده است، ابتدا uv را با سرویسی که فایل‌هایی
را میزبانی می‌کند که Bazel باید fetch کند، احراز هویت کنید:

```console
$ uv auth login https://packages.example.com
```

سپس، Bazel را طوری پیکربندی کنید که
[`uv auth helper`](../../concepts/authentication/cli.md#using-credentials-with-external-tools) را
برای میزبان‌های منطبق فراخوانی کند:

```text title=".bazelrc"
common --credential_helper=packages.example.com=%workspace%/bazel/uv-auth-helper
common --credential_helper=files.example.com=%workspace%/bazel/uv-auth-helper
```

الگوهای میزبان را با میزبان‌هایی جایگزین کنید که index و فایل‌هایی را سرو می‌کنند که Bazel قرار است
fetch کند.

در پایان، script مربوط به wrapper که در `.bazelrc` به آن اشاره شده است را اضافه کنید:

```bash title="bazel/uv-auth-helper"
#!/usr/bin/env bash
exec uv --preview-features auth-helper auth helper --protocol=bazel "$@"
```

این script باید قابل اجرا باشد:

```console
$ chmod +x bazel/uv-auth-helper
```
