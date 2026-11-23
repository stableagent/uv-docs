# عیب‌یابی خطاهای build {#troubleshooting-build-failures}

وقتی wheel سازگاری (یک distribution از پیش ساخته‌شده از package) در دسترس نباشد، uv باید packageها
را build کند. build کردن packageها می‌تواند به دلایل زیادی ناموفق شود که بعضی از آن‌ها ممکن است
به خود uv مربوط نباشند.

## شناخت یک خطای build {#recognizing-a-build-failure}

می‌توان با تلاش برای نصب یک نسخهٔ قدیمی از numpy روی یک نسخهٔ جدید و پشتیبانی‌نشده از Python، یک
مثال از خطای build تولید کرد:

```console
$ uv pip install -p 3.13 'numpy<1.20'
Resolved 1 package in 62ms
  × Failed to build `numpy==1.19.5`
  ├─▶ The build backend returned an error
  ╰─▶ Call to `setuptools.build_meta:__legacy__.build_wheel()` failed (exit status: 1)

      [stderr]
      Traceback (most recent call last):
        File "<string>", line 8, in <module>
          from setuptools.build_meta import __legacy__ as backend
        File "/home/konsti/.cache/uv/builds-v0/.tmpi4bgKb/lib/python3.13/site-packages/setuptools/__init__.py", line 9, in <module>
          import distutils.core
      ModuleNotFoundError: No module named 'distutils'

      hint: `distutils` was removed from the standard library in Python 3.12. Consider adding a constraint (like `numpy >1.19.5`) to avoid building a version of `numpy` that depends
      on `distutils`.
```

توجه کنید که پیام خطا با عبارت "The build backend returned an error" شروع می‌شود.

خطای build شامل `[stderr]` (و `[stdout]`، در صورت وجود) از build backend‌ای است که برای build
استفاده شده بود. logهای خطا از خود uv نیستند.

پیام پس از `╰─▶` یک راهنمایی است که uv برای کمک به حل خطاهای رایج build ارائه می‌دهد. برای همهٔ
خطاهای build راهنمایی در دسترس نخواهد بود.

## تأیید اینکه خطای build مخصوص uv است {#confirming-that-a-build-failure-is-specific-to-uv}

خطاهای build معمولاً به سیستم شما و build backend مربوط‌اند. به‌ندرت پیش می‌آید که یک خطای build
مخصوص uv باشد. می‌توانید با تلاش برای بازتولید آن با pip تأیید کنید که خطای build به uv مربوط
نیست:

```console
$ uv venv -p 3.13 --seed
$ source .venv/bin/activate
$ pip install --use-pep517 --no-cache --force-reinstall 'numpy==1.19.5'
Collecting numpy==1.19.5
  Using cached numpy-1.19.5.zip (7.3 MB)
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
ERROR: Exception:
Traceback (most recent call last):
  ...
  File "/Users/example/.cache/uv/archive-v0/3783IbOdglemN3ieOULx2/lib/python3.13/site-packages/pip/_vendor/pyproject_hooks/_impl.py", line 321, in _call_hook
    raise BackendUnavailable(data.get('traceback', ''))
pip._vendor.pyproject_hooks._impl.BackendUnavailable: Traceback (most recent call last):
  File "/Users/example/.cache/uv/archive-v0/3783IbOdglemN3ieOULx2/lib/python3.13/site-packages/pip/_vendor/pyproject_hooks/_in_process/_in_process.py", line 77, in _build_backend
    obj = import_module(mod_path)
  File "/Users/example/.local/share/uv/python/cpython-3.13.0-macos-aarch64-none/lib/python3.13/importlib/__init__.py", line 88, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "<frozen importlib._bootstrap>", line 1387, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1310, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 488, in _call_with_frames_removed
  File "<frozen importlib._bootstrap>", line 1387, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1331, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 935, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 1022, in exec_module
  File "<frozen importlib._bootstrap>", line 488, in _call_with_frames_removed
  File "/private/var/folders/6p/k5sd5z7j31b31pq4lhn0l8d80000gn/T/pip-build-env-vdpjme7d/overlay/lib/python3.13/site-packages/setuptools/__init__.py", line 9, in <module>
    import distutils.core
ModuleNotFoundError: No module named 'distutils'
```

