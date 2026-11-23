# قابلیت‌ها {#features}

uv قابلیت‌های ضروری برای توسعهٔ Python را فراهم می‌کند — از نصب Python و کار روی script های
ساده تا کار روی project های بزرگی که از چند Python version و platform پشتیبانی می‌کنند.

رابط uv را می‌توان به بخش‌هایی تقسیم کرد که به‌صورت مستقل یا با هم قابل استفاده‌اند.

## Python version ها {#python-versions}

نصب و مدیریت خود Python.

- `uv python install`: نصب Python version ها.
- `uv python list`: مشاهدهٔ Python version های موجود.
- `uv python find`: پیدا کردن یک Python version نصب‌شده.
- `uv python pin`: پین کردن project جاری برای استفاده از یک Python version مشخص.
- `uv python uninstall`: حذف نصب یک Python version.

برای شروع، [راهنمای نصب Python](../guides/install-python.md) را ببینید.

## Script ها {#scripts}

اجرای script های مستقل Python؛ مثلاً `example.py`.

- `uv run`: اجرای یک script.
- `uv add --script`: افزودن یک dependency به یک script.
- `uv remove --script`: حذف یک dependency از یک script.

برای شروع، [راهنمای اجرای script ها](../guides/scripts.md) را ببینید.

## Project ها {#projects}

ساختن و کار روی project های Python؛ یعنی با یک `pyproject.toml`.

- `uv init`: ساختن یک project جدید Python.
- `uv add`: افزودن یک dependency به project.
- `uv remove`: حذف یک dependency از project.
- `uv sync`: همگام‌سازی dependency های project با environment.
- `uv lock`: ساختن یک lockfile برای dependency های project.
- `uv run`: اجرای یک دستور در environment پروژه.
- `uv tree`: مشاهدهٔ درخت dependency برای project.
- `uv build`: ساختن project به‌صورت archive های distribution.
- `uv publish`: منتشر کردن project در یک package index.

برای شروع، [راهنمای project ها](../guides/projects.md) را ببینید.

## Tool ها {#tools}

اجرا و نصب tool های منتشرشده در index های package Python؛ مثلاً `ruff` یا `black`.

- `uvx` / `uv tool run`: اجرای یک tool در یک environment موقت.
- `uv tool install`: نصب یک tool برای کل کاربر.
- `uv tool uninstall`: حذف نصب یک tool.
- `uv tool list`: فهرست tool های نصب‌شده.
- `uv tool update-shell`: به‌روزرسانی shell برای گنجاندن executable های tool.

برای شروع، [راهنمای tool ها](../guides/tools.md) را ببینید.

## رابط pip {#the-pip-interface}

مدیریت دستی environment ها و package ها — برای استفاده در workflow های قدیمی یا مواردی که
دستورهای سطح بالا کنترل کافی فراهم نمی‌کنند.

ساختن virtual environment ها (جایگزین `venv` و `virtualenv`):

- `uv venv`: ساختن یک virtual environment جدید.

برای جزئیات، مستندات [استفاده از environment ها](../pip/environments.md) را ببینید.

مدیریت package ها در یک environment (جایگزین [`pip`](https://github.com/pypa/pip) و
[`pipdeptree`](https://github.com/tox-dev/pipdeptree)):

- `uv pip install`: نصب package ها در environment جاری.
- `uv pip show`: نمایش جزئیات یک package نصب‌شده.
- `uv pip freeze`: فهرست package های نصب‌شده و version های آن‌ها.
- `uv pip check`: بررسی اینکه environment جاری package های سازگار دارد.
- `uv pip list`: فهرست package های نصب‌شده.
- `uv pip uninstall`: حذف نصب package ها.
- `uv pip tree`: مشاهدهٔ درخت dependency برای environment.

برای جزئیات، مستندات [مدیریت package ها](../pip/packages.md) را ببینید.

قفل کردن package ها در یک environment (جایگزین [`pip-tools`](https://github.com/jazzband/pip-tools)):

- `uv pip compile`: کامپایل requirement ها به یک lockfile.
- `uv pip sync`: همگام‌سازی یک environment با یک lockfile.

برای جزئیات، مستندات [قفل کردن environment ها](../pip/compile.md) را ببینید.

!!! important

    این دستورها دقیقاً رابط‌ها و رفتار tool هایی را که بر اساس آن‌ها ساخته شده‌اند پیاده
    نمی‌کنند. هرچه بیشتر از workflow های رایج فاصله بگیرید، احتمال اینکه با تفاوت‌هایی روبه‌رو
    شوید بیشتر می‌شود. برای جزئیات، [راهنمای سازگاری با pip](../pip/compatibility.md) را ببینید.

## ابزارهای کمکی {#utility}

مدیریت و بررسی وضعیت uv؛ مانند cache و پوشه‌های storage، یا انجام به‌روزرسانی خودکار:

- `uv cache clean`: حذف ورودی‌های cache.
- `uv cache prune`: حذف ورودی‌های قدیمی cache و همهٔ environment های متمرکز project.
- `uv cache dir`: نمایش مسیر پوشهٔ cache مربوط به uv.
- `uv tool dir`: نمایش مسیر پوشهٔ tool مربوط به uv.
- `uv python dir`: نمایش مسیر Python version های نصب‌شدهٔ uv.
- `uv self update`: به‌روزرسانی uv به آخرین version.

## قدم‌های بعدی {#next-steps}

برای آشنایی با هر قابلیت، [راهنماها](../guides/index.md) را بخوانید، برای جزئیات عمیق‌تر
دربارهٔ قابلیت‌های uv صفحه‌های [مفاهیم](../concepts/index.md) را ببینید، یا اگر به مشکلی
برخوردید یاد بگیرید چطور [کمک بگیرید](./help.md).
