# ذخیره‌سازی {#storage}

## پوشه‌های ذخیره‌سازی {#storage-directories}

uv از پوشه‌های سطح بالای زیر برای ذخیره‌سازی استفاده می‌کند.

برای هر مکان، uv وجود environment variableها را به ترتیبی که داده شده بررسی می‌کند و از اولین
مسیری که پیدا شود استفاده می‌کند.

مسیرهای پوشه‌های ذخیره‌سازی به platform وابسته‌اند. uv در Linux و macOS از قراردادهای
[XDG](https://specifications.freedesktop.org/basedir-spec/latest/) و در Windows از طرح
[Known Folder](https://learn.microsoft.com/en-us/windows/win32/shell/known-folders) پیروی می‌کند.

### پوشهٔ موقت {#temporary-directory}

پوشهٔ موقت برای داده‌های موقت استفاده می‌شود.

=== "Unix"

    1. `$TMPDIR`
    1. `/tmp`

=== "Windows"

    1. `%TMP%`
    1. `%TEMP%`
    1. `%USERPROFILE%`

### پوشهٔ cache {#cache-directory}

پوشهٔ cache برای داده‌های دورریختنی‌ای استفاده می‌شود که نگه داشتن آن‌ها بین فراخوانی‌ها مفید
است.

=== "Unix"

    1. `$XDG_CACHE_HOME/uv`
    1. `$HOME/.cache/uv`

=== "Windows"

    1. `%LOCALAPPDATA%\uv\cache`
    1. `uv\cache` درون [`FOLDERID_LocalAppData`](https://learn.microsoft.com/en-us/windows/win32/shell/knownfolderid#FOLDERID_LocalAppData)

### پوشهٔ داده‌های ماندگار {#persistent-data-directory}

پوشهٔ داده‌های ماندگار برای داده‌های غیر دورریختنی استفاده می‌شود.

=== "Unix"

    1. `$XDG_DATA_HOME/uv`
    1. `$HOME/.local/share/uv`
    1. `$CWD/.uv`

=== "Windows"

    1. `%APPDATA%\uv\data`
    1. `.\.uv`

### پوشه‌های پیکربندی {#configuration-directories}

پوشه‌های پیکربندی برای ذخیرهٔ تغییرات تنظیمات uv استفاده می‌شوند.

پیکربندی در سطح کاربر

=== "Unix"

    1. `$XDG_CONFIG_HOME/uv`
    1. `$HOME/.config/uv`

=== "Windows"

    1. `%APPDATA%\uv`
    1. `uv` درون [`FOLDERID_RoamingAppData`](https://learn.microsoft.com/en-us/windows/win32/shell/knownfolderid#FOLDERID_RoamingAppData)

پیکربندی در سطح سیستم

=== "Unix"

    1. `$XDG_CONFIG_DIRS/uv`
    1. `/etc/uv`

=== "Windows"

    1. `%PROGRAMDATA%\uv`
    1. `uv` درون [`FOLDERID_AppDataProgramData`](https://learn.microsoft.com/en-us/windows/win32/shell/knownfolderid#FOLDERID_AppDataProgramData)

### پوشهٔ اجرایی {#executable-directory}

پوشهٔ اجرایی برای ذخیرهٔ فایل‌هایی استفاده می‌شود که کاربر می‌تواند آن‌ها را اجرا کند؛ یعنی
پوشه‌ای که باید در `PATH` باشد.

=== "Unix"

    1. `$XDG_BIN_HOME`
    1. `$XDG_DATA_HOME/../bin`
    1. `$HOME/.local/bin`

=== "Windows"

    1. `%XDG_BIN_HOME%`
    1. `%XDG_DATA_HOME%\..\bin`
    1. `%USERPROFILE%\.local\bin`

## انواع داده {#types-of-data}

### cache مربوط به dependencyها {#dependency-cache}

uv از یک cache محلی استفاده می‌کند تا از دانلود و build دوبارهٔ dependencyها جلوگیری کند.

به‌طور پیش‌فرض، cache در [پوشهٔ cache](#cache-directory) ذخیره می‌شود، اما می‌توان آن را با
آرگومان‌های خط فرمان، environment variableها یا تنظیمات، آن‌طور که در
[مستندات cache](../concepts/cache.md#cache-directory) آمده، جایگزین کرد. وقتی cache غیرفعال
باشد، cache در یک [پوشهٔ موقت](#temporary-directory) ذخیره می‌شود.

برای نمایش مسیر فعلی پوشهٔ cache، از `uv cache dir` استفاده کنید.

!!! important

    برای عملکرد بهینه، پوشهٔ cache باید روی همان سیستم فایلی باشد که virtual environmentها روی
    آن قرار دارند.

### نسخه‌های Python {#python-versions}

uv می‌تواند [نسخه‌های Python](../concepts/python-versions.md) مدیریت‌شده را نصب کند، مثلاً با
`uv python install`.

به‌طور پیش‌فرض، نسخه‌های Python مدیریت‌شده توسط uv در یک زیرپوشهٔ `python/` از
[پوشهٔ داده‌های ماندگار](#persistent-data-directory) ذخیره می‌شوند، مثلاً
`~/.local/share/uv/python`.

برای نمایش پوشهٔ نصب Python، از `uv python dir` استفاده کنید.

برای جایگزین کردن پوشهٔ نصب، از environment variable `UV_PYTHON_INSTALL_DIR` استفاده کنید.

!!! note

    تغییر محل نصب Python به‌طور خودکار در virtual environmentهای موجود اعمال نمی‌شود؛ آن‌ها
    همچنان به مکان قبلی اشاره می‌کنند و باید به‌صورت دستی به‌روز شوند (مثلاً با بازسازی آن‌ها).

### فایل‌های اجرایی Python {#python-executables}

uv فایل‌های اجرایی را برای [نسخه‌های Python](#python-versions) نصب می‌کند، مثلاً `python3.13`.

به‌طور پیش‌فرض، فایل‌های اجرایی Python در [پوشهٔ اجرایی](#executable-directory) ذخیره می‌شوند.

برای نمایش پوشهٔ فایل‌های اجرایی Python، از `uv python dir --bin` استفاده کنید.

برای جایگزین کردن پوشهٔ فایل‌های اجرایی Python، از environment variable `UV_PYTHON_BIN_DIR`
استفاده کنید.

### toolها {#tools}

uv می‌تواند packageهای Python را به‌عنوان [toolهای خط فرمان](../concepts/tools.md) با
`uv tool install` نصب کند.

به‌طور پیش‌فرض، toolها در یک زیرپوشهٔ `tools/` از
[پوشهٔ داده‌های ماندگار](#persistent-data-directory) نصب می‌شوند، مثلاً
`~/.local/share/uv/tools`.

برای نمایش پوشهٔ نصب toolها، از `uv tool dir` استفاده کنید.

برای پیکربندی پوشهٔ نصب، از environment variable `UV_TOOL_DIR` استفاده کنید.

### فایل‌های اجرایی toolها {#tool-executables}

uv فایل‌های اجرایی را برای [toolهای](#tools) نصب‌شده نصب می‌کند، مثلاً `ruff`.

به‌طور پیش‌فرض، فایل‌های اجرایی toolها در [پوشهٔ اجرایی](#executable-directory) ذخیره می‌شوند.

برای نمایش پوشهٔ فایل‌های اجرایی toolها، از `uv tool dir --bin` استفاده کنید.

برای پیکربندی پوشهٔ فایل‌های اجرایی toolها، از environment variable `UV_TOOL_BIN_DIR` استفاده
کنید.

### فایل اجرایی uv {#the-uv-executable}

وقتی از [نصب‌کنندهٔ مستقل](./installer.md) uv برای نصب uv استفاده می‌کنید، فایل‌های اجرایی `uv`
و `uvx` در [پوشهٔ اجرایی](#executable-directory) نصب می‌شوند.

برای پیکربندی پوشهٔ نصب uv، از environment variable `UV_INSTALL_DIR` استفاده کنید.

### فایل‌های پیکربندی {#configuration-files}

رفتار uv را می‌توان از طریق فایل‌های TOML پیکربندی کرد.

فایل‌های پیکربندی در [پوشه‌های پیکربندی](#configuration-directories) پیدا می‌شوند.

برای جزئیات بیشتر، [مستندات فایل‌های پیکربندی](../concepts/configuration-files.md) را ببینید.

### virtual environmentهای project {#project-virtual-environments}

هنگام کار روی [projectها](../concepts/projects/index.md)، uv برای هر project یک virtual environment
اختصاصی می‌سازد.

به‌طور پیش‌فرض، virtual environmentهای project در `.venv` در ریشهٔ project یا workspace ساخته
می‌شوند؛ یعنی در کنار `pyproject.toml`.

برای جایگزین کردن این مکان، از environment variable `UV_PROJECT_ENVIRONMENT` استفاده کنید. برای
جزئیات بیشتر، [مستندات environment مربوط به projectها](../concepts/projects/config.md#project-environment-path)
را ببینید.

با [قابلیت پیش‌نمایش `centralized-project-envs`](../concepts/preview.md)، uv environmentهای
پیش‌فرض project را در [پوشهٔ cache](#cache-directory) ذخیره می‌کند. می‌توان آن‌ها را با
`uv cache clean` یا `uv cache prune` حذف کرد و هنگام نیاز بعدی دوباره ساخته می‌شوند. برای جزئیات،
مستندات [environmentهای متمرکز project](../concepts/projects/layout.md#centralized-project-environments)
را ببینید.

### virtual environmentهای script {#script-virtual-environments}

هنگام اجرای [scriptهای دارای metadata درون‌خطی](../guides/scripts.md)، uv برای هر script یک virtual
environment اختصاصی در [پوشهٔ cache](#cache-directory) می‌سازد.
