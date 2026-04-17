# Tailwind CSS Standard Scale - Canonical Reference

This file is the canonical source for every sizing, typography, spacing, radius, opacity, elevation, layering, and breakpoint value used by the pencil-atelier plugin. Every skill and agent in the plugin ships an identical copy of this file under its own `references/` folder, so whichever consumer is active has local access to the full spec without crossing execution contexts.

The values mirror Tailwind CSS v4's default theme verbatim. The plugin treats that scale as a universal 4px-grid standard, applied whether or not the companion project actually uses Tailwind.

Tailwind source of truth:

- https://tailwindcss.com/docs/font-size
- https://tailwindcss.com/docs/font-weight
- https://tailwindcss.com/docs/line-height
- https://tailwindcss.com/docs/letter-spacing
- https://tailwindcss.com/docs/border-radius
- https://tailwindcss.com/docs/border-width
- https://tailwindcss.com/docs/opacity
- https://tailwindcss.com/docs/box-shadow
- https://tailwindcss.com/docs/z-index
- https://tailwindcss.com/docs/theme
- https://tailwindcss.com/docs/responsive-design

---

## 1. Spacing (padding, margin, gap, width, height, inset, translate, space-x/y)

Tailwind v4 defines a single base unit (`--spacing: 0.25rem = 4px`) and derives every spacing class dynamically:

```
size-in-px(n) = n x 4
```

Any non-negative integer `n` is a valid v4 class: `p-17`, `w-29`, `gap-67`, `m-101` all work.

### Canonical documented values

| Class   | rem   | px  |
| ------- | ----- | --- |
| `*-0`   | 0     | 0   |
| `*-0.5` | 0.125 | 2   |
| `*-1`   | 0.25  | 4   |
| `*-1.5` | 0.375 | 6   |
| `*-2`   | 0.5   | 8   |
| `*-2.5` | 0.625 | 10  |
| `*-3`   | 0.75  | 12  |
| `*-3.5` | 0.875 | 14  |
| `*-4`   | 1     | 16  |
| `*-5`   | 1.25  | 20  |
| `*-6`   | 1.5   | 24  |
| `*-7`   | 1.75  | 28  |
| `*-8`   | 2     | 32  |
| `*-9`   | 2.25  | 36  |
| `*-10`  | 2.5   | 40  |
| `*-11`  | 2.75  | 44  |
| `*-12`  | 3     | 48  |
| `*-14`  | 3.5   | 56  |
| `*-16`  | 4     | 64  |
| `*-20`  | 5     | 80  |
| `*-24`  | 6     | 96  |
| `*-28`  | 7     | 112 |
| `*-32`  | 8     | 128 |
| `*-36`  | 9     | 144 |
| `*-40`  | 10    | 160 |
| `*-44`  | 11    | 176 |
| `*-48`  | 12    | 192 |
| `*-52`  | 13    | 208 |
| `*-56`  | 14    | 224 |
| `*-60`  | 15    | 240 |
| `*-64`  | 16    | 256 |
| `*-72`  | 18    | 288 |
| `*-80`  | 20    | 320 |
| `*-96`  | 24    | 384 |

### Allowed pixel values (plugin rule)

`0, 2, 4, 6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60, 64, 72, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 384`

Any multiple of 4 up to 384 is allowed, plus the fractional-derived values 2, 6, 10, 14.

### Off-scale snap targets

- 3 -> 4
- 5 -> 4 or 6 (prefer 4)
- 7 -> 8 or 6 (prefer 8)
- 9 -> 8 or 12 (prefer 8)
- 11 -> 12
- 13 -> 12 or 14 (prefer 12)
- 15 -> 16
- 17 -> 16 or 20 (prefer 16)
- 18 -> 16 or 20 (prefer 16 for compact, 20 for breathing room)
- 22 -> 24
- 26 -> 24 or 28 (prefer 24)
- 30 -> 28 or 32 (prefer 32)
- 34 -> 32 or 36 (prefer 32)

---

## 2. Font sizes

| Class       | rem   | px  |
| ----------- | ----- | --- |
| `text-xs`   | 0.75  | 12  |
| `text-sm`   | 0.875 | 14  |
| `text-base` | 1     | 16  |
| `text-lg`   | 1.125 | 18  |
| `text-xl`   | 1.25  | 20  |
| `text-2xl`  | 1.5   | 24  |
| `text-3xl`  | 1.875 | 30  |
| `text-4xl`  | 2.25  | 36  |
| `text-5xl`  | 3     | 48  |
| `text-6xl`  | 3.75  | 60  |
| `text-7xl`  | 4.5   | 72  |
| `text-8xl`  | 6     | 96  |
| `text-9xl`  | 8     | 128 |

