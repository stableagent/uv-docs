# مثال‌های قابل تکرار {#reproducible-examples}

## چرا مثال‌های قابل تکرار مهم هستند {#why-reproducible-examples-are-important}

یک مثال حداقلی قابل تکرار (MRE) برای رفع باگ‌ها ضروری است. بدون مثالی که بتوان با آن مشکل را
بازتولید کرد، یک نگهدارنده نمی‌تواند آن را اشکال‌زدایی کند یا آزمایش کند که آیا رفع شده است یا نه.
اگر مثال حداقلی نباشد؛ یعنی اگر محتوای زیادی را شامل شود که به issue مربوط نیست، ممکن است یک
نگهدارنده برای شناسایی علت ریشه‌ای مشکل زمان بسیار بیشتری صرف کند.

## نحوهٔ نوشتن یک مثال قابل تکرار {#how-to-write-a-reproducible-example}

هنگام نوشتن یک مثال قابل تکرار، هدف ارائهٔ همهٔ زمینه‌ای است که برای بازتولید مثال شما توسط شخص
دیگری لازم است. این موارد شامل این‌هاست:

- platform‌ای که استفاده می‌کنید (مثلاً سیستم‌عامل و معماری)
- هر وضعیت مرتبط با سیستم (مثلاً environment variableهایی که صراحتاً تنظیم شده‌اند)
- نسخهٔ uv
- نسخهٔ toolهای مرتبط دیگر
- فایل‌های مرتبط (`uv.lock`، `pyproject.toml` و غیره)
- دستورهایی که باید اجرا شوند

