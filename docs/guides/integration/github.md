---
title: استفاده از uv در GitHub Actions
description:
  راهنمای استفاده از uv در GitHub Actions، شامل نصب، راه‌اندازی Python، نصب dependencyها و موارد
  دیگر.
---

# استفاده از uv در GitHub Actions {#using-uv-in-github-actions}

## نصب {#installation}

برای استفاده با GitHub Actions، action رسمی
[`astral-sh/setup-uv`](https://github.com/astral-sh/setup-uv) را توصیه می‌کنیم که uv را نصب
می‌کند، آن را به PATH اضافه می‌کند، (به‌صورت اختیاری) cache را پایدار نگه می‌دارد و موارد دیگر، با
پشتیبانی از همهٔ platformهای پشتیبانی‌شده توسط uv.

برای نصب آخرین نسخهٔ uv:

```yaml title="example.yml" hl_lines="11 12"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
```

بهترین روش این است که به یک نسخهٔ مشخص از uv پین کنید، مثلاً با:

```yaml title="example.yml" hl_lines="14 15"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
        with:
          # Install a specific version of uv.
          version: "0.13.9"
```

## راه‌اندازی Python {#setting-up-python}

می‌توان Python را با دستور `python install` نصب کرد:

```yaml title="example.yml" hl_lines="14 15"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0

      - name: Set up Python
        run: uv python install
```

این دستور به نسخهٔ Python پین‌شده در project احترام می‌گذارد.

به‌جای آن، می‌توان از action رسمی GitHub با نام `setup-python` استفاده کرد. این کار می‌تواند
سریع‌تر باشد، زیرا GitHub نسخه‌های Python را در کنار runner در cache نگه می‌دارد.

گزینهٔ
[`python-version-file`](https://github.com/actions/setup-python/blob/main/docs/advanced-usage.md#using-the-python-version-file-input)
را تنظیم کنید تا از نسخهٔ پین‌شده برای project استفاده شود:

```yaml title="example.yml" hl_lines="14"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: "Set up Python"
        uses: actions/setup-python@5fda3b95a4ea91299a34e894583c3862153e4b97 # v7.0.0
        with:
          python-version-file: ".python-version"

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
```

یا فایل `pyproject.toml` را مشخص کنید تا پین نادیده گرفته شود و از آخرین نسخهٔ سازگار با constraint
مربوط به `requires-python` در project استفاده شود:

```yaml title="example.yml" hl_lines="14"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: "Set up Python"
        uses: actions/setup-python@5fda3b95a4ea91299a34e894583c3862153e4b97 # v7.0.0
        with:
          python-version-file: "pyproject.toml"

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
```

## چند نسخهٔ Python {#multiple-python-versions}

هنگام استفاده از یک matrix برای تست چند نسخهٔ Python، نسخهٔ Python را با استفاده از
`astral-sh/setup-uv` تنظیم کنید که مشخصهٔ نسخهٔ Python را در فایل‌های `pyproject.toml` یا
`.python-version` بازنویسی می‌کند:

```yaml title="example.yml" hl_lines="17 18"
jobs:
  build:
    name: continuous-integration
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version:
          - "3.10"
          - "3.11"
          - "3.12"

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Install uv and set the Python version
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
        with:
          python-version: ${{ matrix.python-version }}
```

اگر از action مربوط به `setup-uv` استفاده نمی‌کنید، می‌توانید environment variable مربوط به
`UV_PYTHON` را تنظیم کنید:

```yaml title="example.yml" hl_lines="12"
jobs:
  build:
    name: continuous-integration
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version:
          - "3.10"
          - "3.11"
          - "3.12"
    env:
      UV_PYTHON: ${{ matrix.python-version }}
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
```

## Sync کردن و اجرا کردن {#syncing-and-running}

وقتی uv و Python نصب شدند، می‌توان project را با `uv sync` نصب کرد و دستورها را با `uv run` در
environment اجرا کرد:

```yaml title="example.yml" hl_lines="15 17-22"
name: Example

jobs:
  uv-example:
    name: python
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0

      - name: Install the project
        run: uv sync --locked --all-extras --dev

      - name: Run tests
        # For example, using `pytest`
        run: uv run pytest tests
```

!!! tip

    می‌توان از
    [تنظیم `UV_PROJECT_ENVIRONMENT`](../../concepts/projects/config.md#project-environment-path)
    برای نصب در system environment مربوط به Python به‌جای ساختن یک virtual environment استفاده کرد.

## کش کردن {#caching}

ممکن است ذخیرهٔ cache مربوط به uv در اجراهای workflow، زمان CI را بهبود دهد.

action مربوط به [`astral-sh/setup-uv`](https://github.com/astral-sh/setup-uv) پشتیبانی داخلی برای
پایدار نگه داشتن cache دارد:

```yaml title="example.yml"
- name: Enable caching
  uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
  with:
    enable-cache: true
```

به‌جای آن، می‌توانید cache را به‌صورت دستی با action مربوط به `actions/cache` مدیریت کنید:

```yaml title="example.yml"
jobs:
  install_job:
    env:
      # Configure a constant location for the uv cache
      UV_CACHE_DIR: /tmp/.uv-cache

    steps:
      # ... setup up Python and uv ...

      - name: Restore uv cache
        uses: actions/cache@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.1.0
        with:
          path: /tmp/.uv-cache
          key: uv-${{ runner.os }}-${{ hashFiles('uv.lock') }}
          restore-keys: |
            uv-${{ runner.os }}-${{ hashFiles('uv.lock') }}
            uv-${{ runner.os }}

      # ... install packages, run tests, etc ...

      - name: Minimize uv cache
        run: uv cache prune --ci
```

دستور `uv cache prune --ci` برای کاهش اندازهٔ cache استفاده می‌شود و برای CI بهینه شده است. تأثیر
آن بر عملکرد به packageهایی که نصب می‌شوند، بستگی دارد.

!!! tip

    اگر از `uv pip` استفاده می‌کنید، به‌جای `uv.lock` از `requirements.txt` در کلید cache استفاده
    کنید.

!!! note

    [post-job-hook]: https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/running-scripts-before-or-after-a-job

    هنگام استفاده از runnerهای self-hosted غیر موقت، پوشهٔ پیش‌فرض cache می‌تواند بدون حد رشد کند.
    در این صورت، ممکن است به‌اشتراک‌گذاشتن cache بین jobها بهینه نباشد. به‌جای آن، cache را به داخل
    GitHub Workspace منتقل کنید و پس از پایان job با استفاده از یک
    [Post Job Hook][post-job-hook] آن را حذف کنید.

    ```yaml
    install_job:
      env:
        # Configure a relative location for the uv cache
        UV_CACHE_DIR: ${{ github.workspace }}/.cache/uv
    ```

    استفاده از post job hook نیازمند تنظیم environment variable مربوط به
    `ACTIONS_RUNNER_HOOK_JOB_STARTED` روی runner مربوط به self-hosted، به مسیر یک script مربوط به
    پاک‌سازی مانند script زیر است.

    ```sh title="clean-uv-cache.sh"
    #!/usr/bin/env sh
    uv cache clean
    ```

## استفاده از `uv pip` {#using-uv-pip}

اگر به‌جای interface مربوط به project در uv از رابط مربوط به `uv pip` استفاده می‌کنید، uv
به‌طور پیش‌فرض به یک virtual environment نیاز دارد. برای اجازهٔ نصب packageها در system environment،
فلگ `--system` را در همهٔ فراخوانی‌های `uv` به کار ببرید یا متغیر `UV_SYSTEM_PYTHON` را تنظیم کنید.

متغیر `UV_SYSTEM_PYTHON` را می‌توان در دامنه‌های مختلف تعریف کرد.

با تعریف آن در سطح بالایی، برای کل workflow فعالش کنید:

```yaml title="example.yml"
env:
  UV_SYSTEM_PYTHON: 1

jobs: ...
```

یا برای یک job مشخص در workflow فعالش کنید:

```yaml title="example.yml"
jobs:
  install_job:
    env:
      UV_SYSTEM_PYTHON: 1
    ...
```

یا برای یک step مشخص در یک job فعالش کنید:

```yaml title="example.yml"
steps:
  - name: Install requirements
    run: uv pip install -r requirements.txt
    env:
      UV_SYSTEM_PYTHON: 1
```

برای غیرفعال‌کردن دوبارهٔ آن، می‌توان در هر فراخوانی uv از فلگ `--no-system` استفاده کرد.

## مخزن‌های خصوصی {#private-repos}

اگر project شما [dependency](../../concepts/projects/dependencies.md#git)هایی روی مخزن‌های خصوصی
GitHub دارد، باید یک [personal access token (PAT)][PAT] پیکربندی کنید تا uv بتواند آن‌ها را fetch
کند.

پس از ساختن یک PAT که دسترسی خواندن به مخزن‌های خصوصی را دارد، آن را به‌عنوان یک [repository
secret] اضافه کنید.

سپس، می‌توانید از CLI مربوط به [`gh`](https://cli.github.com/) (که به‌طور پیش‌فرض در runnerهای
GitHub Actions نصب شده است) برای پیکربندی یک
[credential helper برای Git](../../concepts/authentication/git.md#git-credential-helpers) استفاده
کنید تا از PAT برای درخواست‌ها به مخزن‌های میزبانی‌شده در `github.com` استفاده شود.

برای مثال، اگر repository secret خود را `MY_PAT` نامیدید:

```yaml title="example.yml"
steps:
  - name: Register the personal access token
    run: echo "${{ secrets.MY_PAT }}" | gh auth login --with-token
  - name: Configure the Git credential helper
    run: gh auth setup-git
```

[PAT]:
  https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
[repository secret]:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository

## منتشر کردن در PyPI {#publishing-to-pypi}

می‌توان از uv برای build و منتشر کردن package خود در PyPI از GitHub Actions استفاده کرد. ما یک مثال
مستقل در کنار این راهنما در
[astral-sh/trusted-publishing-examples](https://github.com/astral-sh/trusted-publishing-examples)
ارائه می‌دهیم. این workflow از [Trusted Publishing](https://docs.pypi.org/trusted-publishers/)
استفاده می‌کند، پس نیازی به پیکربندی credential نیست.

در workflow نمونه، از یک script برای تست اینکه source distribution و wheel هر دو کار می‌کنند و
فایلی را جا نینداخته‌ایم، استفاده می‌کنیم. این مرحله توصیه می‌شود، اما اختیاری است.

!!! important

    این workflow نمونه از دو job جداگانه (`build` و `publish`) استفاده می‌کند تا مرحلهٔ publish
    (که از طریق `id-token: write` به یک publishing credential دسترسی دارد) مجوزهای خود را با مرحلهٔ
    build به اشتراک نگذارد. این کار سطح حملهٔ مربوط به supply chain را کاهش می‌دهد.

ابتدا، یک release workflow به project خود اضافه کنید:

```yaml title=".github/workflows/release.yml"
name: "Publish release to PyPI"

on:
  push:
    tags:
      # Publish on version tags, e.g. v0.1.0
      - "v[0-9]+.[0-9]+.[0-9]+"
      - "v[0-9]+.[0-9]+.[0-9]+rc[0-9]+"
      - "v[0-9]+.[0-9]+.[0-9]+[ab][0-9]+"

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false

      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
        with:
          enable-cache: false

      - name: Build
        run: uv build

      # Optional, but recommended: run smoke tests on distributions
      - name: Smoke test (wheel)
        run: uv run --isolated --no-project --with dist/*.whl tests/smoke_test.py
      - name: Smoke test (source distribution)
        run: uv run --isolated --no-project --with dist/*.tar.gz tests/smoke_test.py

      - name: Upload distributions as artifacts
        uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7.0.1
        with:
          name: dist
          path: dist/

  publish:
    needs:
      - build
    runs-on: ubuntu-latest
    environment:
      name: pypi
    permissions:
      id-token: write
    steps:
      - name: Install uv
        uses: astral-sh/setup-uv@c771a70e6277c0a99b617c7a806ffedaca235ff9 # v9.0.0
        with:
          enable-cache: false

      - name: Download distributions artifact
        uses: actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8.0.1
        with:
          name: dist
          path: dist/

      - name: Generate PEP 740 attestations
        uses: astral-sh/attest-action@39b6ce43875c196a579e31635b11aa2a2036384a # v0.0.7

      - name: Publish
        run: uv publish
```

سپس، environment تعریف‌شده در workflow را در مخزن GitHub، در بخش «Settings» -> «Environments»
ایجاد کنید.

![کادر محاوره‌ای تنظیمات GitHub که نشان می‌دهد چگونه environment مربوط به «pypi» را در بخش «Settings» -> «Environments» اضافه کنید](../../assets/github-add-environment.png)

یک [Trusted Publisher](https://docs.pypi.org/trusted-publishers/adding-a-publisher/) به project
مربوط به PyPI خود در تنظیمات project، در بخش «Publishing» اضافه کنید. مطمئن شوید همهٔ فیلدها با
پیکربندی GitHub شما مطابقت دارند.

![کادر محاوره‌ای تنظیمات انتشار project مربوط به PyPI که نشان می‌دهد چگونه همهٔ فیلدها را برای یک پیکربندی trusted publisher تنظیم کنید](../../assets/pypi-add-trusted-publisher.png)

پس از ذخیره:

![کادر محاوره‌ای تنظیمات انتشار project مربوط به PyPI که تنظیمات پیکربندی‌شدهٔ trusted publishing را نشان می‌دهد](../../assets/pypi-with-trusted-publisher.png)

در پایان، یک release را tag کنید و آن را push کنید. مطمئن شوید با `v` شروع می‌شود تا با الگوی
موجود در workflow مطابقت داشته باشد.

```console
$ git tag -a v0.1.0 -m v0.1.0
$ git push --tags
```
