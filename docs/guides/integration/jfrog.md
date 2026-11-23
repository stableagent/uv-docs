---
title: JFrog Artifactory
description: استفاده از uv با JFrog Artifactory برای نصب و منتشر کردن packageهای Python.
---

# JFrog Artifactory {#jfrog-artifactory}

uv می‌تواند packageها را از JFrog Artifactory نصب کند، چه با استفاده از username و password و چه با
یک JWT token.

برای استفاده از آن، index را به project خود اضافه کنید:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "private-registry"
url = "https://<organization>.jfrog.io/artifactory/api/pypi/<repository>/simple"
```

## احراز هویت با username و password {#authenticate-with-username-and-password}

```console
$ export UV_INDEX_PRIVATE_REGISTRY_USERNAME="<username>"
$ export UV_INDEX_PRIVATE_REGISTRY_PASSWORD="<password>"
```

## احراز هویت با JWT token {#authenticate-with-jwt-token}

```console
$ export UV_INDEX_PRIVATE_REGISTRY_USERNAME=""
$ export UV_INDEX_PRIVATE_REGISTRY_PASSWORD="$JFROG_JWT_TOKEN"
```

!!! note

    در نام‌های environment variable، عبارت `PRIVATE_REGISTRY` را با نام واقعی index تعریف‌شده در
    `pyproject.toml` خود جایگزین کنید.

## منتشر کردن packageها {#publishing-packages}

یک `publish-url` به تعریف index خود اضافه کنید:

```toml title="pyproject.toml"
[[tool.uv.index]]
name = "private-registry"
url = "https://<organization>.jfrog.io/artifactory/api/pypi/<repository>/simple"
publish-url = "https://<organization>.jfrog.io/artifactory/api/pypi/<repository>"
```

!!! important

    اگر با JFrog از `--token "$JFROG_TOKEN"` یا `UV_PUBLISH_TOKEN` استفاده کنید، خطای
    401 Unauthorized دریافت می‌کنید، زیرا JFrog به username خالی نیاز دارد، اما uv هنگام استفاده از
    `--token` مقدار `__token__` را به‌عنوان username می‌فرستد.

برای احراز هویت، token خود را به‌عنوان password بدهید و username را روی یک رشتهٔ خالی تنظیم کنید:

```console
$ uv publish --index <index_name> -u "" -p "$JFROG_TOKEN"
```

به‌جای آن، می‌توانید environment variableها را تنظیم کنید:

```console
$ export UV_PUBLISH_USERNAME=""
$ export UV_PUBLISH_PASSWORD="$JFROG_TOKEN"
$ uv publish --index private-registry
```

!!! note

    environment variableهای مربوط به publish (یعنی `UV_PUBLISH_USERNAME` و `UV_PUBLISH_PASSWORD`)
    نام index را شامل نمی‌شوند.
