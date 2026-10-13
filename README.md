# image-gen (archived)

> **ARCHIVED 2026-10-13 — abandoned because of the cost.** Generating images through paid APIs (OpenAI / OpenRouter / Gemini) was not worth the money, so this skill and its `imagegen` CLI were deleted from `~/.claude`. Kept here only for reference.

A Claude Code skill that wrapped the `imagegen` CLI (`image_gen.py`):

- auto-routes by model name to OpenAI, OpenRouter or Google Gemini
- saves the raw API response to disk before parsing, so a failed extraction does not mean a paid re-run
- `Skill.md` carries a 10-part cinematic prompt framework

## Files

- `Skill.md` — the skill definition
- `image_gen.py` — the CLI (was symlinked to `~/.local/bin/imagegen`)
