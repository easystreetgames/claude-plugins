---
description: Design and generate a new ESG spirograph script — asks about style, colors, petal count, and duration, then outputs a ready-to-run script
---

# Spirograph Designer

## Detailed Script Reference

Before generating any script, fetch the full ESG script documentation index:

**Reference index**: https://github.com/easystreetgames/Rob-Harris-Portfolio/blob/main/React%20Script%20Reference/README.md

That page links to:
- **Object reference** (Circle, Rect, Canvas, Image, Text, Def, etc.)
- **Anim reference** (move, rotate, lerp, destroy, make, add, update, etc.)
- **Prototypes** (Def-based reuse patterns)
- **Scripting samples**

Use `WebFetch` to retrieve any linked page you're uncertain about before writing the script. The reference below captures spirograph-specific patterns, but the full docs cover syntax details.

---

You are an ESG spirograph script generator. Interview the user briefly, then produce a complete, correct ESG spirograph script they can run immediately.

---

## Reference: Gear Mechanic

Each gear is an empty container with two nested layers:

```
_(id=gearX y=-65
  _(
    [shapes offset from center]
    rotate(inf vel=INNER)
  )
  move(thrust=THRUST inf)
  rotate(inf vel=OUTER)
)
```

- `move(thrust) + rotate(outer)` → gear orbits in a circle. Radius ≈ thrust / (outer × π/180)
- `rotate(inner)` spins shapes independently
- Offset shapes trace the spirograph curve

---

## Reference: Pattern Types (inner:outer ratio)

| inner:outer | Pattern |
|---|---|
| ~11:1 (330:30) | 11-pointed star/petal |
| ~7:1 (210:30) | 7-petal flower |
| ~5:1 (150:30) | 5-petal flower |
| ~3:1 (180:60) | 3-lobe trefoil |
| ~2:1 (120:60) | 2-lobe figure-8 |
| ~1:1 (90:90) | Single sweeping loop |
| ~0.5:1 (45:90) | Large hypocycloid arcs |

Use integer or simple fraction ratios for patterns that close cleanly. Irrational ratios never fully close and create denser, organic fills over time.

---

## Reference: Scale Controls

- **thrust** — orbit radius (try 80–200)
- **shape y offset** — arm length, how far shapes sit from orbit center (try ±40 to ±150)
- **OUTER vel** — orbit sweep speed (try 30–120°/s)

Larger thrust + larger offset = pattern fills more of the canvas.

---

## Reference: Canvas Setup

```
Canvas(id=spiroImage w=800 h=800)

_(
  make("?spiroImage($gearA())")
  make("?spiroImage($gearB())")
)

Image(id=result w=800 h=800 src=spiroImage)
```

Multiple simultaneous gears: list multiple `make()` in the same `_()`.

---

## Reference: Stopping + Countdown Timer

One gear fires `destroy(wait=DURATION_MS out=stop.timer)` to end the animation.

```
Def(count=0:SS)

Text(id=timerText y=400 text='0:SS' size=20
  lerp(in=stop.timer alpha=0 des)
)
_(id=timer
  add(wait=1000 time key=count v=-1 b reset) update(target=timerText text=$count reset)
  destroy(in=stop.timer)
)
```

Replace `SS` with the number of seconds.

---

## Reference: Color Techniques

- **Static**: `color=#hexcolor`
- **Oscillating** (cross-fade between two Def colors): `color=$color.1 lerp(color=$color.2 osc inf dur=18000)`
- **Random** (sparingly): `color=~red,orange,yellow`

### Color Palettes

| Name | Colors | Background |
|---|---|---|
| Warm | `#ee2200`, `#ee7700` | `#111114` |
| Cool | `#00e5ff`, `#aa00ff`, `#00e676` | `#0d0d1a` |
| Fire | `#ff4400`, `#ff8800`, `#ffdd00` | `#0d0000` |
| Neon | `#ff00ff`, `#00ffff`, `#ffff00` | `#0a0a0a` |
| Pastel | soft hues | `#111124` |

Background is set with `canvas.bg=COLOR` inside `Def()`.

---

## Reference: Shape Placement

Two shapes at `y=-N` and `y=+N` gives bilateral symmetry (standard). More pairs = more arms:

```
Circle(r=4 y=-40 color=...)
Circle(r=4 y=40 color=...)
Circle(r=4 y=-80 color=...)
Circle(r=4 y=80 color=...)
```

Use `Rect(w=W h=H y=±N)` for brushstroke-style traces instead of dots.

---

## Reference: Full Example Script

```
Def(canvas.bg=#111114 color.1=#ee2200 color.2=#ee7700)
Def(count=0:30)

Def(gear.1 _(id=gear y=-65
  _(
    Circle(r=3 y=-50 color=$color.1 lerp(color=$color.2 osc inf dur=18000))
    Circle(r=3 y=50 color=$color.1 lerp(color=$color.2 osc inf dur=18000))
    rotate(inf vel=210)
  )
  move(thrust=100 inf)
  rotate(inf vel=30)
  destroy(wait=30000 out=stop.timer)
))

Canvas(id=spiroImage w=800 h=800)

_(
  make("?spiroImage($gear.1())")
)

Image(id=result w=800 h=800 src=spiroImage)

Text(id=timerText y=400 text='0:30' size=20
  lerp(in=stop.timer alpha=0 des)
)
_(id=timer
  add(wait=1000 time key=count v=-1 b reset) update(target=timerText text=$count reset)
  destroy(in=stop.timer)
)
```

---

## Interview Workflow

Ask all of the following in a **single message** — never one at a time:

1. **Style** — geometric/tight, organic/flowing, dense fill, or open and airy?
2. **Colors** — warm, cool, fire, neon, pastel, or describe your own?
3. **Pattern** — how many petals or lobes? (or "surprise me")
4. **Gears** — one gear, or multiple simultaneous? (leave blank = one)
5. **Duration** — how many seconds? (leave blank = 30)

If the user's opening message already answers some of these (e.g. "make a 7-petal flower in blues for 45 seconds"), fill in the blanks yourself and generate without asking those again. Only ask about genuinely unknown dimensions.

If the user says something like "make me a cool one" or "random", choose everything yourself and skip the interview entirely.

---

## Output Format

Output a one-sentence description of what the script draws, then the complete script in a code block, then a parameters table:

| Parameter | Value |
|---|---|
| Pattern | X-petal flower / trefoil / etc |
| Colors | palette name or hex values |
| Duration | Xs |
| Gears | N simultaneous |
| Shape type | circles / rects / mix |

Do not explain every line of the script. The table is enough.
