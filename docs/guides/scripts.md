---
title: اجرای scriptها
description:
  راهنمای استفاده از uv برای اجرای scriptهای Python، شامل پشتیبانی از inline dependency metadata،
  scriptهای تکرارپذیر و موارد دیگر.
---

# اجرای scriptها {#running-scripts}

یک script در Python فایلی است که برای اجرای مستقل در نظر گرفته شده است، مثلاً با
`python <script>.py`. استفاده از uv برای اجرای scriptها تضمین می‌کند که dependencyهای script بدون
مدیریت دستی environmentها مدیریت شوند.

!!! note

    اگر با environmentهای Python آشنا نیستید: هر نصب Python یک environment دارد که packageها
    می‌توانند در آن نصب شوند. معمولاً توصیه می‌شود برای جداسازی packageهای موردنیاز هر script،
    [_virtual_ environments](https://docs.python.org/3/library/venv.html) ایجاد کنید. uv به‌طور
    خودکار virtual environmentها را برای شما مدیریت می‌کند و رویکرد
    [اعلانی](#declaring-script-dependencies) را برای dependencyها ترجیح می‌دهد.

## اجرای یک script بدون dependency {#running-a-script-without-dependencies}

اگر script شما dependency ندارد، می‌توانید آن را با `uv run` اجرا کنید:

```python title="example.py"
print("Hello world")
```

```console
$ uv run example.py
Hello world
```

<!-- TODO(zanieb): Once we have a `python` shim, note you can execute it with `python` here -->

به همین ترتیب، اگر script شما به یک module در کتابخانهٔ استاندارد وابسته است، کار دیگری لازم نیست:

```python title="example.py"
import os

print(os.path.expanduser("~"))
```

```console
$ uv run example.py
/Users/astral
```

می‌توان به script آرگومان داد:

```python title="example.py"
import sys

print(" ".join(sys.argv[1:]))
```

```console
$ uv run example.py test
test

$ uv run example.py hello world!
hello world!
```

همچنین، script شما می‌تواند مستقیماً از stdin خوانده شود:

```console
$ echo 'print("hello world!")' | uv run -
```

یا، اگر shell شما از [here-documentها](https://en.wikipedia.org/wiki/Here_document) پشتیبانی می‌کند:

```bash
uv run - <<EOF
print("hello world!")
EOF
```

توجه کنید که اگر از `uv run` در یک _project_ استفاده کنید، یعنی در پوشه‌ای که یک
`pyproject.toml` دارد، پیش از اجرای script، project فعلی نصب می‌شود. اگر script شما به project
وابسته نیست، برای رد کردن این کار از فلگ `--no-project` استفاده کنید:

```console
$ # Note: the `--no-project` flag must be provided _before_ the script name.
$ uv run --no-project example.py
```

برای جزئیات بیشتر دربارهٔ کار در projectها، به [راهنمای projectها](./projects.md) مراجعه کنید.

## اجرای یک script با dependencyها {#running-a-script-with-dependencies}

وقتی script شما به packageهای دیگری نیاز دارد، آن‌ها باید در environmentای نصب شوند که script در
آن اجرا می‌شود. uv ترجیح می‌دهد این environmentها را در صورت نیاز ایجاد کند، به‌جای استفاده از یک
virtual environment بلندمدت با dependencyهایی که به‌صورت دستی مدیریت می‌شوند. این کار به اعلان
صریح dependencyهای موردنیاز script نیاز دارد. به‌طور کلی، توصیه می‌شود برای اعلان dependencyها از
یک [project](./projects.md) یا [inline metadata](#declaring-script-dependencies) استفاده کنید، اما
uv از درخواست dependency در هر فراخوانی نیز پشتیبانی می‌کند.

برای مثال، script زیر به `rich` نیاز دارد.

```python title="example.py"
import time
from rich.progress import track

for i in track(range(20), description="For example:"):
    time.sleep(0.05)
```

اگر این script بدون مشخص کردن dependency اجرا شود، با خطا مواجه می‌شود:

```console
$ uv run --no-project example.py
Traceback (most recent call last):
  File "/Users/astral/example.py", line 2, in <module>
    from rich.progress import track
ModuleNotFoundError: No module named 'rich'
```

dependency را با گزینهٔ `--with` درخواست کنید:

```console
$ uv run --with rich example.py
For example: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:01
```

اگر به نسخه‌های خاصی نیاز است، می‌توان به dependency درخواستی constraint اضافه کرد:

```console
$ uv run --with 'rich>12,<13' example.py
```

با تکرار گزینهٔ `--with` می‌توان چند dependency درخواست کرد.

توجه کنید که اگر `uv run` در یک _project_ استفاده شود، این dependencyها _علاوه بر_ dependencyهای
project لحاظ می‌شوند. برای انصراف از این رفتار، از فلگ `--no-project` استفاده کنید.

## ایجاد یک script در Python {#creating-a-python-script}

Python به‌تازگی یک قالب استاندارد برای
[inline script metadata](https://packaging.python.org/en/latest/specifications/inline-script-metadata/#inline-script-metadata)
اضافه کرده است. این قالب امکان انتخاب Python versionها و تعریف dependencyها را فراهم می‌کند. برای
مقداردهی اولیهٔ scriptها با inline metadata از `uv init --script` استفاده کنید:

```console
$ uv init --script example.py --python 3.12
```

## اعلان dependencyهای script {#declaring-script-dependencies}

قالب inline metadata امکان می‌دهد dependencyهای یک script در خود script اعلان شوند.

uv افزودن و به‌روزرسانی inline script metadata را برای شما پشتیبانی می‌کند. برای اعلان dependencyهای
script از `uv add --script` استفاده کنید:

```console
$ uv add --script example.py 'requests<3' 'rich'
```

این کار یک بخش `script` در بالای script اضافه می‌کند که dependencyها را با TOML اعلان می‌کند:

```python title="example.py"
# /// script
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json")
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])
```

uv به‌طور خودکار یک environment با dependencyهای لازم برای اجرای script ایجاد می‌کند، مثلاً:

```console
$ uv run example.py
[
│   ('1', 'PEP Purpose and Guidelines'),
│   ('2', 'Procedure for Adding New Modules'),
│   ('3', 'Guidelines for Handling Bug Reports'),
│   ('4', 'Deprecation of Standard Modules'),
│   ('5', 'Guidelines for Language Evolution'),
│   ('6', 'Bug Fix Releases'),
│   ('7', 'Style Guide for C Code'),
│   ('8', 'Style Guide for Python Code'),
│   ('9', 'Sample Plaintext PEP Template'),
│   ('10', 'Voting Guidelines')
]
```

!!! important

    هنگام استفاده از inline script metadata، حتی اگر `uv run` در یک
    [_project_](../concepts/projects/run.md) استفاده شود، dependencyهای project نادیده گرفته
    می‌شوند. فلگ `--no-project` لازم نیست.

uv به الزامات Python version نیز احترام می‌گذارد:

```python title="example.py"
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///

# Use some syntax added in Python 3.12
type Point = tuple[float, float]
print(Point)
```

!!! note

    فیلد `dependencies` حتی اگر خالی باشد نیز باید ارائه شود.

`uv run` نسخهٔ موردنیاز Python را جستجو و استفاده می‌کند. اگر Python version نصب نشده باشد، دانلود
می‌شود — برای جزئیات بیشتر به مستندات [Python versionها](../concepts/python-versions.md) مراجعه
کنید.

## استفاده از shebang برای ایجاد یک فایل اجرایی {#using-a-shebang-to-create-an-executable-file}

می‌توان یک shebang اضافه کرد تا script بدون استفاده از `uv run` اجرایی شود — این کار اجرای
scriptهایی را که در `PATH` شما یا در پوشهٔ فعلی هستند آسان می‌کند.

برای مثال، فایلی به نام `greet` با محتوای زیر ایجاد کنید:

```python title="greet"
#!/usr/bin/env -S uv run --script

print("Hello, world!")
```

مطمئن شوید script شما اجرایی است، مثلاً با `chmod +x greet`، سپس script را اجرا کنید:

```console
$ ./greet
Hello, world!
```

اعلان dependencyها در این زمینه نیز پشتیبانی می‌شود، برای مثال:

```python title="example"
#!/usr/bin/env -S uv run --script
#
# /// script
# requires-python = ">=3.12"
# dependencies = ["httpx"]
# ///

import httpx

print(httpx.get("https://example.com"))
```

## استفاده از indexهای جایگزین package {#using-alternative-package-indexes}

اگر می‌خواهید برای resolve کردن dependencyها از یک [package index](../concepts/indexes.md) جایگزین
استفاده کنید، می‌توانید index را با گزینهٔ `--index` ارائه دهید:

```console
$ uv add --index "https://example.com/simple" --script example.py 'requests<3' 'rich'
```

این کار داده‌های package را در inline metadata قرار می‌دهد:

```python
# [[tool.uv.index]]
# url = "https://example.com/simple"
```

اگر برای دسترسی به package index به احراز هویت نیاز دارید، به مستندات
[package index](../concepts/indexes.md) مراجعه کنید.

## قفل کردن dependencyها {#locking-dependencies}

uv قفل کردن dependencyها برای scriptهای PEP 723 را با قالب فایل `uv.lock` پشتیبانی می‌کند. برخلاف
projectها، scriptها باید به‌صورت صریح با `uv lock` قفل شوند:

```console
$ uv lock --script example.py
```

اجرای `uv lock --script` یک فایل `.lock` در کنار script ایجاد می‌کند (مثلاً `example.py.lock`).

پس از قفل شدن، عملیات‌های بعدی مانند `uv run --script`، `uv add --script`، `uv export --script`
و `uv tree --script` از dependencyهای قفل‌شده دوباره استفاده می‌کنند و در صورت نیاز lockfile را
به‌روزرسانی می‌کنند.

اگر چنین lockfileای وجود نداشته باشد، دستورهایی مانند `uv export --script` همچنان طبق انتظار کار
می‌کنند، اما lockfile ایجاد نمی‌کنند.

## بهبود تکرارپذیری {#improving-reproducibility}

علاوه بر قفل کردن dependencyها، uv از یک فیلد `exclude-newer` در بخش `tool.uv` از inline script
metadata پشتیبانی می‌کند تا uv را به در نظر گرفتن تنها distributionهایی که پیش از یک تاریخ مشخص
منتشر شده‌اند محدود کند. این کار برای بهبود تکرارپذیری script شما هنگام اجرا در زمانی دیگر مفید
است.

تاریخ باید به‌صورت یک timestamp به قالب [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html)
مشخص شود (مثلاً `2006-12-02T02:07:43Z`).

```python title="example.py"
# /// script
# dependencies = [
#   "requests",
# ]
# [tool.uv]
# exclude-newer = "2023-10-16T00:00:00Z"
# ///

import requests

print(requests.__version__)
```

## استفاده از Python versionهای مختلف {#using-different-python-versions}

uv امکان می‌دهد در هر فراخوانی script، Python versionهای دلخواه درخواست شوند، برای مثال:

```python title="example.py"
import sys

print(".".join(map(str, sys.version_info[:3])))
```

```console
$ # Use the default Python version, may differ on your machine
$ uv run example.py
3.12.6
```

```console
$ # Use a specific Python version
$ uv run --python 3.10 example.py
3.10.15
```

برای جزئیات بیشتر دربارهٔ درخواست Python versionها، به مستندات
[درخواست Python version](../concepts/python-versions.md#requesting-a-version) مراجعه کنید.

## استفاده از scriptهای GUI {#using-gui-scripts}

در Windows، `uv` script شما را که با پسوند `.pyw` تمام می‌شود با `pythonw` اجرا می‌کند:

```python title="example.pyw"
from tkinter import Tk, ttk

root = Tk()
root.title("uv")
frm = ttk.Frame(root, padding=10)
frm.grid()
ttk.Label(frm, text="Hello World").grid(column=0, row=0)
root.mainloop()
```

```console
PS> uv run example.pyw
```

![نتیجهٔ اجرا](../assets/uv_gui_script_hello_world.png){: style="height:50px;width:150px"}

به همین ترتیب، با dependencyها نیز کار می‌کند:

```python title="example_pyqt.pyw"
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QLabel, QGridLayout

app = QApplication(sys.argv)
widget = QWidget()
grid = QGridLayout()

text_label = QLabel()
text_label.setText("Hello World!")
grid.addWidget(text_label)

widget.setLayout(grid)
widget.setGeometry(100, 100, 200, 50)
widget.setWindowTitle("uv")
widget.show()
sys.exit(app.exec_())
```

```console
PS> uv run --with PyQt5 example_pyqt.pyw
```

![نتیجهٔ اجرا](../assets/uv_gui_script_hello_world_pyqt.png){: style="height:50px;width:150px"}

## گام‌های بعدی {#next-steps}

برای آشنایی بیشتر با `uv run`، به [مرجع دستور](../reference/cli.md#uv-run) مراجعه کنید.

یا، برای یادگیری [اجرا و نصب toolها](./tools.md) با uv ادامه دهید.
