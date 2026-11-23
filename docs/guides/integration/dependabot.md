---
title: استفاده از uv با Dependabot
description: راهنمای استفاده از uv با bot مربوط به dependency در Dependabot.
---

# Dependabot {#dependabot}

به‌روز نگه داشتن منظم dependencyها بهترین روش محسوب می‌شود تا در معرض آسیب‌پذیری‌ها قرار نگیرید،
ناسازگاری‌های بین dependencyها را محدود کنید و از ارتقاهای پیچیده هنگام ارتقا از نسخه‌ای خیلی قدیمی
جلوگیری کنید.

Dependabot پشتیبانی از uv را اعلام کرده است، اما برخی موردهای استفاده هنوز کار نمی‌کنند. برای به‌روزرسانی‌ها
به [astral-sh/uv#2512](https://github.com/astral-sh/uv/issues/2512) مراجعه کنید.

Dependabot از به‌روزرسانی فایل‌های `uv.lock` پشتیبانی می‌کند. برای فعال‌کردن آن، `package-ecosystem`
مربوط به uv را به فهرست `updates` خود در `dependabot.yml` اضافه کنید:

```yaml title="dependabot.yml"
version: 2

updates:
  - package-ecosystem: "uv"
    directory: "/"
    schedule:
      interval: "weekly"
```

## Dependency cooldown {#dependency-cooldown}

اگر از گزینهٔ [`exclude-newer`](../../reference/settings.md#exclude-newer) استفاده می‌کنید، توصیه
می‌شود گزینهٔ معادل
[`cooldown`](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#cooldown-)
را نیز در Dependabot تنظیم کنید تا در نهایت با pull requestهایی مواجه نشوید که uv نتواند
dependencyهای آن‌ها را lock کند.

برای مثال، اگر `exclude-newer` را روی `1 week` تنظیم کرده‌اید، می‌توانید این را تنظیم کنید:

```yaml title="dependabot.yml"
version: 2

updates:
  - package-ecosystem: "uv"
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 7
```
