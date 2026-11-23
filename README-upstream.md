# uv {#uv}

<a href="https://pypi.python.org/pypi/uv"><img src="https://img.shields.io/pypi/v/uv.svg" alt="آخرین version در PyPI" /></a>
<a href="https://pypi.python.org/pypi/uv"><img src="https://img.shields.io/pypi/pyversions/uv.svg" alt="Python version های پشتیبانی‌شده" /></a>
<a href="https://discord.gg/astral-sh"><img src="https://img.shields.io/badge/Discord-%235865F2.svg?logo=discord&logoColor=white" alt="Discord" /></a>

یک مدیر package و project بسیار سریع برای Python که با Rust نوشته شده است.

<p align="center">
  <picture align="center">
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/astral-sh/uv/assets/1309177/03aa9163-1c79-4a87-a31d-7a9311ed9310">
    <source media="(prefers-color-scheme: light)" srcset="https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d">
    <img alt="نمودار میله‌ای با نتایج benchmark را نشان می‌دهد." src="https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d">
  </picture>
</p>

<p align="center">
  <i>نصب dependencies های <a href="https://trio.readthedocs.io/">Trio</a> با یک cache گرم.</i>
</p>

## نکات برجسته {#highlights}

- یک tool واحد که جایگزین `pip`، `pip-tools`، `pipx`، `poetry`، `pyenv`، `twine`، `virtualenv` و
  موارد دیگر می‌شود.
