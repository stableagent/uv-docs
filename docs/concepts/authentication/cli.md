# CLI ابزار `uv auth` {#the-uv-auth-cli}

uv یک رابط سطح بالا برای ذخیره و بازیابی credentialها از سرویس‌ها ارائه می‌دهد.

## ورود به یک سرویس {#logging-in-to-a-service}

برای افزودن credentialهای یک سرویس، از دستور `uv auth login` استفاده کنید:

```console
$ uv auth login example.com
```

با این کار، از شما credentialها خواسته می‌شود.

credentialها را می‌توان با گزینه‌های `--username` و `--password` یا با گزینهٔ
`--token` برای سرویس‌هایی که از `__token__` یا یک نام کاربری دلخواه استفاده می‌کنند نیز ارائه کرد.

!!! note

    توصیه می‌کنیم رمز محرمانه را از طریق stdin ارائه کنید. از `-` استفاده کنید تا نشان دهید مقدار باید از
    stdin خوانده شود؛ برای نمونه، برای `--password`:

    ```console
    $ echo 'my-password' | uv auth login example.com --password -
    ```

    همین الگو را می‌توان با `--token` نیز به کار برد.

پس از افزوده‌شدن credentialها، uv از آن‌ها برای عملیات packaging استفاده می‌کند که نیازمند دریافت محتوا
از سرویس موردنظر است. در حال حاضر، فقط احراز هویت Basic در HTTPS پشتیبانی می‌شود. credentialها
هنوز برای درخواست‌های Git استفاده نمی‌شوند.

!!! note

    uv credentialها را اعتبارسنجی نمی‌کند، بنابراین دستور حتی اگر آن‌ها نادرست باشند هم موفق می‌شود.

## خروج از یک سرویس {#logging-out-of-a-service}

برای حذف credentialها، از دستور `uv auth logout` استفاده کنید:

```console
$ uv auth logout example.com
```

!!! note

    credentialها از حافظهٔ محلی حذف می‌شوند، اما در سرور راه دور باطل نمی‌شوند.

## نمایش credentialهای یک سرویس {#showing-credentials-for-a-service}

برای نمایش credential ذخیره‌شده برای یک URL مشخص، از دستور `uv auth token` استفاده کنید:

```console
$ uv auth token example.com
```

اگر برای ورود از نام کاربری استفاده شده بود، آن را نیز باید ارائه کنید؛ برای نمونه:

```console
$ uv auth token --username foo example.com
```

## استفاده از credentialها با ابزارهای خارجی {#using-credentials-with-external-tools}

`uv auth helper` به ابزارهایی که از credential helperها پشتیبانی می‌کنند اجازه می‌دهد credentialهای HTTP را از uv درخواست کنند.
در حال حاضر، uv از
[پروتکل credential helper در Bazel](https://github.com/bazelbuild/proposals/blob/main/designs/2022-06-07-bazel-credential-helpers.md) پشتیبانی می‌کند.

این دستور برای آن است که ابزارهای خارجی آن را فراخوانی کنند. یک درخواست JSON را از stdin می‌خواند و
یک پاسخ JSON را در stdout می‌نویسد. وقتی credentialهای مطابق در دسترس باشند، پاسخ شامل
هدر `Authorization` است:

```console
$ echo '{"uri": "https://example.com/path"}' | uv --preview-features auth-helper auth helper --protocol=bazel get
{"headers":{"Authorization":["Basic ..."]}}
```

اگر credentialای یافت نشود، uv مجموعه‌ای خالی از هدرها برمی‌گرداند:

```json
{ "headers": {} }
```

!!! note

    `uv auth helper` آزمایشی است. برای غیرفعال‌کردن هشدار، از `--preview-features auth-helper` یا
    `UV_PREVIEW_FEATURES=auth-helper` استفاده کنید.

[راهنمای یکپارچه‌سازی Bazel](../../guides/integration/bazel.md) توضیح می‌دهد چگونه از این دستور
با Bazel استفاده کنید.

## پیکربندی backend ذخیره‌سازی {#configuring-the-storage-backend}

credentialها در [مخزن credentialها در uv](./http.md#the-uv-credentials-store) ذخیره می‌شوند.

به‌صورت پیش‌فرض، credentialها در یک فایل متنی ساده نوشته می‌شوند. یک backend ذخیره‌سازی رمزگذاری‌شدهٔ بومیِ سیستم را
می‌توان با `UV_PREVIEW_FEATURES=native-auth` فعال کرد.
