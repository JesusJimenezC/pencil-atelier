# Worked example - PricingTier card

End-to-end pass of `/pencil-design` over a single component. Demonstrates the MCP retrieval pipeline, the Semantic brief, and how R1-R4 bind to concrete tool calls. Pixel values and hex codes are illustrative; the shapes of the calls are what matters.

---

## The prompt

> "Add a pricing tier card to our marketing page library. Show plan name, price, feature list, and a primary CTA. Three tiers: Free, Pro, Enterprise. Keep it consistent with our existing design language - we use Manrope, teal accent, soft rounded corners, flat surfaces with light borders."

---

## Step A - Understand the brief (pipeline 1, 2)

```
get_editor_state({ include_schema: false })
  -> activeFile: "design/marketing.pen"
  -> selection: []
  -> topLevel: ["Home__desktop-xl-1280", "Home__mobile-base-375", "About__desktop-xl-1280", "PricingTier" (none yet)]

get_guidelines({ category: "typography" })
  -> "Manrope, weight range 400-700, letter-spacing 0.01em on headings."
```

Brief references existing tokens and an established visual language. Task scope: one reusable component, three instance variants. No viewport frame → **R4 does not apply** here.

---

## Step B - Capture the semantic brief

```
Atmosphere: restrained, confident, editorial
Palette roles:
  brand   = $brand        (#0F766E) [exists]
  bg      = $bg           (#FFFFFF) [exists]
  surface = $surface      (#FAFAF9) [exists]
  fg      = $fg           (#1F2937) [exists]
  muted   = $muted        (#6B7280) [exists]
  border  = $border       (#E5E7EB) [exists]
  accent  = $brand-bg     (#CCFBF1) [missing -> declare as soft teal wash]
Geometry: soft -> radii from {8, 12}
Depth: flat -> shadows: none; rely on $border for separation
Typography: geometric -> Manrope; body 400, heading 600, price 700
```

One missing role identified: `$brand-bg` for the "featured" tier's soft background tint. Record it; declare in Step C.

---

## Step C - Pre-flight tokens (pipeline 3, 4, 7)

```
get_variables(filePath: "design/marketing.pen")
  -> $brand=#0F766E, $bg=#FFFFFF, $surface=#FAFAF9, $fg=#1F2937,
     $muted=#6B7280, $border=#E5E7EB,
     $spacing-xs=4, $spacing-sm=8, $spacing-md=16, $spacing-lg=24, $spacing-xl=32,
     $radius-sm=8, $radius-md=12, $radius-full=9999,
     $font-sm=14, $font-base=16, $font-lg=18, $font-2xl=24, $font-4xl=36

search_all_unique_properties({
  parents: ["Home__desktop-xl-1280", "About__desktop-xl-1280"],
  properties: ["fillColor","textColor","cornerRadius","padding","gap","fontSize"]
})
  -> fillColor: {$bg, $surface, $brand}
  -> cornerRadius: {8, 12}
  -> padding: {16, 24, 32}
  -> gap: {8, 16, 24}
  -> fontSize: {14, 16, 18, 24, 36}
```

All values in use belong to the R3 scale and the existing token vocabulary. The brief's palette roles match what is already declared **except** `$brand-bg`. Declare it:

```
set_variables({
  filePath: "design/marketing.pen",
  variables: { "$brand-bg": { light: "#CCFBF1", dark: "#134E4A" } }
})
```

---

## Step D - Locate canvas space (pipeline 6)

Component library area, not viewport. Dispatch navigator:

```
Task(pencil-navigator, shape: find-empty-canvas, width: 360, height: 520)
  -> EMPTY: x=2800 y=200 w=360 h=520
```

Three tiers side by side with 24 px gap → total ~1200 px wide canvas slot. Pick anchor `x=2800 y=200`.

---

## Step E - Compose with `batch_design` (pipeline 8)

Two-pass composition: (1) build the `PricingTier` origin once as a reusable component, (2) drop three `ref` instances with per-instance overrides.

### Pass 1 - the origin

