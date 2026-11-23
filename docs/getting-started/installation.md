# نصب uv {#installing-uv}

## روش‌های نصب {#installation-methods}

uv را با installer های مستقل ما یا package manager دلخواه خود نصب کنید.

### نصب‌کنندهٔ مستقل {#standalone-installer}

uv یک installer مستقل برای دانلود و نصب uv ارائه می‌دهد:

=== "macOS and Linux"

    از `curl` برای دانلود script استفاده کنید و آن را با `sh` اجرا کنید:

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

    اگر سیستم شما `curl` ندارد، می‌توانید از `wget` استفاده کنید:

    ```console
    $ wget -qO- https://astral.sh/uv/install.sh | sh
    ```

    با گنجاندن version موردنظر در URL، یک version مشخص درخواست کنید:

    ```console
    $ curl -LsSf https://astral.sh/uv/0.13.9/install.sh | sh
    ```

=== "Windows"

    از `irm` برای دانلود script استفاده کنید و آن را با `iex` اجرا کنید:

    ```pwsh-session
    PS> powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```

    تغییر [execution policy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4#powershell-execution-policies) امکان اجرای یک script از اینترنت را می‌دهد.

    با گنجاندن version موردنظر در URL، یک version مشخص درخواست کنید:

    ```pwsh-session
    PS> powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/0.13.9/install.ps1 | iex"
    ```

!!! tip

    می‌توانید script نصب را پیش از استفاده بررسی کنید:

    === "macOS and Linux"

        ```console
        $ curl -LsSf https://astral.sh/uv/install.sh | less
        ```

    === "Windows"

        ```pwsh-session
        PS> powershell -c "irm https://astral.sh/uv/install.ps1 | more"
        ```

    همچنین، می‌توانید installer یا binary ها را مستقیم از [GitHub](#github-releases) دانلود کنید.

برای جزئیات سفارشی‌سازی نصب uv خود، مستندات مرجع [installer](../reference/installer.md) را
ببینید.

### PyPI {#pypi}

برای راحتی، uv در [PyPI](https://pypi.org/project/uv/) منتشر شده است.

اگر از PyPI نصب می‌کنید، توصیه می‌کنیم uv را در یک environment مجزا نصب کنید؛ مثلاً با `pipx`:

```console
$ pipx install uv
```

با این حال، می‌توان از `pip` هم استفاده کرد:

```console
$ pip install uv
```

!!! note

    uv با distribution های ازپیش‌ساخته‌شده (wheel) برای platform های زیادی عرضه می‌شود؛ اگر
    برای یک platform مشخص wheel در دسترس نباشد، uv از source ساخته می‌شود که به یک Rust toolchain
    نیاز دارد. برای جزئیات ساخت uv از source، [راهنمای راه‌اندازی مشارکت](https://github.com/astral-sh/uv/blob/main/CONTRIBUTING.md#setup)
    را ببینید.

### Homebrew {#homebrew}

uv در package های اصلی Homebrew در دسترس است.

```console
$ brew install uv
```

### MacPorts {#macports}

uv از طریق [MacPorts](https://ports.macports.org/port/uv/) در دسترس است.

```console
$ sudo port install uv
```

### WinGet {#winget}

uv از طریق [WinGet](https://winstall.app/apps/astral-sh.uv) در دسترس است.

```console
$ winget install --id=astral-sh.uv  -e
```

### Scoop {#scoop}

uv از طریق [Scoop](https://scoop.sh/#/apps?q=uv) در دسترس است.

```console
$ scoop install main/uv
```

### Docker {#docker}

uv یک image Docker در
[`ghcr.io/astral-sh/uv`](https://github.com/astral-sh/uv/pkgs/container/uv) ارائه می‌دهد.

برای جزئیات بیشتر، راهنمای ما دربارهٔ [استفاده از uv در Docker](../guides/integration/docker.md) را
ببینید.

### GitHub Releases {#github-releases}

artifact های انتشار uv را می‌توان مستقیم از
[GitHub Releases](https://github.com/astral-sh/uv/releases) دانلود کرد.

هر صفحهٔ انتشار شامل binary هایی برای همهٔ platform های پشتیبانی‌شده و همچنین دستورالعمل‌هایی
برای استفاده از installer مستقل از طریق `github.com` به‌جای `astral.sh` است.

### Cargo {#cargo}

uv از طریق [crates.io](https://crates.io) در دسترس است.

```console
$ cargo install --locked uv
```

!!! note

    این روش uv را از source می‌سازد که به یک Rust toolchain سازگار نیاز دارد.

## به‌روزرسانی uv {#upgrading-uv}

وقتی uv با installer مستقل نصب شده باشد، می‌تواند در صورت نیاز خودش را به‌روز کند:

```console
$ uv self update
```

!!! tip

    به‌روزرسانی uv، installer را دوباره اجرا می‌کند و می‌تواند profile های shell شما را تغییر
    دهد. برای غیرفعال کردن این رفتار، `UV_NO_MODIFY_PATH=1` را تنظیم کنید.

وقتی روش نصب دیگری استفاده شود، به‌روزرسانی خودکار غیرفعال است. به‌جای آن از روش ارتقای package
manager استفاده کنید؛ مثلاً با `pip`:

```console
$ pip install --upgrade uv
```

## تکمیل خودکار shell {#shell-autocompletion}

!!! tip

    می‌توانید `echo $SHELL` را اجرا کنید تا shell خود را مشخص کنید.

برای فعال کردن تکمیل خودکار shell برای دستورهای uv، یکی از موارد زیر را اجرا کنید:

=== "Bash"

    ```bash
    echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uv generate-shell-completion fish | source' > ~/.config/fish/completions/uv.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uv generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uv generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

برای فعال کردن تکمیل خودکار shell برای uvx، یکی از موارد زیر را اجرا کنید:

=== "Bash"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uvx --generate-shell-completion fish | source' > ~/.config/fish/completions/uvx.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uvx --generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uvx --generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

سپس shell را دوباره راه‌اندازی کنید یا فایل پیکربندی shell را source کنید.

## حذف نصب {#uninstallation}

اگر نیاز دارید uv را از سیستم خود حذف کنید، این مراحل را دنبال کنید:

1.  پاک کردن داده‌های ذخیره‌شده (اختیاری):

    ```console
    $ uv cache clean
    $ rm -r "$(uv python dir)"
    $ rm -r "$(uv tool dir)"
    ```

    !!! tip

        پیش از حذف binary ها، شاید بخواهید داده‌هایی را که uv ذخیره کرده است حذف کنید. برای
        جزئیات محل ذخیرهٔ داده‌ها توسط uv، [مرجع storage](../reference/storage.md) را ببینید.

2.  binary های uv و uvx و uvw را حذف کنید:

    === "macOS and Linux"

        ```console
        $ rm ~/.local/bin/uv ~/.local/bin/uvx
        ```

    === "Windows"

        ```pwsh-session
        PS> rm $HOME\.local\bin\uv.exe
        PS> rm $HOME\.local\bin\uvx.exe
        PS> rm $HOME\.local\bin\uvw.exe
        ```

    !!! note

        پیش از 0.5.0، uv در `~/.cargo/bin` نصب می‌شد. برای حذف نصب، می‌توان binary ها را از
        آنجا حذف کرد. ارتقا از یک version قدیمی‌تر، binary ها را به‌صورت خودکار از `~/.cargo/bin`
        حذف نمی‌کند.

## قدم‌های بعدی {#next-steps}

[قدم‌های اول](./first-steps.md) را ببینید یا مستقیم به [راهنماها](../guides/index.md) بروید تا
استفاده از uv را شروع کنید.
