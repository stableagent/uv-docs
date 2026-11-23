---
title: استفاده از uv با AWS Lambda
description:
  راهنمای کامل استفاده از uv با AWS Lambda برای مدیریت dependencyهای Python و مستقر کردن
  functionهای serverless از طریق کانتینرهای Docker یا آرشیوهای zip.
---

# استفاده از uv با AWS Lambda {#using-uv-with-aws-lambda}

[AWS Lambda](https://aws.amazon.com/lambda/) یک سرویس محاسبات serverless است که به شما امکان
می‌دهد کد را بدون تهیه یا مدیریت سرور اجرا کنید.

می‌توانید از uv با AWS Lambda برای مدیریت dependencyهای Python خود، build کردن package مربوط به
استقرار و deploy کردن functionهای Lambda خود استفاده کنید.

!!! tip

    برای نمونه‌ای از بهترین روش‌ها هنگام استفاده از uv برای deploy کردن یک برنامه در AWS Lambda، به
    project مربوط به [`uv-aws-lambda-example`](https://github.com/astral-sh/uv-aws-lambda-example) مراجعه کنید.

## شروع کار {#getting-started}

برای شروع، فرض می‌کنیم یک برنامهٔ حداقلی FastAPI با ساختار زیر داریم:

```plaintext
project
├── pyproject.toml
└── app
    ├── __init__.py
    └── main.py
```

که در آن، `pyproject.toml` شامل موارد زیر است:

```toml title="pyproject.toml"
[project]
name = "uv-aws-lambda-example"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    # FastAPI is a modern web framework for building APIs with Python.
    "fastapi",
    # Mangum is a library that adapts ASGI applications to AWS Lambda and API Gateway.
    "mangum",
]

[dependency-groups]
dev = [
    # In development mode, include the FastAPI development server.
    "fastapi[standard]>=0.115",
]
```

و فایل `main.py` شامل موارد زیر است:

```python title="app/main.py"
import logging

from fastapi import FastAPI
from mangum import Mangum

logger = logging.getLogger()
logger.setLevel(logging.INFO)

app = FastAPI()
handler = Mangum(app)


@app.get("/")
async def root() -> str:
    return "Hello, world!"
```

می‌توانیم این برنامه را به‌صورت محلی با این دستور اجرا کنیم:

```console
$ uv run fastapi dev
```

از آنجا، باز کردن آدرس http://127.0.0.1:8000/ در یک مرورگر وب، عبارت «Hello, world!» را نمایش
می‌دهد.

## مستقر کردن یک ایمیج Docker {#deploying-a-docker-image}

برای مستقر کردن در AWS Lambda، باید یک ایمیج کانتینر بسازیم که کد برنامه و dependencyها را در یک
پوشهٔ خروجی واحد در بر می‌گیرد.

اصول شرح‌داده‌شده در [راهنمای Docker](./docker.md) را دنبال می‌کنیم (به‌ویژه، یک build چند مرحله‌ای)
تا مطمئن شویم ایمیج نهایی تا حد ممکن کوچک و سازگار با cache است.

در مرحلهٔ اول، یک پوشهٔ واحد را با همهٔ کد برنامه و dependencyها پر می‌کنیم. در مرحلهٔ دوم، این
پوشه را به ایمیج نهایی کپی می‌کنیم و toolهای build و دیگر فایل‌های غیرضروری را حذف می‌کنیم.

```dockerfile title="Dockerfile"
FROM ghcr.io/astral-sh/uv:0.13.9 AS uv

# First, bundle the dependencies into the task root.
FROM public.ecr.aws/lambda/python:3.13 AS builder

# Enable bytecode compilation, to improve cold-start performance.
ENV UV_COMPILE_BYTECODE=1

# Disable installer metadata, to create a deterministic layer.
ENV UV_NO_INSTALLER_METADATA=1

# Enable copy mode to support bind mount caching.
ENV UV_LINK_MODE=copy

# Bundle the dependencies into the Lambda task root via `uv pip install --target`.
#
# Omit any local packages (`--no-emit-workspace`) and development dependencies (`--no-dev`).
# This keeps the layer cached across changes to the application code or `pyproject.toml`, as long as
# `uv.lock` is unchanged.
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    uv export --frozen --no-emit-workspace --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

FROM public.ecr.aws/lambda/python:3.13

# Copy the runtime dependencies from the builder stage.
COPY --from=builder ${LAMBDA_TASK_ROOT} ${LAMBDA_TASK_ROOT}

# Copy the application code.
COPY ./app ${LAMBDA_TASK_ROOT}/app

# Set the AWS Lambda handler.
CMD ["app.main.handler"]
```

!!! tip

    برای مستقر کردن در runtimeهای AWS Lambda مبتنی بر ARM، عبارت
    `public.ecr.aws/lambda/python:3.13` را با `public.ecr.aws/lambda/python:3.13-arm64` جایگزین
    کنید.

می‌توانیم ایمیج را مثلاً با این دستور build کنیم:

```console
$ uv lock
$ docker build -t fastapi-app .
```

مزیت‌های اصلی این ساختار Dockerfile به شرح زیر است:

1. **حداقل اندازهٔ ایمیج.** با استفاده از یک build چند مرحله‌ای، می‌توانیم مطمئن شویم که ایمیج
   نهایی فقط کد برنامه و dependencyها را در بر می‌گیرد. برای مثال، خود باینری uv در ایمیج نهایی
   گنجانده نمی‌شود.
2. **حداکثر استفادهٔ مجدد از cache.** با نصب جداگانهٔ dependencyهای برنامه از کد برنامه، می‌توانیم
   مطمئن شویم که cache مربوط به لایهٔ مربوط به Docker فقط وقتی باطل می‌شود که dependencyها تغییر
   کنند.

به‌طور مشخص، بازسازی ایمیج پس از تغییر کد منبع برنامه می‌تواند از لایه‌های cacheشده استفادهٔ
مجدد کند و به buildهایی در حد میلی‌ثانیه منجر شود:

```console
 => [internal] load build definition from Dockerfile                                                                 0.0s
 => => transferring dockerfile: 1.31kB                                                                               0.0s
 => [internal] load metadata for public.ecr.aws/lambda/python:3.13                                                   0.3s
 => [internal] load metadata for ghcr.io/astral-sh/uv:latest                                                         0.3s
 => [internal] load .dockerignore                                                                                    0.0s
 => => transferring context: 106B                                                                                    0.0s
 => [uv 1/1] FROM ghcr.io/astral-sh/uv:latest@sha256:ea61e006cfec0e8d81fae901ad703e09d2c6cf1aa58abcb6507d124b50286f  0.0s
 => [builder 1/2] FROM public.ecr.aws/lambda/python:3.13@sha256:f5b51b377b80bd303fe8055084e2763336ea8920d12955b23ef  0.0s
 => [internal] load build context                                                                                    0.0s
 => => transferring context: 185B                                                                                    0.0s
 => CACHED [builder 2/2] RUN --mount=from=uv,source=/uv,target=/bin/uv     --mount=type=cache,target=/root/.cache/u  0.0s
 => CACHED [stage-2 2/3] COPY --from=builder /var/task /var/task                                                     0.0s
 => CACHED [stage-2 3/3] COPY ./app /var/task                                                                        0.0s
 => exporting to image                                                                                               0.0s
 => => exporting layers                                                                                              0.0s
 => => writing image sha256:6f8f9ef715a7cda466b677a9df4046ebbb90c8e88595242ade3b4771f547652d                         0.0
```

پس از build کردن، می‌توانیم ایمیج را به
[Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) push کنیم، مثلاً با:

```console
$ aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
$ docker tag fastapi-app:latest aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest
$ docker push aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest
```

در پایان، می‌توانیم ایمیج را با استفاده از AWS Management Console یا AWS CLI در AWS Lambda deploy
کنیم، مثلاً:

```console
$ aws lambda create-function \
   --function-name myFunction \
   --package-type Image \
   --code ImageUri=aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest \
   --role arn:aws:iam::111122223333:role/my-lambda-role
```

که در آن،
[execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-api)
از طریق دستور زیر ساخته می‌شود:

```console
$ aws iam create-role \
   --role-name my-lambda-role \
   --assume-role-policy-document '{"Version": "2012-10-17", "Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

یا یک function موجود را با این دستور به‌روز کنید:

```console
$ aws lambda update-function-code \
   --function-name myFunction \
   --image-uri aws_account_id.dkr.ecr.region.amazonaws.com/fastapi-app:latest \
   --publish
```

برای تست Lambda، می‌توانیم آن را از طریق AWS Management Console یا AWS CLI فراخوانی کنیم، مثلاً:

```console
$ aws lambda invoke \
   --function-name myFunction \
   --payload file://event.json \
   --cli-binary-format raw-in-base64-out \
   response.json
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST"
}
```

که در آن، `event.json` شامل event payloadای است که به function مربوط به Lambda داده می‌شود:

```json title="event.json"
{
  "httpMethod": "GET",
  "path": "/",
  "requestContext": {},
  "version": "1.0"
}
```

و `response.json` شامل پاسخ function مربوط به Lambda است:

```json title="response.json"
{
  "statusCode": 200,
  "headers": {
    "content-length": "14",
    "content-type": "application/json"
  },
  "multiValueHeaders": {},
  "body": "\"Hello, world!\"",
  "isBase64Encoded": false
}
```

برای جزئیات، به
[مستندات AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/python-image.html) مراجعه کنید.

### پشتیبانی از Workspace {#workspace-support}

اگر یک project شامل dependencyهای محلی باشد (مثلاً از طریق
[Workspaceها](../../concepts/projects/workspaces.md))، آن‌ها هم باید در package مربوط به deployment
گنجانده شوند.

با گسترش مثال بالا شروع می‌کنیم تا یک dependency به یک کتابخانهٔ توسعه‌یافته به‌صورت محلی با نام
`library` را در بر بگیرد.

ابتدا، خود کتابخانه را می‌سازیم:

```console
$ uv init --lib library
$ uv add ./library
```

اجرای `uv init` در پوشهٔ `project`، به‌صورت خودکار `project` را به یک workspace تبدیل می‌کند و
`library` را به‌عنوان یک workspace member اضافه می‌کند:

```toml title="pyproject.toml"
[project]
name = "uv-aws-lambda-example"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    # FastAPI is a modern web framework for building APIs with Python.
    "fastapi",
    # A local library.
    "library",
    # Mangum is a library that adapts ASGI applications to AWS Lambda and API Gateway.
    "mangum",
]

