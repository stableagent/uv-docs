# سازگاری با `pip` و `pip-tools` {#compatibility-with-pip-and-pip-tools}

uv به‌عنوان یک جایگزین مستقیم برای گردش‌کارهای رایج `pip` و `pip-tools` طراحی شده است.

به‌طور غیررسمی، هدف این است که کاربران فعلی `pip` و `pip-tools` بتوانند بدون ایجاد تغییرات معنادار در گردش‌کارهای packaging خود به uv مهاجرت کنند؛ و در بیشتر موارد، جایگزین کردن `pip install` با `uv pip install` باید «فقط کار کند».

با این حال، uv قرار _نیست_ یک کپی _دقیق_ از `pip` باشد، و هرچه از گردش‌کارهای رایج `pip` بیشتر فاصله بگیرید، احتمال اینکه با تفاوت‌هایی در رفتار روبه‌رو شوید بیشتر می‌شود. در برخی موارد، آن تفاوت‌ها ممکن است شناخته‌شده و عمدی باشند؛ در موارد دیگر، ممکن است نتیجهٔ جزئیات پیاده‌سازی باشند؛ و در موارد دیگر، ممکن است باگ باشند.

این سند تفاوت‌های شناخته‌شدهٔ بین uv و `pip` را همراه با دلیل، راه‌حل‌های جایگزین و بیانیه‌ای دربارهٔ قصد سازگاری در آینده شرح می‌دهد.

## فایل‌های پیکربندی و environment variable‌ها {#configuration-files-and-environment-variables}

uv فایل‌های پیکربندی یا environment variableهایی را که مخصوص `pip` هستند، مانند `pip.conf` یا `PIP_INDEX_URL`، نمی‌خواند.

خواندن فایل‌های پیکربندی و environment variableهایی که برای toolهای دیگر در نظر گرفته شده‌اند، معایب زیادی دارد:

1. نیازمند سازگاری باگ‌به‌باگ با tool هدف است، چون کاربران در نهایت به باگ‌های موجود در قالب، parser و غیره وابسته می‌شوند.
2. اگر tool هدف قالب را به نحوی _تغییر دهد_، آنگاه uv نیز ملزم به تغییر آن به شیوه‌های معادل می‌شود.
3. اگر آن پیکربندی به نحوی نسخه‌بندی شده باشد، uv باید بداند کاربر انتظار دارد از _کدام نسخهٔ_ tool هدف استفاده کند.
4. مانع از آن می‌شود که uv تنظیمات یا پیکربندی‌هایی را معرفی کند که در tool هدف وجود ندارند، چون در غیر این صورت `pip.conf` (یا موارد مشابه) دیگر با `pip` قابل استفاده نخواهد بود.
5. می‌تواند به سردرگمی کاربر منجر شود، چون uv تنظیماتی را می‌خواند که در واقع بر رفتار آن تأثیری ندارند، و بسیاری از کاربران ممکن است انتظار _نداشته باشند_ که uv فایل‌های پیکربندیِ در نظر گرفته‌شده برای toolهای دیگر را بخواند.

در عوض، uv از environment variableهای خودش پشتیبانی می‌کند، مانند `UV_INDEX_URL`. uv همچنین از پیکربندی پایدار در یک فایل `uv.toml` یا در بخش `[tool.uv.pip]` در فایل `pyproject.toml` پشتیبانی می‌کند. برای اطلاعات بیشتر، به [فایل‌های پیکربندی](../concepts/configuration-files.md) مراجعه کنید.

## سازگاری pre-release‌ها {#pre-release-compatibility}

به‌صورت پیش‌فرض (`if-necessary`)، uv نسخه‌های پایدار را به pre-releaseها ترجیح می‌دهد و فقط زمانی به pre-releaseها بازمی‌گردد که همهٔ کاندیداهای پایداری که constraintهای فعال را برآورده می‌کنند، در جریان resolution رد شوند.

برای مثال، فرض کنید فقط `c==1.0` و `c==2.0a1` در دسترس هستند. با هم، `c>=1` و `a -> c>=0.5a1` هر دو نسخه را مجاز می‌دانند، اما uv بدون توجه به اینکه کدام requirement اول کشف شود، `c==1.0` را انتخاب می‌کند. اگر requirement فعال دیگری `c==1.0` را رد کند، uv به `c==2.0a1` بازمی‌گردد.

برای در نظر گرفتن pre-releaseها برای هر package بدون ترجیح دادن اول کاندیداهای پایدار، از `--prerelease allow` استفاده کنید، یا برای حذف کامل آن‌ها از `--prerelease disallow` استفاده کنید.

