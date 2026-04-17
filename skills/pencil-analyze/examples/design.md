# Design System: Marketing Site

A product marketing surface for a developer-tools SaaS. Hero + feature grid + pricing + documentation entry points. Designed mobile-first; desktop layouts are restrained and editorial rather than dense.

## 1. Visual Theme & Atmosphere

The marketing site reads as **restrained, confident, editorial**. Whitespace does the heavy lifting - large section paddings (64-96 px) frame content without competing with it. Typography is geometric and uniform; visual hierarchy emerges from weight and size, not from color or decoration. The single teal accent is reserved for primary actions and the featured pricing tier, so it retains its signal value wherever it appears.

**Key Characteristics:**
- Flat surfaces with 1 px hairline borders; no shadows in the default state
- Soft rounded corners (8-12 px) across every card and input
- Single brand accent used sparingly; neutrals dominate
- Editorial typographic hierarchy with generous line-height on long copy
- Mobile-first grid with restrained desktop expansion (1280 max content width)

## 2. Color Palette & Roles

### Primary Foundation
- **$bg** (`#FFFFFF`) - Page background. Pure white for maximum legibility on long-form marketing copy. *Used in 4 viewport frames.*
- **$surface** (`#FAFAF9`) - Secondary surface for cards and content blocks. Barely-warm off-white that establishes layering without visual weight. *Used in 14 frames.*

### Accent & Interactive
- **$brand** (`#0F766E`) - Deep muted teal. The single vibrant color in the palette. Reserved for primary CTAs, active navigation, and the featured pricing tier border. *Used in 9 frames.*
- **$brand-bg** (`#CCFBF1`) - Soft teal wash, exclusive to the featured pricing tier background. A decision that expresses hierarchy without adding a second brand color. *Used in 1 frame.*

### Typography & Text Hierarchy
- **$fg** (`#1F2937`) - Primary text. Charcoal near-black, softer than pure black. *Used in 22 frames.*
- **$muted** (`#6B7280`) - Secondary text for descriptions, metadata, captions. *Used in 15 frames.*

### Structural
- **$border** (`#E5E7EB`) - Hairline borders for cards, inputs, dividers. Subtle separation in lieu of shadows. *Used in 11 frames.*

### Functional States
- **$success** (`#10B981`) - Availability indicators, positive confirmations.
- **$danger** (`#EF4444`) - Error states, destructive actions.

### Unclear or Missing Tokens

_None. All colors in frames map to declared tokens; all declared tokens have a clear role._

## 3. Typography

**Primary family:** Manrope
**Character:** geometric - modern, even letterforms with slight humanist warmth
**Weight range:** 400, 500, 600, 700
**Font sizes in use:** 14, 16, 18, 24, 36 px

### Hierarchy

- **H1 (hero)** - 36 px, weight 700, line-height 1.25. Reserved for page-level hero copy.
- **H2 (section)** - 24 px, weight 600, line-height 1.25. Establishes content zones.
- **H3 (component title)** - 18 px, weight 600, line-height 1.375. Used on card heads and pricing-tier names.
- **Body** - 16 px, weight 400, line-height 1.625. Long-form prose and descriptions.
- **Small / Meta** - 14 px, weight 400, line-height 1.5. Captions, metadata, helper text.
- **CTA** - 16 px, weight 600. Sits inside primary and secondary buttons.

Line-height 1.625 on body copy reinforces the editorial atmosphere; tighter 1.25 on headings keeps hero composition compact.

## 4. Geometry & Depth

**Geometry character:** soft - radii in use: 8 px, 12 px
**Depth character:** flat - shadows in use: (none), borders in use: 1 px

Every card, input, pricing tier, and button corner sits in the 8-12 px radius slice. The design never mixes `subtle` (4 px) or `generous` (16-24 px) elements - the soft slice is applied uniformly, which is the main source of visual coherence across otherwise varied components.

Depth is handled entirely through hairline borders against the two near-white surfaces. No shadows exist in the design. Separation reads as *architectural* (planes meeting at a line) rather than *atmospheric* (layers floating over each other). This is the single most consequential atmosphere commitment - changing depth later would require revisiting every component.

## 5. Component Stylings

- **PricingTier** (origin) - 3 refs. Column layout: plan name (H3), tagline (meta), price (H1-like 36 px), feature list, CTA. Featured variant overrides background to `$brand-bg`; otherwise identical to siblings.
- **FeatureCard** (origin) - 6 refs. Icon + H3 + short description. Used in the feature grid on Home.
- **NavItem** (origin) - 5 refs. Label + optional active-indicator underline in `$brand`.
- **Button** (origin) - 12 refs. Primary variant on `$brand`; secondary variant outlined with `$border`. Both use 8 px radius and 16 px body weight 600.
- **Input** (origin) - 4 refs. 1 px border, 8 px radius, focus state shifts border to `$brand`.
- **SectionHeading** (origin) - 4 refs. H2 + eyebrow line + optional description. Anchors each marketing section.

## 6. Layout & Scale

**Spacing values in use:** 4, 8, 12, 16, 24, 32, 48, 64, 96 px
**Breakpoints designed:** mobile (375 px), tablet (768 px), desktop (1280 px)

Section padding on desktop is 64-96 px; component interior padding is 24-32 px; element gaps inside flex containers are 8-16 px. Page edge padding scales 24 → 48 → 48 px across the three breakpoints. Hero sections use the largest padding in the system (96 px top and bottom) to establish their importance.

### Scale Parity

- spacing: 9 distinct values - all on-scale
- fontSize: 5 distinct values - all on-scale
- cornerRadius: 2 distinct values - all on-scale
- lineHeight: 3 distinct values - all on-scale
- breakpoints: 3 distinct values - all on-scale

## 7. Token Vocabulary

| Token | Role | Value | Uses |
|-------|------|-------|------|
| `$bg` | bg | `#FFFFFF` | 4 |
| `$surface` | surface | `#FAFAF9` | 14 |
| `$brand` | brand | `#0F766E` | 9 |
| `$brand-bg` | brand-bg | `#CCFBF1` | 1 |
| `$fg` | fg | `#1F2937` | 22 |
| `$muted` | muted | `#6B7280` | 15 |
| `$border` | border | `#E5E7EB` | 11 |
| `$success` | success | `#10B981` | 1 |
| `$danger` | danger | `#EF4444` | 1 |
| `$spacing-xs` | spacing | 4 | 8 |
| `$spacing-sm` | spacing | 8 | 19 |
| `$spacing-md` | spacing | 16 | 28 |
| `$spacing-lg` | spacing | 24 | 14 |
| `$spacing-xl` | spacing | 32 | 9 |
| `$spacing-2xl` | spacing | 48 | 5 |
| `$spacing-3xl` | spacing | 64 | 3 |
| `$spacing-4xl` | spacing | 96 | 2 |
| `$radius-sm` | radius | 8 | 16 |
| `$radius-md` | radius | 12 | 6 |
| `$font-sm` | font | 14 | 10 |
| `$font-base` | font | 16 | 24 |
| `$font-lg` | font | 18 | 7 |
| `$font-2xl` | font | 24 | 4 |
| `$font-4xl` | font | 36 | 2 |

## 8. Usage notes for downstream skills

- `/pencil-design` should honor the soft geometry slice (`{8, 12}`) - do not introduce 16 or 24 px radii without revisiting this brief.
- `/pencil-design` should honor the flat depth commitment - do not introduce shadows without revisiting this brief.
- `/pencil-to-code` should emit role-named CSS variables or Tailwind theme entries from the token vocabulary - do not hardcode hex values from this brief into components.
