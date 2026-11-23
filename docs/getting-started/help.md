# دریافت کمک {#getting-help}

## منوهای راهنما {#help-menus}

از فلگ `--help` می‌توان برای مشاهدهٔ منوی راهنما برای یک دستور استفاده کرد؛ مثلاً برای `uv`:

```console
$ uv --help
```

برای مشاهدهٔ منوی راهنما برای یک دستور مشخص؛ مثلاً برای `uv init`:

```console
$ uv init --help
```

هنگام استفاده از فلگ `--help`، uv یک منوی راهنمای خلاصه نمایش می‌دهد. برای مشاهدهٔ یک منوی
راهنمای طولانی‌تر برای یک دستور، از `uv help` استفاده کنید:

```console
$ uv help
```

برای مشاهدهٔ منوی راهنمای طولانی برای یک دستور مشخص؛ مثلاً برای `uv init`:

```console
$ uv help init
```

هنگام استفاده از منوی راهنمای طولانی، uv تلاش می‌کند از `less` یا `more` برای صفحه‌بندی خروجی
استفاده کند تا همهٔ آن یک‌باره نمایش داده نشود. برای خروج از pager، `q` را فشار دهید.

## نمایش خروجی verbose {#displaying-verbose-output}

از فلگ `-v` می‌توان برای نمایش خروجی verbose برای یک دستور استفاده کرد؛ مثلاً برای `uv sync`:

```console
$ uv sync -v
```

فلگ `-v` را می‌توان برای افزایش verbosity تکرار کرد؛ مثلاً:

```console
$ uv sync -vv
```

اغلب، خروجی verbose شامل اطلاعات بیشتری دربارهٔ علت رفتار uv به شیوه‌ای خاص است.

## مشاهدهٔ version {#viewing-the-version}

هنگام درخواست کمک، مهم است که version ی uv را که استفاده می‌کنید مشخص کنید — گاهی مشکل در یک
version جدیدتر از قبل حل شده است.

برای بررسی version نصب‌شده:

```console
$ uv self version
```

موارد زیر هم معتبرند:

```console
$ uv --version      # Same output as `uv self version`
$ uv -V             # Will not include the build commit and date
```

!!! note

    پیش از uv 0.7.0، به‌جای `uv self version` از `uv version` استفاده می‌شد.

## عیب‌یابی مشکلات {#troubleshooting-issues}

مستندات مرجع شامل یک [راهنمای عیب‌یابی](../reference/troubleshooting/index.md) برای مشکلات رایج
است.

## ثبت issue در GitHub {#open-an-issue-on-github}

[ردیاب issue](https://github.com/astral-sh/uv/issues) در GitHub مکان خوبی برای گزارش باگ‌ها و
درخواست قابلیت‌هاست. اول مطمئن شوید که issue های مشابه را جستجو کرده‌اید، چون معمول است که شخص
دیگری هم با همان مشکل روبه‌رو شده باشد.

## گفتگو در Discord {#chat-on-discord}

Astral یک [سرور Discord](https://discord.com/invite/astral-sh) دارد که مکان خوبی برای پرسیدن
سؤال‌ها، یادگیری بیشتر دربارهٔ uv و تعامل با سایر اعضای جامعه است.
