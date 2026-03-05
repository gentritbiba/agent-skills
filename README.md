# Agent Skills

A collection of skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) agents.

## Skills

| Skill | Description |
|-------|-------------|
| [gemini-image-gen](./gemini-image-gen/SKILL.md) | Generate images (PNG/JPEG) and SVG icons/logos using Gemini CLI |

## Installation

Copy any skill folder into your Claude Code skills directory:

```bash
# Copy a skill to your project
cp -r gemini-image-gen /path/to/your/project/.claude/skills/

# Or to your global skills
cp -r gemini-image-gen ~/.claude/skills/
```

## Contributing

PRs welcome! Each skill should be a folder containing a `SKILL.md` file with frontmatter:

```yaml
---
name: skill-name
description: When to trigger this skill
---
```

## License

MIT