برای اطمینان از اینکه بازتولید شما حداقلی است، تا حد ممکن dependencyها، تنظیمات و فایل‌ها را حذف
کنید. حتماً پیش از به اشتراک گذاشتن بازتولید خود، آن را آزمایش کنید. توصیه می‌کنیم logهای مفصل
(verbose) را از بازتولید خود ضمیمه کنید؛ ممکن است آن‌ها روی ماشین شما به شکلی حیاتی متفاوت باشند.
استفاده از یک [Gist](https://gist.github.com) برای logهای خیلی طولانی می‌تواند مفید باشد.

در ادامه، چند [استراتژی](#strategies-for-reproducible-examples) مشخص برای ساخت و به اشتراک گذاشتن
مثال‌های قابل تکرار را پوشش می‌دهیم.

!!! tip

    یک راهنمای عالی دربارهٔ اصول ساخت MRE در
    [Stack Overflow](https://stackoverflow.com/help/minimal-reproducible-example) وجود دارد.

## استراتژی‌هایی برای مثال‌های قابل تکرار {#strategies-for-reproducible-examples}

### ایمیج Docker {#docker-image}

نوشتن یک ایمیج Docker اغلب بهترین راه برای به اشتراک گذاشتن یک مثال قابل تکرار است، چون کاملاً
خودکفاست. این یعنی وضعیت سیستم شخص بازتولیدکننده روی مشکل تأثیری ندارد.

!!! note

    استفاده از یک ایمیج Docker فقط وقتی امکان‌پذیر است که issue روی Linux قابل بازتولید باشد.
    هنگام استفاده از macOS، ابتدا بررسی کنید که آیا issue روی Linux هم رخ می‌دهد یا نه. بعضی
    باگ‌ها مخصوص سیستم‌عامل هستند. با اینکه استفاده از Docker برای اجرای containerهای Windows
    امکان‌پذیر است، رایج نیست. انتظار می‌رود این نوع باگ‌ها به‌جای آن به‌صورت یک
    [script](#script) گزارش شوند.

هنگام نوشتن یک MRE با Docker و uv، بهتر است با یکی از
[ایمیج‌های Docker مربوط به uv](../../guides/integration/docker.md#available-images) شروع کنید.
هنگام انجام این کار، حتماً به یک نسخهٔ مشخص از uv پین کنید.

```Dockerfile
FROM ghcr.io/astral-sh/uv:0.13.9-debian-slim
```

با اینکه ایمیج‌های Docker از سیستم جدا هستند، build به‌طور پیش‌فرض از معماری سیستم شما استفاده
می‌کند. هنگام به اشتراک گذاشتن یک بازتولید، می‌توانید platform را صراحتاً تنظیم کنید تا مطمئن
شوید بازتولیدکننده رفتار مورد انتظار را می‌گیرد. uv ایمیج‌هایی را برای `linux/amd64` (مثلاً Intel
یا AMD) و `linux/arm64` (مثلاً Apple M Series یا ARM) منتشر می‌کند.

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.13.9-debian-slim
```

ایمیج‌های Docker برای بازتولید issueهایی که می‌توان با دستورها ساخت، بهترین‌اند؛ مثلاً:

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.13.9-debian-slim

RUN uv init /mre
WORKDIR /mre
RUN uv add pydantic
RUN uv sync
RUN uv run -v python -c "import pydantic"
```

با این حال، می‌توانید فایل‌ها را به‌صورت درون‌خطی هم در ایمیج بنویسید:

```Dockerfile
FROM --platform=linux/amd64 ghcr.io/astral-sh/uv:0.13.9-debian-slim

COPY <<EOF /mre/pyproject.toml
[project]
name = "example"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["pydantic"]
EOF

WORKDIR /mre
RUN uv lock
```

اگر نیاز دارید فایل‌های زیادی بنویسید، بهتر است یک
[مخزن Git](#git-repository) بسازید و منتشر کنید. می‌توانید این رویکردها را ترکیب کنید و یک
`Dockerfile` را در مخزن بگنجانید.

هنگام به اشتراک گذاشتن یک بازتولید Docker، ضمیمه کردن logهای build مفید است. می‌توانید با
غیرفعال کردن caching و خروجی فانتزی، خروجی بیشتری از گام‌های build ببینید:

```console
docker build . --progress plain --no-cache
```

### script {#script}

هنگام گزارش باگ‌های خاص platform که نمی‌توان آن‌ها را در یک [container](#docker-image) بازتولید
کرد، بهترین کار این است که یک script شامل دستورهایی که می‌توان با آن‌ها باگ را بازتولید کرد،
ضمیمه کنید؛ مثلاً:

```bash
uv init
uv add pydantic
uv sync
uv run -v python -c "import pydantic"
```

اگر بازتولید شما به فایل‌های زیادی نیاز دارد، برای به اشتراک گذاشتن آن‌ها از یک
[مخزن Git](#git-repository) استفاده کنید.

علاوه بر script، logهای _مفصل_ (verbose) (یعنی با فلگ `-v`) مربوط به خطا و پیام کامل خطا را
ضمیمه کنید.

هرگاه یک script به وضعیت خارجی وابسته باشد، حتماً آن اطلاعات را به اشتراک بگذارید. برای مثال،
اگر script را روی Windows نوشته‌اید و از یک Python version استفاده می‌کند که با `choco` نصب
کرده‌اید و روی PowerShell 6.2 اجرا می‌شود، لطفاً آن را در گزارش بگنجانید.

### مخزن Git {#git-repository}

هنگام به اشتراک گذاشتن یک بازتولید به‌صورت مخزن Git، یک [script](#script) را ضمیمه کنید که مشکل
را بازتولید می‌کند یا، حتی بهتر، یک [Dockerfile](#docker-image). گام اول script باید clone کردن
مخزن و checkout کردن یک commit مشخص باشد:

```console
$ git clone https://github.com/<user>/<project>.git
$ cd <project>
$ git checkout <commit>
$ <commands to produce error>
```

می‌توانید به‌سرعت یک مخزن جدید در [رابط کاربری GitHub](https://github.com/new) یا با `gh` در CLI
بسازید:

```console
$ gh repo create uv-mre-1234 --clone
```

هنگام استفاده از یک مخزن Git برای بازتولید، لطفاً به خاطر داشته باشید که محتوا را با حذف فایل‌ها
یا تنظیماتی که برای بازتولید مشکل شما لازم نیستند، _حداقلی_ کنید.
