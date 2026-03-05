# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains Claude Code skills focused on Django — covering Django ORM patterns, queries, and various Django packages. Skills are reference documents that Claude uses automatically (or on demand) to provide accurate, best-practice guidance for specific Django topics.

## Stack

- **ORM / Framework:** Django (4.2+ recommended)
- **Database:** SQLite (for local development / testing)

## Skills

Skills live in `skills/<name>/SKILL.md` inside this repo and are installed to `~/skills/<name>/SKILL.md` (global, available across all projects). Each skill targets a specific Django package or ORM pattern and is invoked automatically by Claude when relevant, or manually via `/<name>`.

| Skill | Purpose | Key Topics |
|-------|---------|------------|
| `django-storages-s3` | Configure django-storages with AWS S3 | S3 setup, `STORAGES` dict (Django 4.2+), custom backends, presigned URLs, presigned uploads, CloudFront CDN, IAM policies, testing/mocking |

### Installing a Skill from GitHub

```bash
# Clone the repo (one-time)
git clone git@github.com:awais786/claude-skills.git ~/claude-skills

# Copy a specific skill to the global skills directory
cp -r ~/claude-skills/skills/django-storages-s3 ~/skills/
```

Or copy all skills at once:

```bash
cp -r ~/claude-skills/skills/. ~/skills/
```

### Adding a New Skill

1. Create `skills/<name>/SKILL.md`
2. Add a YAML frontmatter block with `name` and `description` fields
3. Write reference content covering: installation, configuration, common patterns, code examples, testing, and pitfalls
4. Copy to `~/skills/<name>/` to activate
5. Register it in the skills table above and in `README.md`

### Skill Conventions

- Each skill **must** have YAML frontmatter with `name` and `description`
- Prefer Django 4.2+ patterns (e.g. `STORAGES` dict) with legacy fallbacks noted
- Include a **Testing & Mocking** section so users can test without external services
- Include a **Common Pitfalls** section for known gotchas
- Use `os.environ` or explicitly name the env library used (e.g. `django-environ`)
- All code examples should be copy-paste ready
