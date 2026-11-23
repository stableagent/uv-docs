# نصب‌کنندهٔ uv {#the-uv-installer}

## تغییر مسیر نصب {#changing-the-installation-path}

به‌طور پیش‌فرض، uv در [پوشهٔ اجرایی](./storage.md#executable-directory) کاربر نصب می‌شود.

برای تغییر مسیر نصب، از `UV_INSTALL_DIR` استفاده کنید:

=== "macOS and Linux"

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR="/custom/path" sh
    ```

=== "Windows"

    ```pwsh-session
    PS> powershell -ExecutionPolicy ByPass -c {$env:UV_INSTALL_DIR = "C:\Custom\Path";irm https://astral.sh/uv/install.ps1 | iex}
    ```

!!! note

    تغییر مسیر نصب فقط روی جایی که باینری uv نصب می‌شود تأثیر می‌گذارد. uv همچنان داده‌هایش
    (cache، نصب‌های Python، toolها و غیره) را در مکان‌های پیش‌فرض ذخیره می‌کند. برای جزئیات این
    مکان‌ها و نحوهٔ سفارشی‌سازی آن‌ها، [مرجع storage](./storage.md) را ببینید.

## غیرفعال کردن تغییرات shell {#disabling-shell-modifications}

نصب‌کننده ممکن است پروفایل‌های shell شما را هم به‌روز کند تا مطمئن شود باینری uv در `PATH` شما
قرار دارد. برای غیرفعال کردن این رفتار، از `UV_NO_MODIFY_PATH` استفاده کنید. برای مثال:

```console
$ curl -LsSf https://astral.sh/uv/install.sh | env UV_NO_MODIFY_PATH=1 sh
```

اگر با `UV_NO_MODIFY_PATH` نصب شده باشد، عملیات‌های بعدی، مثل `uv self update`، پروفایل‌های shell
شما را تغییر نمی‌دهند.

## نصب‌های مدیریت‌نشده {#unmanaged-installations}

در environmentهای موقت مثل CI، از `UV_UNMANAGED_INSTALL` استفاده کنید تا uv را در یک مسیر مشخص
نصب کنید و در عین حال جلوی تغییر پروفایل‌های shell یا environment variableها توسط نصب‌کننده را
بگیرید:

```console
$ curl -LsSf https://astral.sh/uv/install.sh | env UV_UNMANAGED_INSTALL="/custom/path" sh
```

استفاده از `UV_UNMANAGED_INSTALL` همچنین به‌روزرسانی‌های خودکار (از طریق `uv self update`) را
غیرفعال می‌کند.

## ارسال گزینه‌ها به script نصب {#passing-options-to-the-installation-script}

استفاده از environment variableها توصیه می‌شود، چون در همهٔ platformها یکسان هستند. با این حال،
می‌توان گزینه‌ها را مستقیماً به script نصب ارسال کرد. برای مثال، برای دیدن گزینه‌های موجود:

```console
$ curl -LsSf https://astral.sh/uv/install.sh | sh -s -- --help
```
