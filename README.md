# claude-skills

A collection of [Claude Code](https://claude.ai/code) skills for Django packages and ORM patterns. Each skill provides Claude with structured reference material so it can give accurate, best-practice guidance for specific Django topics.

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI or IDE extension installed
- Django 4.2+ recommended (legacy fallbacks are included where applicable)

## Installation

```bash
# Clone the repo
git clone git@github.com:awais786/claude-skills.git ~/claude-skills

# Copy a specific skill
cp -r ~/claude-skills/.claude/skills/django-storages-s3 ~/.claude/skills/

# Or install all skills at once
cp -r ~/claude-skills/.claude/skills/. ~/.claude/skills/
```

Skills are installed to `~/.claude/skills/<name>/` and become globally available across all your projects.

## Available Skills

| Skill | Description | Key Topics |
|-------|-------------|------------|
| [`django-storages-s3`](.claude/skills/django-storages-s3/SKILL.md) | Configure django-storages with AWS S3 | S3 setup, `STORAGES` dict (4.2+), custom backends, presigned URLs/uploads, CloudFront CDN, IAM, testing |

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
.claude/skills/<name>/SKILL.md
```

Every `SKILL.md` must include:

1. **YAML frontmatter** with `name` and `description`
2. **Package Installation** — pip commands and Django config
3. **Core Settings** — copy-paste-ready configuration
4. **Code Examples** — models, views, helpers
5. **Testing & Mocking** — how to test without external services
6. **Common Pitfalls** — known gotchas and how to avoid them

## Contributing

1. Create `.claude/skills/<name>/SKILL.md` following the structure above
2. Use Django 4.2+ patterns as the default, with legacy fallbacks noted
3. Use `os.environ` (or explicitly name the env library) — no bare `env()` calls
4. Make all code examples copy-paste ready
5. Add the skill to the **Available Skills** table above and in [`CLAUDE.md`](CLAUDE.md)
6. Submit a PR

## License

MIT
