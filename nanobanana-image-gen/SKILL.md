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
- `curl` available
- One of: `node`/`bun`, `python3`, or `base64` CLI (for decoding)

## Quick Reference

| Task | Model | Method |
|------|-------|--------|
| PNG image | `nano-banana-pro-preview` | Direct Gemini REST API |
| PNG fallback | `gemini-2.0-flash-exp-image-generation` | Direct Gemini REST API |

## Command

Use the first available runtime. All produce identical results.

### Option 1: Node.js / Bun (preferred — most portable)

```bash
RESPONSE=$(curl -s "https://generativelanguage.googleapis.com/v1beta/models/nano-banana-pro-preview:generateContent?key=${GEMINI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "Generate an image of [DESCRIPTION]"}]}],
    "generationConfig": {"responseModalities": ["TEXT", "IMAGE"]}
  }')

node -e "
const data = JSON.parse(process.argv[1]);
if (data.error) { console.error(JSON.stringify(data.error, null, 2)); process.exit(1); }
for (const part of data.candidates[0].content.parts) {
  if (part.inlineData) {
    require('fs').writeFileSync('/tmp/[FILENAME].png', Buffer.from(part.inlineData.data, 'base64'));
    console.log('Image saved to /tmp/[FILENAME].png');
  } else if (part.text) { console.log(part.text); }
}" "$RESPONSE"
```

### Option 2: Python 3

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
    print(json.dumps(data, indent=2)[:500]); sys.exit(1)
for part in data['candidates'][0]['content']['parts']:
    if 'inlineData' in part:
        with open('/tmp/[FILENAME].png', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        print('Image saved to /tmp/[FILENAME].png')
    elif 'text' in part:
        print(part['text'])
"
```

### Option 3: Pure shell (jq + base64 CLI)

```bash
RESPONSE=$(curl -s "https://generativelanguage.googleapis.com/v1beta/models/nano-banana-pro-preview:generateContent?key=${GEMINI_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "Generate an image of [DESCRIPTION]"}]}],
    "generationConfig": {"responseModalities": ["TEXT", "IMAGE"]}
  }')

# Check for errors
if echo "$RESPONSE" | jq -e '.error' >/dev/null 2>&1; then
  echo "$RESPONSE" | jq '.error'; exit 1
fi

# Extract and decode image
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[] | select(.inlineData) | .inlineData.data' | base64 -d > /tmp/[FILENAME].png
echo "Image saved to /tmp/[FILENAME].png"

# Print any text parts
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[] | select(.text) | .text'
```

### Runtime Detection

Pick the right option automatically:

```bash
if command -v node &>/dev/null; then
  # Use Option 1 (node)
elif command -v python3 &>/dev/null; then
  # Use Option 2 (python3)
elif command -v jq &>/dev/null; then
  # Use Option 3 (jq + base64)
else
  echo "Need node, python3, or jq+base64 to decode the response"
  exit 1
fi
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
2. Detect available runtime (node/bun > python3 > jq+base64)
3. Build the curl command with user's prompt
4. Try `nano-banana-pro-preview` first
5. If error, fall back to `gemini-2.0-flash-exp-image-generation`
6. Verify the output file exists
7. Show the image with `![description](/tmp/filename.png)`

## Common Issues

| Problem | Solution |
|---------|----------|
| `PERMISSION_DENIED` | Check `GEMINI_API_KEY` is set and valid |
| `responseModalities` error | Must use `["TEXT", "IMAGE"]`, not just `["IMAGE"]` |
| Model not found | Check model name — use `nano-banana-pro-preview` exactly |
| Empty response | Model may have content-filtered the prompt — simplify it |
| Rate limited (429) | Fall back to `gemini-2.0-flash-exp-image-generation` or wait |
| No runtime available | Install node, python3, or jq — at least one is needed |