حالت `explicit` pre-releaseها را فقط برای requirementهای first-party که شامل یک شناسهٔ pre-release هستند در نظر می‌گیرد (با ترجیح نسخه‌های پایدار و بازگشت به pre-releaseها فقط در صورت لزوم)، در حالی که pre-releaseها را برای همهٔ packageهای دیگر ممنوع می‌کند.

!!! note

    پیش از pip 26.0، این رفتار یکسان نبود.

مدل‌سازی pre-releaseها [به‌طور مشهوری دشوار](https://pubgrub-rs-guide.pages.dev/limitations/prerelease_versions) است، چون requirementهای dependency در جریان resolution به‌تدریج کشف می‌شوند. uv در حالی که کاندیداهای پایدار را پیش از pre-releaseها امتحان می‌کند، جهان کاندیداها را برای هر package ثابت نگه می‌دارد، بنابراین بازگشت به عقب می‌تواند بدون بی‌اعتبار کردن ناسازگاری‌های آموخته‌شدهٔ PubGrub به یک pre-release برسد.

## package‌هایی که در چند index وجود دارند {#packages-that-exist-on-multiple-indexes}

هم در uv و هم در `pip`، کاربران می‌توانند چند index مربوط به package را مشخص کنند تا نسخه‌های در دسترس یک package مشخص را از آن‌ها جست‌وجو کنند. با این حال، uv و `pip` در نحوهٔ برخورد با packageهایی که در چند index وجود دارند تفاوت دارند.

برای مثال، تصور کنید یک شرکت نسخهٔ داخلی `requests` را در یک index خصوصی (`--extra-index-url`) منتشر می‌کند، اما به‌صورت پیش‌فرض اجازهٔ نصب packageها از PyPI را نیز می‌دهد. در این حالت، `requests` خصوصی با [`requests`](https://pypi.org/project/requests/) عمومی در PyPI تداخل پیدا می‌کند.

وقتی uv یک package را در چند index جست‌وجو می‌کند، indexها را به ترتیب بررسی می‌کند (`--extra-index-url` را به index پیش‌فرض ترجیح می‌دهد) و به‌محض اینکه موردی مطابق پیدا کند، جست‌وجو را متوقف می‌کند. این یعنی اگر یک package در چند index وجود داشته باشد، uv نسخه‌های کاندیدای آن را به موارد موجود در اولین indexای که شامل package است محدود می‌کند.

در همین حال، `pip` نسخه‌های کاندیدا را از همهٔ indexها ترکیب می‌کند و بهترین نسخه را از مجموعهٔ ترکیبی انتخاب می‌کند، هرچند دربارهٔ ترتیبی که indexها را جست‌وجو می‌کند [هیچ تضمینی نمی‌دهد](https://github.com/pypa/pip/issues/5045#issuecomment-369521345) و انتظار دارد که packageها تا حد نام و نسخه یکتا باشند، حتی در indexهای مختلف.

رفتار uv به‌گونه‌ای است که اگر یک package در یک index داخلی وجود داشته باشد، همیشه باید از همان index داخلی نصب شود و هرگز از PyPI نصب نشود. هدف، جلوگیری از حمله‌های «dependency confusion» است که در آن‌ها مهاجم یک package مخرب را با همان نام یک package داخلی در PyPI منتشر می‌کند و به این ترتیب باعث می‌شود package مخرب به‌جای package داخلی نصب شود. برای مثال، به [حملهٔ `torchtriton`](https://pytorch.org/blog/compromised-nightly-dependency/) از دسامبر ۲۰۲۲ مراجعه کنید.

از نسخهٔ v0.1.39، کاربران می‌توانند با گزینهٔ خط فرمان `--index-strategy` یا environment variable به نام `UV_INDEX_STRATEGY` رفتار به سبک `pip` را برای چند index انتخاب کنند؛ این گزینه مقادیر زیر را پشتیبانی می‌کند:

- `first-index` (پیش‌فرض): جست‌وجوی هر package در همهٔ indexها، با محدود کردن نسخه‌های کاندیدا به موارد موجود در اولین indexای که شامل package است و با اولویت دادن به indexهای `--extra-index-url` نسبت به URL پیش‌فرض index.
- `unsafe-first-match`: جست‌وجوی هر package در همهٔ indexها، اما با ترجیح اولین indexای که نسخهٔ سازگار دارد، حتی اگر نسخه‌های جدیدتری در indexهای دیگر موجود باشند.
- `unsafe-best-match`: جست‌وجوی هر package در همهٔ indexها و انتخاب بهترین نسخه از مجموعهٔ ترکیبی نسخه‌های کاندیدا.

اگرچه `unsafe-best-match` نزدیک‌ترین رفتار به `pip` است، کاربران را در معرض خطر حمله‌های «dependency confusion» قرار می‌دهد.

uv همچنین از پین کردن packageها به indexهای اختصاصی پشتیبانی می‌کند (به [_Indexها_](../concepts/indexes.md#pinning-a-package-to-an-index) مراجعه کنید)، به‌گونه‌ای که یک package مشخص _همیشه_ از یک index خاص نصب می‌شود.

## ایزولاسیون build در PEP 517 {#pep-517-build-isolation}

uv به‌صورت پیش‌فرض از ایزولاسیون build مطابق [PEP 517](https://peps.python.org/pep-0517/) استفاده می‌کند (مشابه `pip install --use-pep517`)، با پیروی از `pypa/build` و با پیش‌بینی اینکه `pip` در آینده به‌صورت پیش‌فرض به buildهای PEP 517 روی آورد ([pypa/pip#9175](https://github.com/pypa/pip/issues/9175)).

اگر نصب یک package به دلیل نبود یک dependency زمان build ناموفق بود، سعی کنید از نسخهٔ جدیدتری از package استفاده کنید؛ اگر مشکل ادامه داشت، با maintainer package یک issue ثبت کنید و از او بخواهید تنظیمات packaging را به‌روز کند تا dependencyهای صحیح زمان build در PEP 517 اعلام شوند.

به‌عنوان یک راه فرار، می‌توانید dependencyهای build یک package را از پیش نصب کنید و سپس `uv pip install` را با `--no-build-isolation` اجرا کنید، مانند:

```shell
uv pip install wheel && uv pip install --no-build-isolation biopython==1.77
```

برای فهرست packageهایی که مشخص است در ایزولاسیون build در PEP 517 ناموفق می‌شوند، به [#2252](https://github.com/astral-sh/uv/issues/2252) مراجعه کنید.

## dependency‌های URL انتقالی {#transitive-url-dependencies}

اگرچه uv پشتیبانی درجه‌یک از dependencyهای URL دارد (مثلاً `ruff @ https://...`)، در برخورد با dependencyهای URL _انتقالی_ در دو جنبه با pip تفاوت دارد.

اول اینکه، uv فرض می‌کند dependencyهای غیر URL، dependencyهای URL را وارد resolution نمی‌کنند. به عبارت دیگر، فرض می‌کند dependencyهایی که از یک registry دریافت می‌شوند، خودشان به URLها وابسته نیستند. اگر یک dependency غیر URL یک dependency URL را _معرفی کند_، uv در جریان resolution آن dependency URL را رد می‌کند. (توجه کنید که PyPI اجازه نمی‌دهد packageهای منتشرشده به dependencyهای URL وابسته باشند؛ registryهای دیگر ممکن است سهل‌گیرتر باشند.)

دوم اینکه، اگر یک constraint (`--constraint`) یا override (`--override`) با استفاده از یک dependency مستقیم URL تعریف شود و package محدودشده خودش یک dependency مستقیم URL داشته باشد، uv _ممکن است_ آن dependency مستقیم URL انتقالی را در جریان resolution رد کند، اگر URL در جای دیگری از مجموعهٔ requirementهای ورودی ارجاع داده نشده باشد.

اگر uv یک dependency URL انتقالی را رد کند، بهترین کار این است که dependency URL را به‌عنوان یک dependency مستقیم در فایل `pyproject.toml` یا `requirement.in` مربوطه ارائه دهید، چون constraintهای بالا برای dependencyهای مستقیم اعمال نمی‌شوند.

## virtual environment‌ها به‌صورت پیش‌فرض {#virtual-environments-by-default}

`uv pip install` و `uv pip sync` طوری طراحی شده‌اند که به‌صورت پیش‌فرض با virtual environmentها کار کنند.

به‌طور مشخص، uv همیشه packageها را در virtual environment فعال فعلی نصب می‌کند، یا به دنبال یک virtual environment به نام `.venv` در پوشهٔ فعلی یا هر پوشهٔ والد می‌گردد (حتی اگر فعال نشده باشد).

این با `pip` تفاوت دارد؛ `pip` اگر هیچ virtual environmentای فعال نباشد، packageها را در یک environment سراسری نصب می‌کند و به دنبال virtual environmentهای غیرفعال نمی‌گردد.

در uv، می‌توانید با ارائهٔ یک مسیر به یک فایل اجرایی Python از طریق گزینهٔ `--python /path/to/python`، یا با فلگ `--system` که در اولین interpreter Python یافت‌شده در `PATH` نصب می‌کند، مانند `pip`، در environmentهای غیرمجازی نصب کنید.

به عبارت دیگر، uv پیش‌فرض را برعکس می‌کند و برای نصب در Python سیستم نیازمند انتخاب صریح است؛ کاری که می‌تواند به خرابی‌ها و پیچیدگی‌های دیگر منجر شود و فقط باید در شرایط محدود انجام شود.

برای اطلاعات بیشتر، به [«استفاده از environment‌های دلخواه Python»](./environments.md#using-arbitrary-python-environments) مراجعه کنید.

## استراتژی resolution {#resolution-strategy}

برای یک مجموعهٔ مشخص از specifierهای dependency، اغلب هیچ مجموعهٔ «صحیح» یکتایی از packageها برای نصب وجود ندارد. در عوض، مجموعه‌های معتبر زیادی از packageها وجود دارند که specifierها را برآورده می‌کنند.

نه `pip` و نه uv هیچ تضمینی دربارهٔ مجموعهٔ _دقیق_ packageهایی که نصب خواهند شد نمی‌دهند؛ فقط تضمین می‌کنند که resolution سازگار، قطعی و مطابق با specifierها خواهد بود. به همین دلیل، در برخی موارد `pip` و uv resolutionهای متفاوتی می‌دهند؛ با این حال، هر دو resolution _باید_ به یک اندازه معتبر باشند.

برای مثال، در نظر بگیرید:

```requirements title="requirements.in"
starlette
fastapi
```

در زمان نوشتن، جدیدترین نسخهٔ `starlette` برابر `0.37.2` و جدیدترین نسخهٔ `fastapi` برابر `0.110.0` است. با این حال، `fastapi==0.110.0` به `starlette` نیز وابسته است و یک کران بالا معرفی می‌کند: `starlette>=0.36.3,<0.37.0`.

اگر یک resolver در اولویت قرار دادن جدیدترین نسخهٔ `starlette` را در نظر بگیرد، باید از نسخهٔ قدیمی‌تری از `fastapi` استفاده کند که کران بالا روی `starlette` را حذف می‌کند. در عمل، این کار نیازمند بازگشت به `fastapi==0.1.17` است:

```requirements title="requirements.txt"
# This file was autogenerated by uv via the following command:
#    uv pip compile requirements.in
annotated-types==0.6.0
    # via pydantic
anyio==4.3.0
    # via starlette
fastapi==0.1.17
idna==3.6
    # via anyio
pydantic==2.6.3
    # via fastapi
pydantic-core==2.16.3
    # via pydantic
sniffio==1.3.1
    # via anyio
starlette==0.37.2
    # via fastapi
typing-extensions==4.10.0
    # via
    #   pydantic
    #   pydantic-core
```

در مقابل، اگر یک resolver جدیدترین نسخهٔ `fastapi` را در اولویت قرار دهد، باید از نسخهٔ قدیمی‌تری از `starlette` استفاده کند که کران بالا را برآورده کند. در عمل، این کار نیازمند بازگشت به `starlette==0.36.3` است:

```requirements title="requirements.txt"
# This file was autogenerated by uv via the following command:
#    uv pip compile requirements.in
annotated-types==0.6.0
    # via pydantic
anyio==4.3.0
    # via starlette
fastapi==0.110.0
idna==3.6
    # via anyio
pydantic==2.6.3
    # via fastapi
pydantic-core==2.16.3
    # via pydantic
sniffio==1.3.1
    # via anyio
starlette==0.36.3
    # via fastapi
typing-extensions==4.10.0
    # via
    #   fastapi
    #   pydantic
    #   pydantic-core
```

وقتی resolutionهای uv به شیوه‌ای نامطلوب با `pip` تفاوت دارند، اغلب نشانهٔ این است که specifierها بیش از حد آزاد هستند و کاربر باید سفت کردن آن‌ها را در نظر بگیرد. برای مثال، در مورد `starlette` و `fastapi`، کاربر می‌تواند `fastapi>=0.110.0` را الزامی کند.

## `pip check` {#pip-check}

در حال حاضر، `uv pip check` تشخیص‌های زیر را نشان می‌دهد:

- یک package فایل `METADATA` ندارد، یا فایل `METADATA` قابل تجزیه نیست.
- یک package یک کران پایین `Requires-Python` دارد که Python version مربوط به interpreter در حال اجرا را حذف می‌کند. کران‌های بالا [نادیده گرفته می‌شوند](#requires-python-upper-bounds)، همان‌طور که در جریان resolution نادیده گرفته می‌شوند.
- یک package به packageای وابسته است که نصب نشده است.
- یک package به packageای وابسته است که نصب شده است، اما در نسخه‌ای ناسازگار.
- چند نسخه از یک package در virtual environment نصب شده‌اند.

در برخی موارد، `uv pip check` تشخیص‌هایی را نشان می‌دهد که `pip check` نشان نمی‌دهد، و برعکس. برای مثال، برخلاف `uv pip check`، `pip check` وقتی چند نسخه از یک package در environment فعلی نصب شده باشند، هشدار _نمی‌دهد_.

## `--user` و scheme نصب `user` {#user-and-the-user-install-scheme}

uv از فلگ `--user` پشتیبانی نمی‌کند؛ فلگی که packageها را بر اساس scheme نصب `user` نصب می‌کند. در عوض، توصیه می‌کنیم برای جداسازی نصب packageها از virtual environmentها استفاده کنید.

علاوه بر این، pip اگر تشخیص دهد کاربر مجوز نوشتن در پوشهٔ هدف را ندارد، به scheme نصب `user` بازمی‌گردد، همان‌طور که در برخی سیستم‌ها هنگام نصب در Python سیستم اتفاق می‌افتد. uv چنین بازگشتی را پیاده‌سازی نمی‌کند.

برای اطلاعات بیشتر، به [#2077](https://github.com/astral-sh/uv/issues/2077) مراجعه کنید.

## اعمال `--only-binary` {#only-binary-enforcement}

از آرگومان `--only-binary` برای محدود کردن نصب به distributionهای باینری از پیش ساخته‌شده استفاده می‌شود. وقتی `--only-binary :all:` داده شود، هم pip و هم uv از build کردن distributionهای منبع از PyPI و registryهای دیگر خودداری می‌کنند.

با این حال، وقتی یک dependency به‌عنوان یک URL مستقیم ارائه شود (مثلاً `uv pip install https://...`)، pip گزینهٔ `--only-binary` را اعمال _نمی‌کند_ و distributionهای منبع را برای همهٔ چنین packageهایی build می‌کند.

در همین حال، uv گزینهٔ `--only-binary` را برای dependencyهای مستقیم URL اعمال _می‌کند_، با یک استثنا: با توجه به `uv pip install https://... --only-binary flask`، اگر uv نتواند نام package را از پیش استنباط کند، distribution منبع در URL داده‌شده را build _خواهد کرد_، چون uv نمی‌تواند بدون build کردن metadata آن تعیین کند که آیا package در چنین مواردی «مجاز» است یا نه.

هم pip و هم uv اجازه می‌دهند requirementهای قابل ویرایش حتی وقتی `--only-binary` داده شده است، build و نصب شوند. برای مثال، `uv pip install -e . --only-binary :all:` مجاز است.

## اعمال `--no-binary` {#no-binary-enforcement}

از آرگومان `--no-binary` برای محدود کردن نصب به distributionهای منبع استفاده می‌شود. وقتی `--no-binary` داده شود، uv از نصب distributionهای باینری از پیش ساخته‌شده خودداری می‌کند، اما distributionهای باینری‌ای را که از قبل در cache محلی موجود هستند _دوباره استفاده می‌کند_.

علاوه بر این، و برخلاف pip، resolver در uv وقتی `--no-binary` داده شود، همچنان metadata را از distributionهای باینری از پیش ساخته‌شده می‌خواند.

## اعمال `manylinux_compatible` {#manylinuxcompatible-enforcement}

[PEP 600](https://peps.python.org/pep-0600/#package-installers) سازوکاری را توصیف می‌کند که از طریق آن توزیع‌کنندگان Python می‌توانند با تعریف یک تابع `manylinux_compatible` روی module کتابخانهٔ استاندارد `_manylinux`، از سازگاری `manylinux` خارج شوند.

uv به `manylinux_compatible` احترام می‌گذارد، اما فقط در برابر نسخهٔ فعلی glibc آزمایش می‌کند و مقدار بازگشتی `manylinux_compatible` را به‌صورت سراسری اعمال می‌کند.

به عبارت دیگر، اگر `manylinux_compatible` مقدار `True` برگرداند، uv سیستم را سازگار با `manylinux` در نظر می‌گیرد؛ اگر مقدار `False` برگرداند، uv سیستم را ناسازگار با `manylinux` در نظر می‌گیرد، بدون اینکه `manylinux_compatible` را برای هر نسخهٔ glibc فراخوانی کند.

این رویکرد پیاده‌سازی کاملی از spec نیست، اما با پیاده‌سازی‌های رایج و کلی `manylinux_compatible` مانند [`no-manylinux`](https://pypi.org/project/no-manylinux/) سازگار است:

```python
from __future__ import annotations

manylinux1_compatible = False
manylinux2010_compatible = False
manylinux2014_compatible = False


def manylinux_compatible(*_, **__):  # PEP 600
    return False
```

## کامپایل بایت‌کد {#bytecode-compilation}

برخلاف `pip`، uv به‌صورت پیش‌فرض هنگام نصب فایل‌های `.py` را به فایل‌های `.pyc` کامپایل نمی‌کند (یعنی uv پوشه‌های `__pycache__` را ایجاد یا پر نمی‌کند). برای فعال کردن کامپایل بایت‌کد هنگام نصب، فلگ `--compile-bytecode` را به `uv pip install` یا `uv pip sync` بدهید، یا environment variable به نام `UV_COMPILE_BYTECODE` را روی `1` تنظیم کنید.

نادیده گرفتن کامپایل بایت‌کد می‌تواند در گردش‌کارها نامطلوب باشد؛ برای مثال، توصیه می‌کنیم کامپایل بایت‌کد را در [buildهای Docker](../guides/integration/docker.md) فعال کنید تا زمان راه‌اندازی بهبود یابد (به قیمت افزایش زمان build).

از آنجا که کامپایل بایت‌کد هشدارهای مختلف صادرشده توسط interpreter Python را سرکوب می‌کند، در موارد نادر ممکن است هنگام اجرای کد Pythonای که با uv نصب شده است، پیام‌های `SyntaxWarning` یا `DeprecationWarning` ببینید که هنگام استفاده از `pip` ظاهر نمی‌شوند. این‌ها هشدارهای معتبری هستند، اما معمولاً در فرایند کامپایل بایت‌کد پنهان می‌مانند و می‌توان آن‌ها را نادیده گرفت، در بالادست اصلاح کرد یا با فعال کردن کامپایل بایت‌کد در uv به همین ترتیب سرکوب کرد.

## سخت‌گیری و اعمال spec {#strictness-and-spec-enforcement}

uv معمولاً سخت‌گیرتر از `pip` است و اغلب packageهایی را که `pip` نصب می‌کند رد می‌کند. برای مثال، uv indexهای HTML با fragmentهای نامعتبر URL را رد می‌کند (به [PEP 503](https://peps.python.org/pep-0503/) مراجعه کنید)، در حالی که `pip` چنین fragmentهایی را نادیده می‌گیرد.

در برخی موارد، uv برای packageهای محبوبی که مشخص است مشکلات خاصی در تطابق با spec دارند، رفتار سهل‌گیرانه‌ای پیاده می‌کند.

اگر uv یک package را که `pip` به دلیل نقض spec نصب می‌کند رد کرد، بهترین کار این است که اول سعی کنید نسخهٔ جدیدتری از package را نصب کنید؛ و اگر این کار ناموفق بود، مشکل را به maintainer package گزارش دهید.

## گزینه‌ها و subcommandهای خط فرمان `pip` {#pip-command-line-options-and-subcommands}

uv از مجموعهٔ کامل گزینه‌های خط فرمان و subcommandهای `pip` پشتیبانی نمی‌کند، هرچند از زیرمجموعهٔ بزرگی از آن‌ها پشتیبانی می‌کند.

گزینه‌ها و subcommandهای گمشده بر اساس تقاضای کاربر و پیچیدگی پیاده‌سازی اولویت‌بندی می‌شوند و معمولاً در issueهای جداگانه پیگیری می‌شوند. برای مثال:

- [`--trusted-host`](https://github.com/astral-sh/uv/issues/1339)
- [`--user`](https://github.com/astral-sh/uv/issues/2077)

اگر با یک گزینه یا subcommand گمشده روبه‌رو شدید، لطفاً در issue tracker جست‌وجو کنید تا ببینید آیا قبلاً گزارش شده است یا نه، و اگر نشده است، ثبت یک issue جدید را در نظر بگیرید. برای ابراز علاقهٔ خود، می‌توانید به issueهای موجود رأی مثبت بدهید.

## احراز هویت registry {#registry-authentication}

uv از گزینه‌های `auto` یا `import` در `pip` برای `--keyring-provider` پشتیبانی نمی‌کند. در حال حاضر، فقط گزینهٔ `subprocess` پشتیبانی می‌شود.

برخلاف `pip`، uv احراز هویت keyring را به‌صورت پیش‌فرض فعال نمی‌کند.

uv می‌تواند از credentialهایی استفاده کند که همراه یک درخواست ارائه شده‌اند یا برای URL آن پیش از اینکه سرور یک HTTP 401 برگرداند، cache شده‌اند. برای قواعد حاکم بر جست‌وجوی credential، به [credential providerها](../concepts/indexes.md#using-credential-providers) مراجعه کنید.

## پشتیبانی از `egg` {#egg-support}

uv از قابلیت‌هایی که در `pip` قدیمی یا منسوخ محسوب می‌شوند پشتیبانی نمی‌کند. برای مثال، uv از distributionهای به سبک `.egg` پشتیبانی نمی‌کند.

با این حال، uv تا حدی از (۱) distributionهای به سبک `.egg-info` (که گاهی در imageهای Docker و environmentهای Conda یافت می‌شوند) و (۲) distributionهای قدیمی و قابل ویرایش به سبک `.egg-link` پشتیبانی می‌کند.

به‌طور مشخص، uv از نصب distributionهای جدید به سبک `.egg-info` یا `.egg-link` پشتیبانی نمی‌کند، اما در جریان resolution به distributionهای موجود از این دست احترام می‌گذارد، آن‌ها را با `uv pip list` و `uv pip freeze` فهرست می‌کند و با `uv pip uninstall` حذف نصب می‌کند.

## build constraint‌ها {#build-constraints}

وقتی constraintها از طریق `--constraint` (یا `UV_CONSTRAINT`) ارائه شوند، uv هنگام حل dependencyهای build (یعنی برای build کردن یک distribution منبع)، constraintها را اعمال _نمی‌کند_. در عوض، build constraintها باید از طریق تنظیم اختصاصی `--build-constraint` (یا `UV_BUILD_CONSTRAINT`) ارائه شوند.

در همین حال، pip وقتی constraintها از طریق `PIP_CONSTRAINT` مشخص شده باشند، آن‌ها را برای dependencyهای build اعمال می‌کند، اما وقتی از طریق `--constraint` در خط فرمان ارائه شده باشند، اعمال نمی‌کند.

برای مثال، برای اینکه مطمئن شوید `setuptools 60.0.0` برای build کردن هر packageای که dependency build روی `setuptools` دارد استفاده می‌شود، به‌جای `--constraint` از `--build-constraint` استفاده کنید.

## پیش‌فرض‌های `pip compile` {#pip-compile-defaults}

چند تفاوت کوچک اما قابل‌توجه در رفتارهای پیش‌فرض `pip compile` و `pip-tools` وجود دارد.

به‌صورت پیش‌فرض، uv requirementهای کامپایل‌شده را در یک فایل خروجی نمی‌نویسد. در عوض، uv نیاز دارد که کاربر یک فایل خروجی را به‌صورت صریح با گزینهٔ `-o` یا `--output-file` مشخص کند.

به‌صورت پیش‌فرض، uv هنگام خروجی دادن requirementهای کامپایل‌شده، extraها را حذف می‌کند. به عبارت دیگر، uv به‌صورت پیش‌فرض از `--strip-extras` استفاده می‌کند، در حالی که `pip-compile` به‌صورت پیش‌فرض از `--no-strip-extras` استفاده می‌کند. قرار است `pip-compile` این پیش‌فرض را در نسخهٔ اصلی بعدی (v8.0.0) تغییر دهد و در آن زمان هر دو tool به‌صورت پیش‌فرض از `--strip-extras` استفاده خواهند کرد. برای حفظ extraها با uv، فلگ `--no-strip-extras` را به `uv pip compile` بدهید.

به‌صورت پیش‌فرض، uv هیچ URL مربوط به index را در فایل خروجی نمی‌نویسد، در حالی که `pip-compile` هر `--index-url` یا `--extra-index-url` را که با پیش‌فرض (PyPI) مطابقت نداشته باشد، خروجی می‌دهد. برای گنجاندن URLهای index در فایل خروجی، فلگ `--emit-index-url` را به `uv pip compile` بدهید. برخلاف `pip-compile`، وقتی `--emit-index-url` داده شود، uv همهٔ URLهای index را شامل می‌شود، از جمله URL پیش‌فرض index.

## کران‌های بالای `requires-python` {#requires-python-upper-bounds}

هنگام ارزیابی بازه‌های `requires-python` برای dependencyها، uv فقط کران‌های پایین را در نظر می‌گیرد و کران‌های بالا را کاملاً نادیده می‌گیرد. برای مثال، `>=3.8, <4` به‌عنوان `>=3.8` در نظر گرفته می‌شود. رعایت کران‌های بالا روی `requires-python` اغلب به resolutionهایی منجر می‌شود که از نظر صوری درست اما از نظر عملی نادرست هستند، چون مثلاً resolverها به اولین نسخهٔ منتشرشده‌ای بازمی‌گردند که کران بالا را حذف کرده است (به [محدودیت‌های بالای `Requires-Python`](https://discuss.python.org/t/requires-python-upper-limits/12663) مراجعه کنید).

## specifierهای `requires-python` {#requires-python-specifiers}

هنگام ارزیابی Python versionها در برابر specifierهای `requires-python`، uv نسخهٔ کاندیدا را تا اجزای major و minor و patch کوتاه می‌کند و شناسه‌های pre-release و post-release را نادیده می‌گیرد.

برای مثال، یک project که `requires-python: >=3.13` را اعلام می‌کند، Python 3.13.0b1 را می‌پذیرد. اگرچه 3.13.0b1 از نظر دقیق بزرگ‌تر از 3.13 نیست، وقتی شناسهٔ pre-release حذف شود، بزرگ‌تر از 3.13 است.

اگرچه این کار دقیقاً با [PEP 440](https://peps.python.org/pep-0440/) مطابقت ندارد، با [pip](https://github.com/pypa/pip/blob/24.1.1/src/pip/_internal/resolution/resolvelib/candidates.py#L540) سازگار _است_.

## اولویت package {#package-priority}

معمولاً با توجه به مجموعه‌ای از requirementها، راه‌حل‌های ممکن زیادی وجود دارد و یک resolver باید بین آن‌ها انتخاب کند. resolver در uv و resolver در pip مجموعهٔ متفاوتی از اولویت‌های package دارند. اگرچه هر دو resolver ترتیب ارائه‌شده توسط کاربر را به‌عنوان یکی از اولویت‌های خود استفاده می‌کنند، pip [اولویت‌های](https://pip.pypa.io/en/stable/topics/more-dependency-resolution/#the-resolver-algorithm) بیشتری دارد که uv ندارد. بنابراین، uv بیشتر از pip تحت تأثیر تغییر در ترتیب کاربر قرار می‌گیرد.

برای مثال، `uv pip install foo bar` نسخه‌های جدیدتر `foo` را نسبت به `bar` در اولویت قرار می‌دهد و می‌تواند به resolution متفاوتی نسبت به `uv pip install bar foo` منجر شود. به همین ترتیب، این رفتار برای ترتیب requirementها در فایل‌های ورودی `uv pip compile` نیز صدق می‌کند.

## اعتبارسنجی نام فایل wheel و metadata {#wheel-filename-and-metadata-validation}

به‌صورت پیش‌فرض، uv wheelهایی را رد می‌کند که نام فایل آن‌ها با metadata داخل فایل ناسازگار است. برای مثال، یک wheel به نام `foo-1.0.0-py3-none-any.whl` که metadataای شامل نسخهٔ `1.0.1` دارد، توسط uv رد می‌شود، اما توسط pip پذیرفته می‌شود.

برای اینکه uv را مجبور کنید چنین wheelهایی را بپذیرد، مقدار `UV_SKIP_WHEEL_FILENAME_CHECK=1` را در environment تنظیم کنید.

## نرمال‌سازی نام package {#package-name-normalization}

به‌صورت پیش‌فرض، uv نام packageها را نرمال می‌کند تا با [شکل‌های مطابق PEP 503](https://packaging.python.org/en/latest/specifications/name-normalization/#name-normalization) آن‌ها مطابقت داشته باشند و از آن نام‌های نرمال‌شده در همهٔ زمینه‌های خروجی استفاده می‌کند. این با pip تفاوت دارد؛ pip تمایل دارد نام package را دقیقاً همان‌طور که در registry منتشر شده است حفظ کند.

برای مثال، `uv pip list` نام‌های نرمال‌شدهٔ packageها را نمایش می‌دهد (مثلاً `docstring-parser`)، در حالی که `pip list` نام‌های نرمال‌نشدهٔ packageها را نمایش می‌دهد (مثلاً `docstring_parser`):

```shell
(venv) $ diff --side-by-side  <(pip list) <(uv pip list)
Package          Version					Package          Version
---------------- -------					---------------- -------
docstring_parser 0.16					      |	docstring-parser 0.16
jaraco.classes   3.4.0					      |	jaraco-classes   3.4.0
more-itertools   10.7.0				    		more-itertools   10.7.0
pip              25.1					    	pip              25.1
PyMuPDFb         1.24.10				      |	pymupdfb         1.24.10
PyPDF2           3.0.1					      |	pypdf2           3.0.1
```