[dependency-groups]
dev = [
    # In development mode, include the FastAPI development server.
    "fastapi[standard]",
]

[tool.uv.workspace]
members = ["library"]

[tool.uv.sources]
lib = { workspace = true }
```

به‌طور پیش‌فرض، `uv init --lib` یک package می‌سازد که یک تابع `hello` را صادر می‌کند. کد منبع
برنامه را تغییر می‌دهیم تا آن تابع را فراخوانی کند:

```python title="app/main.py"
import logging

from fastapi import FastAPI
from mangum import Mangum

from library import hello

logger = logging.getLogger()
logger.setLevel(logging.INFO)

app = FastAPI()
handler = Mangum(app)


@app.get("/")
async def root() -> str:
    return hello()
```

می‌توانیم برنامهٔ تغییرکرده را به‌صورت محلی با این دستور اجرا کنیم:

```console
$ uv run fastapi dev
```

و تأیید کنیم که باز کردن آدرس http://127.0.0.1:8000/ در یک مرورگر وب، عبارت «Hello from library!»
را نمایش می‌دهد (به‌جای «Hello, World!»)

در پایان، Dockerfile را به‌روز می‌کنیم تا کتابخانهٔ محلی را در package مربوط به deployment بگنجانیم:

```dockerfile title="Dockerfile"
FROM ghcr.io/astral-sh/uv:0.13.9 AS uv

