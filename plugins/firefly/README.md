# firefly

Adobe Firefly prompt generation plugin for Claude Code.

## Install

```
/plugin install firefly@esgames-plugins
```

## Commands

| Command | Description |
|---|---|
| `/firefly:firefly-prompt` | Interview-driven prompt builder — describes your image idea and get a ready-to-paste Firefly prompt with content type setting and iteration tips |

## How it works

The skill interviews you for up to 8 dimensions (subject, style, lighting, composition, mood, color palette, detail level, intended use). It batches missing questions into a single message, so if you give enough detail upfront it skips straight to output.

Output includes:
- A ready-to-paste prompt in Firefly's preferred formula
- A recommended Content type setting (Photo / Art / Graphic)
- 2–3 tips specific to your prompt (style presets, slider advice, reference image suggestions)
