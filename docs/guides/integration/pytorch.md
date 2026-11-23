---
title: استفاده از uv با PyTorch
description:
  راهنمای استفاده از uv با PyTorch، شامل نصب PyTorch، پیکربندی buildهای هر platform و هر شتاب‌دهنده
  و نصب extensionهای PyTorch با قابلیت GPU.
---

# استفاده از uv با PyTorch {#using-uv-with-pytorch}

اکوسیستم [PyTorch](https://pytorch.org/) یک انتخاب محبوب برای تحقیق و توسعه در زمینهٔ یادگیری عمیق
است. می‌توانید از uv برای مدیریت projectهای PyTorch و dependencyهای PyTorch در نسخه‌های مختلف Python
و environmentها استفاده کنید، حتی با کنترل انتخاب شتاب‌دهنده (مثلاً CPU-only در برابر CUDA).

!!! note

    برخی از قابلیت‌های شرح‌داده‌شده در این راهنما به uv نسخهٔ 0.5.3 یا جدیدتر نیاز دارند. توصیه
    می‌کنیم پیش از پیکربندی PyTorch، ارتقا دهید.

## نصب PyTorch {#installing-pytorch}

از دیدگاه بسته‌بندی، PyTorch چند ویژگی غیرمعمول دارد:

- بسیاری از wheelهای PyTorch روی یک index اختصاصی میزبانی می‌شوند، نه Python Package Index (PyPI).
  به همین دلیل، نصب PyTorch اغلب نیازمند پیکربندی project برای استفاده از index مربوط به PyTorch
  است.
- PyTorch برای هر شتاب‌دهنده، buildهای مجزا تولید می‌کند (مثلاً CPU-only ،CUDA). از آنجا که هیچ
  سازوکار استانداردی برای مشخص‌کردن این شتاب‌دهنده‌ها هنگام publish یا نصب وجود ندارد، PyTorch
  آن‌ها را در specifier مربوط به نسخهٔ محلی رمزگذاری می‌کند. به همین دلیل، نسخه‌های PyTorch اغلب
  شبیه `2.11.0+cpu` ،`2.11.0+cu130` و غیره به نظر می‌رسند.
- buildهای مربوط به شتاب‌دهنده‌های مختلف در indexهای مختلف منتشر می‌شوند. برای مثال، buildهای `+cpu`
  در آدرس https://download.pytorch.org/whl/cpu منتشر می‌شوند، در حالی که buildهای `+cu130` در آدرس
  https://download.pytorch.org/whl/cu130 منتشر می‌شوند.

به همین دلیل، پیکربندی لازم برای بسته‌بندی بسته به platformهایی که باید پشتیبانی کنید و
شتاب‌دهنده‌هایی که می‌خواهید فعال کنید، متفاوت خواهد بود.

برای شروع، پیکربندی (پیش‌فرض) زیر را در نظر بگیرید که با اجرای `uv init --python 3.14` و سپس
`uv add torch torchvision` تولید می‌شود.

در این حالت، PyTorch از PyPI نصب می‌شود که wheelهای CPU-only را برای Windows و macOS و wheelهای با
شتاب GPU را روی Linux میزبانی می‌کند (که از PyTorch نسخهٔ 2.11.0، CUDA نسخهٔ 13.0 را هدف می‌گیرد):

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14"
dependencies = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
]
```

این یک پیکربندی معتبر برای projectهایی است که می‌خواهند از buildهای CPU روی Windows و macOS و از
buildهای با قابلیت CUDA روی Linux استفاده کنند. با این حال، اگر نیاز دارید platformها یا
شتاب‌دهنده‌های متفاوتی را پشتیبانی کنید، باید project را مطابق آن پیکربندی کنید.

## استفاده از یک index مربوط به PyTorch {#using-a-pytorch-index}

در برخی موارد، ممکن است بخواهید از یک گونهٔ مشخص PyTorch در همهٔ platformها استفاده کنید. برای
مثال، ممکن است بخواهید از buildهای CPU-only در Linux هم استفاده کنید.

در چنین مواردی، اولین قدم این است که index مربوط به PyTorch را به `pyproject.toml` خود اضافه کنید:

=== "CPU-only"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-cpu"
    url = "https://download.pytorch.org/whl/cpu"
    explicit = true
    ```

=== "CUDA 11.8"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-cu118"
    url = "https://download.pytorch.org/whl/cu118"
    explicit = true
    ```

=== "CUDA 12.6"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-cu126"
    url = "https://download.pytorch.org/whl/cu126"
    explicit = true
    ```

