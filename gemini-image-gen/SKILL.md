---
name: gemini-image-gen
description: Use when asked to generate images (PNG/JPEG), SVG icons, SVG logos, or any visual asset - delegates to Gemini CLI for image generation and SVG code generation via gemini-3-pro model
---

# Generating Images & SVGs with Gemini CLI

## Overview

Gemini CLI can generate two types of visual output:
- **SVG code**: Text-based vector graphics output directly from Gemini models (no API key needed)
- **Raster images (PNG)**: Generated via the `nanobanana` extension (requires `GEMINI_API_KEY` env var)

## When to Use

- User asks to "generate an image", "create an icon", "make a logo", "draw something"
- User needs SVG code for icons, logos, illustrations, diagrams
- User needs PNG/JPEG images (photos, artwork, UI assets)

## Quick Reference

| Task | Model | Command Pattern |
|------|-------|----------------|
| SVG code | `gemini-3-pro-preview` | `gemini -m gemini-3-pro-preview -e none -p "..." --output-format text` |
| SVG code (fallback) | `gemini-2.5-flash` | `gemini -m gemini-2.5-flash -e none -p "..." --output-format text` |
| PNG image | `gemini-2.5-flash` | `gemini -m gemini-2.5-flash -p "Generate image... Save to [path]" -y` |

## Model Fallback Chain

Always try the best model first, then fall back to cheaper/faster models on rate limit (429) errors.

**SVG models** (in order): `gemini-3-pro-preview` → `gemini-2.5-flash`
**Image models** (in order): `gemini-2.5-flash` → `gemini-2.5-flash` (retry after delay)

**Fallback procedure:**
1. Run command with primary model, capture stderr via `2>&1`
2. Check output for `429`, `RESOURCE_EXHAUSTED`, or `capacity` → means rate limited
3. Retry with the next model in the chain
4. If all models fail, tell the user and suggest waiting

## SVG Generation — via Code Output (NOT Nanobanana)

SVGs are generated as **text code output** by the model itself — NOT via nanobanana.
Use `-e none` to disable extensions so Gemini outputs SVG code instead of trying to use image tools.

```bash
# Try primary model first
OUTPUT=$(gemini -m gemini-3-pro-preview -e none -p "Create an SVG icon of [description]. Output ONLY the raw SVG code, no markdown fences, no explanation." --output-format text 2>&1)

# Check for rate limiting - fall back to cheaper model
if echo "$OUTPUT" | grep -qi "429\|RESOURCE_EXHAUSTED\|capacity\|quota"; then
  OUTPUT=$(gemini -m gemini-2.5-flash -e none -p "Create an SVG icon of [description]. Output ONLY the raw SVG code, no markdown fences, no explanation." --output-format text 2>&1)
fi

# Save result
echo "$OUTPUT" | sed '/^```/d' > output.svg
```

Or run them as separate commands - try the primary model first, and if you see rate limit errors in the output, immediately run the fallback.

### SVG Prompt Tips

- Always end with: "Output ONLY the raw SVG code, no markdown fences, no explanation."
- Specify dimensions: "200x200", "24x24 icon size"
- Specify colors explicitly: "Use #FF6B35 orange and #1A1A2E dark blue"
- Specify style: "flat design", "line art", "gradient", "minimalist"
- For logos, include text requirements: "Include the text 'AppName' below the icon"

### Post-Processing SVG Output

Gemini sometimes wraps output in markdown code fences. Strip them:

```bash
# Generate and clean SVG output
gemini -m gemini-3-pro-preview -e none -p "..." --output-format text | sed '/^```/d' > output.svg
```

After generating, read the file to verify valid SVG and show/return the code to the user.

## Image Generation (PNG) — via Nanobanana Extension

PNG image generation uses **nanobanana**, a Gemini CLI extension that provides a `generate_image` tool.
This is the ONLY way to generate real images — do NOT try to use Pillow, Python scripts, or `-e none`.

### How it works

1. Gemini CLI loads the `nanobanana` extension automatically on startup (you'll see `Loading extension: nanobanana` in output)
2. Gemini calls the `generate_image` tool provided by nanobanana
3. Nanobanana generates the image and saves it to the requested path

### Requirements

- `GEMINI_API_KEY` environment variable must be set (configured in `~/.zshrc`)
- Get a key from https://aistudio.google.com/apikey if not set
- Do **NOT** pass `-e none` — that disables all extensions including nanobanana

### Save location

Unless the user specifies a different path, always save generated images to `/tmp/`. Example: `/tmp/prishtina.png`, `/tmp/logo.png`.

### Command

```bash
gemini -m gemini-2.5-flash -p "Generate a beautiful image of [description]. Save it to /tmp/[filename].png" -y --output-format text 2>&1
```

### Key rules

- **NO `-e none`** — extensions must be enabled so nanobanana loads
- **`-y` is required** — auto-approves the nanobanana tool call (YOLO mode)
- **`--output-format text`** — keeps output clean
- If you see `Loading extension: nanobanana` in stdout, it's working
- If you see "No valid API key found", the `GEMINI_API_KEY` env var is missing
- Best for: photorealistic images, artwork, cityscapes, illustrations, any visual content

## Prompt Handling

**Default: pass the user's prompt as-is.** Always forward the user's words to Gemini without rewriting, embellishing, or adding details. Only append the save path instruction.

Examples of prompts to pass verbatim:
- "generate an image of Prishtina" → pass as-is
- "generate an image of a cat sitting on a rooftop at sunset" → pass as-is
- "make a logo with a blue mountain" → pass as-is

**Only fill in details when the user explicitly delegates content to you** — e.g. "create some images to add to this page" or "generate a few images for this blog post". In those cases you choose subjects, composition, and style.

## Workflow for Claude

When asked to generate visual assets:

1. **Determine type**: SVG (vector code) or PNG (raster image)?
2. **SVG path**:
   - Run with `gemini-3-pro-preview` first (best quality)
   - If output contains rate limit error → immediately retry with `gemini-2.5-flash`
   - Strip markdown fences, save to file, verify valid SVG
3. **PNG path** (uses nanobanana extension):
   - Run gemini **without** `-e none` and with `-y` so nanobanana loads and auto-approves
   - Confirm `Loading extension: nanobanana` appears in output
   - Specify save path in prompt, verify file exists after
4. **Always verify**: Read/check the output file exists and is valid before reporting success
5. **Show SVG code**: For SVGs, read the file and present the code to the user

## Common Issues

| Problem | Solution |
|---------|----------|
| Model rate limited (429) | Use fallback model (gemini-2.5-flash for SVGs) or wait and retry |
| SVG wrapped in markdown fences | Pipe through `sed '/^```/d'` |
| gemini hangs in headless mode | Add timeout: `timeout 120 gemini -p "..."` |
| SVG has no xmlns attribute | Add to prompt: "Include xmlns='http://www.w3.org/2000/svg'" |
| PNG: "No valid API key found" | Set `GEMINI_API_KEY` env var (get from https://aistudio.google.com/apikey) |
| PNG: nanobanana extension not loading | Don't use `-e none` for PNG generation — extensions must be enabled |
