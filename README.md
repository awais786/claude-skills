# claude-skills

A collection of Claude Code skills for Django packages and ORM patterns.

## Installation

```bash
# Clone the repo
git clone git@github.com:awais786/claude-skills.git ~/claude-skills

# Copy a specific skill
cp -r ~/claude-skills/.claude/skills/django-storages-s3 ~/.claude/skills/

# Or install all skills
cp -r ~/claude-skills/.claude/skills/. ~/.claude/skills/
```

Then invoke any skill in Claude Code with `/<skill-name>`.

## Available Skills

| Skill | Description |
|-------|-------------|
| `django-storages-s3` | Configure django-storages with AWS S3, custom backends, presigned URLs |
