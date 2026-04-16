---
topic: design
role: reference
scope: spacing
audience: ui-designer
---

# Spacing and Rhythm

Modular spacing scales, logical properties, container queries, subgrid, modern viewport units, intrinsic sizing, fluid spacing, vertical rhythm. Reference for every spatial decision.



## 1. Modular spacing scale

A four-pixel base unit (`0.25rem`) is the dominant 2026 default — every modern UI framework converges on it (Tailwind, Radix, Material 3, Polaris, Carbon).

| Step | Tailwind class | rem | px (16-base) | Use |
|---|---|---|---|---|
| 0 | `gap-0` | `0` | 0 | Reset |
| 1 | `gap-1` | `0.25rem` | 4 | Icon-to-text gap |
| 2 | `gap-2` | `0.5rem` | 8 | Inline form spacing |
| 3 | `gap-3` | `0.75rem` | 12 | Tight stack |
| 4 | `gap-4` | `1rem` | 16 | Card padding (compact) |
| 5 | `gap-5` | `1.25rem` | 20 | Card padding (default) |
| 6 | `gap-6` | `1.5rem` | 24 | Section gap (small) |
| 8 | `gap-8` | `2rem` | 32 | Section gap (default) |
| 10 | `gap-10` | `2.5rem` | 40 | Section gap (large) |
| 12 | `gap-12` | `3rem` | 48 | Major section break |
| 16 | `gap-16` | `4rem` | 64 | Page section gap |
| 20 | `gap-20` | `5rem` | 80 | Hero padding (mobile) |
| 24 | `gap-24` | `6rem` | 96 | Hero padding (desktop) |

Tailwind v4 default scale uses `--spacing: 0.25rem` and lets every utility take any positive multiple (`p-13` works). Override the base unit in `@theme` if your design system standardizes on a different rhythm:

```css
@import "tailwindcss";

@theme {
  --spacing: 0.25rem;       /* default */
  /* OR a denser data-app rhythm: */
  /* --spacing: 0.125rem; */ /* 2px base */
}
```

## 2. Logical properties for i18n safety

Every directional property has a writing-mode-aware equivalent. Use these by default — RTL languages (Arabic, Hebrew, Farsi) and vertical CJK get correct visual flow with zero markup change.

| Physical | Logical |
|---|---|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` | `margin-block-start` |
| `margin-bottom` | `margin-block-end` |
| `padding-left/right` | `padding-inline-start` / `padding-inline-end` |
| `padding-left` + `padding-right` | `padding-inline` (shorthand) |
| `padding-top` + `padding-bottom` | `padding-block` |
| `width` | `inline-size` |
| `max-width` | `max-inline-size` |
| `height` | `block-size` |
| `text-align: left` | `text-align: start` |
| `border-left` | `border-inline-start` |
| `inset: 0 0 0 0` | `inset-block: 0; inset-inline: 0` |

```css
/* Old (LTR-only) */
.card { padding: 1rem 1.5rem; max-width: 65ch; margin-left: auto; }

/* New (writing-mode-aware) */
.card {
  padding-block: 1rem;
  padding-inline: 1.5rem;
  max-inline-size: 65ch;
  margin-inline-start: auto;
}
```

Reference: [MDN Logical properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values).

## 3. Container queries over media queries

Container queries respond to a parent's size, not the viewport. The same component lays out correctly whether dropped in a sidebar, a 3-column grid, or a full-bleed hero. The single biggest layout shift of 2025-2026.

```css
.card-host {
  container-type: inline-size;
  container-name: card;
}

/* Default (narrow) */
.card { display: grid; grid-template-columns: 1fr; gap: 1rem; }

@container card (min-width: 24rem) {
  .card { grid-template-columns: 8rem 1fr; }
}

