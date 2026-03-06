---
name: django-storages-s3
description: Configure Django with django-storages and AWS S3 for static and media files
---

## Package Installation

```bash
pip install django-storages[s3] boto3
```

Add to `INSTALLED_APPS`:
```python
INSTALLED_APPS = [
    ...
    "storages",
]
```

---

## Core Settings (settings.py)

Load credentials from environment variables using `os.environ` or a library like `django-environ`:

```python
import os

AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = os.environ.get("AWS_SECRET_ACCESS_KEY")
AWS_STORAGE_BUCKET_NAME = os.environ.get("AWS_STORAGE_BUCKET_NAME")
AWS_S3_REGION_NAME = os.environ.get("AWS_S3_REGION_NAME", "us-east-1")

AWS_S3_CUSTOM_DOMAIN = f"{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com"
AWS_DEFAULT_ACL = None           # Recommended: let bucket policy control ACL
AWS_S3_FILE_OVERWRITE = False    # Avoid overwriting files with the same name
AWS_QUERYSTRING_AUTH = False     # Set True for private files (presigned URLs)
```

> **Tip:** On AWS infrastructure (EC2, ECS, Lambda), omit `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` entirely — boto3 will use the attached IAM role automatically.

### Django 4.2+ (recommended) — `STORAGES` dict

```python
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "bucket_name": AWS_STORAGE_BUCKET_NAME,
            "location": "media",
            "file_overwrite": False,
        },
    },
    "staticfiles": {
        "BACKEND": "storages.backends.s3boto3.S3StaticS3Storage",
        "OPTIONS": {
            "bucket_name": AWS_STORAGE_BUCKET_NAME,
            "location": "static",
        },
    },
}

MEDIA_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/media/"
STATIC_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/static/"
```

### Django < 4.2 (legacy)

```python
DEFAULT_FILE_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"
STATICFILES_STORAGE = "storages.backends.s3boto3.S3StaticS3Storage"
MEDIA_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/media/"
STATIC_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/static/"
```

> **Note:** Always use separate `location` prefixes (e.g. `media/` and `static/`) or separate buckets so that static and media files are never mixed.

---

## Custom Storage Backends

### Option A — Via `STORAGES` dict (Django 4.2+)

```python
STORAGES = {
    "default": { ... },      # media uploads
    "staticfiles": { ... },  # static files
    "public_images": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "bucket_name": "mybucket-public",
            "default_acl": "public-read",
            "querystring_auth": False,
            "file_overwrite": False,
            "location": "media/public",
        },
    },
    "private_files": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "bucket_name": "mybucket-private",
            "default_acl": "private",
            "querystring_auth": True,
            "custom_domain": None,
            "file_overwrite": False,
            "location": "media/private",
        },
    },
}
```

Use in models (Django 4.2+):

```python
from django.core.files.storage import storages

class Document(models.Model):
    image = models.ImageField(storage=storages["public_images"])
    contract = models.FileField(storage=storages["private_files"])
```

### Option B — Manual helper (all Django versions)

Define named backends as dicts in `settings.py`:

```python
PUBLIC_IMAGE_BACKEND = {
    "class": "storages.backends.s3boto3.S3Boto3Storage",
    "options": {
        "bucket_name": "mybucket-public",
        "default_acl": "public-read",
        "querystring_auth": False,
        "file_overwrite": False,
        "location": "media/public",
    },
}

PRIVATE_FILE_BACKEND = {
    "class": "storages.backends.s3boto3.S3Boto3Storage",
    "options": {
        "bucket_name": "mybucket-private",
        "default_acl": "private",
        "querystring_auth": True,   # .url() returns presigned URLs automatically
        "custom_domain": None,      # Must be None for presigned URLs to work
        "file_overwrite": False,
        "location": "media/private",
    },
}
```

Helper to instantiate a backend from a settings dict:

```python
# myapp/storages.py
from django.conf import settings
from django.utils.module_loading import import_string

def get_storage(setting_name):
    config = getattr(settings, setting_name)
    storage_class = import_string(config["class"])
    return storage_class(**config.get("options", {}))
```

Use in models:

```python
from myapp.storages import get_storage

class Document(models.Model):
    image = models.ImageField(storage=get_storage("PUBLIC_IMAGE_BACKEND"))
    contract = models.FileField(storage=get_storage("PRIVATE_FILE_BACKEND"))
```

---

## Presigned URLs

### Automatic (recommended)

For a backend with `querystring_auth=True` (e.g. `private_files` above), calling `instance.contract.url` automatically returns a presigned URL — no manual boto3 call needed.

### Manual (for custom expiry or non-model files)

```python
import boto3
from django.conf import settings

def get_presigned_url(s3_key: str, expiry_seconds: int = 3600) -> str:
    """Generate a time-limited URL for a private S3 object."""
    client = boto3.client("s3", region_name=settings.AWS_S3_REGION_NAME)
    return client.generate_presigned_url(
        "get_object",
        Params={"Bucket": settings.AWS_STORAGE_BUCKET_NAME, "Key": s3_key},
        ExpiresIn=expiry_seconds,
    )
```