# First, bundle the dependencies into the task root.
FROM public.ecr.aws/lambda/python:3.13 AS builder

# Enable bytecode compilation, to improve cold-start performance.
ENV UV_COMPILE_BYTECODE=1

# Disable installer metadata, to create a deterministic layer.
ENV UV_NO_INSTALLER_METADATA=1

# Enable copy mode to support bind mount caching.
ENV UV_LINK_MODE=copy

# Bundle the dependencies into the Lambda task root via `uv pip install --target`.
#
# Omit any local packages (`--no-emit-workspace`) and development dependencies (`--no-dev`).
# This keeps the layer cached across changes to the application code or `pyproject.toml`, as long as
# `uv.lock` is unchanged.
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    uv export --frozen --no-emit-workspace --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

# If you have a workspace, copy it over and install it too.
#
# By omitting `--no-emit-workspace`, `library` will be copied into the task root. Using a separate
# `RUN` command ensures that all third-party dependencies are cached separately and remain
# robust to changes in the workspace.
RUN --mount=from=uv,source=/uv,target=/bin/uv \
    --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=library,target=library \
    uv export --frozen --no-dev --no-editable -o requirements.txt && \
    uv pip install -r requirements.txt --target "${LAMBDA_TASK_ROOT}"

FROM public.ecr.aws/lambda/python:3.13