@container card (min-width: 40rem) {
  .card { grid-template-columns: 12rem 1fr 8rem; }
}
```

```html
<aside class="card-host"><Card/></aside>          <!-- 1fr layout -->
<main class="card-host"><Card/></main>            <!-- 12rem 1fr 8rem layout -->
```

| Container query type | Detail |
|---|---|
| Size queries | `@container (min-width: ...)` — geometry-driven |
| Style queries | `@container style(--theme: dark) { ... }` — value-driven (Chrome 111+) |
| Container units | `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax` — sized to the container, not viewport |

`@container card` is the named form; unnamed `@container` matches the closest containment ancestor. References: [MDN @container](https://developer.mozilla.org/en-US/docs/Web/CSS/@container), [web.dev container queries](https://web.dev/learn/css/container-queries/).

## 4. Subgrid for shared alignment

When grandchildren must align to the parent grid, `grid-template-rows: subgrid` and `grid-template-columns: subgrid` propagate the parent's tracks. Browser baseline: 2024 (all evergreen).

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(18rem, 1fr));
  grid-auto-rows: min-content;
  gap: 1.5rem;
}

.card {
  grid-row: span 4;            /* claim 4 rows of the parent */
  display: grid;
  grid-template-rows: subgrid; /* inherit those 4 row tracks */
  gap: 0.5rem;
}

.card > .image    { grid-row: 1; }
.card > .title    { grid-row: 2; }
.card > .body     { grid-row: 3; }
.card > .actions  { grid-row: 4; }
```

Result: every card's image/title/body/actions sit at the same Y position across the row, regardless of body length. Without subgrid you need uniform content lengths or JS height-syncing.

Reference: [MDN Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid).

## 5. Modern viewport units

| Unit | Meaning | Use |
|---|---|---|
| `vh` | 1% of viewport height | Avoid on mobile — fixed at "largest" causes content under URL bar |
| `svh` | 1% of *small* viewport (URL bar shown) | Conservative full-height; never overflows |
| `lvh` | 1% of *large* viewport (URL bar hidden) | Hero with parallax that should fill when bars retract |
| `dvh` | 1% of *dynamic* viewport (recalcs) | Default for full-screen modals, sticky panels |
| `dvi` / `dvb` | Dynamic inline / block | Writing-mode-aware versions |

```css
/* Full-screen modal — adjusts as the URL bar shows/hides */
.modal {
  position: fixed;
  inset: 0;
  block-size: 100dvh;
  inline-size: 100dvw;
}

/* Hero that's always at least short-viewport-tall */
.hero { min-block-size: 100svh; }
```

