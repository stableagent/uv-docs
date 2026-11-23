# ایجاد پروژه‌ها {#creating-projects}

uv از ایجاد یک project با `uv init` پشتیبانی می‌کند.

هنگام ایجاد projectها، uv از دو قالب پایه پشتیبانی می‌کند: [**applicationها**](#applications) و
[**libraryها**](#libraries). به‌طور پیش‌فرض، uv یک project برای یک application ایجاد می‌کند. فلگ `--lib`
را می‌توان برای ایجاد یک project برای یک library به‌جای آن استفاده کرد.

در هر دو حالت، uv ترجیح می‌دهد یک [build system](./config.md#build-systems) تعریف کند و فایل‌های منبع
را در یک پوشهٔ اختصاصی `src/<project_name>/` قرار دهد. تعریف یک build system امکان استفاده از ویژگی‌های
مختلف بسته‌بندی Python را فراهم می‌کند، مانند افزودن entry pointهای خط فرمان، و از نقاط رایج
سردرگمی در سیستم import Python جلوگیری می‌کند. استفاده از build system را می‌توان با استفاده از
گزینه‌های [`--no-package`](#creating-a-project-without-a-build-system) یا
[`--bare`](#creating-a-minimal-project) غیرفعال کرد.

!!! note

    پیش از نسخهٔ 0.12، uv به‌طور پیش‌فرض برای applicationها build system تعریف نمی‌کرد.

## پوشهٔ هدف {#target-directory}

uv یک project را در پوشهٔ کاری ایجاد می‌کند. برای ایجاد یکی در یک پوشهٔ هدف، یک
نام بدهید، برای مثال `uv init foo`. پوشهٔ کاری را می‌توان با گزینهٔ `--directory` تغییر داد،
که باعث می‌شود مسیر پوشهٔ هدف نسبت به پوشهٔ کاری مشخص‌شده تفسیر شود. اگر از قبل یک project در پوشهٔ هدف وجود داشته باشد، یعنی اگر یک
`pyproject.toml` وجود داشته باشد، uv با یک خطا خارج می‌شود.

## Applicationها {#applications}

projectهای application برای وب‌سرورها، scriptها و رابط‌های خط فرمان مناسب‌اند.

Applicationها هدف پیش‌فرض `uv init` هستند، اما می‌توان آن‌ها را با فلگ `--app` نیز مشخص کرد:

```console
$ uv init example-app
```

کد منبع در یک پوشهٔ `src` با یک پوشهٔ module و یک فایل `__init__.py` قرار دارد:

```console
$ tree example-app
example-app/
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_app
        └── __init__.py
```

یک [build system](./config.md#build-systems) تعریف شده است، بنابراین project در
environment نصب خواهد شد:

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-app = "example_app:main"

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

!!! tip

    می‌توان از گزینهٔ `--build-backend` برای درخواست یک build system جایگزین استفاده کرد.

یک تعریف [command](./config.md#entry-points) نیز گنجانده شده است:

```toml title="pyproject.toml" hl_lines="9 10"
[project]
name = "example-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-app = "example_app:main"

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

می‌توان دستور را با `uv run` اجرا کرد:

```console
$ cd example-app
$ uv run example-app
Hello from example-app!
```

## Libraryها {#libraries}

یک library توابع و آبجکت‌هایی را برای مصرف projectهای دیگر فراهم می‌کند. Libraryها برای این در نظر گرفته شده‌اند که
ساخته و توزیع شوند، برای مثال با بارگذاری آن‌ها در PyPI.

می‌توان libraryها را با استفاده از فلگ `--lib` ایجاد کرد:

```console
$ uv init --lib example-lib
```

!!! note

    Libraryها همیشه به یک project بسته‌بندی‌شده نیاز دارند.

یک نشانگر `py.typed` گنجانده شده است تا به مصرف‌کنندگان نشان دهد که می‌توان typeها را از library خواند:

```console
$ tree example-lib
example-lib/
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_lib
        ├── py.typed
        └── __init__.py
```

!!! note

    یک چیدمان `src` هنگام توسعهٔ libraryها به‌ویژه ارزشمند است. این چیدمان تضمین می‌کند که library
    از هر فراخوانی `python` در ریشهٔ project جدا باشد و کد library توزیع‌شده
    به‌خوبی از بقیهٔ منبع project جدا شود.

یک [build system](./config.md#build-systems) تعریف شده است، بنابراین project در
environment نصب خواهد شد:

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-lib"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[build-system]
requires = ["uv_build>=0.13.9,<0.14"]
build-backend = "uv_build"
```

!!! tip

    می‌توانید با استفاده از `--build-backend` همراه با `hatchling`،
    `uv_build`، `flit-core`، `pdm-backend`، `setuptools`، `maturin` یا `scikit-build-core` یک قالب build backend متفاوت را انتخاب کنید. اگر می‌خواهید یک [library با extension moduleها](#projects-with-extension-modules) ایجاد کنید، به یک backend جایگزین نیاز دارید.

module ایجادشده یک تابع API ساده را تعریف می‌کند:

```python title="__init__.py"
def hello() -> str:
    return "Hello from example-lib!"
```

و می‌توانید آن را با استفاده از `uv run` import و اجرا کنید:

```console
$ cd example-lib
$ uv run python -c "import example_lib; print(example_lib.hello())"
Hello from example-lib!
```

## Projectها با extension moduleها {#projects-with-extension-modules}

بیشتر projectهای Python «Python خالص» هستند، یعنی moduleهایی را به زبان‌های دیگر مانند
C، C++، FORTRAN یا Rust تعریف نمی‌کنند. با این حال، projectهای دارای extension module اغلب برای کدهای
حساس به عملکرد استفاده می‌شوند.

ایجاد یک project با یک extension module نیازمند انتخاب یک build system جایگزین است. uv
از ایجاد projectها با build systemهای زیر که از ساخت extension moduleها پشتیبانی می‌کنند، پشتیبانی می‌کند:

- [`maturin`](https://www.maturin.rs) برای projectهای دارای Rust
- [`scikit-build-core`](https://github.com/scikit-build/scikit-build-core) برای projectهای دارای C، C++،
  FORTRAN، Cython

build system را با فلگ `--build-backend` مشخص کنید:

```console
$ uv init --build-backend maturin example-ext
```

!!! note

    استفاده از `--build-backend` به‌معنای `--package` است.

project علاوه بر فایل‌های معمول project Python، شامل یک فایل `Cargo.toml` و یک فایل `lib.rs` است:

```console
$ tree example-ext
example-ext/
├── .python-version
├── Cargo.toml
├── README.md
├── pyproject.toml
└── src
    ├── lib.rs
    └── example_ext
        ├── __init__.py
        └── _core.pyi
```

!!! note

    اگر از `scikit-build-core` استفاده می‌کنید، به‌جای آن پیکربندی CMake و یک فایل `main.cpp` را خواهید دید.

library مربوط به Rust یک تابع ساده را تعریف می‌کند:

```rust title="src/lib.rs"
use pyo3::prelude::*;

#[pymodule]
mod _core {
    use pyo3::prelude::*;

    #[pyfunction]
    fn hello_from_bin() -> String {
        "Hello from example-ext!".to_string()
    }
}
```

و module Python آن را import می‌کند:

```python title="src/example_ext/__init__.py"
from example_ext._core import hello_from_bin


def main() -> None:
    print(hello_from_bin())
```

می‌توان دستور را با `uv run` اجرا کرد:

```console
$ cd example-ext
$ uv run example-ext
Hello from example-ext!
```

!!! important

    هنگام ایجاد یک project با maturin یا scikit-build-core، uv [`tool.uv.cache-keys`](../../reference/settings.md#cache-keys) را
    پیکربندی می‌کند تا انواع رایج فایل منبع را شامل شود. برای اجبار به بازسازی، برای مثال هنگام تغییر فایل‌های خارج از
    `cache-keys` یا هنگام عدم استفاده از `cache-keys`، از `--reinstall` استفاده کنید.

## ایجاد یک project بدون build system {#creating-a-project-without-a-build-system}

در حالی که تعریف یک build system به‌طور کلی تجربهٔ بهتری فراهم می‌کند، مواردی وجود دارد که در آن‌ها
حذف آن و تعریف moduleهای Python به‌طور مستقیم در پوشهٔ سطح بالا آسان‌تر است.

از فلگ `--no-package` برای غیرفعال کردن استفاده از build system استفاده کنید:

```console
$ uv init --no-package example-app
```

project شامل یک `pyproject.toml`، یک فایل نمونه (`main.py`)، یک readme و یک فایل
pin نسخهٔ Python (`.python-version`) است.

```console
$ tree example-app
example-app/
├── .python-version
├── README.md
├── main.py
└── pyproject.toml
```

فایل `pyproject.toml` شامل metadata پایه است اما build system تعریف نمی‌کند. project یک
[package](./config.md#project-packaging) نیست و در environment نصب نخواهد شد:

```toml title="pyproject.toml"
[project]
name = "example-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []
```

فایل نمونه یک تابع `main` را با مقداری کد قالب استاندارد تعریف می‌کند:

```python title="main.py"
def main():
    print("Hello from example-app!")


if __name__ == "__main__":
    main()
```

فایل‌های Python را می‌توان با `uv run` اجرا کرد:

```console
$ cd example-app
$ uv run main.py
Hello from example-app!
```

## ایجاد یک project حداقلی {#creating-a-minimal-project}

اگر فقط می‌خواهید یک `pyproject.toml` ایجاد کنید، از گزینهٔ `--bare` استفاده کنید:

```console
$ uv init example-bare --bare
```

uv از ایجاد یک فایل pin نسخهٔ Python، یک README و هر پوشه یا فایل منبعی صرف‌نظر می‌کند.
علاوه بر این، uv یک سیستم کنترل نسخه (یعنی `git`) را مقداردهی اولیه نمی‌کند.

```console
$ tree example-bare
example-bare
└── pyproject.toml
```

uv همچنین metadata اضافی به `pyproject.toml` اضافه نمی‌کند، مانند `description` یا `authors`.

```toml
[project]
name = "example-bare"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []
```

می‌توان از گزینهٔ `--bare` همراه با گزینه‌های دیگر مانند `--lib` یا `--build-backend` استفاده کرد — در این
موارد uv همچنان یک build system را پیکربندی می‌کند اما ساختار فایل مورد انتظار را ایجاد نمی‌کند.

هنگامی که از `--bare` استفاده می‌شود، همچنان می‌توان ویژگی‌های اضافی را به‌صورت اختیاری استفاده کرد:

```console
$ uv init example-bare --bare --description "Hello world" --author-from git --vcs git --pin-python
```
