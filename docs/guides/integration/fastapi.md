---
title: استفاده از uv با FastAPI
description:
  راهنمای استفاده از uv با FastAPI برای مدیریت dependencyهای Python، اجرای برنامه‌ها و استقرار
  با Docker.
---

# استفاده از uv با FastAPI {#using-uv-with-fastapi}

[FastAPI](https://github.com/fastapi/fastapi) یک فریم‌ورک وب مدرن و پُرسرعت برای Python است.
می‌توانید از uv برای مدیریت project مربوط به FastAPI خود استفاده کنید، از جمله نصب dependencyها،
مدیریت environmentها، اجرای برنامه‌های FastAPI و موارد دیگر.

!!! note

    می‌توانید کد منبع این راهنما را در مخزن [uv-fastapi-example](https://github.com/astral-sh/uv-fastapi-example) مشاهده کنید.

## مهاجرت یک project موجود در FastAPI {#migrating-an-existing-fastapi-project}

به‌عنوان مثال، برنامهٔ نمونه‌ای را در نظر بگیرید که در
[مستندات FastAPI](https://fastapi.tiangolo.com/tutorial/bigger-applications/) تعریف شده و به این
شکل ساختار یافته است:

```plaintext
project
└── app
    ├── __init__.py
    ├── main.py
    ├── dependencies.py
    ├── routers
    │   ├── __init__.py
    │   ├── items.py
    │   └── users.py
    └── internal
        ├── __init__.py
        └── admin.py
```

برای استفاده از uv با این برنامه، در داخل پوشهٔ `project` این دستور را اجرا کنید:

```console
$ uv init --no-package
```

این دستور یک فایل `pyproject.toml` برای یک
[project تخت و بدون package](../../concepts/projects/init.md#creating-a-project-without-a-build-system) ایجاد می‌کند.

سپس، یک dependency به FastAPI اضافه کنید:

```console
$ uv add fastapi --extra standard
```

اکنون باید ساختار زیر را داشته باشید:

```plaintext
project
├── pyproject.toml
└── app
    ├── __init__.py
    ├── main.py
    ├── dependencies.py
    ├── routers
    │   ├── __init__.py
    │   ├── items.py
    │   └── users.py
    └── internal
        ├── __init__.py
        └── admin.py
```

و محتوای فایل `pyproject.toml` باید چیزی شبیه به این باشد:

```toml title="pyproject.toml"
[project]
name = "uv-fastapi-example"
version = "0.1.0"
description = "FastAPI project"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "fastapi[standard]",
]
```

از آنجا می‌توانید برنامهٔ FastAPI را با این دستور اجرا کنید:

```console
$ uv run fastapi dev
```

`uv run` به‌صورت خودکار dependencyهای project را resolve و lock می‌کند (یعنی یک `uv.lock` در کنار
`pyproject.toml` ایجاد می‌کند)، یک virtual environment می‌سازد و دستور را در آن environment اجرا
می‌کند.

برنامه را با باز کردن http://127.0.0.1:8000/?token=jessica در یک مرورگر وب تست کنید.

## استقرار {#deployment}

برای مستقر کردن برنامهٔ FastAPI با Docker، می‌توانید از `Dockerfile` زیر استفاده کنید:

```dockerfile title="Dockerfile"
FROM python:3.12-slim

# Install uv.
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# Copy the application into the container.
COPY . /app

# Install the application dependencies.
WORKDIR /app
RUN uv sync --frozen --no-cache

# Run the application.
CMD ["/app/.venv/bin/fastapi", "run", "app/main.py", "--port", "80", "--host", "0.0.0.0"]
```

ایمیج Docker را با این دستور build کنید:

```console
$ docker build -t fastapi-app .
```

کانتینر Docker را به‌صورت محلی با این دستور اجرا کنید:

```console
$ docker run -p 8000:80 fastapi-app
```

برای بررسی اینکه برنامه به‌درستی در حال اجراست، به آدرس http://127.0.0.1:8000/?token=jessica در
مرورگر خود بروید.

!!! tip

    برای اطلاعات بیشتر دربارهٔ استفاده از uv با Docker، به [راهنمای Docker](./docker.md) مراجعه کنید.
