---
title: استفاده از uv با marimo
description:
  راهنمای کامل استفاده از uv با نوت‌بوک‌های marimo برای محاسبات تعاملی، اجرای script و
  برنامه‌های داده‌ای.
---

# استفاده از uv با marimo {#using-uv-with-marimo}

[marimo](https://github.com/marimo-team/marimo) یک نوت‌بوک متن‌باز Python است که محاسبات تعاملی را
با قابلیت بازتولید و استفادهٔ مجدد نرم‌افزارهای سنتی در هم می‌آمیزد و به شما امکان می‌دهد با Git
نسخه‌بندی کنید، به‌صورت script اجرا کنید و به‌عنوان برنامه به اشتراک بگذارید. از آنجا که نوت‌بوک‌های
marimo به‌صورت scriptهای خالص Python ذخیره می‌شوند، می‌توانند به‌خوبی با uv یکپارچه شوند.

می‌توانید marimo را به‌راحتی به‌عنوان یک tool مستقل، به‌صورت scriptهای خودکفا، در projectها و در
environmentهای غیر project استفاده کنید.

## استفاده از marimo به‌عنوان یک tool مستقل {#using-marimo-as-a-standalone-tool}

برای دسترسی موردی به نوت‌بوک‌های marimo، در هر زمان یک سرور marimo را در یک environment ایزوله با این
دستور شروع کنید:

```console
$ uvx marimo edit
```

یک نوت‌بوک مشخص را با این دستور شروع کنید:

```console
$ uvx marimo edit my_notebook.py
```

## استفاده از marimo با inline script metadata {#using-marimo-with-inline-script-metadata}

از آنجا که نوت‌بوک‌های marimo به‌صورت scriptهای Python ذخیره می‌شوند، می‌توانند dependencyهای خود
را با استفاده از inline script metadata، از طریق [پشتیبانی uv از scriptها](../../guides/scripts.md)،
در خود جای دهند. برای مثال، برای افزودن `numpy` به‌عنوان یک dependency به نوت‌بوک خود، از این دستور
استفاده کنید:

```console
$ uv add --script my_notebook.py numpy
```

برای ویرایش تعاملی یک نوت‌بوک که inline script metadata دارد، از این دستور استفاده کنید:

```console
$ uvx marimo edit --sandbox my_notebook.py
```

marimo به‌صورت خودکار از uv استفاده می‌کند تا نوت‌بوک شما را در یک virtual environment ایزوله، با
dependencyهای script شما، شروع کند. packageهایی که از رابط کاربری marimo نصب شوند، به‌صورت خودکار به
script metadata مربوط به نوت‌بوک اضافه می‌شوند.

به‌صورت اختیاری می‌توانید این نوت‌بوک‌ها را به‌عنوان scriptهای Python اجرا کنید، بدون اینکه یک
نشست تعاملی باز کنید:

```console
$ uv run my_notebook.py
```

## استفاده از marimo در یک project {#using-marimo-within-a-project}

اگر در یک [project](../../concepts/projects/index.md) کار می‌کنید، می‌توانید یک نوت‌بوک marimo را
با دسترسی به virtual environment مربوط به project، با دستور زیر شروع کنید (با این فرض که marimo یک
dependency مربوط به project است):

```console
$ uv run marimo edit my_notebook.py
```

برای اینکه packageهای بیشتری در دسترس نوت‌بوک شما قرار بگیرند، یا آن‌ها را با `uv add` به project
خود اضافه کنید، یا از رابط کاربری داخلی نصب package در marimo استفاده کنید که از طرف شما `uv add`
را فراخوانی می‌کند.

اگر marimo یک dependency مربوط به project نیست، همچنان می‌توانید یک نوت‌بوک را با دستور زیر اجرا
کنید:

```console
$ uv run --with marimo marimo edit my_notebook.py
```

با این کار می‌توانید moduleهای project خود را هنگام ویرایش نوت‌بوک import کنید. با این حال،
packageهایی که از طریق رابط کاربری marimo هنگام اجرا به این روش نصب شوند، به project شما اضافه
نمی‌شوند و ممکن است در فراخوانی‌های بعدی marimo ناپدید شوند.

## استفاده از marimo در یک environment غیر project {#using-marimo-in-a-non-project-environment}

برای اجرای marimo در یک virtual environment که به یک
[project](../../concepts/projects/index.md) مرتبط نیست، marimo را مستقیماً به environment اضافه
کنید:

```console
$ uv venv
$ uv pip install numpy
$ uv pip install marimo
$ uv run marimo edit
```

از اینجا، دستور `import numpy` در نوت‌بوک کار می‌کند و نصب‌کنندهٔ رابط کاربری marimo، packageها را
از طرف شما با `uv pip install` به environment اضافه می‌کند.

## اجرای نوت‌بوک‌های marimo به‌عنوان script {#running-marimo-notebooks-as-scripts}

صرف‌نظر از اینکه dependencyهای شما چگونه مدیریت می‌شوند (با inline script metadata، در یک project
یا با یک environment غیر project)، می‌توانید نوت‌بوک‌های marimo را به‌عنوان script با این دستور اجرا
کنید:

```console
$ uv run my_notebook.py
```

این کار نوت‌بوک شما را به‌عنوان یک script مربوط به Python اجرا می‌کند، بدون اینکه یک نشست تعاملی در
مرورگر شما باز کند.
