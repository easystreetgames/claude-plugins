---
name: firefly-prompt
description: Generate an optimized Adobe Firefly image prompt from user input — interview the user for subject, style, lighting, composition, and mood, then output a ready-to-paste prompt.
---

# Adobe Firefly Prompt Generator

Interview the user to build an optimized Adobe Firefly image generation prompt. Ask only what you don't already know from their initial message.

## Interview — collect these dimensions

Ask about missing dimensions in a single message (batch questions, don't drip one at a time):

1. **Subject** — what is the primary subject or scene? (person, object, landscape, concept)
2. **Style** — photorealistic, oil painting, digital art, watercolor, anime, cinematic, illustration, etc.
3. **Lighting** — golden hour, studio lighting, cinematic, dramatic shadows, soft glow, neon, natural daylight
4. **Composition / camera** — close-up, wide shot, aerial, eye-level, portrait orientation, rule of thirds
5. **Mood / atmosphere** — eerie, peaceful, epic, cozy, tense, mythic, adventurous
6. **Color palette** — warm, cool, monochrome, vibrant, muted, specific colors or hex values
7. **Detail level** — minimalist, hyper-detailed, impressionistic, photorealistic textures
8. **Intended use** *(optional)* — wallpaper, book cover, game asset, social post, poster

Skip any dimension the user already covered. If they gave enough to build a solid prompt, skip straight to output.

## Output format

After collecting the dimensions, output exactly this:

---

**Your Adobe Firefly prompt:**

```
[assembled prompt here]
```

**Content type setting:** [Photo / Art / Graphic]

**Tips for this prompt:**
- [2–3 short, specific tips for Firefly settings or iteration — e.g. which style preset to try, strength slider advice, reference image suggestion]

---

## Prompt assembly rules

Use the formula: `[Subject]. [Setting/environment]. [Lighting]. [Color palette]. [Style]. [Composition]. [Mood/atmosphere]. [Detail level].`

- Write in plain descriptive English — no "generate" or "create"
- Use commas and periods to separate elements clearly; short clauses beat long sentences
- Front-load the subject — Firefly weights earlier words more heavily
- Keep total prompt under 200 words
- Use specific, concrete terms: "warm amber backlight" not "nice lighting"; "lone figure in a hooded cloak" not "a person"
- Avoid negations in the prompt text — use Firefly's negative prompt field for exclusions instead
- If the style is photorealistic, suggest Content type = Photo; if artistic, suggest Art

## Example

User: "a warrior standing on a cliff at sunset, epic feel"

Output:
```
Lone armored warrior standing on a rocky cliff edge at sunset. Vast mountain range in the background, dramatic orange and crimson sky. Cinematic backlighting, long golden shadows. Epic, mythic mood. Wide shot, low camera angle looking up. Ultra-detailed armor with weathered textures, photorealistic.
```
Content type: Photo
Tips: Try the "Cinematic" style preset at 60–70% strength. Add a reference image of real mountain ranges to anchor the background. Use the "Dramatic" lighting effect to reinforce the backlight.