### Allowed pixel values

`12, 14, 16, 18, 20, 24, 30, 36, 48, 60, 72, 96, 128`

Legibility floor: never emit sizes below 12.

### Off-scale snap targets

- 10, 11 -> 12
- 13, 15 -> 14
- 17 -> 18
- 19, 21 -> 20
- 22, 23 -> 24
- 25, 26 -> 24 or 30 (context-dependent)
- 27, 28, 29 -> 30
- 32, 34 -> 30 or 36
- 38 -> 36
- 40, 42 -> 36 or 48
- 52 -> 48
- 64 -> 60 or 72

---

## 3. Font weights

| Class             | Value |
| ----------------- | ----- |
| `font-thin`       | 100   |
| `font-extralight` | 200   |
| `font-light`      | 300   |
| `font-normal`     | 400   |
| `font-medium`     | 500   |
| `font-semibold`   | 600   |
| `font-bold`       | 700   |
| `font-extrabold`  | 800   |
| `font-black`      | 900   |

### Allowed values

`100, 200, 300, 400, 500, 600, 700, 800, 900`

Always multiples of 100 in the 100-900 range. No intermediate weights.

---

## 4. Line heights

### Named multipliers (`leading-*`)

| Class             | Value |
| ----------------- | ----- |
| `leading-none`    | 1     |
| `leading-tight`   | 1.25  |
| `leading-snug`    | 1.375 |
| `leading-normal`  | 1.5   |
| `leading-relaxed` | 1.625 |
| `leading-loose`   | 2     |

### Allowed multipliers

`1, 1.25, 1.375, 1.5, 1.625, 2`

Numeric `leading-{n}` also exists (uses the spacing scale, producing rem-based line heights). Prefer named multipliers for consistency.

### Off-scale snap targets

- 1.1, 1.15, 1.2 -> 1.25
- 1.3 -> 1.25 or 1.375
- 1.4 -> 1.375 or 1.5
- 1.6 -> 1.625
- 1.7, 1.75, 1.8 -> 1.625 or 2

---

## 5. Letter spacing

| Class              | Value (em) |
| ------------------ | ---------- |
| `tracking-tighter` | -0.05      |
| `tracking-tight`   | -0.025     |
| `tracking-normal`  | 0          |
| `tracking-wide`    | 0.025      |
| `tracking-wider`   | 0.05       |
| `tracking-widest`  | 0.1        |

### Allowed values (em)

`-0.05, -0.025, 0, 0.025, 0.05, 0.1`

---

## 6. Border radius

Tailwind v4:

| Class          | rem   | px   |
| -------------- | ----- | ---- |
| `rounded-none` | 0     | 0    |
| `rounded-xs`   | 0.125 | 2    |
| `rounded-sm`   | 0.25  | 4    |
| `rounded-md`   | 0.375 | 6    |
| `rounded-lg`   | 0.5   | 8    |
| `rounded-xl`   | 0.75  | 12   |
| `rounded-2xl`  | 1     | 16   |
| `rounded-3xl`  | 1.5   | 24   |
| `rounded-4xl`  | 2     | 32   |
| `rounded-full` | --    | 9999 |

### Allowed pixel values

`0, 2, 4, 6, 8, 12, 16, 24, 32, 9999`

Asymmetric arrays (`[0,8,8,0]`, `[12,0,0,12]`, etc.) are valid for single-side rounding (`rounded-r-*`, `rounded-l-*`, `rounded-t-*`, `rounded-b-*`).

### v3 to v4 rename notice

- `rounded-sm` (2px in v3) -> `rounded-xs` in v4.
- `rounded` (4px in v3) -> `rounded-sm` in v4.

When migrating a v3 codebase, apply the rename.

### Off-scale snap targets

- 3 -> 4
- 5, 7 -> 6
- 10 -> 8 or 12
- 14 -> 12 or 16
- 20 -> 16 or 24
- 28 -> 24 or 32; if width == 2x radius -> 9999 (pill/circle)

---

## 7. Border widths

| Class      | px  |
| ---------- | --- |
| `border-0` | 0   |
| `border`   | 1   |
| `border-2` | 2   |
| `border-4` | 4   |
| `border-8` | 8   |

