---
name: nanobanana-image-gen
description: Use when asked to generate images (PNG/JPEG) using NanoBanana (Google's nano-banana-pro-preview model) via direct Gemini API calls - no CLI needed, just GEMINI_API_KEY
---

# Generating Images with NanoBanana (Gemini API)

## Overview

NanoBanana generates raster images (PNG) by calling the Gemini REST API directly with the `nano-banana-pro-preview` model. No Gemini CLI or extensions needed — just a `GEMINI_API_KEY` environment variable.

## When to Use

- User asks to "generate an image", "create a picture", "make an illustration"
- User explicitly asks for nanobanana
- User needs PNG/JPEG raster images (photos, artwork, illustrations, UI assets)
- For SVG/vector output, use the `gemini-image-gen` skill instead

## Requirements

- `GEMINI_API_KEY` environment variable must be set
- Get a key from https://aistudio.google.com/apikey if not set
- `python3` available (for JSON/base64 processing)
- `curl` available

## Quick Reference

| Task | Model | Method |
|------|-------|--------|
| PNG image | `nano-banana-pro-preview` | Direct Gemini REST API |
| PNG fallback | `gemini-2.0-flash-exp-image-generation` | Direct Gemini REST API |

## Command

```bash
curl -s "https://generativelanguage.googleapis.com/v1beta/models/nano-banana-pro-preview:generateContent?key=${GEMINI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "Generate an image of [DESCRIPTION]"}]}],
    "generationConfig": {"responseModalities": ["TEXT", "IMAGE"]}
  }' | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
if 'error' in data:
    print(json.dumps(data, indent=2)[:500])
    sys.exit(1)
for part in data['candidates'][0]['content']['parts']:
    if 'inlineData' in part:
        img_data = base64.b64decode(part['inlineData']['data'])
        with open('/tmp/[FILENAME].png', 'wb') as f:
            f.write(img_data)
        print('Image saved to /tmp/[FILENAME].png')
    elif 'text' in part:
        print(part['text'])
"
```

## Model Fallback Chain

Always try the best model first, then fall back on failure.

**Image models** (in order): `nano-banana-pro-preview` → `gemini-2.0-flash-exp-image-generation`

**Fallback procedure:**
1. Run with `nano-banana-pro-preview`
2. If response contains `error` → retry with `gemini-2.0-flash-exp-image-generation`
3. If all models fail, tell the user and suggest waiting

## Key Rules

- **`responseModalities` must be `["TEXT", "IMAGE"]`** — using only `["IMAGE"]` will fail
- Always save to `/tmp/` unless the user specifies a different path
- Pass the user's prompt as-is — don't rewrite or embellish unless they delegate content to you
- After generating, verify the file exists and show it with `![description](/tmp/filename.png)`

## Prompt Handling

**Default: pass the user's prompt as-is.** Forward the user's words to the API without rewriting.

Examples:
- "generate an image of a dog" → `"Generate an image of a dog"`
- "create a sunset over mountains" → `"Generate an image of a sunset over mountains"`

**Only add details when the user explicitly delegates** — e.g. "generate some images for this page".

## Workflow for Claude

1. Check `GEMINI_API_KEY` is set
2. Build the curl command with user's prompt
3. Try `nano-banana-pro-preview` first
4. If error, fall back to `gemini-2.0-flash-exp-image-generation`
5. Verify the output file exists
6. Show the image with `![description](/tmp/filename.png)`

## Common Issues

| Problem | Solution |
|---------|----------|
| `PERMISSION_DENIED` | Check `GEMINI_API_KEY` is set and valid |
| `responseModalities` error | Must use `["TEXT", "IMAGE"]`, not just `["IMAGE"]` |
| Model not found | Check model name — use `nano-banana-pro-preview` exactly |
| Empty response | Model may have content-filtered the prompt — simplify it |
| Rate limited (429) | Fall back to `gemini-2.0-flash-exp-image-generation` or wait |
