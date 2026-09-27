---
name: design-tokens
description: How to design and implement a tokenized design system using the W3C DTCG spec (stable since Oct 2025) with a three-tier architecture (primitive → semantic → component). Used by the designer to author tokens and by the engineer to wire them into Tailwind v4 + shadcn/ui. The single source of truth for colors, spacing, typography, radii, shadows, and motion.
---

# Design Tokens — Tokenized Design System

A design system without tokens is a styleguide with extra steps. Tokens are the **only** source of truth for design decisions; components and pages consume tokens, never raw values.

This skill enforces:

1. **W3C DTCG** specification (stable, October 2025) as the conceptual model.
2. **Three-tier architecture**: primitive → semantic → component.
3. **Tailwind v4 + CSS variables** as the runtime implementation for this project.
4. **No hardcoded design values** anywhere in the codebase.

---

## The Three Tiers

Most token systems fail at the **semantic** layer, not at primitives or components. The semantic layer is where intent lives — without it you cannot theme, you cannot maintain consistency, and changes ripple chaotically.

### Tier 1 — Primitive (raw values, no meaning)

```
color.blue.500   = oklch(0.55 0.22 250)
color.gray.900   = oklch(0.18 0.01 280)
space.4          = 16px
font.size.4      = 18px
radius.md        = 8px
```

Rules:
- Named by scale, not by usage.
- No reference to UI purpose.
- Lives in `tokens/primitive.json`.

### Tier 2 — Semantic (intent, theming layer)

```
color.bg.default         → color.gray.50
color.bg.muted           → color.gray.100
color.text.primary       → color.gray.900
color.text.secondary     → color.gray.600
color.border.default     → color.gray.200
color.action.primary     → color.blue.500
color.action.primary.hover → color.blue.600
color.feedback.error     → color.red.500
space.gutter.inline      → space.4
space.gutter.block       → space.6
radius.control           → radius.md
```

Rules:
- Named by **role**, never by appearance (`color.text.primary`, never `color.dark-gray`).
- Every semantic token references **exactly one primitive** (or another semantic in the same tier — rare, document it).
- Light and dark themes redefine the **same semantic tokens** to different primitives. Components do not change.
- Lives in `tokens/semantic.json`.

> **The test for a good semantic token:** if you renamed all primitives to nonsense (`color.frob.500`), would the semantic layer still make sense to read? If yes, the names are good.

### Tier 3 — Component (purpose-specific)

```
button.primary.bg              → color.action.primary
button.primary.bg.hover        → color.action.primary.hover
button.primary.text            → color.text.on-action
button.primary.radius          → radius.control
button.primary.padding.inline  → space.4
input.border.default           → color.border.default
input.border.focus             → color.action.primary
card.bg                        → color.bg.elevated
card.radius                    → radius.lg
card.shadow                    → shadow.sm
```

Rules:
- Created **only** when a component has bespoke needs that semantic tokens cannot express directly.
- Each references a semantic token (never a primitive).
- If two components would use the exact same component token, it probably belongs in the semantic tier.
- Lives in `tokens/component.json`.

**Default: don't create component tokens.** Reach for them only when justified.

---

## Token Categories (Minimum Coverage)

A complete system covers at least:

| Category | Primitives | Semantic examples |
|---|---|---|
| Color | scales of gray, brand, success, warning, error, info | bg.default, bg.muted, text.primary, text.secondary, border, action.primary, feedback.* |
| Spacing | 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 / 96 | gutter.inline, gutter.block, layout.gap, stack.tight, stack.loose |
| Typography | font.family.{sans,mono}, font.size.1..7, font.weight.{regular,medium,bold}, line-height.{tight,normal,loose}, tracking.* | display, h1..h4, body, body.small, caption, code |
| Radius | none, sm, md, lg, xl, full | control, surface, pill |
| Shadow | shadow.1..5 | elevation.low, elevation.medium, elevation.high |
| Z-index | z.1..z.5 | overlay, dropdown, modal, toast, tooltip |
| Motion | duration.{fast,base,slow}, easing.{standard,enter,exit} | transition.control, transition.surface |
| Breakpoints | bp.sm/md/lg/xl/2xl | (used by layout) |

Designer **must** define all categories above. Missing categories block handoff.

---

## File Layout (Designer Output)

Under `docs/<rfp-slug>/tokens/`:

```
tokens/
├── primitive.json     # tier 1
├── semantic.json      # tier 2
├── component.json     # tier 3 (may be empty)
└── README.md          # naming conventions, theme list, deviation log
```

Use the DTCG JSON format. Minimal example:

```json
{
  "color": {
    "blue": {
      "500": { "$value": "oklch(0.55 0.22 250)", "$type": "color" }
    }
  },
  "space": {
    "4": { "$value": "16px", "$type": "dimension" }
  }
}
```

Semantic tokens reference primitives:

```json
{
  "color": {
    "bg": {
      "default": { "$value": "{color.blue.50}", "$type": "color" }
    }
  }
}
```

The engineer compiles these to CSS variables.

---

## Engineer Implementation — Tailwind v4 + CSS Variables

Tokens become CSS custom properties at `:root` (light) and `.dark` (dark). Tailwind v4 exposes them via `@theme inline`.

### `app/globals.css`

```css
@import "tailwindcss";

:root {
  /* primitives — usually not exposed to Tailwind directly, but kept as variables */
  --color-blue-500: oklch(0.55 0.22 250);
  --color-gray-50: oklch(0.98 0.005 280);
  --color-gray-900: oklch(0.18 0.01 280);

  /* semantic — these are what components consume */
  --color-bg-default: var(--color-gray-50);
  --color-text-primary: var(--color-gray-900);
  --color-action-primary: var(--color-blue-500);
  --radius: 0.5rem;
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
}

.dark {
  --color-bg-default: var(--color-gray-950);
  --color-text-primary: var(--color-gray-50);
  /* semantic names stay; primitive bindings change */
}

@theme inline {
  --color-background: var(--color-bg-default);
  --color-foreground: var(--color-text-primary);
  --color-primary: var(--color-action-primary);
  --radius-sm: var(--radius-sm);
  --radius-md: var(--radius-md);
  --radius-lg: var(--radius-lg);
}
```

Now `bg-background`, `text-foreground`, `bg-primary`, `rounded-md` all resolve through the semantic layer.

### Component code

```tsx
// ✅ correct — consumes semantic token via Tailwind utility
<button className="bg-primary text-primary-foreground rounded-md px-4 py-2">

// ❌ wrong — hardcoded color, bypasses the system
<button className="bg-blue-500 text-white rounded-lg px-4 py-2">

// ❌ wrong — references primitive directly
<button style={{ background: 'var(--color-blue-500)' }}>
```

---

## Theming (Light / Dark / Brand variants)

- Add a new theme by redefining **only semantic tokens** in a new class (`.dark`, `.brand-acme`, etc.).
- Primitives stay shared.
- Components never change.

```css
.brand-acme {
  --color-action-primary: var(--color-purple-500);
  --color-action-primary-hover: var(--color-purple-600);
}
```

Switching themes is a class toggle on `<html>`.

---

## Anti-Patterns (Reject in Review)

- Tailwind classes with literal colors (`bg-blue-500`, `text-red-600`) outside of `globals.css`.
- Inline `style={{ color: '#...' }}` anywhere.
- Component code referencing primitive tokens (`var(--color-blue-500)`).
- Semantic tokens named by appearance (`color.dark-gray`) instead of role (`color.text.primary`).
- Creating a component token before checking if a semantic token suffices.
- Spacing values like `mt-[18px]` — should be `mt-4` from the scale.
- Font sizes / line heights set arbitrarily (`text-[15px]`).
- Adding a new color outside the documented scale.

---

## Definition of Done (Tokens)

- [ ] All three tier files exist; primitives and semantics are non-empty.
- [ ] Every category listed above has at least minimum coverage.
- [ ] Every semantic token references a primitive (or a documented semantic).
- [ ] Light + dark themes defined.
- [ ] `globals.css` compiled from tokens; Tailwind utilities resolve to semantic vars.
- [ ] Grep finds zero hardcoded color/spacing/radius literals in `components/` and `app/`.
- [ ] `tokens/README.md` documents naming convention and any deviations.

---

## Tooling (Optional but Recommended)

- [Style Dictionary](https://amzn.github.io/style-dictionary/) to compile DTCG JSON to CSS variables.
- [Token Studio](https://tokens.studio/) if designers work in Figma and export DTCG JSON.
- For the training: hand-authored JSON + a small script (or manual transcription) is fine.