Side-specific variants: `border-t-*`, `border-r-*`, `border-b-*`, `border-l-*`, `border-x-*`, `border-y-*`.

### Allowed pixel values

`0, 1, 2, 4, 8`

1px hairlines are permitted as an exception to the 4px spacing grid.

---

## 8. Opacity

| Class         | Value |
| ------------- | ----- |
| `opacity-0`   | 0     |
| `opacity-5`   | 0.05  |
| `opacity-10`  | 0.10  |
| `opacity-15`  | 0.15  |
| `opacity-20`  | 0.20  |
| `opacity-25`  | 0.25  |
| `opacity-30`  | 0.30  |
| `opacity-35`  | 0.35  |
| `opacity-40`  | 0.40  |
| `opacity-45`  | 0.45  |
| `opacity-50`  | 0.50  |
| `opacity-55`  | 0.55  |
| `opacity-60`  | 0.60  |
| `opacity-65`  | 0.65  |
| `opacity-70`  | 0.70  |
| `opacity-75`  | 0.75  |
| `opacity-80`  | 0.80  |
| `opacity-85`  | 0.85  |
| `opacity-90`  | 0.90  |
| `opacity-95`  | 0.95  |
| `opacity-100` | 1     |

### Allowed values

Multiples of 5 from 0 through 100 (percentages), equivalently 0.00 through 1.00 in increments of 0.05.

---

## 9. Box shadow (elevation)

| Class          | Semantics                    |
| -------------- | ---------------------------- |
| `shadow-none`  | No shadow                    |
| `shadow-sm`    | Subtle elevation             |
| `shadow`       | Default elevation            |
| `shadow-md`    | Medium elevation             |
| `shadow-lg`    | Large elevation              |
| `shadow-xl`    | Extra large elevation        |
| `shadow-2xl`   | Double extra large elevation |
| `shadow-inner` | Inset shadow                 |

Tailwind emits specific `box-shadow` CSS values for each named step. The plugin treats these as the canonical seven elevation levels plus the inner variant.

### Allowed levels

`none, sm, default, md, lg, xl, 2xl, inner`

Do not emit custom shadow definitions unless a brand requirement justifies it.

---

## 10. Z-index

| Class    | Value |
| -------- | ----- |
| `z-0`    | 0     |
| `z-10`   | 10    |
| `z-20`   | 20    |
| `z-30`   | 30    |
| `z-40`   | 40    |
| `z-50`   | 50    |
| `z-auto` | auto  |

### Allowed values

`0, 10, 20, 30, 40, 50, auto`

Use only these layers. If higher values seem necessary, refactor the stacking context before reaching for a custom z-index.

---

## 11. Breakpoints

### Responsive prefixes

| Prefix    | rem | px   |
| --------- | --- | ---- |
| (default) | --  | 0+   |
| `sm:`     | 40  | 640  |
| `md:`     | 48  | 768  |
| `lg:`     | 64  | 1024 |
| `xl:`     | 80  | 1280 |
| `2xl:`    | 96  | 1536 |

Mobile design base: 375 px (iPhone SE / standard phone portrait). Tailwind emits no prefix for this range (default).

### Device-label mapping

Semantic device labels bound to each breakpoint for frame naming (see pencil-design R4) and human-readable queries:

| Device label | Breakpoint token | Width (px) | Typical target                      |
| ------------ | ---------------- | ---------- | ----------------------------------- |
| `mobile`     | `base`           | 375        | Phone portrait (iPhone baseline)    |
| `mobile-lg`  | `sm`             | 640        | Large phone / small tablet portrait |
| `tablet`     | `md`             | 768        | Tablet portrait (iPad baseline)     |
| `laptop`     | `lg`             | 1024       | Tablet landscape / small laptop     |
| `desktop`    | `xl`             | 1280       | Standard desktop / laptop           |
| `desktop-lg` | `2xl`            | 1536       | Large desktop / wide monitor        |

### Allowed viewport widths

`375, 640, 768, 1024, 1280, 1536`

Anything outside these six is off-scale for a viewport/screen frame.

---

## Synchronization note

This file is maintained as three identical copies, one per consumer:

- `agents/references/tailwind-scale.md` (consumed by `pencil-auditor`)
- `skills/pencil-design/references/tailwind-scale.md` (consumed by `/pencil-design`)
- `skills/pencil-to-code/references/tailwind-scale.md` (consumed by `/pencil-to-code`)

When the scale changes, update all three copies in the same commit. Divergence between copies is a bug.
