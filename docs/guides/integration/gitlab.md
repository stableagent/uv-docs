---
title: استفاده از uv در GitLab CI/CD
description: راهنمای استفاده از uv در GitLab CI/CD، شامل نصب، راه‌اندازی Python، نصب dependencyها و موارد دیگر.
---

# استفاده از uv در GitLab CI/CD {#using-uv-in-gitlab-cicd}

## استفاده از ایمیج uv {#using-the-uv-image}

Astral [ایمیج‌های Docker](docker.md#available-images) را با uv از پیش نصب‌شده ارائه می‌دهد.
از یک نوع (variant) دارای shell استفاده کنید، مثل ایمیج مبتنی بر Debian زیر، تا GitLab Runner بتواند
`script` مربوط به job را اجرا کند.

```yaml title=".gitlab-ci.yml"
variables:
  UV_VERSION: "0.13.9"
  PYTHON_VERSION: "3.12"
  BASE_LAYER: trixie-slim
  # GitLab CI creates a separate mountpoint for the build directory,
  # so we need to copy instead of using hard links.
  UV_LINK_MODE: copy

uv:
  image: ghcr.io/astral-sh/uv:$UV_VERSION-python$PYTHON_VERSION-$BASE_LAYER
  script:
    # your `uv` commands
```

## کش کردن {#caching}

پایدار نگه داشتن cache مربوط به uv بین اجراهای workflow می‌تواند عملکرد را بهبود دهد.

```yaml
uv-install:
  variables:
    UV_CACHE_DIR: .uv-cache
  cache:
    - key:
        files:
          - uv.lock
      paths:
        - $UV_CACHE_DIR
  script:
    # Your `uv` commands
  after_script:
    - uv cache prune --ci
```

برای جزئیات بیشتر دربارهٔ پیکربندی کش کردن، به [مستندات کش کردن در GitLab](https://docs.gitlab.com/ee/ci/caching/) مراجعه کنید.

استفاده از `uv cache prune --ci` در پایان job برای کاهش اندازهٔ cache توصیه می‌شود. برای جزئیات
بیشتر، به [مستندات cache در uv](../../concepts/cache.md#caching-in-continuous-integration) مراجعه کنید.

## استفاده از `uv pip` {#using-uv-pip}

اگر به‌جای رابط مربوط به project در uv از رابط مربوط به `uv pip` استفاده می‌کنید، uv
به‌طور پیش‌فرض به یک virtual environment نیاز دارد. برای اجازهٔ نصب packageها در system environment،
فلگ `--system` را در همهٔ فراخوانی‌های uv به کار ببرید یا متغیر `UV_SYSTEM_PYTHON` را تنظیم کنید.

متغیر `UV_SYSTEM_PYTHON` را می‌توان در دامنه‌های مختلف تعریف کرد. دربارهٔ نحوهٔ کار
[متغیرها و اولویت آن‌ها در GitLab، اینجا](https://docs.gitlab.com/ee/ci/variables/) بیشتر بخوانید.

با تعریف آن در سطح بالایی، برای کل workflow فعالش کنید:

```yaml title=".gitlab-ci.yml"
variables:
  UV_SYSTEM_PYTHON: 1

# [...]
```

برای غیرفعال‌کردن دوبارهٔ آن، می‌توان در هر فراخوانی uv از فلگ `--no-system` استفاده کرد.

هنگام پایدار نگه داشتن cache، شاید بخواهید به‌جای `uv.lock` از `requirements.txt` یا
`pyproject.toml` به‌عنوان فایل‌های کلید cache استفاده کنید.
