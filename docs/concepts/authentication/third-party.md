# سرویس‌های شخص ثالث {#third-party-services}

## احراز هویت با indexهای package جایگزین {#authentication-with-alternative-package-indexes}

برای احراز هویت با indexهای جایگزین محبوب package در Python، به راهنماهای اختصاصی مراجعه کنید:

- [Azure Artifacts](../../guides/integration/azure.md)
- [Google Artifact Registry](../../guides/integration/google.md)
- [AWS CodeArtifact](../../guides/integration/aws.md)
- [JFrog Artifactory](../../guides/integration/jfrog.md)

## پشتیبانی از Hugging Face {#hugging-face-support}

uv از احراز هویت خودکار برای Hugging Face Hub پشتیبانی می‌کند. به‌طور مشخص، اگر environment variable به نام `HF_TOKEN`
تنظیم شده باشد، uv آن را به درخواست‌های ارسالی به `huggingface.co` منتقل می‌کند.

این کار به‌ویژه برای دسترسی به scriptهای خصوصی در Hugging Face Datasets مفید است. برای نمونه، می‌توانید
دستور زیر را اجرا کنید تا script به نام `main.py` را از یک dataset خصوصی اجرا کنید:

```console
$ HF_TOKEN=hf_... uv run https://huggingface.co/datasets/<user>/<name>/resolve/<branch>/main.py
```

می‌توانید احراز هویت خودکار Hugging Face را با تنظیم environment variable به نام `UV_NO_HF_TOKEN=1` غیرفعال کنید.
