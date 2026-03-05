# claude-skills

A collection of [Claude Code](https://claude.ai/code) skills for Django packages and ORM patterns.

Each skill is a self-contained Markdown reference that Claude uses to give accurate, best-practice guidance on a specific topic — right inside your editor.

## How It Works

Claude Code skills are **local Markdown files** — there is no central registry, app store, or platform to publish them to. You share skills via **Git repositories** and install them by copying files to your machine.

| Method | Scope | Path |
|--------|-------|------|
| **Global** (all projects) | Per-user | `~/skills/<name>/SKILL.md` |
| **Project** (one repo) | Per-repo | `<project>/skills/<name>/SKILL.md` |

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI or IDE extension
- Django 4.2+ recommended (legacy fallbacks included)

## Quick Install

**One-liner (all skills, global):**

```bash
git clone git@github.com:awais786/claude-skills.git ~/claude-skills \
  && mkdir -p ~/.claude/skills \
  && cp -r ~/claude-skills/skills/. ~/.claude/skills/
```

**Single skill:**

```bash
git clone git@github.com:awais786/claude-skills.git ~/claude-skills
cp -r ~/claude-skills/skills/django-storages-s3 ~/.claude/skills/
```

**Into a specific project (project-scoped):**

```bash
cp -r ~/claude-skills/skills/django-storages-s3 ./your-project/skills/
```

Once installed, Claude picks up the skill automatically when the topic is relevant.

## Available Skills

| Skill | Description | Key Topics |
|-------|-------------|------------|
| [`django-storages-s3`](skills/django-storages-s3/SKILL.md) | Configure django-storages with AWS S3 | S3 setup, `STORAGES` dict (4.2+), custom backends, presigned URLs/uploads, CloudFront CDN, IAM, testing |

### `django-storages-s3`

Everything you need to integrate `django-storages` with AWS S3 in a Django project:

- **Core Settings** — `os.environ`-based config, IAM role tips
- **Django 4.2+ `STORAGES` dict** — modern approach with legacy fallback
- **Custom Storage Backends** — public vs. private buckets, named backends
- **Presigned URLs** — automatic via `querystring_auth`, manual via boto3, and presigned upload URLs for direct browser-to-S3 uploads
- **File Upload Views** — form-based upload example
- **CloudFront CDN** — serve media/static via CloudFront
- **IAM Policy** — minimal required S3 permissions
- **Testing & Mocking** — `InMemoryStorage`, `@override_settings`, and `moto`
- **Common Pitfalls** — ACLs, deprecated settings, large uploads, Content-Type

## Skill Structure

Each skill follows a consistent format:

```
skills/<name>/SKILL.md
```

Every `SKILL.md` must include:

1. **YAML frontmatter** with `name` and `description`
2. **Package Installation** — pip commands and Django config
3. **Core Settings** — copy-paste-ready configuration
4. **Code Examples** — models, views, helpers
5. **Testing & Mocking** — how to test without external services
6. **Common Pitfalls** — known gotchas and how to avoid them

## Sharing & Distribution

Skills are distributed via **GitHub repos** — there is no Vercel, npm, or marketplace to submit them to. To share your skills:

1. Push this repo to GitHub (public or private)
2. Others clone and copy the skills they want
3. Optionally, include install instructions in your project's README

> **Note:** If Anthropic introduces a skill marketplace or registry in the future, this README will be updated with submission instructions.

## Contributing

1. Create `skills/<name>/SKILL.md` following the structure above
2. Use Django 4.2+ patterns as the default, with legacy fallbacks noted
3. Use `os.environ` (or explicitly name the env library) — no bare `env()` calls
4. Make all code examples copy-paste ready
5. Add the skill to the **Available Skills** table above and in [`CLAUDE.md`](CLAUDE.md)
6. Submit a PR

## License

MIT

MIT

