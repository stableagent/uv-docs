---
title: Azure Artifacts
description: استفاده از uv با Azure Artifacts برای نصب و منتشر کردن packageهای Python.
---

# Azure Artifacts {#azure-artifacts}

uv می‌تواند packageها را از
[Azure Artifacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops&tabs=nuget%2Cnugetserver)
نصب کند، چه با استفاده از یک
[Personal Access Token](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows)
(PAT) و چه با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring).

برای استفاده از Azure Artifacts، index را به project خود اضافه کنید:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "private-registry"
url = "https://pkgs.dev.azure.com/<ORGANIZATION>/<PROJECT>/_packaging/<FEED>/pypi/simple/"
```

## احراز هویت با access token مربوط به Azure {#authenticate-with-an-azure-access-token}

اگر یک personal access token (PAT) در دسترس باشد (مثلاً
[`$(System.AccessToken)` در یک Azure pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/variables?view=azure-devops&tabs=yaml#systemaccesstoken))،
می‌توان credentialها را از طریق طرح احراز هویت «Basic» در HTTP ارائه کرد. PAT را در فیلد
password مربوط به URL قرار دهید. یک username هم باید گنجانده شود، اما می‌تواند هر رشته‌ای باشد.

برای مثال، با token ذخیره‌شده در environment variable مربوط به `$AZURE_ARTIFACTS_TOKEN`،
credentialهای مربوط به index را با این دستور تنظیم کنید:

```bash
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=dummy
export UV_INDEX_PRIVATE_REGISTRY_PASSWORD="$AZURE_ARTIFACTS_TOKEN"
```

!!! note

    عبارت `PRIVATE_REGISTRY` باید با نام index تعریف‌شده در `pyproject.toml` شما مطابقت داشته باشد.

## احراز هویت با `keyring` و `artifacts-keyring` {#authenticate-with-keyring-and-artifacts-keyring}

همچنین می‌توانید با استفاده از package مربوط به
[`keyring`](https://github.com/jaraco/keyring) و
[افزونهٔ مربوط به `artifacts-keyring`](https://github.com/Microsoft/artifacts-keyring) به Artifacts
احراز هویت کنید. از آنجا که این دو package برای احراز هویت به Azure Artifacts لازم‌اند، باید از
منبعی غیر از Artifacts از پیش نصب شده باشند.

افزونهٔ مربوط به `artifacts-keyring`،
[ابزار Azure Artifacts Credential Provider](https://github.com/microsoft/artifacts-credprovider) را
در بر می‌گیرد. این credential provider از چند حالت احراز هویت مختلف، از جمله ورود تعاملی، پشتیبانی
می‌کند — برای اطلاعات دربارهٔ پیکربندی، به [مستندات ابزار](https://github.com/microsoft/artifacts-credprovider) مراجعه کنید.

uv فقط از استفاده از package مربوط به `keyring` در
[subprocess mode](../../reference/settings.md#keyring-provider) پشتیبانی می‌کند. فایل اجرایی
`keyring` باید در `PATH` باشد، یعنی به‌صورت سراسری یا در environment فعال نصب شده باشد. CLI مربوط به
`keyring` به یک username در URL نیاز دارد و آن باید `VssSessionToken` باشد.

```bash
# Pre-install keyring and the Artifacts plugin from the public PyPI
uv tool install keyring --with artifacts-keyring

# Enable keyring authentication
export UV_KEYRING_PROVIDER=subprocess

# Set the username for the index
export UV_INDEX_PRIVATE_REGISTRY_USERNAME=VssSessionToken
```

!!! note

    می‌توان از تنظیم
    [`tool.uv.keyring-provider`](../../reference/settings.md#keyring-provider) برای فعال‌کردن
    keyring در `uv.toml` یا `pyproject.toml` استفاده کرد.

    به همین ترتیب، username مربوط به index را می‌توان مستقیماً به URL مربوط به index اضافه کرد.

## منتشر کردن packageها {#publishing-packages}

اگر می‌خواهید packageهای خود را در Azure Artifacts هم منتشر کنید، می‌توانید از `uv publish` به
شکلی که در [راهنمای build و publish کردن](../package.md) توضیح داده شده است، استفاده کنید.

ابتدا، یک `publish-url` به indexای که می‌خواهید packageها را در آن منتشر کنید، اضافه کنید. برای مثال:

```toml title="pyproject.toml" hl_lines="4"
[[tool.uv.index]]
name = "private-registry"
url = "https://pkgs.dev.azure.com/<ORGANIZATION>/<PROJECT>/_packaging/<FEED>/pypi/simple/"
publish-url = "https://pkgs.dev.azure.com/<ORGANIZATION>/<PROJECT>/_packaging/<FEED>/pypi/upload/"
```

سپس، credentialها را پیکربندی کنید (اگر از keyring استفاده نمی‌کنید):

```console
$ export UV_PUBLISH_USERNAME=dummy
$ export UV_PUBLISH_PASSWORD="$AZURE_ARTIFACTS_TOKEN"
```

و package را منتشر کنید:

```console
$ uv publish --index private-registry
```

برای استفاده از `uv publish` بدون افزودن `publish-url` به project، می‌توانید `UV_PUBLISH_URL` را
تنظیم کنید:

```console
$ export UV_PUBLISH_URL=https://pkgs.dev.azure.com/<ORGANIZATION>/<PROJECT>/_packaging/<FEED>/pypi/upload/
$ uv publish
```

توجه کنید که این روش ترجیح داده نمی‌شود، زیرا uv نمی‌تواند پیش از آپلود artifactها بررسی کند که
آیا package از قبل منتشر شده است یا نه.
