# credentialهای HTTP {#http-credentials}

uv هنگام درخواست از registryهای package، از credentialها روی HTTP پشتیبانی می‌کند.

احراز هویت می‌تواند از منبع‌های زیر، به ترتیب اولویت، تأمین شود:

- URL، برای نمونه `https://<user>:<password>@<hostname>/...`
- یک فایل پیکربندی [netrc](#netrc-files)
- credentials store در uv
- یک [keyring provider](#keyring-providers) (به‌صورت پیش‌فرض غیرفعال)

احراز هویت ممکن است برای hostهایی که در بسترهای زیر مشخص شده‌اند استفاده شود:

- `[index]`
- `index-url`
- `extra-index-url`
- `find-links`
- `package @ https://...`

## فایل‌های netrc {#netrc-files}

فایل‌های [`.netrc`](https://everything.curl.dev/usingcurl/netrc.html) یک قالب متنی سادهٔ قدیمی
برای ذخیرهٔ credentialها در یک سیستم هستند.

خواندن credentialها از فایل‌های `.netrc` همیشه فعال است. uv اگر environment variable به نام `NETRC`
تعریف شده باشد، از مسیر فایل موجود در آن استفاده می‌کند و در غیر این صورت از `~/.netrc` استفاده می‌کند.

## مخزن credentialها در uv {#the-uv-credentials-store}

uv می‌تواند credentialها را با استفاده از دستورهای [`uv auth`](./cli.md) از یک مخزن بخواند و در آن بنویسد.

credentialها در یک فایل متنی ساده در پوشهٔ state مربوط به uv ذخیره می‌شوند؛ برای نمونه،
`~/.local/share/uv/credentials/credentials.toml` در Unix. در حال حاضر قرار نیست این فایل به‌صورت
دستی ویرایش شود.

!!! note

    یک مکانیزم ذخیره‌سازی امن و بومیِ سیستم در [پیش‌نمایش](../preview.md) است — هنوز
    آزمایشی است و به‌طور فعال در حال توسعه است. در آینده، این مکانیزم به مکانیزم ذخیره‌سازی
    پیش‌فرض تبدیل خواهد شد.

    وقتی فعال باشد، uv از مکانیزم ذخیره‌سازی secret بومیِ سیستم عامل شما استفاده می‌کند. در
    macOS، از Keychain Services استفاده می‌کند. در Windows، از Windows Credential Manager استفاده می‌کند. در
    Linux، از Secret Service API مبتنی بر DBus استفاده می‌کند.

    در حال حاضر، uv فقط در مخزن بومی به‌دنبال credentialهایی می‌گردد که خودش به مخزن رمزها اضافه کرده است —
    credentialهایی را که برنامه‌های دیگر ذخیره کرده‌اند بازیابی نمی‌کند.

    برای استفاده از این مکانیزم ذخیره‌سازی، `UV_PREVIEW_FEATURES=native-auth` را تنظیم کنید.

## keyring providerها {#keyring-providers}

یک keyring provider مفهومی از `pip` است که امکان بازیابی credentialها از یک رابط
مطابق با package محبوب [keyring](https://github.com/jaraco/keyring) در Python را فراهم می‌کند.

keyring provider از نوع «subprocess» دستور `keyring` را فراخوانی می‌کند تا credentialها را دریافت کند. uv در
حال حاضر از انواع دیگر keyring provider پشتیبانی نمی‌کند.

برای استفاده از provider، `--keyring-provider subprocess` ،`UV_KEYRING_PROVIDER=subprocess` یا
`tool.uv.keyring-provider = "subprocess"` را تنظیم کنید.

## پایداری credentialها {#persistence-of-credentials}

اگر احراز هویت برای یک index URL واحد یا یک net location واحد (scheme، host و port) یافت شود، برای
مدت اجرای دستور در cache نگه داشته می‌شود و برای درخواست‌های دیگر به آن index یا net location استفاده می‌شود.
احراز هویت در فراخوانی‌های مختلف uv در cache نگه داشته نمی‌شود.

هنگام استفاده از `uv add`، uv credentialهای index را در `pyproject.toml` یا `uv.lock` ذخیره _نمی‌کند_.
این فایل‌ها اغلب در کنترل نسخه و توزیع‌ها گنجانده می‌شوند، بنابراین به‌طور کلی گنجاندن credentialها در
آن‌ها ناامن است. با این حال، uv credentialها را برای URLهای مستقیم ذخیره _می‌کند_؛ یعنی
`package @ https://username:password:example.com/foo.whl`، چون در حال حاضر راه دیگری برای
ارائهٔ آن credentialها وجود ندارد.

اگر هنگام `uv add` credentialها به یک index URL متصل شده بودند، ممکن است uv در عملیات بعدی نتواند dependencyها را از
indexهایی دریافت کند که به احراز هویت نیاز دارند. برای جزئیات دربارهٔ احراز هویت پایدار
برای indexها، به
[مستندات احراز هویت index](../indexes.md#authentication) مراجعه کنید.

## بیشتر بدانید {#learn-more}

برای جزئیات دربارهٔ احراز هویت index URLها، به [مستندات احراز هویت index](../indexes.md#authentication) مراجعه کنید.

برای جزئیات دربارهٔ تفاوت‌ها با `pip`، به [راهنمای سازگاری `pip`](../../pip/compatibility.md#registry-authentication) مراجعه کنید.
