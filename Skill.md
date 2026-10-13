---
name: image-gen
description: Generate images via OpenAI, OpenRouter, or Google Gemini through the `imagegen` CLI. Auto-routes by model name and auto-saves raw API response to avoid paid re-runs when extraction fails. Use when user asks to generate/create an image via API. Includes a 10-part cinematic prompt framework for crafting high-quality prompts.
version: 1.2.0
---

# Image Generation

Use the `imagegen` CLI (install per the [repo README](https://github.com/StevenLi-phoenix/imagegen-cli) — symlinks `image_gen.py` into `~/.local/bin/imagegen`).

## Usage

```bash
# Default: OpenRouter / openai/gpt-5.4-image-2 (no org verification needed)
imagegen "your prompt here"

# Auto-routes by model name — no --provider needed
imagegen "prompt" --model gpt-image-2                  # → OpenAI direct
imagegen "prompt" --model gemini-3-pro-image-preview   # → Gemini direct
imagegen "prompt" --model imagen-4.0-generate-preview  # → Gemini Imagen :predict
imagegen "prompt" --model openai/gpt-5.4-image-2       # → OpenRouter

# OpenAI direct with gpt-image-2 (requires org verification; use openrouter default if unsure)
imagegen "prompt" --provider openai --model gpt-image-2

# Explicit OpenRouter with gpt-5.4-image-2 (same as bare default, no org verification needed)
imagegen "prompt" --provider openrouter --model openai/gpt-5.4-image-2

# Re-extract from cached response (FREE — no API call)
imagegen

# Force a new API call even if cache exists
imagegen "new prompt" --force

# Custom output dir / size / quality
imagegen "your prompt" --out /tmp/myimages --size 1024x1024 --quality medium
```

## What It Does

1. Calls the image generation API
2. Saves raw JSON response to `image_response.json` BEFORE parsing — so failed extraction doesn't lose paid base64 data
3. Extracts and writes PNG file(s) to the output directory
4. Logs token usage and total cost

## Provider Selection Cheat-Sheet

**When in doubt: just run `imagegen "prompt"` with no flags — the default is already correct.**

| Provider | Model | Org Verification | Cost |
|----------|-------|-----------------|------|
| `openrouter` **(default)** | `openai/gpt-5.4-image-2` | Not required | ~$0.22/img |
| `openai` | `gpt-image-2` | **Required** | ~$0.05/img |
| `openai` | `gpt-image-1` | Not required | ~$0.03/img — **only use if gpt-image-2 returns 403** |
| `gemini` | `gemini-3-pro-image-preview` | Google API key | see [pricing](https://ai.google.dev/pricing) |

> **Never default to `gpt-image-1`.** Always prefer `gpt-image-2` (openai direct) or the openrouter default. `gpt-image-1` is only a last-resort fallback when openai returns HTTP 403 for gpt-image-2.

Auto-detection: model with `/` → OpenRouter; starts with `gemini`/`imagen` → Gemini; starts with `dall-e`/`gpt-image` → OpenAI.

## Requirements

- `OPENAI_API_KEY` env var (for openai provider)
- `OPENROUTER_API_KEY` env var (for openrouter provider)
- `GEMINI_API_KEY` env var (for gemini provider, from Google AI Studio)
- Python 3 with `httpx` installed

If `httpx` is missing in the project venv: activate venv, then `uv pip install httpx`.

## Cinematic Prompt Framework

When crafting prompts for `imagegen`, use this 10-part framework to produce cinematic, consistent results. Most bad AI images come from describing only a subject — this framework describes a **shot**: subject, story, style, lighting, camera, detail, quality, and what to avoid.

### The 10 Parts

1. **Subject Definition** — The main subject with 2–4 defining physical traits. The anchor of the image.
2. **Action and Context** — What the subject is doing and a small purpose/motivation.
3. **Environment and Setting** — Location, time of day, key surroundings. Ground the scene.
4. **Mood and Story** — Emotional tone and implied narrative beat.
5. **Visual Style and References** — Aesthetic direction: genre, era, medium, inspirations.
6. **Lighting and Color** — Lighting type + direction + color palette + color grading.
7. **Camera and Composition** — Lens focal length (mm), shot type, angle, framing, depth of field.
8. **Detail and Texture Control** — Materials, surface details, micro textures, realism cues.
9. **Quality and Realism Control** — Realism level, sharpness, fidelity, cinematic polish.
10. **Negative Constraints** — What to prevent: text, watermarks, extra limbs, distortion, artifacts.

### How to Write the Prompt

Combine all 10 parts into **one flowing paragraph** (not numbered, no labels). The prompt should read as a single cohesive description that an image model can directly consume. Target 150–300 words.

**Example** — User asks for "a cat in a coffee shop":

> A fluffy orange tabby cat with bright green eyes and a slightly tilted head, sitting on a worn wooden counter inside a cozy neighborhood coffee shop, lazily watching steam rise from a ceramic latte cup beside it. Late afternoon golden hour light streams through large storefront windows, casting long warm shadows across scattered newspapers and a chalkboard menu in the background. The mood is quiet contentment, a stolen peaceful moment in an otherwise busy city. Rendered in a warm, nostalgic film photography style reminiscent of Kodak Portra 400, with soft grain and slightly lifted blacks. Natural window light from camera-left as key, warm tungsten fill from pendant Edison bulbs overhead, color palette of amber, cream, and deep brown with a subtle teal split-tone in the shadows. Shot on an 85mm lens, medium close-up, shallow depth of field at f/1.8, the background coffee equipment softly bokeh'd, eye-level angle with the cat. Visible wood grain on the counter, tiny dust motes in the light beams, slightly chipped ceramic glaze on the cup, individual fur strands catching the light. Photorealistic, high fidelity, sharp focus on the cat's eyes, subtle film grain, cinematic color grading. No text, no watermark, no extra limbs, no distortion, no blurry face, no UI elements, no over-smoothed textures.

### Quick Construction Loop

1. **Lock the story**: subject + action + environment + mood
2. **Lock the shot**: camera + lighting
3. **Add realism**: materials + micro details + quality
4. **Add guardrails**: negative constraints
5. **Iterate one block at a time** — don't change everything at once

### Common Fixes

| Problem | Fix |
|---------|-----|
| Image looks flat | Add rim light + volumetric haze + contrast grade |
| Weird anatomy | Tighten negatives, simplify pose, add "hands in pockets" or "hands not visible" |
| Too generic | Add 3 specific details a photographer would notice |
| Style drift | Strengthen visual style references, keep them consistent |
| Messy background | Specify "clean background, minimal props, controlled depth of field" |

### Standard Negative Constraints

Always append these unless the user's intent conflicts:

> No text, no watermark, no logo, no signature, no frame, no UI elements. No extra limbs, no extra fingers, no fused hands, no distorted anatomy. No blurry face, no out of focus subject, no low resolution, no compression artifacts. No duplicated subjects, no warped geometry, no unnatural reflections. No over-smoothed skin, no plastic texture, no uncanny eyes.

### When to Use the Framework

- **Always use** when the user gives a short/vague prompt ("generate an image of X")
- **Adapt, don't force** — if the user asks for a cartoon/pixel art/minimalist style, don't impose "cinematic realism"; match the framework to their intent
- **Skip for technical prompts** — if the user provides a fully detailed prompt already, pass it through as-is
- **Show the expanded prompt** to the user before running `imagegen` so they can adjust
