---
name: django-storages-s3
description: Configure and use django-storages with AWS S3 for media and static file storage. Use when setting up S3 storage, writing custom storage backends, handling file uploads, or generating presigned URLs in a Django project.
---

Help the user with django-storages S3 integration: $ARGUMENTS

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

```python
from storages.backends.s3boto3 import S3Boto3Storage

AWS_ACCESS_KEY_ID = env("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = env("AWS_SECRET_ACCESS_KEY")
AWS_STORAGE_BUCKET_NAME = env("AWS_STORAGE_BUCKET_NAME")
AWS_S3_REGION_NAME = env("AWS_S3_REGION_NAME", default="us-east-1")

AWS_S3_CUSTOM_DOMAIN = f"{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com"
AWS_DEFAULT_ACL = None           # Recommended: let bucket policy control ACL
AWS_S3_FILE_OVERWRITE = False    # Avoid overwriting files with the same name
AWS_QUERYSTRING_AUTH = False     # Set True for private files (presigned URLs)
```

```python
DEFAULT_FILE_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"
STATICFILES_STORAGE = "storages.backends.s3boto3.S3Boto3Storage"
MEDIA_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/media/"
STATIC_URL = f"https://{AWS_S3_CUSTOM_DOMAIN}/static/"
```

---

## Custom Storage Backends via Settings

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
        "querystring_auth": True,   # .url returns presigned URLs automatically
        "custom_domain": None,      # Must be None for presigned URLs to work
        "file_overwrite": False,
        "location": "media/private",
    },
}
```

Add a helper to instantiate a backend from a settings dict:

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

Generate a time-limited URL for a private file:

```python
import boto3
from django.conf import settings

def get_presigned_url(s3_key: str, expiry_seconds: int = 3600) -> str:
    client = boto3.client(
        "s3",
        region_name=settings.AWS_S3_REGION_NAME,
        aws_access_key_id=settings.AWS_ACCESS_KEY_ID,
        aws_secret_access_key=settings.AWS_SECRET_ACCESS_KEY,
    )
    return client.generate_presigned_url(
        "get_object",
        Params={"Bucket": settings.AWS_STORAGE_BUCKET_NAME, "Key": s3_key},
        ExpiresIn=expiry_seconds,
    )
```

For a backend with `querystring_auth: True` (e.g. `PRIVATE_FILE_BACKEND`), calling `instance.file.url` automatically returns a presigned URL — no manual boto3 call needed.

---

## File Upload in Views

```python
# forms.py
from django import forms

class UploadForm(forms.Form):
    file = forms.FileField()

# views.py
def upload_view(request):
    if request.method == "POST":
        form = UploadForm(request.POST, request.FILES)
        if form.is_valid():
            instance = MyModel(file=form.cleaned_data["file"])
            instance.save()  # Uploads to S3 automatically
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

## Common Pitfalls

- **`querystring_auth=True` + `custom_domain`**: These conflict — presigned URLs require the default S3 domain, so set `custom_domain = None` on private backends.
- **ACL errors on ACL-disabled buckets**: Set `AWS_DEFAULT_ACL = None` and rely on bucket policies instead of per-object ACLs.
- **`collectstatic` uploading to wrong location**: Ensure the `staticfiles` backend has `location = "static"` to avoid mixing with media.
- **Credentials in settings**: Always load via environment variables or AWS IAM roles, never hardcode.
