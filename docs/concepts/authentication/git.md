# credentialهای Git {#git-credentials}

uv امکان نصب packageها از مخزن‌های خصوصی Git را با استفاده از احراز هویت SSH یا HTTP فراهم می‌کند.

## احراز هویت SSH {#ssh-authentication}

برای احراز هویت با استفاده از یک کلید SSH، از پروتکل `ssh://` استفاده کنید:

- `git+ssh://git@<hostname>/...` (برای نمونه، `git+ssh://git@github.com/astral-sh/uv`)
- `git+ssh://git@<host>/...` (برای نمونه، `git+ssh://git@github.com-key-2/astral-sh/uv`)

احراز هویت SSH نیازمند استفاده از نام کاربری `git` است.

برای جزئیات بیشتر دربارهٔ نحوهٔ پیکربندی SSH، به
[مستندات SSH در GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)
مراجعه کنید.

### احراز هویت HTTP {#http-authentication}

برای احراز هویت روی احراز هویت Basic در HTTP با استفاده از یک رمز عبور یا token:

- `git+https://<user>:<token>@<hostname>/...` (برای نمونه،
  `git+https://git:github_pat_asdf@github.com/astral-sh/uv`)
- `git+https://<token>@<hostname>/...` (برای نمونه، `git+https://github_pat_asdf@github.com/astral-sh/uv`)
- `git+https://<user>@<hostname>/...` (برای نمونه، `git+https://git@github.com/astral-sh/uv`)

!!! note

    هنگام استفاده از یک personal access token در GitHub، نام کاربری دلخواه است. GitHub به شما اجازه نمی‌دهد
    از نام حساب و رمز عبور خود در URLهایی مانند این استفاده کنید، هرچند hostهای دیگر ممکن است این کار را مجاز بدانند.

اگر در URL هیچ credentialای وجود نداشت و احراز هویت لازم بود،
[credential helper در Git](#git-credential-helpers) مورد پرس‌وجو قرار می‌گیرد.

## پایداری credentialها {#persistence-of-credentials}

هنگام استفاده از `uv add`، uv credentialهای Git را در `pyproject.toml` یا `uv.lock` ذخیره _نمی‌کند_.
این فایل‌ها اغلب در کنترل نسخه و توزیع‌ها گنجانده می‌شوند، بنابراین به‌طور کلی گنجاندن credentialها در
آن‌ها ناامن است.

اگر یک credential helper در Git پیکربندی کرده باشید، ممکن است credentialهای شما به‌صورت خودکار ذخیره شوند
و در نتیجه دریافت‌های بعدی آن dependency موفق شوند. با این حال، اگر credential helper در Git ندارید یا
project روی ماشینی استفاده می‌شود که credentialها در آن ذخیره نشده‌اند، uv در دریافت dependency ناموفق خواهد بود.

شما _می‌توانید_ با ارسال گزینهٔ `--raw` به `uv add`، uv را وادار کنید credentialهای Git را ذخیره کند. با این حال، ما
به‌شدت توصیه می‌کنیم به‌جای آن یک [credential helper](#git-credential-helpers) تنظیم کنید.

## credential helperهای Git {#git-credential-helpers}

credential helperهای Git برای ذخیره و بازیابی credentialهای Git استفاده می‌شوند. برای اطلاعات بیشتر، به
[مستندات Git](https://git-scm.com/doc/credential-helpers) مراجعه کنید.

اگر از GitHub استفاده می‌کنید، ساده‌ترین راه برای تنظیم یک credential helper این است که
[CLI ابزار `gh`](https://github.com/cli/cli#installation) را نصب کنید و از این دستور استفاده کنید:

```console
$ gh auth login
```

برای جزئیات بیشتر، به مستندات [`gh auth login`](https://cli.github.com/manual/gh_auth_login) مراجعه کنید.

!!! note

    هنگام استفادهٔ تعاملی از `gh auth login`، credential helper به‌صورت خودکار پیکربندی می‌شود.
    اما هنگام استفاده از `gh auth login --with-token`، مانند راهنمای
    [GitHub Actions در uv](../../guides/integration/github.md#private-repos)،
    دستور [`gh auth setup-git`](https://cli.github.com/manual/gh_auth_setup-git) پس از آن باید
    اجرا شود تا credential helper پیکربندی شود.
