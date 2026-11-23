# مدیریت dependencies {#managing-dependencies}

## فیلدهای dependency {#dependency-fields}

dependencyهای project در چند فیلد تعریف می‌شوند:

- [`project.dependencies`](#project-dependencies): dependencyهای منتشرشده.
- [`project.optional-dependencies`](#optional-dependencies): dependencyهای اختیاری منتشرشده، یا
  «extra»ها.
- [`dependency-groups`](#dependency-groups): dependencyهای محلی برای توسعه.
- [`tool.uv.sources`](#dependency-sources): sourceهای جایگزین برای dependencyها در طول توسعه.

!!! note

    فیلدهای `project.dependencies` و `project.optional-dependencies` را می‌توان حتی اگر قرار
    نیست project منتشر شود هم استفاده کرد. `dependency-groups` یک قابلیت تازه استانداردشده است
    و ممکن است هنوز همهٔ toolها از آن پشتیبانی نکنند.

uv از تغییر dependencyهای project با `uv add` و `uv remove` پشتیبانی می‌کند، اما metadata مربوط
به dependency را می‌توان با ویرایش مستقیم `pyproject.toml` هم به‌روز کرد.

## افزودن dependencyها {#adding-dependencies}

برای افزودن یک dependency:

```console
$ uv add httpx
```

یک ورودی در فیلد `project.dependencies` اضافه خواهد شد:

```toml title="pyproject.toml" hl_lines="4"
[project]
name = "example"
version = "0.1.0"
dependencies = ["httpx>=0.27.2"]
```

می‌توان از فلگ‌های [`--dev`](#development-dependencies)، [`--group`](#dependency-groups) یا
[`--optional`](#optional-dependencies) برای افزودن dependencyها به یک فیلد جایگزین استفاده کرد.

dependency شامل یک constraint خواهد بود، مثلاً `>=0.27.2`، برای جدیدترین نسخهٔ سازگار package.
نوع bound را می‌توان با
[`--bounds`](../../reference/settings.md#add-bounds) تنظیم کرد، یا constraint را می‌توان مستقیم
ارائه داد:

```console
$ uv add "httpx>=0.20"
```

هنگام افزودن یک dependency از یک source غیر از registry پکیج، uv یک ورودی در فیلد sources اضافه
می‌کند. برای مثال، هنگام افزودن `httpx` از GitHub:

```console
$ uv add "httpx @ git+https://github.com/encode/httpx"
```

فایل `pyproject.toml` شامل یک [ورودی source از نوع Git](#git) خواهد بود:

```toml title="pyproject.toml" hl_lines="8-9"
[project]
name = "example"
version = "0.1.0"
dependencies = [
    "httpx",
]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

اگر نتوان از یک dependency استفاده کرد، uv یک خطا نمایش می‌دهد:

```console
$ uv add "httpx>9999"
  × No solution found when resolving dependencies:
  ╰─▶ Because only httpx<=1.0.0b0 is available and your project depends on httpx>9999,
      we can conclude that your project's requirements are unsatisfiable.
```

### وارد کردن dependencyها از فایل‌های requirements {#importing-dependencies-from-requirements-files}

dependencyهای اعلام‌شده در یک فایل `requirements.txt` را می‌توان با گزینهٔ `-r` به project اضافه
کرد:

```
uv add -r requirements.txt
```

برای جزئیات بیشتر، [راهنمای مهاجرت از pip](../../guides/migration/pip-to-project.md#importing-requirements-files)
را ببینید.

## حذف dependencyها {#removing-dependencies}

برای حذف یک dependency:

```console
$ uv remove httpx
```

می‌توان از فلگ‌های `--dev`، `--group` یا `--optional` برای حذف یک dependency از یک table مشخص
استفاده کرد.

اگر برای dependency حذف‌شده یک [source](#dependency-sources) تعریف شده باشد و هیچ ارجاع دیگری به
آن dependency وجود نداشته باشد، آن source هم حذف خواهد شد.

## تغییر dependencyها {#changing-dependencies}

برای تغییر یک dependency موجود، مثلاً برای استفاده از constraint متفاوتی برای `httpx`:

```console
$ uv add "httpx>0.1.0"
```

!!! note

    در این مثال، constraintهای dependency را در `pyproject.toml` تغییر می‌دهیم.
    نسخهٔ قفل‌شدهٔ dependency فقط در صورت لزوم برای برآورده کردن constraintهای جدید
    تغییر می‌کند. برای اینکه نسخهٔ package را مجبور کنید در محدودهٔ constraintها به جدیدترین
    نسخه به‌روز شود، از `--upgrade-package <name>` استفاده کنید، مثلاً:

    ```console
    $ uv add "httpx>0.1.0" --upgrade-package httpx
    ```

    برای جزئیات بیشتر دربارهٔ به‌روزرسانی packageها، مستندات
    [lockfile](./sync.md#upgrading-locked-package-versions) را ببینید.

درخواست یک source متفاوت برای dependency، جدول `tool.uv.sources` را به‌روز می‌کند، مثلاً برای
استفاده از `httpx` از یک مسیر محلی در طول توسعه:

```console
$ uv add "httpx @ ../httpx"
```

## dependencyهای مخصوص platform {#platform-specific-dependencies}

برای اینکه مطمئن شوید یک dependency فقط روی یک platform مشخص یا روی نسخه‌های مشخصی از Python نصب
می‌شود، از [environment markerها](https://peps.python.org/pep-0508/#environment-markers) استفاده
کنید.

برای مثال، برای نصب `jax` روی Linux، اما نه روی Windows یا macOS:

```console
$ uv add "jax; sys_platform == 'linux'"
```

در نتیجه، فایل `pyproject.toml` در تعریف dependency شامل environment marker خواهد بود:

```toml title="pyproject.toml" hl_lines="6"
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["jax; sys_platform == 'linux'"]
```

به همین ترتیب، برای گنجاندن `numpy` در Python 3.11 و نسخه‌های بعدی:

```console
$ uv add "numpy; python_version >= '3.11'"
```

برای فهرست کامل markerها و عملگرهای موجود، مستندات
[environment marker](https://peps.python.org/pep-0508/#environment-markers) مربوط به Python را
ببینید.

!!! tip

    sourceهای dependency را می‌توان [به‌صورت جداگانه برای هر platform هم تغییر داد](#platform-specific-sources).

## dependencyهای project {#project-dependencies}

جدول `project.dependencies` نشان‌دهندهٔ dependencyهایی است که هنگام آپلود در PyPI یا ساخت یک
wheel استفاده می‌شوند. dependencyهای منفرد با سینتکس
[dependency specifierها](https://packaging.python.org/en/latest/specifications/dependency-specifiers/)
مشخص می‌شوند و جدول از استاندارد
[PEP 621](https://packaging.python.org/en/latest/specifications/pyproject-toml/) پیروی می‌کند.

`project.dependencies` فهرست packageهایی را تعریف می‌کند که برای project لازم‌اند، همراه با هر
constraint نسخه‌ای که هنگام نصب آن‌ها استفاده شود. هر ورودی شامل نام یک dependency است و ممکن است
شامل constraintهای نسخه، extraها یا environment markerهایی برای packageهای مخصوص platform باشد.
برای مثال:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
dependencies = [
  # Any version in this range
  "tqdm >=4.66.2,<5",
  # Exactly this version of torch
  "torch ==2.2.2",
  # Install transformers with the torch extra
  "transformers[torch] >=4.39.3,<5",
  # Only install this package on older python versions
  # See "Environment Markers" for more information
  "importlib_metadata >=7.1.0,<8; python_version < '3.10'",
  "mollymawk ==0.1.0"
]
```

## sourceهای dependency {#dependency-sources}

جدول `tool.uv.sources` جدول‌های استاندارد dependency را با sourceهای جایگزین dependency گسترش
می‌دهد که در طول توسعه استفاده می‌شوند.

sourceهای dependency از الگوهای رایجی پشتیبانی می‌کنند که استاندارد `project.dependencies` از
آن‌ها پشتیبانی نمی‌کند، مانند editable installها و مسیرهای نسبی. برای مثال، برای نصب `foo` از یک
پوشه نسبت به ریشهٔ project:

```toml title="pyproject.toml" hl_lines="7"
[project]
name = "example"
version = "0.1.0"
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "./packages/foo" }
```

sourceهای dependency زیر توسط uv پشتیبانی می‌شوند:

- [Index](#index): یک package که از یک package index مشخص resolve می‌شود.
- [Git](#git): یک repository از نوع Git.
- [URL](#url): یک wheel یا source distribution راه دور.
- [Path](#path): یک wheel، source distribution یا پوشهٔ project محلی.
- [Workspace](#workspace-member): یک عضو از workspace فعلی.

!!! important

    sourceها فقط توسط uv رعایت می‌شوند. اگر از tool دیگری استفاده شود، فقط تعریف‌های موجود در
    جدول‌های استاندارد project استفاده خواهند شد. اگر برای توسعه از tool دیگری استفاده می‌شود، هر
    metadata ارائه‌شده در جدول source باید دوباره در قالب آن tool دیگر مشخص شود.

### Index {#index}

برای افزودن یک package از نوع Python از یک index مشخص، از گزینهٔ `--index` استفاده کنید:

```console
$ uv add torch --index pytorch=https://download.pytorch.org/whl/cpu
```

uv، index را در `[[tool.uv.index]]` ذخیره می‌کند و یک ورودی `[tool.uv.sources]` اضافه می‌کند:

```toml title="pyproject.toml"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
```

اگر index از قبل پیکربندی شده باشد، می‌توانید آن را با نام انتخاب کنید (این قابلیت در حالت preview
است):

```console
$ uv add --preview-features index-by-name torch --index pytorch
```

!!! tip

    مثال بالا فقط روی Linux با معماری x86-64 کار می‌کند، به‌دلیل ویژگی‌های خاص index مربوط به
    PyTorch. برای اطلاعات بیشتر دربارهٔ راه‌اندازی PyTorch،
    [راهنمای PyTorch](../../guides/integration/pytorch.md) را ببینید.

استفاده از یک source از نوع `index`، یک package را به index داده‌شده _پین_ می‌کند — این package
از indexهای دیگر دانلود نخواهد شد.

هنگام تعریف یک index، می‌توان یک فلگ `explicit` گنجاند تا نشان دهد که index باید _فقط_ برای
packageهایی استفاده شود که آن را به‌صراحت در `tool.uv.sources` مشخص کرده‌اند. اگر `explicit`
تنظیم نشده باشد، packageهای دیگر هم ممکن است در صورت پیدا نشدن در جای دیگر، از آن index resolve
شوند.

```toml title="pyproject.toml" hl_lines="4"
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

### Git {#git}

برای افزودن یک source از نوع dependency مربوط به Git، یک URL سازگار با Git را با پیشوند `git+`
شروع کنید.

برای مثال:

```console
$ # Install over HTTP(S).
$ uv add git+https://github.com/encode/httpx

$ # Install over SSH.
$ uv add git+ssh://git@github.com/encode/httpx
```

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

می‌توان referenceهای مشخص Git را درخواست کرد، مثلاً یک tag:

```console
$ uv add git+https://github.com/encode/httpx --tag 0.27.0
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.0" }
```

یا یک branch:

```console
$ uv add git+https://github.com/encode/httpx --branch main
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", branch = "main" }
```

یا یک revision (commit):

```console
$ uv add git+https://github.com/encode/httpx --rev 326b9431c761e1ef1e00b9f760d1f654c8db48c6
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", rev = "326b9431c761e1ef1e00b9f760d1f654c8db48c6" }
```

اگر package در ریشهٔ repository نباشد، می‌توان یک `subdirectory` مشخص کرد:

```console
$ uv add git+https://github.com/langchain-ai/langchain#subdirectory=libs/langchain
```

```toml title="pyproject.toml"
[project]
dependencies = ["langchain"]

[tool.uv.sources]
langchain = { git = "https://github.com/langchain-ai/langchain", subdirectory = "libs/langchain" }
```

پشتیبانی از [Git LFS](https://git-lfs.com) را هم می‌توان برای هر source پیکربندی کرد. به‌صورت
پیش‌فرض، objectهای Git LFS دریافت نخواهند شد.

```console
$ uv add --lfs git+https://github.com/astral-sh/lfs-cowsay
```

```toml title="pyproject.toml"
[project]
dependencies = ["lfs-cowsay"]

[tool.uv.sources]
lfs-cowsay = { git = "https://github.com/astral-sh/lfs-cowsay", lfs = true }
```

- وقتی `lfs = true` باشد، uv همیشه objectهای LFS را برای این source از نوع Git دریافت می‌کند.
- وقتی `lfs = false` باشد، uv هرگز objectهای LFS را برای این source از نوع Git دریافت نمی‌کند.
- وقتی این گزینه حذف شود، environment variable مربوط به `UV_GIT_LFS` برای همهٔ sourceهای Git بدون
  پیکربندی صریح `lfs` استفاده می‌شود.

!!! important

    پیش از تلاش برای نصب sourceهایی که از Git LFS استفاده می‌کنند، مطمئن شوید Git LFS روی سیستم
    شما نصب و پیکربندی شده است، در غیر این صورت ممکن است خطای build رخ دهد.

### URL {#url}

برای افزودن یک source از نوع URL، یک URL از نوع `https://` به یک wheel (که به `.whl` ختم
می‌شود) یا یک source distribution (که معمولاً به `.tar.gz` یا `.zip` ختم می‌شود؛ برای همهٔ
قالب‌های پشتیبانی‌شده [اینجا](../../concepts/resolution.md#source-distribution) را ببینید) ارائه
دهید.

برای مثال:

```console
$ uv add "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz"
```

به یک `pyproject.toml` با محتوای زیر منجر می‌شود:

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { url = "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz" }
```

dependencyهای URL را می‌توان به‌صورت دستی هم در `pyproject.toml` با سینتکس `{ url = <url> }`
اضافه یا ویرایش کرد. اگر source distribution در ریشهٔ archive نباشد، می‌توان یک `subdirectory`
مشخص کرد.

### Path {#path}

برای افزودن یک source از نوع path، مسیر یک wheel (که به `.whl` ختم می‌شود)، یک source distribution
(که معمولاً به `.tar.gz` یا `.zip` ختم می‌شود؛ برای همهٔ قالب‌های پشتیبانی‌شده
[اینجا](../../concepts/resolution.md#source-distribution) را ببینید)، یا یک پوشهٔ حاوی
`pyproject.toml` را ارائه دهید.

برای مثال:

```console
$ uv add /example/foo-0.1.0-py3-none-any.whl
```

به یک `pyproject.toml` با محتوای زیر منجر می‌شود:

```toml title="pyproject.toml"
[project]
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "/example/foo-0.1.0-py3-none-any.whl" }
```

مسیر همچنین می‌تواند یک مسیر نسبی باشد:

```console
$ uv add ./foo-0.1.0-py3-none-any.whl
```

یا مسیری به یک پوشهٔ project:

```console
$ uv add ~/projects/bar/
```

!!! important

    هنگام استفاده از یک پوشه به‌عنوان یک dependency از نوع path، uv به‌صورت پیش‌فرض تلاش می‌کند
    هدف را به‌عنوان یک package بسازد و نصب کند. برای جزئیات، مستندات
    [virtual dependency](#virtual-dependencies) را ببینید.

برای dependencyهای path به‌صورت پیش‌فرض از یک [editable installation](#editable-dependencies)
استفاده نمی‌شود. برای پوشه‌های project می‌توان یک editable installation درخواست کرد:

```console
$ uv add --editable ../projects/bar/
```

که به یک `pyproject.toml` با محتوای زیر منجر می‌شود:

```toml title="pyproject.toml"
[project]
dependencies = ["bar"]

[tool.uv.sources]
bar = { path = "../projects/bar", editable = true }
```

!!! tip

    برای چند package در یک repository، [_workspace_ها](./workspaces.md) ممکن است گزینهٔ بهتری
    باشند.

### عضو workspace {#workspace-member}

برای اعلام یک dependency روی یک عضو workspace، نام عضو را با `{ workspace = true }` اضافه کنید.
همهٔ اعضای workspace باید به‌صراحت اعلام شوند. اعضای workspace همیشه
[editable](#editable-dependencies) هستند. برای جزئیات بیشتر دربارهٔ workspaceها، مستندات
[workspace](./workspaces.md) را ببینید.

برای گرفتن source یک dependency از یک workspace متفاوت، `workspace` می‌تواند یک رشتهٔ مسیر هم باشد:

```toml title="pyproject.toml"
[tool.uv.sources]
foo = { workspace = "../other-workspace" }
```

```toml title="pyproject.toml"
[project]
dependencies = ["foo==0.1.0"]

[tool.uv.sources]
foo = { workspace = true }

[tool.uv.workspace]
members = [
  "packages/foo"
]
```

### sourceهای مخصوص platform {#platform-specific-sources}

می‌توانید یک source را به یک platform مشخص یا نسخهٔ مشخصی از Python محدود کنید، با ارائهٔ
environment markerهای سازگار با
[dependency specifierها](https://packaging.python.org/en/latest/specifications/dependency-specifiers/)
برای آن source.

برای مثال، برای گرفتن `httpx` از GitHub، اما فقط روی macOS، از مورد زیر استفاده کنید:

```toml title="pyproject.toml" hl_lines="8"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" }
```

با مشخص کردن marker روی source، uv همچنان `httpx` را روی همهٔ platformها شامل می‌شود، اما source
را روی macOS از GitHub دانلود می‌کند و روی همهٔ platformهای دیگر به PyPI برمی‌گردد.

### چند source {#multiple-sources}

می‌توانید چند source برای یک dependency واحد مشخص کنید، با ارائهٔ فهرستی از sourceها که با
environment markerهای سازگار با [PEP 508](https://peps.python.org/pep-0508/#environment-markers)
از هم متمایز می‌شوند.

برای مثال، برای گرفتن tagهای متفاوت `httpx` روی macOS در برابر Linux:

```toml title="pyproject.toml" hl_lines="6-7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = [
  { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" },
  { git = "https://github.com/encode/httpx", tag = "0.24.1", marker = "sys_platform == 'linux'" },
]
```

این راهبرد به استفاده از indexهای متفاوت بر اساس environment markerها هم گسترش می‌یابد. برای مثال،
برای نصب `torch` از indexهای متفاوت PyTorch بر اساس platform:

```toml title="pyproject.toml" hl_lines="6-7"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", marker = "platform_system == 'Darwin'"},
  { index = "torch-gpu", marker = "platform_system == 'Linux'"},
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu130"
explicit = true
```

### غیرفعال کردن sourceها {#disabling-sources}

برای اینکه به uv دستور دهید جدول `tool.uv.sources` را نادیده بگیرد (مثلاً برای شبیه‌سازی resolve
کردن با metadata منتشرشدهٔ package)، از فلگ `--no-sources` استفاده کنید:

```console
$ uv lock --no-sources
```

استفاده از `--no-sources` همچنین مانع می‌شود uv هر [عضو workspace](#workspace-member)ای را که
می‌تواند یک dependency مشخص را برآورده کند، کشف کند.

## dependencyهای اختیاری {#optional-dependencies}

برای projectهایی که به‌عنوان library منتشر می‌شوند، رایج است که برخی قابلیت‌ها را اختیاری کنند تا
درخت dependency پیش‌فرض کوچک‌تر شود. برای مثال، Pandas یک
[`excel` extra](https://pandas.pydata.org/docs/getting_started/install.html#excel-files) و یک
[`plot` extra](https://pandas.pydata.org/docs/getting_started/install.html#visualization) دارد تا از
نصب تجزیه‌کننده‌های Excel و `matplotlib` جلوگیری شود، مگر اینکه کسی به‌صراحت آن‌ها را درخواست کند.
extraها با سینتکس `package[<extra>]` درخواست می‌شوند، مثلاً `pandas[plot, excel]`.

dependencyهای اختیاری در `[project.optional-dependencies]` مشخص می‌شوند، یک جدول TOML که نام extra
را به dependencyهای آن نگاشت می‌کند و از سینتکس [dependency specifierها](#dependency-specifiers)
پیروی می‌کند.

dependencyهای اختیاری می‌توانند ورودی‌هایی در `tool.uv.sources` داشته باشند، درست مانند dependencyهای
معمولی.

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "1.0.0"

[project.optional-dependencies]
plot = [
  "matplotlib>=3.6.3"
]
excel = [
  "odfpy>=1.4.1",
  "openpyxl>=3.1.0",
  "python-calamine>=0.1.7",
  "pyxlsb>=1.0.10",
  "xlrd>=2.0.1",
  "xlsxwriter>=3.0.5"
]
```

برای افزودن یک dependency از نوع اختیاری، از گزینهٔ `--optional <extra>` استفاده کنید:

```console
$ uv add httpx --optional network
```

!!! note

    اگر dependencyهای اختیاری دارید که با یکدیگر در تضادند، resolution ناموفق خواهد بود، مگر
    اینکه آن‌ها را به‌صراحت [به‌عنوان متضاد اعلام کنید](./config.md#conflicting-dependencies).

sourceها را می‌توان به‌گونه‌ای هم اعلام کرد که فقط برای یک dependency از نوع اختیاری مشخص اعمال
شوند. برای مثال، برای گرفتن `torch` از indexهای متفاوت PyTorch بر اساس extra از نوع اختیاری به
نام `cpu` یا `gpu`:

```toml title="pyproject.toml"
[project]
dependencies = []

[project.optional-dependencies]
cpu = [
  "torch",
]
gpu = [
  "torch",
]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", extra = "cpu" },
  { index = "torch-gpu", extra = "gpu" },
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu130"
```

## dependencyهای توسعه {#development-dependencies}

برخلاف dependencyهای اختیاری، dependencyهای توسعه فقط محلی‌اند و هنگام انتشار در PyPI یا indexهای
دیگر در requirementهای project گنجانده _نخواهند_ شد. به همین دلیل، dependencyهای توسعه در جدول
`[project]` گنجانده نمی‌شوند.

dependencyهای توسعه می‌توانند ورودی‌هایی در `tool.uv.sources` داشته باشند، درست مانند dependencyهای
معمولی.

برای افزودن یک dependency توسعه، از فلگ `--dev` استفاده کنید:

```console
$ uv add --dev pytest
```

uv برای اعلام dependencyهای توسعه از جدول `[dependency-groups]` استفاده می‌کند (همان‌طور که در
[PEP 735](https://peps.python.org/pep-0735/) تعریف شده است). دستور بالا یک group به نام `dev`
ایجاد می‌کند:

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest >=8.1.1,<9"
]
```

group به نام `dev` به‌صورت ویژه در نظر گرفته می‌شود؛ فلگ‌های `--dev`، `--only-dev` و `--no-dev`
برای تغییر وضعیت گنجاندن یا حذف dependencyهای آن وجود دارند. برای غیرفعال کردن همهٔ groupهای
پیش‌فرض به‌جای آن، `--no-default-groups` را ببینید. علاوه بر این، group به نام `dev`
[به‌صورت پیش‌فرض sync می‌شود](#default-groups).

### groupهای dependency {#dependency-groups}

dependencyهای توسعه را می‌توان با استفاده از فلگ `--group` به چند group تقسیم کرد.

برای مثال، برای افزودن یک dependency توسعه در group به نام `lint`:

```console
$ uv add --group lint ruff
```

که به تعریف زیر از `[dependency-groups]` منجر می‌شود:

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest"
]
lint = [
  "ruff"
]
```

پس از تعریف groupها، می‌توان از گزینه‌های `--all-groups`، `--no-default-groups`، `--group`،
`--only-group` و `--no-group` برای گنجاندن یا حذف dependencyهای آن‌ها استفاده کرد.

!!! tip

    فلگ‌های `--dev`، `--only-dev` و `--no-dev` به‌ترتیب معادل `--group dev`، `--only-group dev`
    و `--no-group dev` هستند.

uv لازم می‌داند که همهٔ groupهای dependency با یکدیگر سازگار باشند و هنگام ایجاد lockfile، همهٔ
groupها را با هم resolve می‌کند.

اگر dependencyهای اعلام‌شده در یک group با dependencyهای group دیگر سازگار نباشند، uv در resolve
کردن requirementهای project با خطا ناموفق خواهد بود.

!!! note

    اگر groupهای dependency دارید که با یکدیگر در تضادند، resolution ناموفق خواهد بود، مگر اینکه
    آن‌ها را به‌صراحت [به‌عنوان متضاد اعلام کنید](./config.md#conflicting-dependencies).

### گروه‌های تو در تو {#nesting-groups}

یک group از نوع dependency می‌تواند groupهای dependency دیگر را هم شامل شود، مثلاً:

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  {include-group = "lint"},
  {include-group = "test"}
]
lint = [
  "ruff"
]
test = [
  "pytest"
]
```

dependencyهای یک group گنجانده‌شده نمی‌توانند با dependencyهای دیگری که در یک group اعلام شده‌اند،
در تضاد باشند.

### groupهای پیش‌فرض {#default-groups}

به‌صورت پیش‌فرض، uv group از نوع dependency به نام `dev` را در environment شامل می‌کند (مثلاً در
طول `uv run` یا `uv sync`). groupهای پیش‌فرضی که باید گنجانده شوند را می‌توان با تنظیم
`tool.uv.default-groups` تغییر داد.

```toml title="pyproject.toml"
[tool.uv]
default-groups = ["dev", "foo"]
```

برای فعال کردن همهٔ groupهای dependency به‌صورت پیش‌فرض، به‌جای فهرست کردن نام groupها از `"all"`
استفاده کنید:

```toml title="pyproject.toml"
[tool.uv]
default-groups = "all"
```

!!! tip

    برای غیرفعال کردن این رفتار در طول `uv run` یا `uv sync`، از `--no-default-groups` استفاده
    کنید. برای حذف یک group پیش‌فرض مشخص، از `--no-group <name>` استفاده کنید.

### group از نوع `requires-python` {#group-requires-python}

به‌صورت پیش‌فرض، groupهای dependency باید با بازهٔ `requires-python` مربوط به project شما سازگار
باشند.

اگر یک group از نوع dependency به بازهٔ متفاوتی از نسخه‌های Python نسبت به project شما نیاز دارد،
می‌توانید یک `requires-python` برای آن group در `[tool.uv.dependency-groups]` مشخص کنید، مثلاً:

```toml title="pyproject.toml" hl_lines="9-10"
[project]
name = "example"
version = "0.0.0"
requires-python = ">=3.10"

[dependency-groups]
dev = ["pytest"]

[tool.uv.dependency-groups]
dev = {requires-python = ">=3.12"}
```

### `dev-dependencies` قدیمی {#legacy-dev-dependencies}

پیش از اینکه `[dependency-groups]` استاندارد شود، uv از فیلد `tool.uv.dev-dependencies` برای مشخص
کردن dependencyهای توسعه استفاده می‌کرد، مثلاً:

```toml title="pyproject.toml"
[tool.uv]
dev-dependencies = [
  "pytest"
]
```

dependencyهای اعلام‌شده در این بخش با محتوای موجود در `dependency-groups.dev` ترکیب خواهند شد. در
نهایت، فیلد `dev-dependencies` منسوخ و حذف خواهد شد.

!!! note

    اگر یک فیلد `tool.uv.dev-dependencies` وجود داشته باشد، `uv add --dev` به‌جای افزودن یک بخش
    جدید `dependency-groups.dev`، از بخش موجود استفاده خواهد کرد.

## dependencyهای build {#build-dependencies}

اگر یک project به‌صورت یک [package از نوع Python](./config.md#build-systems) ساختاربندی شده باشد،
ممکن است dependencyهایی را اعلام کند که برای ساخت project لازم‌اند، اما برای اجرای آن لازم نیستند.
این dependencyها در جدول `[build-system]` زیر `build-system.requires` مشخص می‌شوند و از
[PEP 518](https://peps.python.org/pep-0518/) پیروی می‌کنند.

برای مثال، اگر یک project از `setuptools` به‌عنوان build backend خود استفاده کند، باید `setuptools`
را به‌عنوان یک dependency از نوع build اعلام کند:

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"
```

به‌صورت پیش‌فرض، uv هنگام resolve کردن dependencyهای build به `tool.uv.sources` احترام می‌گذارد.
برای مثال، برای استفاده از یک نسخهٔ محلی از `setuptools` برای ساخت، source را به `tool.uv.sources`
اضافه کنید:

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"

[tool.uv.sources]
setuptools = { path = "./packages/setuptools" }
```

هنگام انتشار یک package، توصیه می‌کنیم `uv build --no-sources` را اجرا کنید تا مطمئن شوید package
وقتی `tool.uv.sources` غیرفعال است، به‌درستی ساخته می‌شود، همان‌طور که هنگام استفاده از toolهای
build دیگر، مانند [`pypa/build`](https://github.com/pypa/build)، چنین است.

## dependencyهای editable {#editable-dependencies}

یک نصب معمولی از یک پوشهٔ حاوی یک package از نوع Python، ابتدا یک wheel می‌سازد و سپس آن wheel را
در virtual environment شما نصب می‌کند و همهٔ فایل‌های source را کپی می‌کند. وقتی فایل‌های source
مربوط به package ویرایش شوند، virtual environment شامل نسخه‌های قدیمی خواهد بود.

نصب‌های editable این مشکل را با افزودن یک پیوند به project در داخل virtual environment (یک فایل
`.pth`) حل می‌کنند، که به interpreter دستور می‌دهد فایل‌های source را مستقیم شامل شود.

برای editableها محدودیت‌هایی وجود دارد (عمدتاً: build backend باید از آن‌ها پشتیبانی کند و
moduleهای native پیش از import دوباره کامپایل نمی‌شوند)، اما آن‌ها برای توسعه مفیدند، چون virtual
environment همیشه از جدیدترین تغییرات package استفاده خواهد کرد.

uv به‌صورت پیش‌فرض از editable installation برای packageهای workspace استفاده می‌کند.

برای افزودن یک dependency از نوع editable، از فلگ `--editable` استفاده کنید:

```console
$ uv add --editable ./path/foo
```

یا، برای انصراف از استفاده از یک dependency از نوع editable در یک workspace:

```console
$ uv add --no-editable ./path/foo
```

## dependencyهای virtual {#virtual-dependencies}

uv اجازه می‌دهد dependencyها «virtual» باشند، به این معنا که خود dependency به‌عنوان یک
[package](./config.md#project-packaging) نصب نمی‌شود، اما dependencyهای آن نصب می‌شوند.

به‌صورت پیش‌فرض، dependencyها هرگز virtual نیستند.

یک dependency با یک [`path` source](#path) می‌تواند virtual باشد، اگر به‌صراحت
[`tool.uv.package = false`](../../reference/settings.md#package) را تنظیم کند. بدون این تنظیم، uv
dependency از نوع path را به‌عنوان یک package معمولی در نظر می‌گیرد و تلاش می‌کند آن را بسازد، حتی
اگر project یک [build system](./config.md#build-systems) اعلام نکرده باشد.

برای اینکه یک dependency به‌عنوان virtual در نظر گرفته شود، `package = false` را روی source تنظیم
کنید:

```toml title="pyproject.toml"
[project]
dependencies = ["bar"]

[tool.uv.sources]
bar = { path = "../projects/bar", package = false }
```

اگر یک dependency مقدار `tool.uv.package = false` را تنظیم کند، می‌توان با اعلام `package = true`
روی source، آن را لغو کرد:

```toml title="pyproject.toml"
[project]
dependencies = ["bar"]

[tool.uv.sources]
bar = { path = "../projects/bar", package = true }
```

به همین ترتیب، یک dependency با یک [`workspace` source](#workspace-member) می‌تواند virtual باشد،
اگر به‌صراحت [`tool.uv.package = false`](../../reference/settings.md#package) را تنظیم کند. بدون
این تنظیم، عضو workspace ساخته خواهد شد، حتی اگر یک
[build system](./config.md#build-systems) اعلام نشده باشد.

اعضای workspace که dependency _نیستند_ می‌توانند به‌صورت پیش‌فرض virtual باشند، مثلاً اگر فایل
`pyproject.toml` مربوط به والد به این صورت باشد:

```toml title="pyproject.toml"
[project]
name = "parent"
version = "1.0.0"
dependencies = []

[tool.uv.workspace]
members = ["child"]
```

و فایل `pyproject.toml` مربوط به فرزند شامل یک build system نباشد:

```toml title="pyproject.toml"
[project]
name = "child"
version = "1.0.0"
dependencies = ["anyio"]
```

در این صورت، عضو workspace به نام `child` نصب نمی‌شود، اما dependency انتقالی `anyio` نصب خواهد
شد.

در مقابل، اگر والد یک dependency روی `child` اعلام کرده باشد:

```toml title="pyproject.toml"
[project]
name = "parent"
version = "1.0.0"
dependencies = ["child"]

[tool.uv.sources]
child = { workspace = true }

[tool.uv.workspace]
members = ["child"]
```

در این صورت، `child` ساخته و نصب می‌شود.

## dependency specifierها {#dependency-specifiers}

uv از [dependency specifierهای](https://packaging.python.org/en/latest/specifications/dependency-specifiers/)
استاندارد استفاده می‌کند که در اصل در [PEP 508](https://peps.python.org/pep-0508/) تعریف شده‌اند.
یک dependency specifier به‌ترتیب از موارد زیر تشکیل شده است:

- نام dependency
- extraهایی که می‌خواهید (اختیاری)
- specifier نسخه
- یک environment marker (اختیاری)

specifierهای نسخه با کاما از هم جدا می‌شوند و با هم جمع می‌شوند، مثلاً `foo >=1.2.3,<2,!=1.4.0` به
این صورت تفسیر می‌شود: «نسخه‌ای از `foo` که دست‌کم 1.2.3 باشد، اما کمتر از 2 باشد و 1.4.0 نباشد».

در صورت لزوم، specifierها با صفرهای انتهایی پر می‌شوند، پس `foo ==2` با foo نسخهٔ 2.0.0 هم مطابقت
دارد.

می‌توان برای آخرین رقم با علامت مساوی از ستاره استفاده کرد، مثلاً `foo ==2.1.*` هر انتشار از سری
2.1 را می‌پذیرد. به همین ترتیب، `~=` جایی مطابقت دارد که آخرین رقم برابر یا بیشتر باشد، مثلاً
`foo ~=1.2` برابر است با `foo >=1.2,<2` و `foo ~=1.2.3` برابر است با `foo >=1.2.3,<1.3`.

extraها در براکت‌های مربع بین نام و نسخه با کاما از هم جدا می‌شوند، مثلاً `pandas[excel,plot] ==2.2`. فضای خالی بین نام extraها نادیده گرفته می‌شود.

برخی dependencyها فقط در environmentهای مشخصی لازم‌اند، مثلاً یک نسخهٔ مشخص از Python یا یک
سیستم‌عامل مشخص. برای مثال، برای نصب backport مربوط به `importlib-metadata` برای module به نام
`importlib.metadata`، از `importlib-metadata >=7.1.0,<8; python_version < '3.10'` استفاده کنید.
برای نصب `colorama` روی Windows (اما حذف آن روی platformهای دیگر)، از
`colorama >=0.4.6,<5; platform_system == "Windows"` استفاده کنید.

markerها با `and`، `or` و پرانتز ترکیب می‌شوند، مثلاً
`aiohttp >=3.7.4,<4; (sys_platform != 'win32' or implementation_name != 'pypy') and python_version >= '3.10'`.
توجه کنید که نسخه‌های داخل markerها باید داخل گیومه باشند، در حالی که نسخه‌های _خارج_ از markerها
_نباید_ داخل گیومه باشند.
