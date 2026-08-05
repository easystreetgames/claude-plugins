# spirograph

Design and generate ESG spirograph animation scripts using gear mechanics, color palettes, and pattern ratios.

## Install

```
/plugin install spirograph@esgames-plugins
```

## Skills

### `/spirograph:design`

Interview-driven script generation. Asks about style, colors, petal count, and duration — then outputs a complete, ready-to-run ESG script.

### `/spirograph:variation`

Takes an existing script (pasted or described) and proposes three distinct variations. Generates whichever you pick.

## How it works

Scripts use a "gear" mechanic: a container that orbits while its inner shapes spin independently. The ratio of inner-to-outer rotation velocity determines the pattern shape (petal count). Shapes trace accumulated paths into an offscreen canvas buffer.

Key design parameters:
- **inner:outer ratio** — petal count (7:1 = 7-petal flower, 3:1 = trefoil, etc.)
- **thrust** — orbit radius
- **shape offset** — arm length
- **color palette** — static, oscillating cross-fade, or random
