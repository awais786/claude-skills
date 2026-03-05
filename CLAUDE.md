# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains Claude Code skills focused on Django — covering Django ORM patterns, queries, and various Django packages. The project uses Django as the ORM layer with SQLite as the database backend.

## Stack

- **ORM / Framework:** Django
- **Database:** SQLite

## Skills

Skills live in `~/.claude/skills/<name>/SKILL.md` (global, available across all projects). Each skill targets a specific Django package or ORM pattern and is invoked automatically by Claude when relevant, or manually via `/<name>`.

| Skill | Purpose |
|-------|---------|
| `django-storages-s3` | Configure django-storages with AWS S3, custom backends, presigned URLs |

### Installing a Skill from GitHub

```bash
# Clone the repo (one-time)
git clone git@github.com:awais786/claude-skills.git ~/claude-skills

# Copy a specific skill to the global skills directory
cp -r ~/claude-skills/.claude/skills/django-storages-s3 ~/.claude/skills/
```

Or copy all skills at once:

```bash
cp -r ~/claude-skills/.claude/skills/. ~/.claude/skills/
```

### Adding a New Skill

1. Create `.claude/skills/<name>/SKILL.md`
2. Add a YAML frontmatter block with `name` and `description`
3. Copy to `~/.claude/skills/<name>/` to activate
4. Register it in the table above