- [۱۰ تا ۱۰۰ برابر سریع‌تر](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md) از `pip`.
- [مدیریت جامع project](#projects) را فراهم می‌کند، همراه با یک
  [lockfile سراسری](https://docs.astral.sh/uv/concepts/projects/layout#the-lockfile).
- [اجرای script ها](#scripts)، با پشتیبانی از
  [metadata درون‌خطی dependency](https://docs.astral.sh/uv/guides/scripts#declaring-script-dependencies).
- [نصب و مدیریت](#python-versions) Python version ها.
- [اجرا و نصب](#tools) tool های منتشرشده به‌عنوان package های Python.
- شامل یک [رابط سازگار با pip](#the-pip-interface) برای افزایش کارایی با یک CLI
  آشنا است.
- از [workspace](https://docs.astral.sh/uv/concepts/projects/workspaces) های به‌سبک Cargo برای
  project های مقیاس‌پذیر پشتیبانی می‌کند.
- بهینه در مصرف فضای دیسک، با یک [cache سراسری](https://docs.astral.sh/uv/concepts/cache) برای
  حذف تکراری dependency ها.
- قابل نصب بدون Rust یا Python با `curl` یا `pip`.
- از macOS و Linux و Windows پشتیبانی می‌کند.

uv از سوی [Astral](https://astral.sh) پشتیبانی می‌شود؛ سازندگان
[Ruff](https://github.com/astral-sh/ruff) و [ty](https://github.com/astral-sh/ty).

## نصب {#installation}

uv را با installer های مستقل ما نصب کنید:

```bash
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash
# On Windows.
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

یا از [PyPI](https://pypi.org/project/uv/):

```bash
# With pip.
pip install uv
```

```bash
# Or pipx.
pipx install uv
```

اگر با installer مستقل نصب شده باشد، uv می‌تواند خودش را به آخرین version به‌روز کند:

```bash
uv self update
```

برای جزئیات و روش‌های نصب جایگزین، [مستندات نصب](https://docs.astral.sh/uv/getting-started/installation/)
را ببینید.

## مستندات {#documentation}

مستندات uv در [docs.astral.sh/uv](https://docs.astral.sh/uv) در دسترس است.

علاوه بر این، مستندات مرجع خط فرمان را می‌توان با `uv help` مشاهده کرد.

## قابلیت‌ها {#features}

### Project ها {#projects}

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
 + ruff==0.5.0

$ uv run ruff check
All checks passed!

$ uv lock
Resolved 2 packages in 0.33ms

$ uv sync
Resolved 2 packages in 0.70ms
Checked 1 package in 0.02ms
```

برای شروع، [مستندات project](https://docs.astral.sh/uv/guides/projects/) را ببینید.

uv از build و منتشر کردن project ها هم پشتیبانی می‌کند، حتی اگر با uv مدیریت نشوند. برای
اطلاعات بیشتر، [راهنمای publish](https://docs.astral.sh/uv/guides/publish/) را ببینید.

### Script ها {#scripts}

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

برای شروع، [مستندات script ها](https://docs.astral.sh/uv/guides/scripts/) را ببینید.

### Tool ها {#tools}

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
 + ruff==0.5.0
Installed 1 executable: ruff

$ ruff --version
ruff 0.5.0
```

برای شروع، [مستندات tool ها](https://docs.astral.sh/uv/guides/tools/) را ببینید.

### Python version ها {#python-versions}

uv Python را نصب می‌کند و امکان جابه‌جایی سریع بین version ها را می‌دهد.

چند Python version نصب کنید:

```console
$ uv python install 3.12 3.13 3.14
Installed 3 versions in 972ms
 + cpython-3.12.12-macos-aarch64-none (python3.12)
 + cpython-3.13.9-macos-aarch64-none (python3.13)
 + cpython-3.14.0-macos-aarch64-none (python3.14)

```

Python version ها را در صورت نیاز دانلود کنید:

```console
$ uv venv --python 3.12.0
Using Python 3.12.0
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

$ uv run --python pypy@3.8 -- python --version
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

برای شروع، [مستندات نصب Python](https://docs.astral.sh/uv/guides/install-python/) را ببینید.

### رابط pip {#the-pip-interface}

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
Using Python 3.12.3
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

برای شروع، [مستندات رابط pip](https://docs.astral.sh/uv/pip/index/) را ببینید.

## مشارکت {#contributing}

ما با اشتیاق از مشارکت‌کنندگان در همهٔ سطوح تجربه پشتیبانی می‌کنیم و دوست داریم شما را درگیر
project ببینیم. برای شروع، [راهنمای مشارکت](https://github.com/astral-sh/uv?tab=contributing-ov-file#contributing)
را ببینید.

## پرسش‌های متداول {#faq}

#### تلفظ uv چگونه است؟ {#how-do-you-pronounce-uv}

به‌صورت «یو – وی» تلفظ می‌شود ([`/juː viː/`](https://en.wikipedia.org/wiki/Help:IPA/English#Key))

#### uv را چطور باید نوشت؟ {#how-should-i-stylize-uv}

فقط «uv»، لطفاً. برای جزئیات، [راهنمای سبک](./STYLE.md#styling-uv) را ببینید.

#### uv از چه platform هایی پشتیبانی می‌کند؟ {#what-platforms-does-uv-support}

سند [پشتیبانی platform](https://docs.astral.sh/uv/reference/platforms/) مربوط به uv را ببینید.

#### آیا uv برای production آماده است؟ {#is-uv-ready-for-production}

بله، uv پایدار است و به‌طور گسترده در production استفاده می‌شود. برای جزئیات، سند
[سیاست versioning](https://docs.astral.sh/uv/reference/versioning/) مربوط به uv را ببینید.

## قدردانی {#acknowledgements}

resolver ی dependency مربوط به uv در پشت صحنه از [PubGrub](https://github.com/pubgrub-rs/pubgrub)
استفاده می‌کند. ما از نگهدارندگان PubGrub، به‌ویژه [Jacob Finkelman](https://github.com/Eh2406)،
بابت پشتیبانی‌شان سپاسگزاریم.

پیاده‌سازی Git در uv بر اساس [Cargo](https://github.com/rust-lang/cargo) است.

برخی از بهینه‌سازی‌های uv از کار بزرگی الهام گرفته شده است که در
[pnpm](https://pnpm.io/)، [Orogene](https://github.com/orogene/orogene) و
[Bun](https://github.com/oven-sh/bun) دیده‌ایم. ما همچنین از [Posy](https://github.com/njsmith/posy)
متعلق به Nathaniel J. Smith چیزهای زیادی آموخته‌ایم و
[trampoline](https://github.com/njsmith/posy/tree/main/src/trampolines/windows-trampolines/posy-trampoline)
آن را برای پشتیبانی از Windows تطبیق داده‌ایم.

## مجوز {#license}

uv تحت یکی از این دو مجوز است:

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
  <https://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/licenses/MIT>)

به انتخاب شما.

مگر اینکه صراحتاً خلاف آن را اعلام کنید، هر مشارکتی که عمداً برای گنجاندن در uv توسط شما ارسال
شود، همان‌طور که در مجوز Apache-2.0 تعریف شده است، تحت هر دو مجوز بالا خواهد بود، بدون هیچ شرط
یا ضابطهٔ اضافی.

<div align="center">
  <a target="_blank" href="https://astral.sh" style="background:none">
    <img src="https://raw.githubusercontent.com/astral-sh/uv/main/assets/svg/Astral.svg" alt="ساخته‌شده توسط Astral">
  </a>
</div>
