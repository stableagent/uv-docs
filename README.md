# داکیومنت رسمی uv — نسخهٔ ریپازیتوری

این ریپازیتوری یک کپی از **داکیومنت رسمی uv** است، برای مطالعهٔ آفلاین و مرجع.

> ⚠️ این ریپو، ریپوی رسمی Astral نیست. مرجع اصلی و به‌روز همیشه این‌هاست:
> - سایت داکیومنت: https://docs.astral.sh/uv/
> - ریپوی رسمی: https://github.com/astral-sh/uv
> - پوشهٔ داکیومنت در ریپوی رسمی: https://github.com/astral-sh/uv/tree/main/docs
>
> اگر جایی بین این کپی و سایت رسمی تناقض بود، حرف آخر را سایت رسمی می‌زند.

## uv چیست؟

`uv` یک پکیج‌منیجر و مدیر پروژهٔ پایتون بسیار سریع است که با Rust نوشته شده و جای `pip` ،`pip-tools` ،`pipx` ،`poetry` ،`pyenv` ،`twine` ،`virtualenv` و چند ابزار دیگر را می‌گیرد.

## منبع این کپی

- منبع: `astral-sh/uv` — شاخهٔ `main`
- کامیت منبع: `264d8b4a5e10d195d2cc2a34dffa807cfbe6298c`
- تاریخ کامیت منبع: 2026-11-21
- پیام کامیت: `Skip distribution selection for packages excluded from installation (#23338)`
- پوشهٔ کپی‌شده: `docs/` — دقیقاً همان پوشه‌ای که سایت `docs.astral.sh/uv` از روی آن ساخته می‌شود
- تعداد فایل‌ها در `docs/`: 93 فایل — 81 فایل Markdown، بقیه تصویر، CSS، JS و فایل‌های قالب سایت
- جزئیات کامل در فایل `SOURCE.md`

## ساختار ریپو

```text
uv-docs/
├── README.md               # همین فایل — معرفی فارسی
├── README-upstream.md      # README رسمی ریپوی astral-sh/uv
├── SOURCE.md               # منبع، کامیت و لایسنس این کپی
├── LICENSE-MIT             # لایسنس MIT پروژهٔ uv
├── LICENSE-APACHE          # لایسنس Apache-2.0 پروژهٔ uv
├── mkdocs.yml              # تنظیمات سایت داکیومنت (از ریپوی رسمی)
└── docs/
    ├── index.md            # صفحهٔ اصلی داکیومنت
    ├── getting-started/    # شروع کار: نصب، اولین قدم‌ها، ویژگی‌ها
    ├── guides/             # راهنماها: پروژه، اسکریپت، ابزار، مهاجرت، یکپارچه‌سازی
    ├── concepts/           # مفاهیم: پروژه، وابستگی، کش، پیکربندی و…
    ├── pip/                # رابط سازگار با pip
    └── reference/          # مرجع: ذخیره‌سازی، سیاست‌ها، عیب‌یابی، مشارکت (CLI و تنظیمات موقع ساخت سایت تولید می‌شوند)
```

## از کجا شروع کنم؟

1. `docs/index.md` — معرفی کلی
2. `docs/getting-started/installation.md` — نصب
3. `docs/getting-started/first-steps.md` — اولین قدم‌ها
4. `docs/guides/projects.md` — کار با پروژه‌ها
5. `docs/guides/install-python.md` — نصب و مدیریت نسخهٔ پایتون
6. مرجع دستورات CLI — در این snapshot به‌صورت فایل `cli.md` نیست؛ موقع ساخت سایت از روی کد uv تولید می‌شود. روی سیستم خودت با `uv help` ببینید، یا در `docs/reference/index.md` لینک آن را ببینید.

## مشاهده به‌صورت سایت

اگر `mkdocs` و افزونه‌های آن را دارید، از ریشهٔ ریپو:

```bash
mkdocs serve
```

در غیر این صورت، فایل‌های `.md` داخل `docs/` را مستقیم با هر Markdown Reader بخوانید؛ نیازی به ساخت سایت نیست.

## به‌روزرسانی

برای گرفتن نسخهٔ جدید داکیومنت رسمی:

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/astral-sh/uv.git /tmp/uv-upstream
cd /tmp/uv-upstream
git sparse-checkout set docs
cp -r docs /path/to/this-repo/docs
```

بعد کامیت جدید را در `SOURCE.md` ثبت کنید.

## لایسنس

محتوای uv تحت دو لایسنس MIT و Apache-2.0 منتشر می‌شود؛ هر دو فایل لایسنس در ریشهٔ همین ریپو هست. این کپی فقط برای مرجع و مطالعه است و مالکیت آن متعلق به Astral Software Inc. و مشارکت‌کنندگان uv است.
