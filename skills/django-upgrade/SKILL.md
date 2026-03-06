---
name: django-upgrade
description: >
  Guides Claude through Django version upgrades with a test-driven workflow. Use this skill whenever
  the user mentions upgrading Django, bumping Django versions, fixing Django deprecation warnings,
  updating tox or GitHub Actions for a new Django version, or running tests against multiple Django
  versions. Also trigger when the user mentions "DeprecationWarning" alongside Django, wants to
  prepare a codebase for a newer Django release, or asks about Django 5.x/6.x migration. Even if
  the user just says "update Django" or "make my project work with Django 5", use this skill.
  Supports any Django version upgrade path (4→5, 5→6, etc.) with per-version reference files.
argument-hint: [source-version] [target-version]
---

# Django Upgrade Skill

A single, version-agnostic skill for upgrading Django projects. The workflow is always the same —
only the deprecations and breaking changes differ per version. This skill handles that by loading
the right reference file based on what the user is upgrading from and to.

## Step 0: Determine the Upgrade Path

Before anything else, figure out the source and target versions. The user will typically say
something like "upgrade from Django 4.2 to 5.0" or "move to Django 5". Extract:

- **Source version**: the current Django version in the project
- **Target version**: what the user wants to upgrade to

If the user doesn't specify one or both:
- **Source**: detect from `requirements.txt`, `pyproject.toml`, `setup.cfg`, `Pipfile`, or
  `pip freeze` output
- **Target**: ask the user, or default to the latest stable release

Then load the correct reference files:

| Upgrade path | Reference file |
|---|---|
| Django 4.x → 5.x | `references/django-4-to-5.md` |
| Django 5.x → 6.x | `references/django-5-to-6.md` |
| Any upgrade | `references/common-patterns.md` (always load — tox, CI, warning triage) |

> **Note:** There is no reference file for 3.x → 4.x. For that path, consult Django's
> official 4.0 release notes directly, then continue with the table above for further hops.

If the upgrade spans multiple major versions (e.g., 3.x → 5.x), do it in steps — one major
version at a time. Each step uses its own reference file.

Also check Python version compatibility:
- Django 5.0+ requires Python 3.10+
- Django 5.2 (LTS) requires Python 3.10+
- Django 6.x — check `references/django-5-to-6.md` for requirements

---

## The Workflow

Follow these phases in order. The philosophy: let your test suite tell you what's broken.
Don't guess — run tests, read warnings, fix what they report.

### Phase 1: Understand the Current State

Get a clear picture before touching anything.

1. **Current Django version** — check `requirements.txt`, `pyproject.toml`, `setup.cfg`,
   `setup.py`, or `Pipfile`
2. **Python version(s)** — check `tox.ini`, `.github/workflows/`, `pyproject.toml`
3. **Test configuration** — locate `tox.ini`, `.github/workflows/`, and pytest config
   (`pytest.ini`, `pyproject.toml [tool.pytest.ini_options]`, or `setup.cfg [tool:pytest]`)
4. **Third-party Django packages** — scan requirements for packages like `djangorestframework`,
   `django-celery-beat`, `django-allauth`, etc. These may have their own version constraints

### Phase 2: Update Test Configuration

The goal is to run existing tests against the new Django version *without changing application
code yet*. This cleanly surfaces all deprecation warnings and failures.

#### tox.ini

Add the new Django version as a factor while keeping the old one:

**Key changes to make:**
- Add new Django version factor in `envlist`
- Add the dependency pin for the new version
- Enable deprecation warnings (pytest hides them by default)

```ini
[tox]
envlist = py{310,311,312}-django{CURRENT,TARGET}

[testenv]
deps =
    djangoCURRENT: Django>=CURRENT_MIN,<CURRENT_MAX
    djangoTARGET: Django>=TARGET_MIN,<TARGET_MAX
    pytest
    pytest-django
commands =
    pytest -W default::DeprecationWarning {posargs}
setenv =
    PYTHONWARNINGS = default::DeprecationWarning
```