```
pt = I("<canvasId>", {
  name: "PricingTier",
  reusable: true,
  layout: "flex", direction: "column", gap: "$spacing-lg",
  padding: ["$spacing-xl","$spacing-xl","$spacing-xl","$spacing-xl"],
  width: 360, height: 520,
  fillColor: "$bg",
  cornerRadius: "$radius-md",
  strokeColor: "$border", strokeWidth: 1
})

head = I(pt, {
  name: "Head",
  layout: "flex", direction: "column", gap: "$spacing-sm"
})
I(head, { name: "PlanName", text: "Plan", fontSize: "$font-lg", fontWeight: 600, textColor: "$fg", slot: ["text"] })
I(head, { name: "Tagline",  text: "One-line description", fontSize: "$font-sm", textColor: "$muted", slot: ["text"] })

price = I(pt, {
  name: "Price",
  layout: "flex", direction: "row", gap: "$spacing-xs", align: "baseline"
})
I(price, { name: "Currency", text: "$", fontSize: "$font-2xl", fontWeight: 600, textColor: "$fg" })
I(price, { name: "Amount",   text: "0",  fontSize: "$font-4xl", fontWeight: 700, textColor: "$fg", slot: ["text"] })
I(price, { name: "Period",   text: "/mo", fontSize: "$font-sm", textColor: "$muted" })

features = I(pt, {
  name: "Features",
  layout: "flex", direction: "column", gap: "$spacing-sm",
  slot: ["children"]
})
I(features, { name: "FeatureItem", text: "Feature", fontSize: "$font-base", textColor: "$fg" })

I(pt, {
  name: "CTA",
  layout: "flex", align: "center", justify: "center",
  padding: ["$spacing-sm","$spacing-lg","$spacing-sm","$spacing-lg"],
  fillColor: "$brand", textColor: "$bg",
  cornerRadius: "$radius-sm",
  text: "Get started",
  fontSize: "$font-base", fontWeight: 600,
  slot: ["text"]
})
```

Every value is a token. `slot: [...]` marks the variable content for refs to override.

### Pass 2 - three instances (refs with overrides)

```
free = C(pt, "<canvasId>", {
  name: "PricingTier__free",
  x: 2800, y: 200,
  descendants: {
    "PlanName": { text: "Free" },
    "Tagline": { text: "For hobby projects" },
    "Amount": { text: "0" },
    "Features": { children: ["Up to 3 projects","Community support","1 GB storage"] },
    "CTA": { text: "Start free" }
  }
})

pro = C(pt, "<canvasId>", {
  name: "PricingTier__pro",
  x: 3184, y: 200,
  fillColor: "$brand-bg",
  descendants: {
    "PlanName": { text: "Pro" },
    "Tagline": { text: "For professionals" },
    "Amount": { text: "19" },
    "Features": { children: ["Unlimited projects","Priority support","50 GB storage","Custom domains"] },
    "CTA": { text: "Upgrade to Pro" }
  }
})

ent = C(pt, "<canvasId>", {
  name: "PricingTier__enterprise",
  x: 3568, y: 200,
  descendants: {
    "PlanName": { text: "Enterprise" },
    "Tagline": { text: "For teams at scale" },
    "Amount": { text: "99" },
    "Features": { children: ["SSO and SCIM","24/7 dedicated support","Unlimited storage","SLA"] },
    "CTA": { text: "Contact sales" }
  }
})
```

Featured tier (`Pro`) overrides `fillColor` to `$brand-bg` - the only palette-role token we declared in Step C. That single override expresses the brief's "draw attention to the featured tier" intent without introducing any off-role color.

---

## Step F - Verify visually (pipeline 9)

```
get_screenshot(nodeId: "<canvasId>", bounds: { x: 2800, y: 200, w: 1152, h: 520 })
```

Confirm: three cards, equal widths, 24 px gaps, flat surfaces with hairline borders, Pro tier sits on soft teal wash, CTA reads as the loudest element on each card.

If the Pro wash reads too loud or the border on Free/Enterprise disappears against the page bg, iterate inside `batch_design` - do not return control to the user on a visual mismatch.

---

## Step G - Self-check against the rules

- [x] **R1** - every color, spacing, radius, font-size cites a token. Zero raw hex in the batch.
- [x] **R2** - extracted on the second repetition (tier #2 triggered the origin; tiers #1, #3 are refs). Three total instances all point at one origin.
- [x] **R3** - every numeric property (paddings 8/16/24/32, radii 8/12, font sizes 14/16/18/24/36) belongs to the scale and to the `soft` geometry subset chosen in the brief.
- [x] **R4** - not applicable; no viewport frame in this pass.
- [x] **Brief** - flat depth (no shadows, borders only), geometric Manrope, teal accent used on CTA + the featured-tier wash only. Every visual choice traces back to the brief.

Clean pass. Hand off.

---

## What this example is deliberately not doing

- No shadows (brief says `flat`). Adding even a `shadow-sm` on hover would violate the depth axis.
- No fourth tier "just in case". Scope is three tiers; composition stops there.
- No per-tier custom font-size. Typography axis is `geometric` uniform; visual hierarchy comes from weight and size *within the declared scale*, not from ad-hoc per-instance values.
- No absolute positioning inside the card. All layout is flex with spacing-scale gaps and paddings.
- No new radius value like `10` because "it looked nicer than 8 or 12". The brief's `soft` slice is `{8, 12}`; anything outside requires re-negotiating the brief.