# Copy the runtime dependencies from the builder stage.
COPY --from=builder ${LAMBDA_TASK_ROOT} ${LAMBDA_TASK_ROOT}

# Copy the application code.
COPY ./app ${LAMBDA_TASK_ROOT}/app

# Set the AWS Lambda handler.
CMD ["app.main.handler"]
```

!!! tip

    برای مستقر کردن در runtimeهای AWS Lambda مبتنی بر ARM، عبارت
    `public.ecr.aws/lambda/python:3.13` را با `public.ecr.aws/lambda/python:3.13-arm64` جایگزین
    کنید.

از آنجا می‌توانیم ایمیج به‌روزشده را مانند قبل build و deploy کنیم.

## مستقر کردن یک آرشیو zip {#deploying-a-zip-archive}

AWS Lambda از مستقر کردن از طریق آرشیوهای zip هم پشتیبانی می‌کند. برای برنامه‌های ساده، آرشیوهای
zip می‌توانند روش ساده‌تر و کارآمدتری نسبت به ایمیج‌های Docker باشند؛ با این حال، آرشیوهای zip به
[۲۵۰ مگابایت](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html#python-package-create-update)
محدودند.

با بازگشت به مثال FastAPI، می‌توانیم dependencyهای برنامه را از طریق دستور زیر در یک پوشهٔ محلی
برای AWS Lambda بگنجانیم:

```console
$ uv export --frozen --no-dev --no-editable -o requirements.txt
$ uv pip install \
   --no-installer-metadata \
   --no-compile-bytecode \
   --python-platform x86_64-manylinux2014 \
   --python 3.13 \
   --target packages \
   -r requirements.txt
```

!!! tip

    برای مستقر کردن در runtimeهای AWS Lambda مبتنی بر ARM، عبارت `x86_64-manylinux2014` را با
    `aarch64-manylinux2014` جایگزین کنید.

با دنبال‌کردن
[مستندات AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html)، می‌توانیم
سپس این dependencyها را به این شکل در یک zip بگنجانیم:

```console
$ cd packages
$ zip -r ../package.zip .
$ cd ..
```

در پایان، می‌توانیم کد برنامه را به آرشیو zip اضافه کنیم:

```console
$ zip -r package.zip app
```

سپس می‌توانیم آرشیو zip را از طریق AWS Management Console یا AWS CLI در AWS Lambda مستقر کنیم،
مثلاً:

```console
$ aws lambda create-function \
   --function-name myFunction \
   --runtime python3.13 \
   --zip-file fileb://package.zip \
   --handler app.main.handler \
   --role arn:aws:iam::111122223333:role/service-role/my-lambda-role
```

که در آن،
[execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-api)
از طریق دستور زیر ساخته می‌شود:

```console
$ aws iam create-role \
   --role-name my-lambda-role \
   --assume-role-policy-document '{"Version": "2012-10-17", "Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
```

یا یک function موجود را با این دستور به‌روز کنید:

```console
$ aws lambda update-function-code \
   --function-name myFunction \
   --zip-file fileb://package.zip
