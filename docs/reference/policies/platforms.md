# پشتیبانی از platform {#platform-support}

uv از platformهای زیر پشتیبانی Tier 1 دارد:

- macOS (Apple Silicon)
- macOS (x86_64)
- Linux (x86_64)
- Windows (x86_64)

uv به‌طور مداوم در برابر platformهای Tier 1 خود ساخته، آزمایش و توسعه داده می‌شود. با الهام از
پروژهٔ Rust، Tier 1 را می‌توان
["تضمین‌شده برای کار کردن"](https://doc.rust-lang.org/beta/rustc/platform-support.html#tier-1)
دانست.

uv از platformهای زیر پشتیبانی Tier 2 دارد
(["تضمین‌شده برای ساخته شدن"](https://doc.rust-lang.org/beta/rustc/platform-support.html#tier-2-with-host-tools)):

- Linux (PPC64LE)
- Linux (RISC-V64)
- Linux (aarch64)
- Linux (armv7)
- Linux (i686)
- Linux (s390x)
- Windows (arm64)

uv از platformهای زیر پشتیبانی Tier 3 دارد
(["بهترین تلاش"](https://doc.rust-lang.org/beta/rustc/platform-support.html#tier-3)):

- FreeBSD (x86_64)
- Windows (i686)

uv باینری‌های رسمی را در GitHub و wheelهای از پیش ساخته‌شده را در [PyPI](https://pypi.org/project/uv/)
برای platformهای Tier 1 و Tier 2 خود ارائه می‌دهد.

platformهای Tier 2 به‌طور مداوم ساخته می‌شوند، اما مجموعهٔ آزمایش uv روی آن‌ها اجرا نمی‌شود و
پایداری آن‌ها در عمل ممکن است متفاوت باشد.

platformهای Tier 3 ممکن است ساخته یا آزمایش نشوند، اما uv patchهایی را برای رفع باگ‌ها می‌پذیرد.

## نسخه‌های Linux {#linux-versions}

در Linux، سازگاری بر اساس نسخهٔ libc تعیین می‌شود.

uv هم distributionهای مبتنی بر glibc و هم distributionهای مبتنی بر musl را منتشر می‌کند.

برای distributionهای Linux مبتنی بر glibc، uv
[wheelهای سازگار با manylinux](https://peps.python.org/pep-0600/) و باینری‌های متناظر را منتشر
می‌کند. این artifactها به در دسترس بودن glibc روی سیستم میزبان وابسته‌اند. در یک wheel tag مربوط
به manylinux، نسخه حداقل نسخهٔ پشتیبانی‌شدهٔ glibc برای آن wheel را مشخص می‌کند؛ برای مثال،
`manylinux_2_17_x86_64` به glibc نسخهٔ 2.17+ نیاز دارد.

wheelها و باینری‌های رسمی uv مبتنی بر glibc برای targetهای زیر منتشر می‌شوند:

- `x86_64-unknown-linux-gnu` (`manylinux_2_17_x86_64`)
- `aarch64-unknown-linux-gnu` (`manylinux_2_28_aarch64`)
- `armv7-unknown-linux-gnueabihf` (`manylinux_2_17_armv7l`)
- `i686-unknown-linux-gnu` (`manylinux_2_17_i686`)
- `powerpc64le-unknown-linux-gnu` (`manylinux_2_17_ppc64le`)
- `riscv64gc-unknown-linux-gnu` (`manylinux_2_31_riscv64`)
- `s390x-unknown-linux-gnu` (`manylinux_2_17_s390x`)

uv همچنین wheelهای مبتنی بر musl و باینری‌های کاملاً به‌صورت ایستا لینک‌شده را برای targetهای زیر
منتشر می‌کند:

- `x86_64-unknown-linux-musl` (`musllinux_1_1_x86_64`)
- `aarch64-unknown-linux-musl` (`musllinux_1_1_aarch64`)
- `armv7-unknown-linux-musleabihf` (`musllinux_1_1_armv7l`)
- `i686-unknown-linux-musl` (`musllinux_1_1_i686`)
- `riscv64gc-unknown-linux-musl` (`musllinux_1_1_riscv64`)
- `arm-unknown-linux-musleabihf` (`linux_armv6l`)

wheelها با tagهای [سازگار با musllinux](https://peps.python.org/pep-0656/) منتشر می‌شوند. با این
حال، باینری‌های `uv` جاسازی‌شده کاملاً به‌صورت ایستا لینک‌شده‌اند و به musl libc روی سیستم میزبان
نیازی ندارند.

[ایمیج‌های رسمی Docker](../../guides/integration/docker.md) این باینری‌های uv مبتنی بر musl را که
کاملاً به‌صورت ایستا لینک‌شده‌اند، برای amd64 و arm64 شامل می‌شوند.

## نسخه‌های Windows {#windows-versions}

حداقل نسخه‌های پشتیبانی‌شدهٔ Windows عبارت‌اند از Windows 10 و Windows Server 2016، مطابق با
[پشتیبانی Tier 1 خود Rust](https://blog.rust-lang.org/2024/02/26/Windows-7.html).

## نسخه‌های macOS {#macos-versions}

uv از macOS نسخهٔ 13+ (Ventura) پشتیبانی می‌کند.

می‌دانیم uv روی macOS 12 کار می‌کند، اما به نصب یک فایل اجرایی `realpath` نیاز دارد.