> **Tip:** When running on AWS infrastructure with an IAM role, `boto3.client("s3")` picks up credentials automatically — no need to pass them explicitly.

### Presigned Upload URL (client-side direct upload)

```python
def get_presigned_upload_url(s3_key: str, content_type: str = "application/octet-stream", expiry: int = 3600) -> dict:
    """Generate a presigned POST URL for direct browser-to-S3 uploads."""
    client = boto3.client("s3", region_name=settings.AWS_S3_REGION_NAME)
    return client.generate_presigned_post(
        Bucket=settings.AWS_STORAGE_BUCKET_NAME,
        Key=s3_key,
        Fields={"Content-Type": content_type},
        Conditions=[{"Content-Type": content_type}],
        ExpiresIn=expiry,
    )
```

---

## File Upload in Views

```python
# forms.py
from django import forms

class UploadForm(forms.Form):
    file = forms.FileField()

# views.py
from django.shortcuts import render, redirect

def upload_view(request):
    if request.method == "POST":
        form = UploadForm(request.POST, request.FILES)
        if form.is_valid():
            instance = MyModel(file=form.cleaned_data["file"])
            instance.save()  # Uploads to S3 automatically
            return redirect("success")
    else:
        form = UploadForm()
    return render(request, "upload.html", {"form": form})
```

---

## CloudFront CDN Integration

For production, serve files via CloudFront instead of directly from S3:

```python
AWS_S3_CUSTOM_DOMAIN = os.environ.get("CLOUDFRONT_DOMAIN")  # e.g. "d1234abcdef.cloudfront.net"

# If using signed CloudFront URLs:
AWS_CLOUDFRONT_KEY_ID = os.environ.get("AWS_CLOUDFRONT_KEY_ID")
AWS_CLOUDFRONT_KEY = os.environ.get("AWS_CLOUDFRONT_KEY")  # PEM private key string
```

With Django 4.2+ `STORAGES`:
```python
STORAGES = {
    "default": {
        "BACKEND": "storages.backends.s3boto3.S3Boto3Storage",
        "OPTIONS": {
            "bucket_name": AWS_STORAGE_BUCKET_NAME,
            "custom_domain": AWS_S3_CUSTOM_DOMAIN,
            "location": "media",
        },
    },
    "staticfiles": {
        "BACKEND": "storages.backends.s3boto3.S3StaticS3Storage",
        "OPTIONS": {
            "bucket_name": AWS_STORAGE_BUCKET_NAME,
            "custom_domain": AWS_S3_CUSTOM_DOMAIN,
            "location": "static",
        },
    },
}
```

---

## IAM Policy (minimum required)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::your-bucket-name"
    }
  ]
}
```

---

## Testing & Mocking

Use `django.core.files.storage.InMemoryStorage` (Django 4.2+) or override the storage in tests to avoid hitting S3:

```python
from django.test import TestCase, override_settings

@override_settings(
    STORAGES={
        "default": {"BACKEND": "django.core.files.storage.InMemoryStorage"},
        "staticfiles": {"BACKEND": "django.contrib.staticfiles.storage.StaticFilesStorage"},
    }
)
class FileUploadTests(TestCase):
    def test_upload(self):
        from django.core.files.uploadedfile import SimpleUploadedFile
        f = SimpleUploadedFile("test.txt", b"hello", content_type="text/plain")
        obj = MyModel.objects.create(file=f)
        self.assertIn("test", obj.file.name)
```

For Django < 4.2, use `@override_settings(DEFAULT_FILE_STORAGE="django.core.files.storage.FileSystemStorage")`.

Alternatively, use the **moto** library to mock S3 entirely:

```bash
pip install moto[s3]
```

```python
import boto3
from moto import mock_aws

@mock_aws
def test_presigned_url():
    conn = boto3.client("s3", region_name="us-east-1")
    conn.create_bucket(Bucket="test-bucket")
    conn.put_object(Bucket="test-bucket", Key="test.txt", Body=b"data")
    url = get_presigned_url("test.txt")
    assert "test.txt" in url
```

---

## Common Pitfalls

- **`querystring_auth=True` + `custom_domain`**: These conflict — presigned URLs require the default S3 domain, so set `custom_domain = None` on private backends.
- **ACL errors on ACL-disabled buckets**: Set `AWS_DEFAULT_ACL = None` and rely on bucket policies instead of per-object ACLs. Since April 2023, new S3 buckets have ACLs disabled by default.
- **`collectstatic` uploading to wrong location**: Ensure the `staticfiles` backend has `location = "static"` to avoid mixing with media files.
- **Credentials in settings**: Always load via environment variables or AWS IAM roles, never hardcode.
- **Using deprecated settings**: `DEFAULT_FILE_STORAGE` and `STATICFILES_STORAGE` are deprecated in Django 4.2+. Use the `STORAGES` dict instead.
- **Large file uploads timing out**: For files > 100 MB, consider using presigned upload URLs for direct browser-to-S3 uploads to bypass your Django server.
- **Missing `Content-Type`**: S3 may default to `binary/octet-stream`. Set `AWS_S3_OBJECT_PARAMETERS = {"ContentType": "auto"}` or let django-storages detect it automatically (the default behavior).