=== "CUDA 12.8"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-cu128"
    url = "https://download.pytorch.org/whl/cu128"
    explicit = true
    ```

=== "CUDA 13.0"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-cu130"
    url = "https://download.pytorch.org/whl/cu130"
    explicit = true
    ```

=== "ROCm 7.2"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-rocm"
    url = "https://download.pytorch.org/whl/rocm7.2"
    explicit = true
    ```

=== "Intel GPUs"

    ```toml
    [[tool.uv.index]]
    name = "pytorch-xpu"
    url = "https://download.pytorch.org/whl/xpu"
    explicit = true
    ```

استفاده از `explicit = true` را توصیه می‌کنیم تا مطمئن شویم index _فقط_ برای `torch` ،`torchvision`
و دیگر packageهای مرتبط با PyTorch استفاده می‌شود، نه dependencyهای عمومی مانند `jinja2` که باید
همچنان از index پیش‌فرض (PyPI) تأمین شوند.

<!-- TODO(tk): Show `uv add --index <name>` once index-by-name is stable. -->

سپس، `pyproject.toml` را به‌روز کنید تا `torch` و `torchvision` به index موردنظر اشاره کنند:

=== "CPU-only"

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-cpu" },
    ]
    torchvision = [
      { index = "pytorch-cpu" },
    ]
    ```