!!! important

    فلگ `--use-pep517` باید همراه با فراخوانی `pip install` گنجانده شود تا همان رفتار build
    isolation تضمین شود. uv همیشه به‌طور پیش‌فرض از
    [build isolation](../../pip/compatibility.md#pep-517-build-isolation) استفاده می‌کند.

    همچنین توصیه می‌کنیم هنگام بازتولید خطاها، گزینه‌های `--force-reinstall` و `--no-cache` را هم
    شامل کنید.

از آنجا که این خطای build در pip هم رخ می‌دهد، احتمالاً یک باگ در uv نیست.

اگر یک خطای build با یک نصب‌کنندهٔ دیگر هم قابل بازتولید باشد، باید در upstream بررسی کنید (در این
مثال، `numpy` یا `setuptools`)، راهی برای اجتناب از build کردن package در وهلهٔ اول پیدا کنید، یا
تنظیمات لازم را در سیستم خود انجام دهید تا build موفق شود.

## چرا uv یک package را build می‌کند؟ {#why-does-uv-build-a-package}

هنگام تولید lockfile چندplatform‌ای، uv باید dependencyهای همهٔ packageها را تعیین کند، حتی
آن‌هایی که فقط روی platformهای دیگر نصب می‌شوند. uv تلاش می‌کند در حین resolution از build کردن
packageها اجتناب کند. اگر برای آن نسخه wheel‌ای وجود داشته باشد از آن استفاده می‌کند، سپس تلاش
می‌کند metadata ایستا را در source distribution پیدا کند (عمدتاً pyproject.toml با
`project.version` و `project.dependencies` و `project.optional-dependencies` ایستا یا METADATA
نسخهٔ 2.2+). اگر هر دو رویکرد ناموفق باشند، uv package را build می‌کند.

هنگام نصب، uv برای هر package به یک wheel برای platform فعلی نیاز دارد. اگر wheel مطابقتی در index
وجود نداشته باشد، uv تلاش می‌کند source distribution را build کند.

می‌توانید بررسی کنید کدام wheelها برای یک project در PyPI در بخش «Download Files» وجود دارند؛
مثلاً [NumPy 2.1.1](https://pypi.org/project/numpy/2.1.1/#files). wheelهایی با نام فایل
`...-py3-none-any.whl` همه‌جا کار می‌کنند و بقیه سیستم‌عامل و platform را در نام فایل دارند. در
مثال لینک‌شدهٔ `numpy` می‌توانید ببینید که distributionهای از پیش ساخته‌شده برای Python 3.10 تا
3.13 روی macOS و Linux و Windows وجود دارد.

## خطاهای رایج build {#common-build-failures}

مثال‌های زیر، خطاهای رایج build و نحوهٔ حل آن‌ها را نشان می‌دهند.

### دستور پیدا نشد {#command-is-not-found}

اگر خطای build به یک دستور گمشده اشاره کند، مثلاً `gcc`:

<!-- docker run --platform linux/x86_64 -it ghcr.io/astral-sh/uv:python3.10-trixie-slim /bin/bash -c "uv pip install --system pysha3==1.0.2" -->

```hl_lines="17"
× Failed to build `pysha3==1.0.2`
├─▶ The build backend returned an error
╰─▶ Call to `setuptools.build_meta:__legacy__.build_wheel` failed (exit status: 1)

    [stdout]
    running bdist_wheel
    running build
    running build_py
    creating build/lib.linux-x86_64-cpython-310
    copying sha3.py -> build/lib.linux-x86_64-cpython-310
    running build_ext
    building '_pysha3' extension
    creating build/temp.linux-x86_64-cpython-310/Modules/_sha3
    gcc -Wno-unused-result -Wsign-compare -DNDEBUG -g -fwrapv -O3 -Wall -fPIC -DPY_WITH_KECCAK=1 -I/root/.cache/uv/builds-v0/.tmp8V4iEk/include -I/usr/local/include/python3.10 -c
    Modules/_sha3/sha3module.c -o build/temp.linux-x86_64-cpython-310/Modules/_sha3/sha3module.o

    [stderr]
    error: command 'gcc' failed: No such file or directory
```

در این صورت، باید آن را با مدیر package سیستم خود نصب کنید؛ مثلاً برای حل خطای بالا:

```console
$ apt install gcc
```

!!! tip

    هنگام استفاده از نسخه‌های Python مدیریت‌شده توسط uv، معمولاً به نصب `clang` به‌جای `gcc`
    نیاز است.

    بسیاری از distributionهای Linux یک package ارائه می‌دهند که همهٔ dependencyهای رایج build را
    شامل می‌شود. می‌توانید بیشتر نیازمندی‌های build را با نصب آن برطرف کنید؛ مثلاً برای Debian
    یا Ubuntu:

    ```console
    $ apt install build-essential
    ```

### header یا library گم شده است {#header-or-library-is-missing}

اگر خطای build به یک header یا library گمشده اشاره کند، مثلاً یک فایل `.h`، باید آن را با مدیر
package سیستم خود نصب کنید.

برای مثال، نصب `pygraphviz` به نصب Graphviz نیاز دارد:

<!-- docker run --platform linux/x86_64 -it ghcr.io/astral-sh/uv:python3.12-trixie /bin/bash -c "uv pip install --system 'pygraphviz'" -->

```hl_lines="18-19"
× Failed to build `pygraphviz==1.14`
├─▶ The build backend returned an error
╰─▶ Call to `setuptools.build_meta.build_wheel` failed (exit status: 1)

  [stdout]
  running bdist_wheel
  running build
  running build_py
  ...
  gcc -fno-strict-overflow -Wsign-compare -DNDEBUG -g -O3 -Wall -fPIC -DSWIG_PYTHON_STRICT_BYTE_CHAR -I/root/.cache/uv/builds-v0/.tmpgLYPe0/include -I/usr/local/include/python3.12 -c pygraphviz/graphviz_wrap.c -o
  build/temp.linux-x86_64-cpython-312/pygraphviz/graphviz_wrap.o

  [stderr]
  ...
  pygraphviz/graphviz_wrap.c:9: warning: "SWIG_PYTHON_STRICT_BYTE_CHAR" redefined
      9 | #define SWIG_PYTHON_STRICT_BYTE_CHAR
        |
  <command-line>: note: this is the location of the previous definition
  pygraphviz/graphviz_wrap.c:3023:10: fatal error: graphviz/cgraph.h: No such file or directory
    3023 | #include "graphviz/cgraph.h"
        |          ^~~~~~~~~~~~~~~~~~~
  compilation terminated.
  error: command '/usr/bin/gcc' failed with exit code 1

  hint: This error likely indicates that you need to install a library that provides `graphviz/cgraph.h` for `pygraphviz@1.14`
```

برای حل این خطا در Debian، package با نام `libgraphviz-dev` را نصب می‌کنید:

```console
$ apt install libgraphviz-dev
```

توجه کنید که نصب package با نام `graphviz` کافی نیست؛ headerهای توسعه باید نصب شوند.

!!! tip

    برای حل خطایی که در آن `Python.h` گم شده است،
    [package با نام `python3-dev`](https://packages.debian.org/trixie/python3-dev) را نصب کنید.

### module گم شده است یا نمی‌توان آن را import کرد {#module-is-missing-or-cannot-be-imported}

اگر خطای build به یک import ناموفق اشاره کند،
[غیرفعال کردن build isolation](../../concepts/projects/config.md#build-isolation) را در نظر
بگیرید.

برای مثال، بعضی packageها فرض می‌کنند که `pip` بدون اعلام آن به‌عنوان یک build dependency در
دسترس است:

<!-- docker run --platform linux/x86_64 -it ghcr.io/astral-sh/uv:python3.12-trixie-slim /bin/bash -c "uv pip install --system chumpy" -->

```hl_lines="7"
  × Failed to build `chumpy==0.70`
  ├─▶ The build backend returned an error
  ╰─▶ Call to `setuptools.build_meta:__legacy__.build_wheel` failed (exit status: 1)

    [stderr]
    Traceback (most recent call last):
      File "<string>", line 9, in <module>
    ModuleNotFoundError: No module named 'pip'

    During handling of the above exception, another exception occurred:

    Traceback (most recent call last):
      File "<string>", line 14, in <module>
      File "/root/.cache/uv/builds-v0/.tmpvvHaxI/lib/python3.12/site-packages/setuptools/build_meta.py", line 334, in get_requires_for_build_wheel
        return self._get_build_requires(config_settings, requirements=[])
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
      File "/root/.cache/uv/builds-v0/.tmpvvHaxI/lib/python3.12/site-packages/setuptools/build_meta.py", line 304, in _get_build_requires
        self.run_setup()
      File "/root/.cache/uv/builds-v0/.tmpvvHaxI/lib/python3.12/site-packages/setuptools/build_meta.py", line 522, in run_setup
        super().run_setup(setup_script=setup_script)
      File "/root/.cache/uv/builds-v0/.tmpvvHaxI/lib/python3.12/site-packages/setuptools/build_meta.py", line 320, in run_setup
        exec(code, locals())
      File "<string>", line 11, in <module>
    ModuleNotFoundError: No module named 'pip'
```

برای حل این خطا، ابتدا dependencyهای build را نصب کنید و سپس build isolation را برای package
غیرفعال کنید:

```console
$ uv pip install pip setuptools
$ uv pip install chumpy --no-build-isolation-package chumpy
```

توجه کنید که باید package گمشده، مثلاً `pip`، _و_ همهٔ dependencyهای build دیگر package، مثلاً
`setuptools`، را نصب کنید.

### نسخهٔ قدیمی package build می‌شود {#old-version-of-the-package-is-built}

اگر یک package در حین resolution در build ناموفق شود و نسخه‌ای که در build ناموفق شده قدیمی‌تر از
نسخه‌ای باشد که می‌خواهید استفاده کنید، سعی کنید یک
[constraint](../settings.md#constraint-dependencies) با یک حد پایین اضافه کنید (مثلاً
`numpy>=1.17`). گاهی به دلیل محدودیت‌های الگوریتمی، resolver مربوط به uv تلاش می‌کند با استفاده
از packageهای غیرمعقول قدیمی، نسخهٔ مناسبی پیدا کند که می‌توان با استفاده از حدهای پایین از آن
جلوگیری کرد.

برای مثال، هنگام حل dependencyهای زیر روی Python 3.10، uv تلاش می‌کند یک نسخهٔ قدیمی از
`apache-beam` را build کند.

```title="requirements.txt"
dill<0.3.9,>=0.2.2
apache-beam<=2.49.0
```

<!-- docker run --platform linux/x86_64 -it ghcr.io/astral-sh/uv:python3.10-trixie-slim /bin/bash -c "printf 'dill<0.3.9,>=0.2.2\napache-beam<=2.49.0' | uv pip compile -" -->

```hl_lines="1"
× Failed to build `apache-beam==2.0.0`
├─▶ The build backend returned an error
╰─▶ Call to `setuptools.build_meta:__legacy__.build_wheel` failed (exit status: 1)

    [stderr]
    ...
```

افزودن یک constraint با حد پایین، مثلاً `apache-beam<=2.49.0,>2.30.0`، این خطای build را حل
می‌کند، چون uv از استفاده از نسخهٔ قدیمی `apache-beam` اجتناب می‌کند.

constraintها را می‌توان برای dependencyهای غیرمستقیم هم با استفاده از فایل‌های `constraints.txt`
یا تنظیم [`constraint-dependencies`](../settings.md#constraint-dependencies) تعریف کرد.

### نسخهٔ قدیمی یک build dependency استفاده می‌شود {#old-version-of-a-build-dependency-is-used}

اگر یک package به این دلیل در build ناموفق شود که `uv` نسخهٔ ناسازگار یا قدیمی‌ای از یک dependency
زمان build را انتخاب می‌کند، می‌توانید constraintها را به‌طور خاص برای build dependencyها اعمال
کنید. می‌توان از تنظیم
[`build-constraint-dependencies`](../settings.md#build-constraint-dependencies) (یا یک فایل
مشابه `build-constraints.txt`) استفاده کرد تا مطمئن شوید `uv` نسخهٔ مناسبی از یک build requirement
معین را انتخاب می‌کند.

برای مثال، issue توصیف‌شده در
[#5551](https://github.com/astral-sh/uv/issues/5551#issuecomment-2256055975) را می‌توان با مشخص
کردن یک build constraint که نسخهٔ `72.0.0` از `setuptools` را کنار می‌گذارد، حل کرد:

```toml title="pyproject.toml"
[tool.uv]
# Prevent setuptools version 72.0.0 from being used as a build dependency.
build-constraint-dependencies = ["setuptools!=72.0.0"]
```

بنابراین build constraint تضمین می‌کند که هر package‌ای که در فرایند build به `setuptools` نیاز
دارد، از استفاده از نسخهٔ مشکل‌ساز اجتناب می‌کند و از خطاهای build ناشی از build dependencyهای
ناسازگار جلوگیری می‌شود.

### package فقط برای یک platform استفاده‌نشده لازم است {#package-is-only-needed-for-an-unused-platform}

اگر قفل کردن به دلیل build کردن یک package از platform‌ای که نیازی به پشتیبانی از آن ندارید
ناموفق شود،
[محدود کردن resolution](../../concepts/projects/config.md#limited-resolution-environments) به
platformهای پشتیبانی‌شدهٔ خود را در نظر بگیرید.

### package از همهٔ نسخه‌های Python پشتیبانی نمی‌کند {#package-does-not-support-all-python-versions}

اگر از بازهٔ بزرگی از نسخه‌های Python پشتیبانی می‌کنید، استفاده از markerها را در نظر بگیرید تا
برای نسخه‌های قدیمی‌تر Python از نسخه‌های قدیمی‌تر و برای نسخه‌های جدیدتر Python از نسخه‌های
جدیدتر استفاده کنید. برای مثال، `numpy` در هر زمان فقط از چهار نسخهٔ minor مربوط به Python
پشتیبانی می‌کند، پس برای پشتیبانی از بازهٔ وسیع‌تری از نسخه‌های Python، مثلاً Python 3.8 تا 3.13،
requirement مربوط به `numpy` باید تقسیم شود:

```
numpy>=1.23; python_version >= "3.10"
numpy<1.23; python_version < "3.10"
```

### package فقط روی یک platform مشخص قابل استفاده است {#package-is-only-usable-on-a-specific-platform}

اگر قفل کردن به دلیل build کردن package‌ای که فقط روی platform دیگری قابل استفاده است ناموفق شود،
می‌توانید [metadata مربوط به dependency را به‌صورت دستی ارائه دهید](../settings.md#dependency-metadata)
تا build را رد کنید. uv نمی‌تواند این اطلاعات را تأید کند، پس مهم است که هنگام استفاده از این
override، metadata صحیح را مشخص کنید.
