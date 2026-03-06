# Django 5.x → 6.x: Deprecations & Breaking Changes

## Python Requirement
Django 6.0 is expected to require **Python 3.12+** (confirm when released).

> **Note:** Update this file with the official Django 6.0 release notes as new breaking
> changes and removals are confirmed. The items below reflect `RemovedInDjango60Warning`
> deprecations from Django 5.x.

---

## Will Be Removed in 6.0 (currently showing RemovedInDjango60Warning)

These are deprecated in Django 5.0/5.1 and will break in 6.0.

### Model Meta Changes

| Deprecated | Fix |
|---|---|
| `index_together` Meta option | Replace with `indexes` using `models.Index()` |

```python
# Old
class Meta:
    index_together = [['field_a', 'field_b']]

# New
class Meta:
    indexes = [
        models.Index(fields=['field_a', 'field_b']),
    ]
```

**Migration note:** You need to create a migration that removes `index_together` and adds
the equivalent `indexes`. Django provides a migration operation for this — or you can do it
in two steps: add the new index, then remove `index_together`.

### Other 6.0 Removals

| Deprecated | Fix |
|---|---|
| `django.utils.text.truncate_html_words()` | `Truncator.words()` |
| `FORMS_URLFIELD_ASSUME_HTTPS` transitional setting | Remove it (HTTPS is the default) |

---

## Will Be Removed in 6.1 (currently showing RemovedInDjango61Warning)

| Deprecated | Fix |
|---|---|
| `LoginRequiredMiddleware` `login_url` and `redirect_field_name` attrs on views | Configure via `settings.py` |

---

## Expected New Features in 6.0

Update this section when Django 6.0 is released. Watch for:
- Composite primary key support improvements
- Further async view/ORM enhancements
- Potential new model field types

---

## Cross-Version Patterns

### Preparing for 6.0 While on 5.x

The best time to fix `RemovedInDjango60Warning` deprecations is while you're still on 5.x:

1. Run your test suite with deprecation warnings enabled
2. Every `RemovedInDjango60Warning` is something that will break in 6.0
3. Fix them now while your code still works — it's much easier than debugging failures after upgrading

### Using django-upgrade CLI

```bash
pip install django-upgrade
django-upgrade --target-version 6.0 **/*.py
# Review the diff!
git diff
```

When Django 6.0 is released, the `django-upgrade` tool will be updated with new fixers.
