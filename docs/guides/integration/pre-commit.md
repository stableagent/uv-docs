---
title: استفاده از uv با pre-commit
description:
  راهنمای استفاده از uv با pre-commit برای به‌روزرسانی خودکار lock fileها، export کردن requirements و
  compile کردن فایل‌های requirements.
---

# استفاده از uv در pre-commit {#using-uv-in-pre-commit}

یک hook رسمی pre-commit در
[`astral-sh/uv-pre-commit`](https://github.com/astral-sh/uv-pre-commit) ارائه شده است.

برای استفاده از uv با pre-commit، یکی از مثال‌های زیر را به فهرست `repos` در فایل
`.pre-commit-config.yaml` اضافه کنید.

برای اینکه مطمئن شوید فایل `uv.lock` شما به‌روز است، حتی اگر فایل `pyproject.toml` تغییر کرده باشد:

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv version.
    rev: 0.13.9
    hooks:
      - id: uv-lock
```

برای همگام نگه داشتن فایل `requirements.txt` با فایل `uv.lock`:

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv version.
    rev: 0.13.9
    hooks:
      - id: uv-export
```

برای compile کردن فایل‌های requirements:

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv version.
    rev: 0.13.9
    hooks:
      # Compile requirements
      - id: pip-compile
        args: [requirements.in, -o, requirements.txt]
```

برای compile کردن فایل‌های requirements جایگزین، `args` و `files` را تغییر دهید:

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv version.
    rev: 0.13.9
    hooks:
      # Compile requirements
      - id: pip-compile
        args: [requirements-dev.in, -o, requirements-dev.txt]
        files: ^requirements-dev\.(in|txt)$
```

برای اجرای hook روی چند فایل به‌صورت هم‌زمان، ورودی‌های بیشتری اضافه کنید:

```yaml title=".pre-commit-config.yaml"
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    # uv version.
    rev: 0.13.9
    hooks:
      # Compile requirements
      - id: pip-compile
        name: pip-compile requirements.in
        args: [requirements.in, -o, requirements.txt]
      - id: pip-compile
        name: pip-compile requirements-dev.in
        args: [requirements-dev.in, -o, requirements-dev.txt]
        files: ^requirements-dev\.(in|txt)$
```