Replace `CURRENT`, `TARGET`, version numbers with actual values from Step 0.
Adapt to the project's existing tox structure — don't rewrite their layout.

#### GitHub Actions Workflow

Find the CI workflow (`.github/workflows/test.yml` or similar) and add the new Django
version to the matrix:

```yaml
strategy:
  matrix:
    python-version: ['3.10', '3.11', '3.12']
    django-version: ['CURRENT', 'TARGET']
```

Ensure the install step pins Django correctly from the matrix variable. If the project
uses tox in CI, just map the matrix to tox environments.

#### pytest-django Warning Filters

Ensure deprecation warnings are visible in pytest output:

```ini
[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "myproject.settings"
filterwarnings = [
    "default::DeprecationWarning",
    "default::PendingDeprecationWarning",
]
```

This is critical — without it, pytest suppresses the warnings you're trying to find.

### Phase 3: Run Tests and Capture Warnings

Run the test suite against the **new** Django version:

```bash
# With tox
tox -e py311-djangoTARGET -- -W default::DeprecationWarning 2>&1 | tee test_output.txt

# Or with pytest directly
PYTHONWARNINGS=default::DeprecationWarning pytest 2>&1 | tee test_output.txt
```

**Parse the output and categorize:**

1. **Django deprecation warnings** (our focus)
   - Look for `RemovedInDjangoXXWarning` (e.g., `RemovedInDjango60Warning`)
   - Look for `DeprecationWarning` from `django.*` modules
2. **Test failures from breaking changes**
   - `ImportError` or `AttributeError` on things that were already removed
3. **Third-party warnings** (flag but don't fix)
   - Warnings originating from packages in `site-packages/`

Present findings to the user as a categorized summary before starting fixes.

### Phase 4: Fix Django Deprecation Warnings

Consult the reference file loaded in Step 0 for specific fixes. General approach:

- **Fix one category at a time** (e.g., all import renames, then all settings changes,
  then all URL patterns)
- **Run tests after each category** to confirm warnings are gone and nothing new broke
- **Use the `django-upgrade` CLI tool** for mechanical bulk fixes when appropriate:
  ```bash
  pip install django-upgrade
  find . -name "*.py" | xargs django-upgrade --target-version TARGET
  ```
  Always review the diff after running it.
- **Only fix Django deprecation warnings** — if a warning comes from a third-party package,
  flag it for the user but don't modify the package's code
- **Prefer Django's recommended replacement** — don't invent creative workarounds

### Phase 5: Verify and Clean Up

1. **Run tests on both versions** — old should still pass, new should pass with
   fewer/no warnings:
   ```bash
   tox -e py311-djangoCURRENT,py311-djangoTARGET
   ```

2. **Check remaining warnings** — any Django deprecation warnings left? Are they from
   your code or from dependencies?

3. **Update version pins** — update `requirements.txt` / `pyproject.toml` to allow
   the new Django version. Don't drop the old version unless the user wants to.

4. **Summary report** — provide the user with:
   - Deprecation warnings found and fixed (with file:line references)
   - Remaining warnings from third-party packages
   - Test results on both old and new versions
   - Next steps recommendation

---

## Common Pitfalls (applies to all upgrades)

- **pytest swallows DeprecationWarnings by default** — always add
  `-W default::DeprecationWarning` or configure `filterwarnings`
- **`DEFAULT_AUTO_FIELD`** — if not set in settings.py, add it. Use `BigAutoField`
  for new projects or `AutoField` to preserve existing PKs.
- **Multi-major-version jumps** — don't jump from 3.x to 5.x directly. Go 3→4, then 4→5.
  Each step uses its own reference file.
- **Third-party packages** may not support the new Django version yet. Check their
  compatibility before blaming your own code for failures.
- **The `django-upgrade` tool is your friend** — it handles many mechanical renames
  automatically. But always review the diff.

---

## When Things Go Wrong

If tests fail on the new version (not just warnings, but actual errors):

1. Check the loaded reference file for known breaking changes
2. Check if a third-party package needs upgrading
3. If too many things break, consider a stepped upgrade path
4. Search Django's official release notes for the specific error
