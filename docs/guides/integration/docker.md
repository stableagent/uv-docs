---
title: استفاده از uv در Docker
description:
  راهنمای کامل استفاده از uv در Docker برای مدیریت dependencyهای Python، با بهینه‌سازی زمان build و
  اندازهٔ image از طریق buildهای چند مرحله‌ای، layerهای میانی و موارد دیگر.
---

# استفاده از uv در Docker {#using-uv-in-docker}

## شروع کار {#getting-started}

!!! tip

    برای نمونه‌ای از بهترین روش‌ها هنگام استفاده از uv برای build کردن یک برنامه در Docker، به
    project مربوط به [`uv-docker-example`](https://github.com/astral-sh/uv-docker-example) مراجعه
    کنید.

uv هم ایمیج‌های Docker مربوط به _distroless_ را ارائه می‌دهد که برای
[کپی‌کردن باینری‌های uv](#installing-uv) در buildهای ایمیج خودتان مفیدند، و هم ایمیج‌هایی مشتق‌شده
از ایمیج‌های پایهٔ محبوب را که برای استفاده از uv در یک کانتینر مفیدند. ایمیج‌های distroless چیزی جز
باینری‌های uv ندارند. در مقابل، ایمیج‌های مشتق‌شده شامل یک سیستم‌عامل با uv از پیش نصب‌شده‌اند.

به‌عنوان مثال، برای اجرای uv در یک کانتینر با استفاده از یک ایمیج مبتنی بر Debian:

```console
$ docker run --rm -it ghcr.io/astral-sh/uv:debian uv --help
```

### ایمیج‌های موجود {#available-images}

ایمیج‌های distroless زیر در دسترس‌اند:

- `ghcr.io/astral-sh/uv:latest`
- `ghcr.io/astral-sh/uv:{major}.{minor}.{patch}`، مثلاً `ghcr.io/astral-sh/uv:0.13.9`
- `ghcr.io/astral-sh/uv:{major}.{minor}`، مثلاً `ghcr.io/astral-sh/uv:0.13` (آخرین نسخهٔ patch)

و ایمیج‌های مشتق‌شدهٔ زیر در دسترس‌اند:

<!-- prettier-ignore-start -->

- مبتنی بر `alpine:3.23`:
    - `ghcr.io/astral-sh/uv:alpine`
    - `ghcr.io/astral-sh/uv:alpine3.23`
- مبتنی بر `alpine:3.22`:
    - `ghcr.io/astral-sh/uv:alpine3.22`
- مبتنی بر `debian:trixie-slim`:
    - `ghcr.io/astral-sh/uv:debian-slim`
    - `ghcr.io/astral-sh/uv:trixie-slim`
- مبتنی بر `buildpack-deps:trixie`:
    - `ghcr.io/astral-sh/uv:debian`
    - `ghcr.io/astral-sh/uv:trixie`
- مبتنی بر `dhi.io/alpine-base:3.23`:
    - `ghcr.io/astral-sh/uv:alpine-dhi`
    - `ghcr.io/astral-sh/uv:alpine3.23-dhi`
- مبتنی بر `dhi.io/debian-base:trixie-debian13`:
    - `ghcr.io/astral-sh/uv:debian-dhi`
    - `ghcr.io/astral-sh/uv:trixie-dhi`
- مبتنی بر `dhi/python:3.x`:
    - `ghcr.io/astral-sh/uv:python3.14-dhi`
    - `ghcr.io/astral-sh/uv:python3.13-dhi`
    - `ghcr.io/astral-sh/uv:python3.12-dhi`
    - `ghcr.io/astral-sh/uv:python3.11-dhi`
    - `ghcr.io/astral-sh/uv:python3.10-dhi`
- مبتنی بر `python3.x-alpine`:
    - `ghcr.io/astral-sh/uv:python3.15-alpine`
    - `ghcr.io/astral-sh/uv:python3.15-alpine3.23`
    - `ghcr.io/astral-sh/uv:python3.14-alpine`
    - `ghcr.io/astral-sh/uv:python3.14-alpine3.23`
    - `ghcr.io/astral-sh/uv:python3.13-alpine`
    - `ghcr.io/astral-sh/uv:python3.13-alpine3.23`
    - `ghcr.io/astral-sh/uv:python3.12-alpine`
    - `ghcr.io/astral-sh/uv:python3.12-alpine3.23`
    - `ghcr.io/astral-sh/uv:python3.11-alpine`
    - `ghcr.io/astral-sh/uv:python3.11-alpine3.23`
- مبتنی بر `python3.x-trixie`:
    - `ghcr.io/astral-sh/uv:python3.15-trixie`
    - `ghcr.io/astral-sh/uv:python3.14-trixie`
    - `ghcr.io/astral-sh/uv:python3.13-trixie`
    - `ghcr.io/astral-sh/uv:python3.12-trixie`
    - `ghcr.io/astral-sh/uv:python3.11-trixie`
- مبتنی بر `python3.x-slim-trixie`:
    - `ghcr.io/astral-sh/uv:python3.15-trixie-slim`
    - `ghcr.io/astral-sh/uv:python3.14-trixie-slim`
    - `ghcr.io/astral-sh/uv:python3.13-trixie-slim`
    - `ghcr.io/astral-sh/uv:python3.12-trixie-slim`
    - `ghcr.io/astral-sh/uv:python3.11-trixie-slim`

<!-- prettier-ignore-end -->

مانند ایمیج distroless، هر ایمیج مشتق‌شده با tagهای نسخهٔ uv به‌صورت
`ghcr.io/astral-sh/uv:{major}.{minor}.{patch}-{base}` و
`ghcr.io/astral-sh/uv:{major}.{minor}-{base}` منتشر می‌شود، مثلاً
`ghcr.io/astral-sh/uv:0.13.9-alpine`.

علاوه بر این، از نسخهٔ `0.8` به بعد، هر ایمیج مشتق‌شده، `UV_TOOL_BIN_DIR` را روی `/usr/local/bin`
تنظیم می‌کند تا `uv tool install` با کاربر پیش‌فرض، مطابق انتظار کار کند.

برای جزئیات بیشتر، به صفحهٔ [GitHub Container](https://github.com/astral-sh/uv/pkgs/container/uv)
مراجعه کنید.

### نصب uv {#installing-uv}

از یکی از ایمیج‌های بالا با uv از پیش نصب‌شده استفاده کنید، یا uv را با کپی‌کردن باینری از ایمیج
رسمی distroless مربوط به Docker نصب کنید:

```dockerfile title="Dockerfile"
FROM python:3.12-slim-trixie
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/
```

یا با نصب‌کننده:

```dockerfile title="Dockerfile"
FROM python:3.12-slim-trixie

# The installer requires curl (and certificates) to download the release archive
RUN apt-get update && apt-get install -y --no-install-recommends curl ca-certificates

# Download the latest installer
ADD https://astral.sh/uv/install.sh /uv-installer.sh

# Run the installer then remove it
RUN sh /uv-installer.sh && rm /uv-installer.sh

# Ensure the installed binary is on the `PATH`
ENV PATH="/root/.local/bin/:$PATH"
```

توجه کنید که این روش به در دسترس بودن `curl` نیاز دارد.

در هر دو حالت، بهترین روش این است که به یک نسخهٔ مشخص از uv پین کنید، مثلاً با:

```dockerfile
COPY --from=ghcr.io/astral-sh/uv:0.13.9 /uv /uvx /bin/
```

!!! tip

    با اینکه مثال Dockerfile در بالا به یک tag مشخص پین می‌کند، می‌توان به یک SHA256 مشخص هم پین
    کرد. پین‌کردن یک SHA256 مشخص در environmentهایی که به buildهای قابل بازتولید نیاز دارند، بهترین
    روش محسوب می‌شود، زیرا tagها می‌توانند روی SHAهای commit مختلف جابه‌جا شوند.

    ```Dockerfile
    COPY --from=ghcr.io/astral-sh/uv@sha256:32a7bb39b496babbd997b0a39a4d66c126bdaf5661b302bf6f06abda8deca6ff /uv /uvx /bin/
    ```

یا با نصب‌کننده:

```dockerfile
ADD https://astral.sh/uv/0.13.9/install.sh /uv-installer.sh
```

### نصب یک project {#installing-a-project}

اگر از uv برای مدیریت project خود استفاده می‌کنید، می‌توانید آن را در ایمیج کپی کنید و نصب کنید:

```dockerfile title="Dockerfile"
# Copy the project into the image
COPY . /app

# Disable development dependencies
ENV UV_NO_DEV=1

# Sync the project into a new environment, asserting the lockfile is up to date
WORKDIR /app
RUN uv sync --locked
```

!!! important

    بهترین روش این است که `.venv` را به یک [فایل `.dockerignore`](https://docs.docker.com/build/concepts/context/#dockerignore-files)
    در مخزن خود اضافه کنید تا از گنجانده‌شدن آن در buildهای ایمیج جلوگیری شود. virtual environment
    مربوط به project به platform محلی شما وابسته است و باید از صفر در ایمیج ساخته شود.

سپس، برای شروع برنامهٔ خود به‌صورت پیش‌فرض:

```dockerfile title="Dockerfile"
# Presuming there is a `my_app` command provided by the project
CMD ["uv", "run", "my_app"]
```

!!! tip

    بهترین روش این است که از [layerهای میانی](#intermediate-layers) استفاده کنید که نصب
    dependencyها و خود project را جدا می‌کنند تا زمان build ایمیج Docker بهبود یابد.

یک مثال کامل را در
[project مربوط به `uv-docker-example`](https://github.com/astral-sh/uv-docker-example/blob/main/Dockerfile) ببینید.

### استفاده از environment {#using-the-environment}

وقتی project نصب شد، می‌توانید یا virtual environment مربوط به project را با قرار دادن پوشهٔ
باینری آن در ابتدای مسیر، _فعال_ کنید:

```dockerfile title="Dockerfile"
ENV PATH="/app/.venv/bin:$PATH"
```

یا می‌توانید برای هر دستوری که به environment نیاز دارد، از `uv run` استفاده کنید:

```dockerfile title="Dockerfile"
RUN uv run some_script.py
```

!!! tip

    به‌جای آن، می‌توان
    [تنظیم `UV_PROJECT_ENVIRONMENT`](../../concepts/projects/config.md#project-environment-path) را
    پیش از sync کردن تنظیم کرد تا در system environment مربوط به Python نصب شود و فعال‌سازی
    environment به‌طور کامل نادیده گرفته شود.

### استفاده از toolهای نصب‌شده {#using-installed-tools}

برای استفاده از toolهای نصب‌شده، مطمئن شوید [پوشهٔ bin مربوط به tool](../../concepts/tools.md#tool-executables) در مسیر قرار دارد:

```dockerfile title="Dockerfile"
ENV PATH=/root/.local/bin:$PATH
RUN uv tool install cowsay
```

```console
$ docker run -it $(docker build -q .) /bin/bash -c "cowsay -t hello"
  _____
| hello |
  =====
     \
      \
        ^__^
        (oo)\_______
        (__)\       )\/\
            ||----w |
            ||     ||
```

!!! note

    مکان پوشهٔ bin مربوط به tool را می‌توان با اجرای دستور `uv tool dir --bin` در کانتینر تعیین
    کرد.

    به‌جای آن، می‌توان آن را روی یک مکان ثابت تنظیم کرد:

    ```dockerfile title="Dockerfile"
    ENV UV_TOOL_BIN_DIR=/opt/uv-bin/
    ```

## توسعه در یک کانتینر {#developing-in-a-container}

هنگام توسعه، mount کردن پوشهٔ project در یک کانتینر مفید است. با این راه‌اندازی، تغییرات project
می‌تواند بلافاصله و بدون rebuild کردن ایمیج، در یک سرویس کانتینری‌شده منعکس شود. با این حال، مهم
است که virtual environment مربوط به project (`.venv`) را در mount نگنجانید، زیرا virtual environment
به platform وابسته است و باید همان نسخه‌ای که برای ایمیج ساخته شده است، نگه داشته شود.

### Mount کردن project با `docker run` {#mounting-the-project-with-docker-run}

project را (در پوشهٔ کاری) با یک [volume ناشناس](https://docs.docker.com/engine/storage/#volumes)
به `/app` bind mount کنید، در حالی که پوشهٔ `.venv` حفظ می‌شود:

```console
$ docker run --rm --volume .:/app --volume /app/.venv [...]
```

!!! tip

    فلگ `--rm` گنجانده شده است تا مطمئن شویم کانتینر و volume ناشناس هنگام خروج کانتینر پاک
    می‌شوند.

یک مثال کامل را در
[project مربوط به `uv-docker-example`](https://github.com/astral-sh/uv-docker-example/blob/main/run.sh) ببینید.

### پیکربندی `watch` با `docker compose` {#configuring-watch-with-docker-compose}

هنگام استفاده از Docker compose، toolهای پیچیده‌تری برای توسعهٔ کانتینر در دسترس‌اند. گزینهٔ
[`watch`](https://docs.docker.com/compose/file-watch/#compose-watch-versus-bind-mounts) نسبت به
bind mount، دانه‌بندی بیشتری فراهم می‌کند و از تحریک به‌روزرسانی‌های سرویس کانتینری‌شده هنگام تغییر
فایل‌ها پشتیبانی می‌کند.

!!! note

    این قابلیت به Compose نسخهٔ 2.22.0 نیاز دارد که همراه با Docker Desktop نسخهٔ 4.24 ارائه
    می‌شود.

`watch` را در
[فایل Docker compose](https://docs.docker.com/compose/compose-application-model/#the-compose-file)
خود پیکربندی کنید تا پوشهٔ project را بدون sync کردن virtual environment مربوط به project، mount
کند و وقتی پیکربندی تغییر می‌کند، ایمیج را rebuild کند:

```yaml title="compose.yaml"
services:
  example:
    build: .

    # ...

    develop:
      # Create a `watch` configuration to update the app
      #
      watch:
        # Sync the working directory with the `/app` directory in the container
        - action: sync
          path: .
          target: /app
          # Exclude the project virtual environment
          ignore:
            - .venv/

        # Rebuild the image on changes to the `pyproject.toml`
        - action: rebuild
          path: ./pyproject.toml
```

سپس، برای اجرای کانتینر با راه‌اندازی توسعه، دستور `docker compose watch` را اجرا کنید.

یک مثال کامل را در
[project مربوط به `uv-docker-example`](https://github.com/astral-sh/uv-docker-example/blob/main/compose.yml) ببینید.

## بهینه‌سازی‌ها {#optimizations}

### Compile کردن bytecode {#compiling-bytecode}

Compile کردن فایل‌های منبع Python به bytecode معمولاً برای ایمیج‌های production مطلوب است، زیرا
تمایل دارد زمان startup را بهبود دهد (به بهای افزایش زمان نصب و اندازهٔ ایمیج).

برای فعال‌کردن compile کردن bytecode، از فلگ `--compile-bytecode` استفاده کنید:

```dockerfile title="Dockerfile"
RUN uv python install --compile-bytecode
RUN uv sync --compile-bytecode
```

به‌جای آن، می‌توانید environment variable مربوط به `UV_COMPILE_BYTECODE` را تنظیم کنید تا مطمئن
شوید همهٔ دستورها در Dockerfile، bytecode را compile می‌کنند:

```dockerfile title="Dockerfile"
ENV UV_COMPILE_BYTECODE=1
```

!!! note

     uv فقط کتابخانهٔ استاندارد مربوط به نسخه‌های Python _مدیریت‌شده_ را هنگام
    `uv python install` compile می‌کند. توزیع‌کنندهٔ نسخه‌های Python مدیریت‌نشده تصمیم می‌گیرد که
    آیا کتابخانهٔ استاندارد از پیش compile شده است یا نه. برای مثال، ایمیج رسمی `python` کتابخانهٔ
    استاندارد compile‌شده نخواهد داشت.

### Caching {#caching}

می‌توان از یک [cache mount](https://docs.docker.com/build/guide/mounts/#add-a-cache-mount) برای
بهبود عملکرد در buildها استفاده کرد:

```dockerfile title="Dockerfile"
ENV UV_LINK_MODE=copy

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync
```

تغییر [`UV_LINK_MODE`](../../reference/settings.md#link-mode) هشدارها دربارهٔ ناتوانی در link کردن
فایل‌ها را ساکت می‌کند، زیرا cache و هدف sync روی file systemهای جداگانه قرار دارند.

اگر cache را mount نمی‌کنید، می‌توان با استفاده از فلگ `--no-cache` یا تنظیم `UV_NO_CACHE`، اندازهٔ
ایمیج را کاهش داد.

به‌طور پیش‌فرض، نصب‌های مدیریت‌شدهٔ Python پیش از نصب، cache نمی‌شوند. تنظیم `UV_PYTHON_CACHE_DIR`
را می‌توان در ترکیب با یک cache mount استفاده کرد:

```dockerfile title="Dockerfile"
ENV UV_PYTHON_CACHE_DIR=/root/.cache/uv/python

RUN --mount=type=cache,target=/root/.cache/uv \
    uv python install
```

!!! note

    مکان پوشهٔ cache را می‌توان با اجرای دستور `uv cache dir` در کانتینر تعیین کرد.

    به‌جای آن، cache را می‌توان روی یک مکان ثابت تنظیم کرد:

    ```dockerfile title="Dockerfile"
    ENV UV_CACHE_DIR=/opt/uv-cache/
    ```

### Layerهای میانی {#intermediate-layers}

اگر از uv برای مدیریت project خود استفاده می‌کنید، می‌توانید با انتقال نصب dependencyهای transitive
به layer خودش، از طریق گزینه‌های `--no-install`، زمان build را بهبود دهید.

دستور `uv sync --no-install-project`، dependencyهای project را نصب می‌کند، اما خود project را نصب
نمی‌کند. از آنجا که project به‌طور مکرر تغییر می‌کند، اما dependencyهای آن معمولاً ثابت‌اند، این کار
می‌تواند صرفه‌جویی زمانی زیادی داشته باشد.

```dockerfile title="Dockerfile"
# Install uv
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Change the working directory to the `app` directory
WORKDIR /app

# Install dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    uv sync --frozen --no-install-project

# Copy the project into the image
COPY . /app

# Sync the project
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked
```

توجه کنید که فقط فایل `uv.lock` برای اولین sync، mount می‌شود. فایل `pyproject.toml` و
_محتوای_ project، پیش از دستور نهایی `uv sync` در ایمیج کپی می‌شوند.

!!! tip

    اگر می‌خواهید packageهای مشخص دیگری را از sync حذف کنید، از
    `--no-install-package <name>` استفاده کنید.

#### Layerهای میانی در workspaceها {#intermediate-layers-in-workspaces}

اگر از یک [workspace](../../concepts/projects/workspaces.md) استفاده می‌کنید، از فلگ
`--no-install-workspace` استفاده کنید تا project _و_ همهٔ memberهای workspace را از sync اولیه
حذف کنید:

```dockerfile title="Dockerfile"
# Install uv
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    uv sync --frozen --no-install-workspace

COPY . /app

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked
```

uv نمی‌تواند بدون فایل‌های `pyproject.toml` مربوط به project و memberهای workspace، تأیید کند که
فایل `uv.lock` به‌روز است، پس به‌جای `--locked` از `--frozen` استفاده می‌کنیم تا بررسی را در sync
اولیه نادیده بگیریم. sync بعدی، پس از کپی‌شدن همهٔ memberهای workspace، همچنان می‌تواند از
`--locked` استفاده کند و بررسی می‌کند که lockfile برای همهٔ memberهای workspace درست است.

### نصب‌های non-editable {#non-editable-installs}

به‌طور پیش‌فرض، uv، projectها و memberهای workspace را در حالت editable نصب می‌کند، به‌طوری که
تغییرات کد منبع بلافاصله در environment منعکس می‌شوند.

هر دو دستور `uv sync` و `uv run` یک فلگ `--no-editable` می‌پذیرند که به uv دستور می‌دهد project را
در حالت non-editable نصب کند و هرگونه وابستگی به کد منبع را حذف کند.

در context مربوط به یک ایمیج Docker چند مرحله‌ای، می‌توان از `--no-editable` برای گنجاندن project
در virtual environment همگام‌شده از یک مرحله استفاده کرد، سپس فقط virtual environment (و نه کد
منبع) را در ایمیج نهایی کپی کرد.

برای مثال:

```dockerfile title="Dockerfile"
# Install uv
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Use the system Python across both stages
ENV UV_PYTHON_DOWNLOADS=0

# Change the working directory to the `app` directory
WORKDIR /app

# Install dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    uv sync --frozen --no-install-project --no-editable

# Copy the project into the intermediate image
COPY . /app

# Sync the project
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-editable

FROM python:3.12-slim

# Copy the environment, but not the source code
COPY --from=builder /app/.venv /app/.venv

# Run the application
CMD ["/app/.venv/bin/hello"]
```

### استفاده موقت از uv {#using-uv-temporarily}

اگر در ایمیج نهایی به uv نیاز نیست، می‌توان باینری را در هر فراخوانی mount کرد:

```dockerfile title="Dockerfile"
RUN --mount=from=ghcr.io/astral-sh/uv,source=/uv,target=/bin/uv \
    uv sync
```

## استفاده از interface مربوط به pip {#using-the-pip-interface}

### نصب یک package {#installing-a-package}

استفاده از system environment مربوط به Python در این context امن است، زیرا کانتینر از پیش ایزوله
شده است. می‌توان از فلگ `--system` برای نصب در system environment استفاده کرد:

```dockerfile title="Dockerfile"
RUN uv pip install --system ruff
```

برای استفاده پیش‌فرض از system environment مربوط به Python، متغیر `UV_SYSTEM_PYTHON` را تنظیم
کنید:

```dockerfile title="Dockerfile"
ENV UV_SYSTEM_PYTHON=1
```

به‌جای آن، می‌توان یک virtual environment ساخت و آن را فعال کرد:

```dockerfile title="Dockerfile"
RUN uv venv /opt/venv
# Use the virtual environment automatically
ENV VIRTUAL_ENV=/opt/venv
# Place entry points in the environment at the front of the path
ENV PATH="/opt/venv/bin:$PATH"
```

هنگام استفاده از یک virtual environment، فلگ `--system` باید از فراخوانی‌های uv حذف شود:

```dockerfile title="Dockerfile"
RUN uv pip install ruff
```

### نصب requirements {#installing-requirements}

برای نصب فایل‌های requirements، آن‌ها را در کانتینر کپی کنید:

```dockerfile title="Dockerfile"
COPY requirements.txt .
RUN uv pip install -r requirements.txt
```

### نصب یک project {#installing-a-project-1}

هنگام نصب یک project در کنار requirements، بهترین روش این است که کپی‌کردن requirements را از بقیهٔ
کد منبع جدا کنید. این کار به dependencyهای project (که اغلب تغییر نمی‌کنند) امکان می‌دهد جداگانه از
خود project (که خیلی مکرر تغییر می‌کند) در cache نگه داشته شوند.

```dockerfile title="Dockerfile"
COPY pyproject.toml .
RUN uv pip install -r pyproject.toml
COPY . .
RUN uv pip install -e .
```

## بررسی provenance مربوط به image {#verifying-image-provenance}

ایمیج‌های Docker در طول فرایند build امضا می‌شوند تا اثباتی از منشأ آن‌ها ارائه دهند. می‌توان از
این attestationها برای بررسی اینکه یک image از یک کانال رسمی تولید شده است، استفاده کرد.

برای مثال، می‌توانید attestationها را با
[ابزار GitHub CLI با نام `gh`](https://cli.github.com/) بررسی کنید:

```console
$ gh attestation verify --owner astral-sh oci://ghcr.io/astral-sh/uv:latest
Loaded digest sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx for oci://ghcr.io/astral-sh/uv:latest
Loaded 1 attestation from GitHub API

The following policy criteria will be enforced:
- OIDC Issuer must match:................... https://token.actions.githubusercontent.com
- Source Repository Owner URI must match:... https://github.com/astral-sh
- Predicate type must match:................ https://slsa.dev/provenance/v1
- Subject Alternative Name must match regex: (?i)^https://github.com/astral-sh/

✓ Verification succeeded!

sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx was attested by:
REPO          PREDICATE_TYPE                  WORKFLOW
astral-sh/uv  https://slsa.dev/provenance/v1  .github/workflows/build-docker.yml@refs/heads/main
```

این به شما می‌گوید که ایمیج Docker مشخص توسط workflow رسمی مربوط به انتشار uv در GitHub ساخته شده
و از آن زمان دست‌کاری نشده است.

attestationهای GitHub بر [زیرساخت sigstore.dev](https://www.sigstore.dev/) بنا شده‌اند. به همین
دلیل می‌توانید از [دستور `cosign`](https://github.com/sigstore/cosign) هم برای بررسی blob مربوط به
attestation در برابر manifest (چند platform) مربوط به `uv` استفاده کنید:

```console
$ REPO=astral-sh/uv
$ gh attestation download --repo $REPO oci://ghcr.io/${REPO}:latest
Wrote attestations to file sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.jsonl.
Any previous content has been overwritten

The trusted metadata is now available at sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.jsonl
$ docker buildx imagetools inspect ghcr.io/${REPO}:latest --format "{{json .Manifest}}" > manifest.json
$ cosign verify-blob-attestation \
    --new-bundle-format \
    --bundle "$(jq -r .digest manifest.json).jsonl"  \
    --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
    --certificate-identity-regexp="^https://github\.com/${REPO}/.*" \
    <(jq -j '.|del(.digest,.size)' manifest.json)
Verified OK
```

!!! tip

    این مثال‌ها از `latest` استفاده می‌کنند، اما بهترین روش این است که attestation را برای یک tag
    نسخهٔ مشخص بررسی کنید، مثلاً `ghcr.io/astral-sh/uv:0.13.9`، یا (حتی بهتر) digest مشخص مربوط به
    image، مانند `ghcr.io/astral-sh/uv@sha256:32a7bb39b496babbd997b0a39a4d66c126bdaf5661b302bf6f06abda8deca6ff`.
