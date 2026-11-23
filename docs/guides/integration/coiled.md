---
title: استفاده از uv با Coiled
description:
  راهنمای کامل استفاده از uv با Coiled برای مدیریت dependencyهای Python و مستقر کردن scriptهای
  serverless.
---

# استفاده از uv با Coiled {#using-uv-with-coiled}

[Coiled](https://coiled.io?utm_source=uv-docs) یک پلتفرم محاسبات ابری serverless و کاربرمحور است که
اجرای کد روی سخت‌افزار ابری (AWS ،GCP و Azure) را آسان می‌کند.

این راهنما نشان می‌دهد چگونه scriptهای Python را روی ابر، با استفاده از uv برای مدیریت dependency و
Coiled برای استقرار ابری، اجرا کنید.

## مدیریت dependencyهای script با uv {#managing-script-dependencies-with-uv}

!!! note

    در سراسر این راهنما از این مثال مشخص استفاده می‌کنیم، اما هر script مربوط به Python را
    می‌توان با uv و Coiled استفاده کرد.

از script زیر به‌عنوان مثال استفاده می‌کنیم:

```python title="process.py" hl_lines="1-8"
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "pandas",
#   "pyarrow",
#   "s3fs",
# ]
# ///

import pandas as pd

df = pd.read_parquet(
    "s3://coiled-data/uber/part.0.parquet",
    storage_options={"anon": True},
)
print(df.head())
```

این script از [`pandas`](https://pandas.pydata.org/docs/) برای بارگذاری یک فایل Parquet میزبانی‌شده
در یک bucket عمومی روی S3 استفاده می‌کند، سپس چند ردیف اول را چاپ می‌کند. این script از
[inline script metadata](https://peps.python.org/pep-0723/) برای فهرست‌کردن dependencyهای خود
استفاده می‌کند.

هنگام اجرای این script به‌صورت محلی، مثلاً با دستور:

```bash
$ uv run process.py
```

uv به‌صورت خودکار یک virtual environment می‌سازد و dependencyهای آن را نصب می‌کند.

برای یادگیری بیشتر دربارهٔ استفاده از inline script metadata با uv، به
[راهنمای script](../scripts.md#declaring-script-dependencies) مراجعه کنید.

## اجرای scriptها روی ابر با Coiled {#running-scripts-on-the-cloud-with-coiled}

استفاده از inline script metadata، script را کاملاً خودکفا می‌کند: این script اطلاعاتی را که برای
اجرای آن لازم است، در خود دارد. این کار اجرای آن روی ماشین‌های دیگر، مثل یک ماشین در ابر، را آسان‌تر
می‌کند.

موردهای استفادهٔ زیادی وجود دارد که در آن‌ها به منابعی فراتر از آنچه در یک ایستگاه کاری محلی در دسترس است
نیاز است، مثلاً:

- پردازش حجم زیادی از داده‌های میزبانی‌شده در ابر
- نیاز به سخت‌افزار شتاب‌دهنده مانند GPU یا یک ماشین بزرگ با حافظهٔ بیشتر
- اجرای همان script با صدها یا هزاران ورودی مختلف، به‌صورت موازی

Coiled اجرای کد روی سخت‌افزار ابری را ساده می‌کند.

ابتدا، با استفاده از
[`coiled login`](https://docs.coiled.io/user_guide/api.html?utm_source=uv-docs#coiled-login) با
Coiled احراز هویت کنید:

```bash
$ uvx coiled login
```

از شما خواسته می‌شود اگر هنوز حساب Coiled ندارید، یک حساب بسازید — شروع استفاده از Coiled رایگان
است.

برای اینکه به Coiled دستور دهید script را روی یک ماشین مجازی در AWS اجرا کند، دو کامنت به بالای
script اضافه کنید:

```python title="process.py" hl_lines="1-2"
# COILED container ghcr.io/astral-sh/uv:debian-slim
# COILED region us-east-2

# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "pandas",
#   "pyarrow",
#   "s3fs",
# ]
# ///

import pandas as pd

df = pd.read_parquet(
    "s3://coiled-data/uber/part.0.parquet",
    storage_options={"anon": True},
)
print(df.head())
```

!!! tip

    با اینکه Coiled از AWS ،GCP و Azure پشتیبانی می‌کند، این مثال فرض می‌کند که AWS استفاده
    می‌شود (به گزینهٔ `region` در بالا مراجعه کنید). اگر در Coiled تازه‌کار هستید، به‌صورت خودکار به
    یک حساب رایگان در حال اجرا روی AWS دسترسی خواهید داشت. اگر روی AWS اجرا نمی‌کنید، می‌توانید یا
    یک `region` معتبر برای ارائه‌دهندهٔ ابری خود استفاده کنید، یا خط مربوط به `region` در بالا را حذف
    کنید.

این کامنت‌ها به Coiled می‌گویند هنگام اجرای script از [ایمیج رسمی Docker مربوط به uv](../integration/docker.md) استفاده کند (تا از در دسترس بودن uv مطمئن شود) و در region مربوط به `us-east-2` در AWS اجرا کند (جایی که این فایل دادهٔ نمونه اتفاقاً در آن قرار دارد) تا از هرگونه خروج داده جلوگیری شود.

برای ارسال یک batch job تا Coiled آن را اجرا کند، از
[`coiled batch run`](https://docs.coiled.io/user_guide/api.html?utm_source=uv-docs#coiled-batch-run)
برای اجرای دستور `uv run` در ابر استفاده کنید:

```bash hl_lines="1"
$ uvx coiled batch run \
    uv run process.py
```

همان فرایندی که پیش‌تر به‌صورت محلی اجرا می‌شد، اکنون روی یک VM ابری راه‌دور در AWS در حال اجراست.

می‌توانید پیشرفت batch job را در رابط کاربری در آدرس
[cloud.coiled.io](https://cloud.coiled.io) یا از ترمینال با استفاده از دستورهای `coiled batch status` ،`coiled batch wait` و `coiled batch logs` پایش کنید.

![رابط کاربری Coiled](https://docs.coiled.io/_images/uv-coiled.png)

توجه کنید که پیکربندی بیشتری هم می‌توانستیم مشخص کنیم، مثلاً نوع نمونه (پیش‌فرض یک ماشین مجازی
۴ هسته‌ای با ۱۶ GiB حافظه است)، اندازهٔ دیسک، اینکه آیا از spot instance استفاده شود یا نه، و موارد
دیگر. برای جزئیات بیشتر، به
[مستندات Coiled Batch](https://docs.coiled.io/user_guide/batch.html?utm_source=uv-docs) مراجعه
کنید.

برای جزئیات بیشتر دربارهٔ Coiled و اینکه چگونه می‌تواند در use caseهای دیگر کمک کند، به
[مستندات Coiled](https://docs.coiled.io?utm_source=uv-docs) مراجعه کنید.
