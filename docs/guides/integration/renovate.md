---
title: استفاده از uv با Renovate
description: راهنمای استفاده از uv با bot مربوط به dependency در Renovate.
---

# Renovate {#renovate}

به‌روز نگه داشتن منظم dependencyها بهترین روش محسوب می‌شود تا در معرض آسیب‌پذیری‌ها قرار نگیرید،
ناسازگاری‌های بین dependencyها را محدود کنید و از ارتقاهای پیچیده هنگام ارتقا از نسخه‌ای خیلی قدیمی
جلوگیری کنید.

uv توسط [Renovate](https://github.com/renovatebot/renovate) پشتیبانی می‌شود.

## خروجی `uv.lock` {#uvlock-output}

Renovate از وجود فایل `uv.lock` برای تشخیص اینکه uv برای مدیریت dependencyها استفاده می‌شود،
استفاده می‌کند و ارتقاهایی را برای
[dependencyهای project](../../concepts/projects/dependencies.md#project-dependencies)،
[dependencyهای optional](../../concepts/projects/dependencies.md#optional-dependencies) و
[dependencyهای development](../../concepts/projects/dependencies.md#development-dependencies)
پیشنهاد می‌دهد. Renovate هر دو فایل `pyproject.toml` و `uv.lock` را به‌روز می‌کند.

lockfile را می‌توان به‌صورت منظم (مثلاً برای به‌روزرسانی dependencyهای transitive) با فعال‌کردن
گزینهٔ
[`lockFileMaintenance`](https://docs.renovatebot.com/configuration-options/#lockfilemaintenance)
تازه‌سازی کرد:

```jsx title="renovate.json5"
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  lockFileMaintenance: {
    enabled: true,
  },
}
```

## متادیتای درون‌خطی script {#inline-script-metadata}

Renovate از به‌روزرسانی dependencyهای تعریف‌شده با استفاده از
[inline script metadata](../scripts.md#declaring-script-dependencies) پشتیبانی می‌کند.

از آنجا که نمی‌تواند به‌صورت خودکار تشخیص دهد کدام فایل‌های Python از inline script metadata
استفاده می‌کنند، مکان‌های آن‌ها باید به‌صراحت با استفاده از
[`managerFilePatterns`](https://docs.renovatebot.com/configuration-options/#managerfilepatterns)
تعریف شوند، به این شکل:

```jsx title="renovate.json5"
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  pep723: {
    managerFilePatterns: [
      "docs/build.py",
      "scripts/**/*.py",
    ],
  },
}
```

!!! note

    Renovate هنوز از به‌روزرسانی lockfile مربوط به
    script پشتیبانی نمی‌کند (https://github.com/renovatebot/renovate/issues/33591)، پس اگر برای
    یک script به این قابلیت متکی هستید، lockfile باید به‌صورت دستی به‌روز شود.

## Dependency cooldown {#dependency-cooldown}

اگر از گزینهٔ [`exclude-newer`](../../reference/settings.md#exclude-newer) استفاده می‌کنید، توصیه
می‌شود گزینهٔ معادل
[`minimumReleaseAge`](https://docs.renovatebot.com/configuration-options/#minimumreleaseage) را
نیز در Renovate تنظیم کنید تا در نهایت با pull requestهایی مواجه نشوید که uv نتواند
dependencyهای آن‌ها را lock کند.

برای مثال، اگر `exclude-newer` را روی `1 week` تنظیم کرده‌اید، می‌توانید این را تنظیم کنید:

```jsx title="renovate.json5"
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",

  // Enable only for PyPI.
  packageRules: [
    {
      matchDatasources: ["pypi"],
      minimumReleaseAge: "1 week",
    },
  ],

  // Or enable for every ecosystem.
  minimumReleaseAge: "1 week",
}
```
