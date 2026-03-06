# Django 4.x → 5.x: Deprecations & Breaking Changes

## Python Requirement
Django 5.0+ requires **Python 3.10 or higher**.

---

## Removed in 5.0 (deprecated since 3.x/4.x)

These cause **ImportError or AttributeError** — they're already gone, not just warnings.

### Import Renames

| Removed | Replacement |
|---|---|
| `django.utils.encoding.force_text()` | `force_str()` |
| `django.utils.encoding.smart_text()` | `smart_str()` |
| `django.utils.translation.ugettext()` | `gettext()` |
| `django.utils.translation.ugettext_lazy()` | `gettext_lazy()` |
| `django.utils.translation.ugettext_noop()` | `gettext_noop()` |
| `django.utils.translation.ungettext()` | `ngettext()` |
| `django.utils.translation.ungettext_lazy()` | `ngettext_lazy()` |
| `django.utils.http.urlquote()` | `urllib.parse.quote()` |
| `django.utils.http.urlquote_plus()` | `urllib.parse.quote_plus()` |
| `django.utils.http.urlunquote()` | `urllib.parse.unquote()` |
| `django.utils.http.urlunquote_plus()` | `urllib.parse.unquote_plus()` |

### URL Configuration

| Removed | Replacement |
|---|---|
| `django.conf.urls.url()` | `django.urls.re_path()` or `path()` |

```python
# Old
from django.conf.urls import url
url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive)

# New
from django.urls import path, re_path
path('articles/<int:year>/', views.year_archive)
re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive)
```

### Other Removals

| Removed | Replacement |
|---|---|
| `HttpRequest.is_ajax()` | `request.headers.get('X-Requested-With') == 'XMLHttpRequest'` |
| `length_is` template filter | Use `length` with comparison |
| `USE_L10N` setting | Localization enabled by default — remove the setting |

---

## New Deprecations in 5.0 (RemovedInDjango60Warning)

These still work in 5.0 but warn. Fix them now.

| Deprecation | Fix |
|---|---|
| `index_together` Meta option | Use `indexes` with `models.Index()` |
| `django.utils.text.truncate_html_words()` | Use `Truncator.words()` |
| `FORMS_URLFIELD_ASSUME_HTTPS` transitional setting | Set to `True`, then remove |

---

## Removed in 5.1

| Removed | Replacement |
|---|---|
| `BaseUserManager.make_random_password()` | `secrets.token_urlsafe()` |

## New Deprecations in 5.1 (RemovedInDjango61Warning)

| Deprecation | Fix |
|---|---|
| `LoginRequiredMiddleware` attributes `login_url`, `redirect_field_name` | Configure in `settings.py` instead |

---

## Removed in 5.2

Items deprecated in 4.1 that are now fully removed. Consult Django 5.2 release notes.

---

## Cross-Version Patterns

### DEFAULT_AUTO_FIELD

If your project was created before Django 3.2 and never set this:

```python
# settings.py — pick one:

# Keep existing integer primary keys (safe, no migration needed):
DEFAULT_AUTO_FIELD = 'django.db.models.AutoField'

# Use BigAutoField (recommended for new projects, may need migrations):
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

Each `AppConfig` can also override with `default_auto_field`.

### Translation Bulk Renames

```
ugettext → gettext
ugettext_lazy → gettext_lazy
ugettext_noop → gettext_noop
ungettext → ngettext
ungettext_lazy → ngettext_lazy
```

### Encoding Bulk Renames

```
force_text → force_str
smart_text → smart_str
```

### Using django-upgrade CLI

```bash
pip install django-upgrade
find . -name "*.py" | xargs django-upgrade --target-version 5.0
# Review the diff!
git diff
```

Handles: import renames, url() → re_path(), @admin.register, on_delete=,
DEFAULT_AUTO_FIELD, and many more mechanical fixes.