=== "CUDA 11.8"

    PyTorch برای macOS، buildهای CUDA منتشر نمی‌کند. به همین دلیل، روی `sys_platform` شرط
    می‌گذاریم تا به uv دستور دهیم از index مربوط به PyTorch روی Linux و Windows استفاده کند، اما در
    macOS به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-cu118", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    torchvision = [
      { index = "pytorch-cu118", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    ```

=== "CUDA 12.6"

    PyTorch برای macOS، buildهای CUDA منتشر نمی‌کند. به همین دلیل، روی `sys_platform` شرط
    می‌گذاریم تا به uv دستور دهیم index مربوط به PyTorch را به Linux و Windows محدود کند و در macOS
    به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-cu126", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    torchvision = [
      { index = "pytorch-cu126", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    ```

=== "CUDA 12.8"

    PyTorch برای macOS، buildهای CUDA منتشر نمی‌کند. به همین دلیل، روی `sys_platform` شرط
    می‌گذاریم تا به uv دستور دهیم index مربوط به PyTorch را به Linux و Windows محدود کند و در macOS
    به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-cu128", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    torchvision = [
      { index = "pytorch-cu128", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    ```

=== "CUDA 13.0"

    PyTorch برای macOS، buildهای CUDA منتشر نمی‌کند. به همین دلیل، روی `sys_platform` شرط
    می‌گذاریم تا به uv دستور دهیم index مربوط به PyTorch را به Linux و Windows محدود کند و در macOS
    به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-cu130", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    torchvision = [
      { index = "pytorch-cu130", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    ```

=== "ROCm 7.2"

    PyTorch برای macOS یا Windows، buildهای ROCm منتشر نمی‌کند. به همین دلیل، روی `sys_platform`
    شرط می‌گذاریم تا به uv دستور دهیم index مربوط به PyTorch را به Linux محدود کند و در macOS و
    Windows به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
    ]
    torchvision = [
      { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
    ]
    # ROCm support relies on both Triton packages, which should also be installed from
    # the PyTorch index (and included in `project.dependencies`).
    pytorch-triton-rocm = [
      { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
    ]
    triton-rocm = [
      { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
    ]
    ```

=== "Intel GPUs"

    PyTorch برای macOS، buildهای مربوط به Intel GPU منتشر نمی‌کند. به همین دلیل، روی `sys_platform`
    شرط می‌گذاریم تا به uv دستور دهیم index مربوط به PyTorch را به Linux و Windows محدود کند و در
    macOS به PyPI برگردد:

    ```toml
    [tool.uv.sources]
    torch = [
      { index = "pytorch-xpu", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    torchvision = [
      { index = "pytorch-xpu", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    # Intel GPU support relies on `triton-xpu`, which should also be installed from the PyTorch index
    # (and included in `project.dependencies`).
    triton-xpu = [
      { index = "pytorch-xpu", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
    ]
    ```

به‌عنوان یک مثال کامل، project زیر از buildهای CPU-only مربوط به PyTorch در همهٔ platformها استفاده
می‌کند:

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14.0"
dependencies = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
]

[tool.uv.sources]
torch = [
    { index = "pytorch-cpu" },
]
torchvision = [
    { index = "pytorch-cpu" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

## پیکربندی شتاب‌دهنده‌ها با environment markerها {#configuring-accelerators-with-environment-markers}

در برخی موارد، ممکن است بخواهید در یک environment (مثلاً macOS و Windows) از buildهای CPU-only و
در environment دیگر (مثلاً Linux) از buildهای با قابلیت CUDA استفاده کنید.

با `tool.uv.sources` می‌توانید از environment markerها برای مشخص‌کردن index موردنظر برای هر platform
استفاده کنید. برای مثال، پیکربندی زیر از buildهای PyTorch با قابلیت CUDA روی Linux و از buildهای
CPU-only در همهٔ platformهای دیگر (مثلاً macOS و Windows) استفاده می‌کند:

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14.0"
dependencies = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
]

[tool.uv.sources]
torch = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu130", marker = "sys_platform == 'linux'" },
]
torchvision = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu130", marker = "sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu130"
url = "https://download.pytorch.org/whl/cu130"
explicit = true
```

به همین ترتیب، پیکربندی زیر از buildهای AMD GPU مربوط به PyTorch روی Linux و از buildهای CPU-only
روی Windows و macOS (با برگشتن به PyPI) استفاده می‌کند:

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14.0"
dependencies = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
  "pytorch-triton-rocm>=3.5.1 ; sys_platform == 'linux'",
  "triton-rocm>=3.6.0 ; sys_platform == 'linux'",
]

[tool.uv.sources]
torch = [
  { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
]
torchvision = [
  { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
]
pytorch-triton-rocm = [
  { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
]
triton-rocm = [
  { index = "pytorch-rocm", marker = "sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-rocm"
url = "https://download.pytorch.org/whl/rocm7.2"
explicit = true
```

یا برای buildهای مربوط به Intel GPU:

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14.0"
dependencies = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
  "triton-xpu>=3.7.0 ; sys_platform == 'win32' or sys_platform == 'linux'",
]

[tool.uv.sources]
torch = [
  { index = "pytorch-xpu", marker = "sys_platform == 'win32' or sys_platform == 'linux'" },
]
torchvision = [
  { index = "pytorch-xpu", marker = "sys_platform == 'win32' or sys_platform == 'linux'" },
]
triton-xpu = [
  { index = "pytorch-xpu", marker = "sys_platform == 'win32' or sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-xpu"
url = "https://download.pytorch.org/whl/xpu"
explicit = true
```

## پیکربندی شتاب‌دهنده‌ها با dependencyهای optional {#configuring-accelerators-with-optional-dependencies}

در برخی موارد، ممکن است بخواهید در بعضی موارد از buildهای CPU-only و در موارد دیگر از buildهای با
قابلیت CUDA استفاده کنید، با این انتخاب که با یک extra ارائه‌شده توسط کاربر تغییر می‌کند (مثلاً
`uv sync --extra cpu` در برابر `uv sync --extra cu130`).

با `tool.uv.sources` می‌توانید از extra markerها برای مشخص‌کردن index موردنظر برای هر extra فعال‌شده
استفاده کنید. برای مثال، پیکربندی زیر از buildهای CPU-only مربوط به PyTorch برای
`uv sync --extra cpu` و از buildهای با قابلیت CUDA برای `uv sync --extra cu130` استفاده می‌کند:

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.14.0"
dependencies = []

[project.optional-dependencies]
cpu = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
]
cu130 = [
  "torch>=2.11.0",
  "torchvision>=0.26.0",
]

[tool.uv]
conflicts = [
  [
    { extra = "cpu" },
    { extra = "cu130" },
  ],
]

[tool.uv.sources]
torch = [
  { index = "pytorch-cpu", extra = "cpu" },
  { index = "pytorch-cu130", extra = "cu130" },
]
torchvision = [
  { index = "pytorch-cpu", extra = "cpu" },
  { index = "pytorch-cu130", extra = "cu130" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu130"
url = "https://download.pytorch.org/whl/cu130"
explicit = true
```

!!! note

    از آنجا که buildهای با شتاب GPU در macOS در دسترس نیستند، پیکربندی بالا هنگام فعال‌بودن extra
    مربوط به `cu130`، در macOS با خطا در نصب مواجه می‌شود.

## نصب extensionهای PyTorch با قابلیت GPU {#installing-gpu-enabled-pytorch-extensions}

بسیاری از packageهای موجود در اکوسیستم PyTorch شامل extensionهای با قابلیت GPU هستند که برای یک
ترکیب مشخص از نسخه‌های CUDA و PyTorch compile شده‌اند. build کردن این packageها از منبع اغلب به
دسترسی به CUDA development toolkit و پیکربندی اضافی build نیاز دارد.

[indexهای Astral GPU](https://wheels.astral.sh/)، wheelهای از پیش buildشده را برای packageهایی
مانند `flash-attn` ،`deepspeed` ،`deep-gemm` ،`torch-scatter` و `vllm`، در طیفی از نسخه‌های Python
،CUDA و PyTorch ارائه می‌دهند.

برای نصب `flash-attn` از index مربوط به CUDA نسخهٔ 12.8، این دستور را اجرا کنید:

```console
$ uv add flash-attn --index astral-cu128=https://wheels.astral.sh/simple/cu128/
```

این دستور `flash-attn` را به dependencyهای project اضافه می‌کند، index مربوط به Astral GPU را
پیکربندی می‌کند و `flash-attn` را به آن index پین می‌کند.

مانند indexهای PyTorch، `explicit = true` را تنظیم کنید تا index مربوط به Astral GPU را به
packageهایی محدود کنید که به‌صراحت به آن پین شده‌اند:

```toml title="pyproject.toml"
[tool.uv.sources]
flash-attn = { index = "astral-cu128" }

[[tool.uv.index]]
name = "astral-cu128"
url = "https://wheels.astral.sh/simple/cu128/"
explicit = true
```

هر index مربوط به Astral GPU، یک نسخهٔ مشخص از CUDA را هدف می‌گیرد و wheelهای آن برای نسخه‌های مشخص
PyTorch build شده‌اند. برای مثال، یک wheel با نسخهٔ محلی `+cu.12.8.torch.2.11` برای CUDA نسخهٔ 12.8
و PyTorch نسخهٔ 2.11 build شده است. index و wheelای را انتخاب کنید که با Python version ،platform
،نسخهٔ CUDA و نصب PyTorch شما مطابقت دارد.

برای packageهای در دسترس، نسخه‌های CUDA و نسخه‌های PyTorch، به
[indexهای Astral GPU](https://wheels.astral.sh/) مراجعه کنید.

## interface مربوط به `uv pip` {#the-uv-pip-interface}

با اینکه مثال‌های بالا بر interface مربوط به project در uv (`uv lock` ،`uv sync` ،`uv run` و غیره)
متمرکزند، PyTorch را می‌توان از طریق رابط مربوط به `uv pip` هم نصب کرد.

خود PyTorch یک [رابط اختصاصی](https://pytorch.org/get-started/locally/) برای تعیین دستور pip
مناسب برای یک پیکربندی هدف مشخص ارائه می‌دهد. برای مثال، می‌توانید PyTorch پایدار و CPU-only را روی
Linux با این دستور نصب کنید:

```shell
$ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

برای استفاده از همان workflow با uv، `pip3` را با `uv pip` جایگزین کنید:

```shell
$ uv pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

## انتخاب خودکار backend {#automatic-backend-selection}

uv از انتخاب خودکار index مناسب PyTorch از طریق آرگومان خط دستور `--torch-backend=auto` (یا
environment variable مربوط به `UV_TORCH_BACKEND=auto`) پشتیبانی می‌کند، به این شکل:

```shell
$ # With a command-line argument.
$ uv pip install torch --torch-backend=auto

$ # With an environment variable.
$ UV_TORCH_BACKEND=auto uv pip install torch
```

وقتی فعال شود، uv برای درایور نصب‌شدهٔ CUDA، نسخه‌های AMD GPU و وجود Intel GPU query می‌کند، سپس
از سازگارترین index مربوط به PyTorch برای همهٔ packageهای مرتبط (مثلاً `torch` ،`torchvision` و
غیره) استفاده می‌کند. اگر چنین GPUای پیدا نشد، uv به index مربوط به CPU-only برمی‌گردد. uv همچنان
به پیکربندی موجود index برای packageهای خارج از اکوسیستم PyTorch احترام می‌گذارد.

همچنین می‌توانید یک backend مشخص (مثلاً CUDA نسخهٔ 13.0) را با `--torch-backend=cu130` (یا
`UV_TORCH_BACKEND=cu130`) انتخاب کنید:

```shell
$ # With a command-line argument.
$ uv pip install torch torchvision --torch-backend=cu130

$ # With an environment variable.
$ UV_TORCH_BACKEND=cu130 uv pip install torch torchvision
```

در حال حاضر، `--torch-backend` فقط در رابط مربوط به `uv pip` در دسترس است.
