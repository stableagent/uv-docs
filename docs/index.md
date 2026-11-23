# uv {#uv}

یک مدیر package و project بسیار سریع برای Python که با Rust نوشته شده است.

<p align="center">
  <img alt="نمودار میله‌ای با نتایج benchmark را نشان می‌دهد." src="https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d#only-light">
</p>

<p align="center">
  <img alt="نمودار میله‌ای با نتایج benchmark را نشان می‌دهد." src="https://github.com/astral-sh/uv/assets/1309177/03aa9163-1c79-4a87-a31d-7a9311ed9310#only-dark">
</p>

<p align="center">
  <i>نصب dependencies های <a href="https://trio.readthedocs.io/">Trio</a> با یک cache گرم.</i>
</p>

## نکات برجسته {#highlights}

- یک tool واحد که جایگزین `pip`، `pip-tools`، `pipx`، `poetry`، `pyenv`، `twine`، `virtualenv` و
  موارد دیگر می‌شود.
- [۱۰ تا ۱۰۰ برابر سریع‌تر](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md) از `pip`.
- [مدیریت جامع project](#projects) را فراهم می‌کند، همراه با یک
  [lockfile سراسری](./concepts/projects/layout.md#the-lockfile).
- [اجرای script ها](#scripts)، با پشتیبانی از
  [metadata درون‌خطی dependency](./guides/scripts.md#declaring-script-dependencies).
- [نصب و مدیریت](#python-versions) Python version ها.
- [اجرا و نصب](#tools) tool های منتشرشده به‌عنوان package های Python.
- شامل یک [رابط سازگار با pip](#the-pip-interface) برای افزایش کارایی با یک CLI
  آشنا است.
- از [workspace](./concepts/projects/workspaces.md) های به‌سبک Cargo برای project های
  مقیاس‌پذیر پشتیبانی می‌کند.
- بهینه در مصرف فضای دیسک، با یک [cache سراسری](./concepts/cache.md) برای حذف تکراری
  dependency ها.
- قابل نصب بدون Rust یا Python با `curl` یا `pip`.
- از macOS و Linux و Windows پشتیبانی می‌کند.

uv از سوی [Astral](https://astral.sh) پشتیبانی می‌شود؛ سازندگان
[Ruff](https://github.com/astral-sh/ruff).

## نصب {#installation}

uv را با نصب‌کنندهٔ مستقل رسمی ما نصب کنید:

=== "macOS and Linux"

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

=== "Windows"

    ```pwsh-session
    PS> powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```

سپس، [قدم‌های اول](./getting-started/first-steps.md) را ببینید یا برای یک مرور کوتاه ادامه دهید.

!!! tip

    uv را می‌توان با pip و Homebrew و موارد دیگر هم نصب کرد. همهٔ روش‌ها را در
    [صفحهٔ نصب](./getting-started/installation.md) ببینید.

## Project ها {#projects}

uv dependency ها و environment های project را مدیریت می‌کند، با پشتیبانی از lockfile ها،
workspace ها و موارد دیگر، مشابه `rye` یا `poetry`:

```console
$ uv init example
Initialized project `example` at `/home/user/example`

$ cd example

$ uv add ruff
Creating virtual environment at: .venv
Resolved 2 packages in 170ms
   Built example @ file:///home/user/example
Prepared 2 packages in 627ms
Installed 2 packages in 1ms
 + example==0.1.0 (from file:///home/user/example)
 + ruff==0.5.4

$ uv run ruff check
All checks passed!

$ uv lock
Resolved 2 packages in 0.33ms

$ uv sync
Resolved 2 packages in 0.70ms
Checked 1 package in 0.02ms
```

برای شروع، [راهنمای project](./guides/projects.md) را ببینید.

uv از build و منتشر کردن project ها هم پشتیبانی می‌کند، حتی اگر با uv مدیریت نشوند. برای
اطلاعات بیشتر، [راهنمای packaging](./guides/package.md) را ببینید.

## Script ها {#scripts}

uv dependency ها و environment ها را برای script های تک‌فایلی مدیریت می‌کند.

یک script جدید بسازید و metadata درون‌خطی که dependency های آن را اعلام می‌کند اضافه کنید:

```console
$ echo 'import requests; print(requests.get("https://astral.sh"))' > example.py

$ uv add --script example.py requests
Updated `example.py`
```

سپس، script را در یک virtual environment مجزا اجرا کنید:

```console
$ uv run example.py
Reading inline script metadata from: example.py
Installed 5 packages in 12ms
<Response [200]>
```

برای شروع، [راهنمای script ها](./guides/scripts.md) را ببینید.

## Tool ها {#tools}

uv tool های خط فرمان ارائه‌شده توسط package های Python را اجرا و نصب می‌کند، مشابه `pipx`.

یک tool را در یک environment موقت با `uvx` اجرا کنید (نام مستعار `uv tool run`):

```console
$ uvx pycowsay 'hello world!'
Resolved 1 package in 167ms
Installed 1 package in 9ms
 + pycowsay==0.0.0.2
  """

  ------------
< hello world! >
  ------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```

یک tool را با `uv tool install` نصب کنید:

```console
$ uv tool install ruff
Resolved 1 package in 6ms
Installed 1 package in 2ms
 + ruff==0.5.4
Installed 1 executable: ruff

$ ruff --version
ruff 0.5.4
```

برای شروع، [راهنمای tool ها](./guides/tools.md) را ببینید.

## Python version ها {#python-versions}

uv Python را نصب می‌کند و امکان جابه‌جایی سریع بین version ها را می‌دهد.

چند Python version نصب کنید:

```console
$ uv python install 3.10 3.11 3.12
Searching for Python versions matching: Python 3.10
Searching for Python versions matching: Python 3.11
Searching for Python versions matching: Python 3.12
Installed 3 versions in 3.42s
 + cpython-3.10.14-macos-aarch64-none
 + cpython-3.11.9-macos-aarch64-none
 + cpython-3.12.4-macos-aarch64-none
```

Python version ها را در صورت نیاز دانلود کنید:

```console
$ uv venv --python 3.12.0
Using CPython 3.12.0
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

$ uv run --python pypy@3.8 -- python
Python 3.8.16 (a9dbdca6fc3286b0addd2240f11d97d8e8de187a, Dec 29 2022, 11:45:30)
[PyPy 7.3.11 with GCC Apple LLVM 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>>
```

از یک Python version مشخص در پوشهٔ جاری استفاده کنید:

```console
$ uv python pin 3.11
Pinned `.python-version` to `3.11`
```

برای شروع، [راهنمای نصب Python](./guides/install-python.md) را ببینید.

## رابط pip {#the-pip-interface}

uv یک جایگزین drop-in برای دستورهای رایج `pip`، `pip-tools` و `virtualenv` فراهم می‌کند.

uv رابط‌های آن‌ها را با قابلیت‌های پیشرفته گسترش می‌دهد؛ مانند override های version
dependency، resolution های مستقل از platform، resolution های قابل بازتولید، استراتژی‌های
جایگزین resolution و موارد دیگر.

بدون تغییر workflow های موجود خود به uv مهاجرت کنید — و با رابط `uv pip`، سرعت ۱۰ تا ۱۰۰
برابری را تجربه کنید.

requirement ها را در یک فایل requirement مستقل از platform کامپایل کنید:

```console
$ uv pip compile requirements.in \
   --universal \
   --output-file requirements.txt
Resolved 43 packages in 12ms
```

یک virtual environment بسازید:

```console
$ uv venv
Using CPython 3.12.3
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate
```

requirement های قفل‌شده را نصب کنید:

```console
$ uv pip sync requirements.txt
Resolved 43 packages in 11ms
Installed 43 packages in 208ms
 + babel==2.15.0
 + black==24.4.2
 + certifi==2024.7.4
 ...
```

برای شروع، [مستندات رابط pip](./pip/index.md) را ببینید.

## بیشتر بدانید {#learn-more}

[قدم‌های اول](./getting-started/first-steps.md) را ببینید یا مستقیم به
[راهنماها](./guides/index.md) بروید تا استفاده از uv را شروع کنید.