```

!!! note

    به‌طور پیش‌فرض، AWS Management Console یک entrypoint مربوط به Lambda با نام
    `lambda_function.lambda_handler` را فرض می‌کند. اگر برنامهٔ شما از یک entrypoint متفاوت استفاده
    می‌کند، باید آن را در AWS Management Console تغییر دهید. برای مثال، برنامهٔ FastAPI بالا از
    `app.main.handler` استفاده می‌کند.

برای تست Lambda، می‌توانیم آن را از طریق AWS Management Console یا AWS CLI فراخوانی کنیم، مثلاً:

```console
$ aws lambda invoke \
   --function-name myFunction \
   --payload file://event.json \
   --cli-binary-format raw-in-base64-out \
   response.json
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST"
}
```

که در آن، `event.json` شامل event payloadای است که به function مربوط به Lambda داده می‌شود:

```json title="event.json"
{
  "httpMethod": "GET",
  "path": "/",
  "requestContext": {},
  "version": "1.0"
}
```

و `response.json` شامل پاسخ function مربوط به Lambda است:

```json title="response.json"
{
  "statusCode": 200,
  "headers": {
    "content-length": "14",
    "content-type": "application/json"
  },
  "multiValueHeaders": {},
  "body": "\"Hello, world!\"",
  "isBase64Encoded": false
}
```

### استفاده از یک Lambda layer {#using-a-lambda-layer}

AWS Lambda هنگام کار با آرشیوهای zip، از deploy کردن چند
[لایهٔ Lambda](https://docs.aws.amazon.com/lambda/latest/dg/python-layers.html) ترکیب‌شده هم
پشتیبانی می‌کند. این layerها از نظر مفهومی شبیه layerها در یک ایمیج Docker هستند و به شما امکان
می‌دهند کد برنامه را از dependencyها جدا کنید.

به‌ویژه، می‌توانیم یک لایهٔ Lambda برای dependencyهای برنامه بسازیم و آن را جدا از خود کد برنامه،
به function مربوط به Lambda متصل کنیم. این راه‌اندازی می‌تواند عملکرد cold-start را برای
به‌روزرسانی‌های برنامه بهبود دهد، زیرا لایهٔ مربوط به dependencyها را می‌توان در استقرارهای مختلف
استفادهٔ مجدد کرد.

برای ساختن یک Lambda layer، مراحل مشابهی را دنبال می‌کنیم، اما دو آرشیو zip جداگانه می‌سازیم: یکی
برای کد برنامه و یکی برای dependencyهای برنامه.

ابتدا، layer مربوط به dependency را می‌سازیم. انتظار می‌رود layerهای Lambda از یک ساختار کمی متفاوت
پیروی کنند، پس به‌جای `--target` از `--prefix` استفاده می‌کنیم:

```console
$ uv export --frozen --no-dev --no-editable -o requirements.txt
$ uv pip install \
   --no-installer-metadata \
   --no-compile-bytecode \
   --python-platform x86_64-manylinux2014 \
   --python 3.13 \
   --prefix packages \
   -r requirements.txt
```

سپس dependencyها را مطابق با layout مورد انتظار برای layerهای Lambda، zip می‌کنیم:

```console
$ mkdir python
$ cp -r packages/lib python/
$ zip -r layer_content.zip python
```

!!! tip

    برای تولید آرشیوهای zip قطعی (deterministic)، به فلگ `-X` برای `zip` فکر کنید تا attributeهای
    گسترش‌یافته و metadata مربوط به file system حذف شوند.

و Lambda layer را منتشر می‌کنیم:

```console
$ aws lambda publish-layer-version --layer-name dependencies-layer \
   --zip-file fileb://layer_content.zip \
   --compatible-runtimes python3.13 \
   --compatible-architectures "x86_64"
```

سپس می‌توانیم function مربوط به Lambda را مانند مثال قبلی بسازیم و dependencyها را حذف کنیم:

```console
$ # Zip the application code.
$ zip -r app.zip app

$ # Create the Lambda function.
$ aws lambda create-function \
   --function-name myFunction \
   --runtime python3.13 \
   --zip-file fileb://app.zip \
   --handler app.main.handler \
   --role arn:aws:iam::111122223333:role/service-role/my-lambda-role
```

در پایان، می‌توانیم layer مربوط به dependencyها را با استفاده از ARN برگردانده‌شده در مرحلهٔ
`publish-layer-version`، به function مربوط به Lambda متصل کنیم:

```console
$ aws lambda update-function-configuration --function-name myFunction \
    --cli-binary-format raw-in-base64-out \
    --layers "arn:aws:lambda:region:111122223333:layer:dependencies-layer:1"
```

وقتی dependencyهای برنامه تغییر می‌کنند، می‌توان لایه را مستقل از برنامه، با republish کردن لایه و
به‌روزرسانی پیکربندی function مربوط به Lambda، به‌روز کرد:

```console
$ # Update the dependencies in the layer.
$ aws lambda publish-layer-version --layer-name dependencies-layer \
   --zip-file fileb://layer_content.zip \
   --compatible-runtimes python3.13 \
   --compatible-architectures "x86_64"

$ # Update the Lambda function configuration.
$ aws lambda update-function-configuration --function-name myFunction \
    --cli-binary-format raw-in-base64-out \
    --layers "arn:aws:lambda:region:111122223333:layer:dependencies-layer:2"
```
