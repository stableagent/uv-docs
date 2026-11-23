---
title: Google Artifact Registry
description: استفاده از uv با Google Artifact Registry برای نصب و منتشر کردن packageهای Python.
---

# Google Artifact Registry {#google-artifact-registry}

uv می‌تواند packageها را از
[Google Artifact Registry](https://cloud.google.com/artifact-registry/docs) نصب کند، چه با استفاده
از یک access token و چه با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring).

!!! note

    این راهنما فرض می‌کند که CLI مربوط به
    [`gcloud`](https://cloud.google.com/sdk/gcloud) نصب شده و احراز هویت شده است.

برای استفاده از Google Artifact Registry، index را به project خود اضافه کنید:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "private-registry"
url = "https://<REGION>-python.pkg.dev/<PROJECT>/<REPOSITORY>/simple/"
```

## احراز هویت با access token مربوط به Google {#authenticate-with-a-google-access-token}

می‌توان credentialها را از طریق طرح احراز هویت «Basic» در HTTP ارائه کرد. access token
را در فیلد password مربوط به URL قرار دهید. username باید `oauth2accesstoken` باشد، در غیر این صورت
احراز هویت ناموفق خواهد بود.

با `gcloud` یک token تولید کنید:

```bash
export ARTIFACT_REGISTRY_TOKEN=$(
    gcloud auth application-default print-access-token
)
```

!!! note

    این یک مثال ساده است؛ ممکن است لازم باشد برای تولید token پارامترهای بیشتری بدهید، مثل
    `--project`.

سپس credentialهای مربوط به index را با این دستور تنظیم کنید:

```bash
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=oauth2accesstoken
export UV_INDEX_PRIVATE_REGISTRY_PASSWORD="$ARTIFACT_REGISTRY_TOKEN"
```

!!! note

    عبارت `PRIVATE_REGISTRY` باید با نام index تعریف‌شده در `pyproject.toml` شما مطابقت داشته باشد.

## احراز هویت با `keyring` و `keyrings.google-artifactregistry-auth` {#authenticate-with-keyring-and-keyringsgoogle-artifactregistry-auth}

همچنین می‌توانید با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring) و
[افزونهٔ مربوط به `keyrings.google-artifactregistry-auth`](https://github.com/GoogleCloudPlatform/artifact-registry-python-tools) به Artifact Registry احراز هویت کنید.
از آنجا که این دو package برای احراز هویت به Artifact Registry لازم‌اند، باید از منبعی غیر از
Artifact Registry از پیش نصب شده باشند.

افزونهٔ مربوط به `keyrings.google-artifactregistry-auth`،
[CLI مربوط به gcloud](https://cloud.google.com/sdk/gcloud) را در بر می‌گیرد تا access tokenهای
کوتاه‌مدت تولید کند، آن‌ها را به‌صورت امن در keyring سیستم ذخیره کند و وقتی منقضی شدند، تازه‌سازی
کند.

uv فقط از استفاده از package مربوط به `keyring` در
[subprocess mode](../../reference/settings.md#keyring-provider) پشتیبانی می‌کند. فایل اجرایی
`keyring` باید در `PATH` باشد، یعنی به‌صورت سراسری یا در environment فعال نصب شده باشد. CLI مربوط به
`keyring` به یک username در URL نیاز دارد و آن باید `oauth2accesstoken` باشد.

```bash
# Pre-install keyring and Artifact Registry plugin from the public PyPI
uv tool install keyring --with keyrings.google-artifactregistry-auth

# Enable keyring authentication
export UV_KEYRING_PROVIDER=subprocess

# Set the username for the index
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=oauth2accesstoken
```

!!! note

    می‌توان از تنظیم
    [`tool.uv.keyring-provider`](../../reference/settings.md#keyring-provider) برای فعال‌کردن
    keyring در `uv.toml` یا `pyproject.toml` استفاده کرد.

    به همین ترتیب، username مربوط به index را می‌توان مستقیماً به URL مربوط به index اضافه کرد.

## منتشر کردن packageها {#publishing-packages}

اگر می‌خواهید packageهای خود را در Google Artifact Registry هم منتشر کنید، می‌توانید از
`uv publish` به شکلی که در [راهنمای build و publish کردن](../package.md) توضیح داده شده است،
استفاده کنید.

ابتدا، یک `publish-url` به indexای که می‌خواهید packageها را در آن منتشر کنید، اضافه کنید. برای مثال:

```toml title="pyproject.toml" hl_lines="4"
[[tool.uv.index]]
name = "private-registry"
url = "https://<REGION>-python.pkg.dev/<PROJECT>/<REPOSITORY>/simple/"
publish-url = "https://<REGION>-python.pkg.dev/<PROJECT>/<REPOSITORY>/"
```

سپس، credentialها را پیکربندی کنید (اگر از keyring استفاده نمی‌کنید):

```console
$ export UV_PUBLISH_USERNAME=oauth2accesstoken
$ export UV_PUBLISH_PASSWORD="$ARTIFACT_REGISTRY_TOKEN"
```

و package را منتشر کنید:

```console
$ uv publish --index private-registry
```

برای استفاده از `uv publish` بدون افزودن `publish-url` به project، می‌توانید `UV_PUBLISH_URL` را
تنظیم کنید:

```console
$ export UV_PUBLISH_URL=https://<REGION>-python.pkg.dev/<PROJECT>/<REPOSITORY>/
$ uv publish
```

توجه کنید که این روش ترجیح داده نمی‌شود، زیرا uv نمی‌تواند پیش از آپلود artifactها بررسی کند که
آیا package از قبل منتشر شده است یا نه.
