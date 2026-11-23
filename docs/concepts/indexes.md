# ایندکس‌های package {#package-indexes}

به‌طور پیش‌فرض، uv برای resolution وابستگی‌ها و نصب package از [Python Package Index (PyPI)](https://pypi.org) استفاده می‌کند. با این حال، می‌توان uv را طوری پیکربندی کرد که از indexهای package دیگر، از جمله indexهای خصوصی، استفاده کند؛ این کار از طریق گزینهٔ پیکربندی `[[tool.uv.index]]` (و `--index`، گزینهٔ خط فرمان متناظر با آن) انجام می‌شود.

## تعریف یک index {#defining-an-index}

برای گنجاندن یک index اضافی هنگام resolution وابستگی‌ها، یک ورودی `[[tool.uv.index]]` به فایل `pyproject.toml` خود اضافه کنید:

```toml
[[tool.uv.index]]
# Optional name for the index.
name = "pytorch"
# Required URL for the index.
url = "https://download.pytorch.org/whl/cpu"
```

indexها به ترتیبی که تعریف شده‌اند اولویت‌بندی می‌شوند، به‌طوری که اولین index فهرست‌شده در فایل پیکربندی، اولین indexای است که هنگام resolution وابستگی‌ها مورد مراجعه قرار می‌گیرد و indexهایی که از طریق خط فرمان ارائه می‌شوند بر indexهای موجود در فایل پیکربندی اولویت دارند.

به‌طور پیش‌فرض، uv فهرست Python Package Index (PyPI) را به‌عنوان index «پیش‌فرض» شامل می‌شود، یعنی indexای که وقتی یک package در هیچ index دیگری پیدا نشد استفاده می‌شود. برای حذف PyPI از فهرست indexها، روی یک ورودی index دیگر `default = true` را تنظیم کنید (یا از گزینهٔ خط فرمان `--default-index` استفاده کنید):

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
default = true
```

index پیش‌فرض همیشه کمترین اولویت را دارد، صرف‌نظر از موقعیت آن در فهرست indexها.

نام‌های index فقط می‌توانند شامل کاراکترهای الفبایی-عددی، خط‌تیره، زیرخط و نقطه باشند و باید ASCII معتبر باشند.

هنگام ارائهٔ یک index در خط فرمان (با `--index` یا `--default-index`) یا از طریق یک environment variable (با `UV_INDEX` یا `UV_DEFAULT_INDEX`)، از URL آن، یک نام پیکربندی‌شده، یا سینتکس `<name>=<url>` استفاده کنید:

```shell
# On the command line.
$ uv lock --index pytorch=https://download.pytorch.org/whl/cpu
# Via an environment variable.
$ UV_INDEX=pytorch=https://download.pytorch.org/whl/cpu uv lock
```

با `--preview-features index-by-name`، نام‌های index پیکربندی‌شده بر مسیرهای منطبق اولویت دارند.

## ایندکس‌های proxy {#proxy-indexes}

!!! note

    ایندکس‌های proxy در وضعیت [پیش‌نمایش](./preview.md) هستند و ممکن است در انتشارهای آینده تغییر کنند.
    برای بی‌صدا کردن هشدار پیش‌نمایش، `--preview-features proxy-index` را پاس دهید یا
    `preview-features = ["proxy-index"]` را در پیکربندی uv خود تنظیم کنید.

ایندکس‌های proxy، resolution و دانلود package را از طریق یک registry جایگزین هدایت می‌کنند، بدون اینکه index بالادستی و URLهای دانلود package ثبت‌شده در lockfileها را تغییر دهند.

این کار زمانی مفید است که یک سیاست محلی از برخی مشارکت‌کنندگان بخواهد از یک registry استفاده کنند که packageهای تازه منتشرشده را حذف می‌کند، در حالی که دیگران مستقیماً از index بالادستی استفاده می‌کنند. پیکربندی رابطهٔ بین proxy و index بالادستی به هر دو گروه اجازه می‌دهد بدون تغییر URLهای آن، یک lockfile مشترک داشته باشند.

برای پیکربندی یک proxy index، یک ورودی `[[tool.uv.proxy-index]]` با مقدار `proxy-for` برابر با نام index بالادستی اضافه کنید. مقدار `url` مربوط به proxy به uv می‌گوید packageها را کجا جستجو کند و مقدار `artifact-base-url` آن به uv می‌گوید آن‌ها را از کجا دانلود کند. در فایل `uv.toml`، به‌جای آن از `[[proxy-index]]` استفاده کنید.

برای استفاده از یک proxy مربوط به PyPI در چند پروژه، هر دو ورودی را به فایل [`uv.toml`](./configuration-files.md) سطح کاربر خود اضافه کنید:

```toml title="uv.toml"
[[index]]
name = "pypi"
url = "https://pypi.org/simple/"
default = true

[[proxy-index]]
name = "pypi-proxy"
url = "https://proxy.example.com/simple/"
artifact-base-url = "https://proxy.example.com/files/"
proxy-for = "pypi"
```

!!! note

    مقدار پیش‌فرض `artifact-base-url` مربوط به PyPI برابر با `https://files.pythonhosted.org/packages/` است.

هنگام دانلود یک فایل ثبت‌شده در `uv.lock`، uv بدون اینکه آن را در فهرست package مربوط به proxy جستجو کند، URL آن را بازنویسی می‌کند تا از proxy استفاده شود. پنهان کردن فایل از فهرست، دانلود را مسدود نمی‌کند: خود proxy باید دانلود را رد کند. uv همچنان می‌تواند از یک نسخهٔ cacheشده استفاده کند.

هنگام نصب از `pylock.toml`، uv فقط در صورتی می‌تواند از proxy استفاده کند که فیلد `packages.index` index بالادستی مربوط به آن package را مشخص کند. بدون این فیلد، uv مستقیماً از URL ثبت‌شدهٔ artifact دانلود می‌کند. دستور `uv pip compile --format pylock.toml` هنگامی که یک فایل را از یک index دارای proxy انتخاب می‌کند، URL بالادستی را در `packages.index` ثبت می‌کند.

برای proxy کردن یک index متفاوت، به index بالادستی یک `name` و یک `artifact-base-url` بدهید:

```toml title="uv.toml"
[[index]]
name = "private"
url = "https://packages.example.com/simple/"
artifact-base-url = "https://downloads.example.com/packages/"

[[proxy-index]]
name = "private-proxy"
url = "https://proxy.example.com/simple/"
artifact-base-url = "https://proxy.example.com/files/"
proxy-for = "private"
```

یک index حداکثر می‌تواند یک proxy داشته باشد. فایل‌های package باید پس از `artifact-base-url` هر index مسیر یکسانی داشته باشند. برای مثال، `https://downloads.example.com/packages/example/package.whl` به `https://proxy.example.com/files/example/package.whl` نگاشت می‌شود. proxyهایی که مسیر فایل‌های package را تغییر می‌دهند پشتیبانی نمی‌شوند. uv هنگام قفل کردن (locking) مسیر بالادستی را بررسی نمی‌کند، بنابراین یک proxy که آن را تغییر دهد می‌تواند یک lockfile تولید کند که فقط از طریق proxy کار می‌کند. برای بررسی، یک نصب frozen را بدون proxy در یک environment تازه با یک cache خالی امتحان کنید.

uv برای هر artifact دارای proxy که در `uv.lock` یا `pylock.toml` می‌نویسد، یک hash پشتیبانی‌شده لازم دارد، از جمله هنگام خروجی گرفتن از یک `uv.lock` موجود. دستور `uv sync --frozen` می‌تواند یک artifact دارای proxy را نصب کند اگر `uv.lock` موجود برای آن hash نداشته باشد، اما وقتی hash وجود داشته باشد، آن را اعتبارسنجی می‌کند.

با فعال بودن پیش‌نمایش `wheel-variants`، uv فرادادهٔ `*-variants.json` را از URL تبلیغ‌شده توسط proxy دریافت می‌کند. نگاشت `artifact-base-url` برای این فراداده اعمال نمی‌شود.

اگر دانلودهای artifact به احراز هویت در یک هاست متفاوت نیاز دارند، credentialها را روی `artifact-base-url` پیکربندی کنید.

## پین کردن یک package به یک index {#pinning-a-package-to-an-index}

می‌توان یک package را با مشخص کردن index در ورودی `tool.uv.sources` آن، به یک index مشخص پین کرد. برای پین کردن یک package به یک index دارای proxy، از نام index بالادستی استفاده کنید. برای مثال، برای اطمینان از اینکه `torch` _همیشه_ از index مربوط به `pytorch` نصب می‌شود، موارد زیر را به فایل `pyproject.toml` خود اضافه کنید:

```toml
[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
```

به همین ترتیب، برای دریافت از یک index متفاوت بر اساس platform، می‌توانید فهرستی از sourceها را ارائه دهید که با environment markerها از هم متمایز شده‌اند:

```toml title="pyproject.toml"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "pytorch-cpu", marker = "sys_platform == 'darwin'"},
  { index = "pytorch-cu130", marker = "sys_platform != 'darwin'"},
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "pytorch-cu130"
url = "https://download.pytorch.org/whl/cu130"
```

می‌توان یک index را با `explicit = true` علامت‌گذاری کرد تا از نصب packageها از آن index جلوگیری شود، مگر اینکه صراحتاً به آن پین شده باشند. برای مثال، برای اطمینان از اینکه `torch` از index مربوط به `pytorch` نصب می‌شود اما همهٔ packageهای دیگر از PyPI نصب می‌شوند، موارد زیر را به فایل `pyproject.toml` خود اضافه کنید:

```toml
[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

indexهای نام‌دار که از طریق `tool.uv.sources` ارجاع داده می‌شوند باید در فایل `pyproject.toml` پروژه تعریف شده باشند؛ indexهایی که از طریق خط فرمان، environment variableها یا پیکربندی سطح کاربر ارائه می‌شوند شناخته نمی‌شوند.

اگر یک index هم با `default = true` و هم با `explicit = true` علامت‌گذاری شود، به‌عنوان یک index صریح در نظر گرفته می‌شود (یعنی فقط از طریق `tool.uv.sources` قابل استفاده است) و در عین حال PyPI را به‌عنوان index پیش‌فرض حذف می‌کند.

## جستجو در چند index {#searching-across-multiple-indexes}

به‌طور پیش‌فرض، uv در اولین indexای که یک package داده‌شده در آن موجود است توقف می‌کند و resolutionها را به موارد موجود در همان index اول محدود می‌کند (`first-index`).

برای مثال، اگر یک index داخلی از طریق `[[tool.uv.index]]` مشخص شود، رفتار uv به‌گونه‌ای است که اگر یک package در آن index داخلی وجود داشته باشد، _همیشه_ از همان index داخلی نصب می‌شود و هرگز از PyPI نصب نمی‌شود. هدف این است که از حملات «dependency confusion» جلوگیری شود، حملاتی که در آن‌ها یک مهاجم یک package مخرب را با همان نام یک package داخلی در PyPI منتشر می‌کند و در نتیجه باعث می‌شود package مخرب به‌جای package داخلی نصب شود. برای مثال، به [حملهٔ `torchtriton`](https://pytorch.org/blog/compromised-nightly-dependency/) از دسامبر ۲۰۲۲ مراجعه کنید.

برای انتخاب رفتارهای جایگزین index، از گزینهٔ خط فرمان `--index-strategy` یا environment variable مربوط به `UV_INDEX_STRATEGY` استفاده کنید که مقادیر زیر را پشتیبانی می‌کند:

- `first-index` (پیش‌فرض): هر package را در همهٔ indexها جستجو کنید و نسخه‌های نامزد را به موارد موجود در اولین indexای که شامل package است محدود کنید.
- `unsafe-first-match`: هر package را در همهٔ indexها جستجو کنید، اما اولین index با یک نسخهٔ سازگار را ترجیح دهید، حتی اگر نسخه‌های جدیدتری در indexهای دیگر موجود باشند.
- `unsafe-best-match`: هر package را در همهٔ indexها جستجو کنید و بهترین نسخه را از مجموعهٔ ترکیبی نسخه‌های نامزد انتخاب کنید.

با اینکه `unsafe-best-match` نزدیک‌ترین رفتار به رفتار pip است، کاربران را در معرض خطر حملات «dependency confusion» قرار می‌دهد.

## احراز هویت {#authentication}

بیشتر indexهای خصوصی package برای دسترسی به packageها به احراز هویت نیاز دارند، معمولاً از طریق نام کاربری و رمز عبور (یا توکن دسترسی).

!!! tip

    برای احراز هویت با ارائه‌دهنده‌های خاص index خصوصی، به راهنماهای اختصاصی مراجعه کنید:
    [Azure Artifacts](../guides/integration/azure.md),
    [Google Artifact Registry](../guides/integration/google.md),
    [AWS CodeArtifact](../guides/integration/aws.md) و
    [JFrog Artifactory](../guides/integration/jfrog.md).

### ارائهٔ مستقیم credentialها {#providing-credentials-directly}

می‌توان credentialها را مستقیماً از طریق environment variableها یا با جاسازی آن‌ها در URL ارائه داد.

برای مثال، با فرض یک index با نام `internal-proxy` که به نام کاربری (`public`) و رمز عبور (`koala`) نیاز دارد، index را (بدون credential) در فایل `pyproject.toml` خود تعریف کنید:

```toml
[[tool.uv.index]]
name = "internal-proxy"
url = "https://example.com/simple"
```

از آنجا می‌توانید environment variableهای `UV_INDEX_INTERNAL_PROXY_USERNAME` و
`UV_INDEX_INTERNAL_PROXY_PASSWORD` را تنظیم کنید، که در آن‌ها `INTERNAL_PROXY` نسخهٔ بزرگ‌شدهٔ
نام index است و کاراکترهای غیرالفبایی-عددی با زیرخط جایگزین شده‌اند:

```sh
export UV_INDEX_INTERNAL_PROXY_USERNAME=public
export UV_INDEX_INTERNAL_PROXY_PASSWORD=koala
```

با ارائهٔ credentialها از طریق environment variableها، می‌توانید از ذخیرهٔ اطلاعات حساس در فایل متنی سادهٔ `pyproject.toml` جلوگیری کنید.

به‌عنوان جایگزین، می‌توان credentialها را مستقیماً در تعریف index جاسازی کرد:

```toml
[[tool.uv.index]]
name = "internal"
url = "https://public:koala@pypi-proxy.corp.dev/simple"
```

به دلایل امنیتی، credentialها _هرگز_ در فایل `uv.lock` ذخیره نمی‌شوند؛ به همین دلیل، uv در زمان نصب _باید_ به URL احراز هویت‌شده دسترسی داشته باشد.

### استفاده از ارائه‌دهنده‌های credential {#using-credential-providers}

علاوه بر ارائهٔ مستقیم credentialها، uv از کشف credentialها از netrc و keyring پشتیبانی می‌کند. برای جزئیات دربارهٔ راه‌اندازی ارائه‌دهنده‌های خاص credential، به مستندات [احراز هویت HTTP](./authentication/http.md) مراجعه کنید.

به‌طور پیش‌فرض، uv پیش از پرس‌وجو از ارائه‌دهنده‌ها، یک درخواست بدون احراز هویت را امتحان می‌کند. اگر درخواست ناموفق باشد، uv به دنبال credentialها می‌گردد. اگر credentialها پیدا شوند، یک درخواست احراز هویت‌شده امتحان می‌شود.

!!! note

    اگر نام کاربری تنظیم شده باشد، uv پیش از ارسال یک درخواست بدون احراز هویت به دنبال credentialها می‌گردد.

برخی indexها (مثلاً GitLab) درخواست‌های بدون احراز هویت را به یک index عمومی، مانند PyPI، هدایت می‌کنند — به این معنی که uv به دنبال credentialها نمی‌گردد. می‌توان این رفتار را برای هر index با استفاده از تنظیم `authenticate` تغییر داد. برای مثال، برای اینکه همیشه به دنبال credentialها بگردد:

```toml hl_lines="4"
[[tool.uv.index]]
name = "example"
url = "https://example.com/simple"
authenticate = "always"
```

وقتی `authenticate` روی `always` تنظیم شود، uv مشتاقانه به دنبال credentialها می‌گردد و اگر credentialها پیدا نشوند خطا می‌دهد.

### نادیده گرفتن کدهای خطا {#ignoring-error-codes}

هنگام استفاده از [استراتژی first-index](#searching-across-multiple-indexes)، اگر کد وضعیت HTTP 401 Unauthorized یا HTTP 403 Forbidden دریافت شود، uv جستجو در indexها را متوقف می‌کند. تنها استثنا این است که uv هنگام جستجو در index مربوط به `pytorch` کدهای 403 را نادیده می‌گیرد (زیرا این index وقتی یک package وجود ندارد، کد 403 برمی‌گرداند).

به‌طور پیش‌فرض، uv در صورتی که هنگام دریافت metadata مربوط به distribution یا یک آرشیو از یک index با خطای HTTP مواجه شود، resolution را هم متوقف می‌کند. نادیده گرفتن آن خطا، نسخهٔ package آسیب‌دیده را به‌عنوان ناموجود علامت‌گذاری می‌کند و به resolver اجازه می‌دهد نسخهٔ دیگری را امتحان کند.

برای پیکربندی اینکه کدام کدهای خطا برای یک index نادیده گرفته شوند، از تنظیم `ignore-error-codes` استفاده کنید. برای مثال، برای نادیده گرفتن کدهای 403 (اما نه 401) برای یک index خصوصی:

```toml
[[tool.uv.index]]
name = "private-index"
url = "https://private-index.com/simple"
authenticate = "always"
ignore-error-codes = [403]
```

uv وقتی با خطای `404 Not Found` مواجه می‌شود، همیشه به جستجو در indexها ادامه می‌دهد. نمی‌توان این رفتار را تغییر داد.

### غیرفعال کردن احراز هویت {#disabling-authentication}

برای جلوگیری از نشت credentialها، می‌توان احراز هویت را برای یک index غیرفعال کرد:

```toml hl_lines="4"
[[tool.uv.index]]
name = "example"
url = "https://example.com/simple"
authenticate = "never"
```

وقتی `authenticate` روی `never` تنظیم شود، uv هرگز برای index داده‌شده به دنبال credentialها نمی‌گردد و اگر credentialها مستقیماً ارائه شوند، خطا می‌دهد.

### سفارشی‌سازی هدرهای کنترل cache {#customizing-cache-control-headers}

به‌طور پیش‌فرض، uv به هدرهای کنترل cache ارائه‌شده توسط index احترام می‌گذارد. برای مثال، PyPI فرادادهٔ package را با هدر `max-age=600` ارائه می‌دهد و به این ترتیب به uv اجازه می‌دهد فرادادهٔ package را به مدت ۱۰ دقیقه cache کند؛ و wheelها و source distributionها را با هدر `max-age=365000000, immutable` ارائه می‌دهد و به این ترتیب به uv اجازه می‌دهد artifactها را برای مدت نامحدود cache کند.

برای بازنویسی هدرهای کنترل cache برای یک index، از تنظیم `cache-control` استفاده کنید:

```toml
[[tool.uv.index]]
name = "example"
url = "https://example.com/simple"
cache-control = { api = "max-age=600", files = "max-age=365000000, immutable" }
```

تنظیم `cache-control` یک object با دو کلید اختیاری می‌پذیرد:

- `api`: کش کردن درخواست‌های Simple API (فرادادهٔ package) را کنترل می‌کند.
- `files`: کش کردن دانلودهای artifact (wheelها و source distributionها) را کنترل می‌کند.

مقادیر این کلیدها رشته‌هایی هستند که از سینتکس
[HTTP Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
پیروی می‌کنند. برای مثال، برای اینکه uv همیشه فرادادهٔ package را دوباره اعتبارسنجی کند، `api = "no-cache"` را تنظیم کنید:

```toml
[[tool.uv.index]]
name = "example"
url = "https://example.com/simple"
cache-control = { api = "no-cache" }
```

این تنظیم بیشتر برای بازنویسی هدرهای پیش‌فرض کنترل cache برای indexهای خصوصی استفاده می‌شود که در غیر این صورت کش کردن را غیرفعال می‌کنند، اغلب به‌صورت ناخواسته. ما معمولاً توصیه می‌کنیم از رویکرد PyPI برای هدرهای کش پیروی کنید، یعنی تنظیم `api = "max-age=600"` و `files = "max-age=365000000, immutable"`.

### الزام یک الگوریتم hash {#requiring-a-hash-algorithm}

وقتی یک index چند hash برای یک distribution تبلیغ می‌کند، uv یک hash واحد را برای ثبت در lockfile انتخاب می‌کند. برای الزام یک الگوریتم مشخص برای distributionهایی که از یک index resolve می‌شوند، از تنظیم `hash-algorithm` استفاده کنید:

```toml
[tool.uv]
preview-features = ["index-hash-algorithm"]

[[tool.uv.index]]
name = "private-index"
url = "https://private-index.com/simple"
hash-algorithm = "sha256"
```

اگر یک distribution قفل‌شده الگوریتم موردنیاز را تبلیغ نکند، uv به‌جای بازگشت به یک الگوریتم hash دیگر، با خطا مواجه می‌شود.

### پیکربندی `exclude-newer` برای یک index {#configuring-exclude-newer-for-an-index}

اگر از [`exclude-newer`](./resolution.md#reproducible-resolutions) استفاده می‌کنید، می‌توانید یک حد زمانی (cutoff) متفاوت برای یک index مشخص پیکربندی کنید:

```toml
[[tool.uv.index]]
name = "internal"
url = "https://internal.example.com/simple"
exclude-newer = "7 days"
```

مقادیر خاص index فقط بر packageهایی که از آن index ارائه می‌شوند تأثیر می‌گذارند. می‌توانید حد زمانی را برای packageهای منفرد در آن index با `exclude-newer-package` بازنویسی کنید:

```toml
[[tool.uv.index]]
name = "internal"
url = "https://internal.example.com/simple"
exclude-newer = "7 days"
exclude-newer-package = { example = false }
```

بازنویسی‌های خاص package در `[tool.uv.exclude-newer-package]` یا در خط فرمان بر بازنویسی‌های خاص package مربوط به index اولویت دارند.

اگر یک index فرادادهٔ `upload-time` ارائه ندهد، می‌توانید حد زمانی را برای آن index کاملاً غیرفعال کنید:

```toml
[[tool.uv.index]]
name = "internal"
url = "https://internal.example.com/simple"
exclude-newer = false
```

## ایندکس‌های «flat» {#flat-indexes}

به‌طور پیش‌فرض، فرض می‌شود ورودی‌های `[[tool.uv.index]]` رجیستری‌هایی به سبک PyPI هستند که [PEP 503](https://peps.python.org/pep-0503/) Simple Repository API را پیاده‌سازی می‌کنند. با این حال، uv از indexهای «flat» هم پشتیبانی می‌کند، که پوشه‌های محلی یا صفحات HTML هستند و فهرست‌های تخت از wheelها و source distributionها را در خود دارند. در pip، چنین indexهایی با استفاده از گزینهٔ `--find-links` مشخص می‌شوند.

برای تعریف یک flat index در فایل `pyproject.toml` خود، از گزینهٔ `format = "flat"` استفاده کنید:

```toml
[[tool.uv.index]]
name = "example"
url = "/path/to/directory"
format = "flat"
```

ایندکس‌های flat از همان مجموعهٔ ویژگی indexهای Simple Repository API پشتیبانی می‌کنند (مثلاً `explicit = true`)؛ همچنین می‌توانید با استفاده از `tool.uv.sources` یک package را به یک flat index پین کنید.

## `--index-url` و `--extra-index-url` {#index-url-and-extra-index-url}

علاوه بر گزینهٔ پیکربندی `[[tool.uv.index]]`، uv برای سازگاری از گزینه‌های خط فرمان `--index-url` و `--extra-index-url` به سبک pip پشتیبانی می‌کند، که در آن‌ها `--index-url` index پیش‌فرض را تعریف می‌کند و `--extra-index-url` indexهای اضافی را تعریف می‌کند.

می‌توان این گزینه‌ها را در ترکیب با گزینهٔ پیکربندی `[[tool.uv.index]]` استفاده کرد و آن‌ها از همان قواعد اولویت‌بندی پیروی می‌کنند:

- index پیش‌فرض همیشه کمترین اولویت را دارد، چه از طریق آرگومان قدیمی `--index-url` تعریف شده باشد، چه از طریق آرگومان توصیه‌شدهٔ `--default-index`، یا از طریق یک ورودی `[[tool.uv.index]]` با `default = true`.
- indexها به ترتیبی که تعریف شده‌اند مورد مراجعه قرار می‌گیرند، چه از طریق آرگومان قدیمی `--extra-index-url`، چه از طریق آرگومان توصیه‌شدهٔ `--index`، یا از طریق ورودی‌های `[[tool.uv.index]]`.

در عمل، می‌توان `--index-url` و `--extra-index-url` را ورودی‌های بی‌نام `[[tool.uv.index]]` در نظر گرفت، که برای اولی `default = true` فعال است. در این زمینه، `--index-url` به `--default-index` نگاشت می‌شود و `--extra-index-url` به `--index` نگاشت می‌شود.
