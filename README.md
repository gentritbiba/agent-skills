# Agent Skills

A collection of skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) agents.

## Skills

| Skill | Description |
|-------|-------------|
| [gemini-image-gen](./gemini-image-gen/SKILL.md) | Generate images (PNG/JPEG) and SVG icons/logos using Gemini CLI |
| [nanobanana-image-gen](./nanobanana-image-gen/SKILL.md) | Generate images (PNG/JPEG) using NanoBanana via direct Gemini API |

## Installation

Install any skill using the [Skills CLI](https://skills.sh/):

```bash
# Install a specific skill
npx skills add gentritbiba/agent-skills@gemini-image-gen
npx skills add gentritbiba/agent-skills@nanobanana-image-gen

# Install globally (no prompts)
npx skills add gentritbiba/agent-skills@gemini-image-gen -g -y
npx skills add gentritbiba/agent-skills@nanobanana-image-gen -g -y
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