The mobile address-bar trap: a `100vh` element overflows by ~60-90px on iOS Safari when the bar reappears. `100dvh` recalculates; `100svh` is the safe lower bound. Reference: [MDN viewport units](https://developer.mozilla.org/en-US/docs/Web/CSS/length#relative_length_units_based_on_viewport).

## 6. Intrinsic sizing

Let content drive size. Avoid `width: 100%` everywhere.

| Function | Behavior |
|---|---|
| `min-content` | Smallest size content can be without overflow (longest unbreakable token) |
| `max-content` | Size needed to fit content on one line |
| `fit-content(<size>)` | `max(min-content, min(<size>, max-content))` |
| `minmax(<min>, <max>)` | Grid track range |
| `auto-fill` / `auto-fit` | Dynamic column count |

```css
/* 3-column grid that collapses to 1 column on narrow containers, no media query */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
  gap: 1rem;
}

/* Sidebar that hugs content, capped at 14rem */
.aside { inline-size: fit-content(14rem); }

/* Tag chip — never wider than its text */
.chip { inline-size: max-content; }
```

`auto-fit` collapses empty tracks; `auto-fill` reserves them. Reference: [MDN minmax](https://developer.mozilla.org/en-US/docs/Web/CSS/minmax).

## 7. Fluid spacing

`clamp()` for spacing breathes the same way `clamp()` for type does. Combined with intrinsic columns, eliminates 80% of media queries.

```css
:root {
  /* min @ 360px viewport, max @ 1280px viewport */
  --space-3xs: clamp(0.25rem, 0.21rem + 0.18vw, 0.375rem);   /*  4 ->  6 */
  --space-2xs: clamp(0.5rem,  0.42rem + 0.36vw, 0.75rem);    /*  8 -> 12 */
  --space-xs:  clamp(0.75rem, 0.63rem + 0.54vw, 1.125rem);   /* 12 -> 18 */
  --space-s:   clamp(1rem,    0.84rem + 0.72vw, 1.5rem);     /* 16 -> 24 */
  --space-m:   clamp(1.5rem,  1.26rem + 1.08vw, 2.25rem);    /* 24 -> 36 */
  --space-l:   clamp(2rem,    1.68rem + 1.43vw, 3rem);       /* 32 -> 48 */
  --space-xl:  clamp(3rem,    2.52rem + 2.15vw, 4.5rem);     /* 48 -> 72 */
  --space-2xl: clamp(4rem,    3.36rem + 2.87vw, 6rem);       /* 64 -> 96 */
  --space-3xl: clamp(6rem,    5.04rem + 4.30vw, 9rem);       /* 96 ->144 */
}

.section { padding-block: var(--space-2xl); }
.card    { padding: var(--space-s) var(--space-m); }
```

[Utopia.fyi](https://utopia.fyi) auto-generates the scale.

## 8. Vertical rhythm

Keep block spacing as multiples of a base unit (typically the body line-height). Helps content feel composed rather than stitched.

| Concept | Detail |
|---|---|
| Base unit | `body` line-height. With `font-size: 1rem` and `line-height: 1.5`, base = `1.5rem` (24px) |
| Heading line-height | `1.2`-`1.3` for tight display; never the body's `1.5` |
| Heading top margin | Multiple of base — e.g. `1.5x base` above `<h2>`, `1x base` below |
| Cap-height alignment | For tabular data, prefer `align-items: baseline` so caps line up across columns |
| Baseline grid | When prose-heavy, snap headings + body to a fixed baseline; `line-height-step` in spec, no browser yet |
| em vs rem | `rem` for scale (page-level), `em` for element-relative spacing (icon-to-text) |

```css
html { font-size: 100%; line-height: 1.5; }   /* base = 24px */

h2 {
  font-size: var(--text-3xl);
  line-height: 1.2;
  margin-block-start: calc(var(--space-l) * 1);
  margin-block-end:   calc(var(--space-s) * 1);
}

p + p { margin-block-start: 1lh; }  /* one line-height between paragraphs */
```

`lh` and `rlh` units (Chrome 110+, Safari 16.4+) tie spacing to actual computed line-height — better than hard-coded rems for content blocks.

## 9. Anti-patterns

| Anti-pattern | Why | Replace with |
|---|---|---|
| `margin-top: 23px` | Magic number, not on the scale | Pick the nearest spacing token |
| Centered everything default (`text-align: center` on body) | Reads as marketing, not as product | Left-align body, center only display headlines |
| 64px grids on mobile | Eats the screen | Fluid spacing — `clamp(2rem, 6vw, 4rem)` |
| Asymmetric padding without intent | Visual instability | Symmetric block + inline padding unless deliberately weighted |
| `width: 100%` everywhere | Defeats intrinsic sizing | Default flow, use `max-inline-size` for caps |
| `vh` on mobile full-height surfaces | Overflows when URL bar retracts | `dvh` (default) or `svh` (conservative) |
| Viewport media queries on a card | Card layout depends on its container, not the page | Container queries with `@container card (min-width: ...)` |
| Nested grids with manual height-syncing JS | Fragile, jank-prone | `grid-template-rows: subgrid` |
| Pixel padding inside fluid type | Whitespace breaks at zoom | Use `em` for icon-to-text gap, `rem` for layout |
| Single global breakpoint set (`sm/md/lg/xl`) on every component | Couples components to viewport | Container queries on portable components, viewport queries only at page shell |
| Forgetting `gap` exists | `margin` between siblings rebuilds gap manually | `display: flex; gap: var(--space-s);` |

( for the full layout decision tree.)
