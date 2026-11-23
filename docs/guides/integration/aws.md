---
title: AWS CodeArtifact
description: استفاده از uv با AWS CodeArtifact برای نصب و منتشر کردن packageهای Python.
---

# AWS CodeArtifact {#aws-codeartifact}

uv می‌تواند packageها را از
[AWS CodeArtifact](https://docs.aws.amazon.com/codeartifact/latest/ug/using-python.html) نصب کند،
چه با استفاده از یک access token و چه با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring).

!!! note

    این راهنما فرض می‌کند که [`awscli`](https://aws.amazon.com/cli/) نصب شده و احراز هویت شده است.

index را می‌توان به این شکل تعریف کرد:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "private-registry"
url = "https://<DOMAIN>-<ACCOUNT_ID>.d.codeartifact.<REGION>.amazonaws.com/pypi/<REPOSITORY>/simple/"
```

## احراز هویت با access token مربوط به AWS {#authenticate-with-an-aws-access-token}

می‌توان credentialها را از طریق طرح احراز هویت «Basic» در HTTP ارائه کرد. access token
را در فیلد password مربوط به URL قرار دهید. username باید `aws` باشد، در غیر این صورت احراز هویت
ناموفق خواهد بود.

با `awscli` یک token تولید کنید:

```bash
export AWS_CODEARTIFACT_TOKEN="$(
    aws codeartifact get-authorization-token \
    --domain <DOMAIN> \
    --domain-owner <ACCOUNT_ID> \
    --query authorizationToken \
    --output text
)"
```

!!! note

    این یک مثال ساده است؛ ممکن است لازم باشد برای تولید token پارامترهای بیشتری بدهید، مثل
    `--region`.

سپس credentialهای مربوط به index را با این دستور تنظیم کنید:

```bash
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=aws
export UV_INDEX_PRIVATE_REGISTRY_PASSWORD="$AWS_CODEARTIFACT_TOKEN"
```

!!! note

    عبارت `PRIVATE_REGISTRY` باید با نام index تعریف‌شده در `pyproject.toml` شما مطابقت داشته باشد.

## احراز هویت با `keyring` و `keyrings.codeartifact` {#authenticate-with-keyring-and-keyringscodeartifact}

همچنین می‌توانید با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring) و
[افزونهٔ مربوط به `keyrings.codeartifact`](https://github.com/jmkeyes/keyrings.codeartifact) به
CodeArtifact احراز هویت کنید. از آنجا که این دو package برای احراز هویت به CodeArtifact لازم‌اند،
باید از منبعی غیر از CodeArtifact از پیش نصب شده باشند.

افزونهٔ مربوط به `keyrings.codeartifact`، [boto3](https://pypi.org/project/boto3/) را در بر می‌گیرد
تا access tokenهای کوتاه‌مدت تولید کند، آن‌ها را به‌صورت امن در keyring سیستم ذخیره کند و وقتی منقضی
شدند، تازه‌سازی کند.

uv فقط از استفاده از package مربوط به `keyring` در
[subprocess mode](../../reference/settings.md#keyring-provider) پشتیبانی می‌کند. فایل اجرایی
`keyring` باید در `PATH` باشد، یعنی به‌صورت سراسری یا در environment فعال نصب شده باشد. CLI مربوط به
`keyring` به یک username در URL نیاز دارد و آن باید `aws` باشد.

```bash
# Pre-install keyring and AWS CodeArtifact plugin from the public PyPI
uv tool install keyring --with keyrings.codeartifact

# Enable keyring authentication
export UV_KEYRING_PROVIDER=subprocess

# Set the username for the index
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=aws
```

!!! note

    می‌توان از تنظیم
    [`tool.uv.keyring-provider`](../../reference/settings.md#keyring-provider) برای فعال‌کردن
    keyring در `uv.toml` یا `pyproject.toml` استفاده کرد.

    به همین ترتیب، username مربوط به index را می‌توان مستقیماً به URL مربوط به index اضافه کرد.

## منتشر کردن packageها {#publishing-packages}

اگر می‌خواهید packageهای خود را در AWS CodeArtifact هم منتشر کنید، می‌توانید از `uv publish` به
شکلی که در [راهنمای build و publish کردن](../package.md) توضیح داده شده است، استفاده کنید.

ابتدا، یک `publish-url` به indexای که می‌خواهید packageها را در آن منتشر کنید، اضافه کنید. برای مثال:

```toml title="pyproject.toml" hl_lines="4"
[[tool.uv.index]]
name = "private-registry"
url = "https://<DOMAIN>-<ACCOUNT_ID>.d.codeartifact.<REGION>.amazonaws.com/pypi/<REPOSITORY>/simple/"
publish-url = "https://<DOMAIN>-<ACCOUNT_ID>.d.codeartifact.<REGION>.amazonaws.com/pypi/<REPOSITORY>/"
```

سپس، credentialها را پیکربندی کنید (اگر از keyring استفاده نمی‌کنید):

```console
$ export UV_PUBLISH_USERNAME=aws
$ export UV_PUBLISH_PASSWORD="$AWS_CODEARTIFACT_TOKEN"
```

و package را منتشر کنید:

```console
$ uv publish --index private-registry
```

برای استفاده از `uv publish` بدون افزودن `publish-url` به project، می‌توانید `UV_PUBLISH_URL` را
تنظیم کنید:

```console
$ export UV_PUBLISH_URL=https://<DOMAIN>-<ACCOUNT_ID>.d.codeartifact.<REGION>.amazonaws.com/pypi/<REPOSITORY>/
$ uv publish
```

توجه کنید که این روش ترجیح داده نمی‌شود، زیرا uv نمی‌تواند پیش از آپلود artifactها بررسی کند که
آیا package از قبل منتشر شده است یا نه.
