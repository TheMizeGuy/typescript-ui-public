---
topic: design
role: reference
scope: typography
audience: ui-designer
---

# Typography

Variable fonts, banned defaults, modular type scales, fluid `clamp()`, optical sizing, web font loading, microtypography, numeral features. Reference for any text decision in a 2026 codebase.

Source vault: `~/Claude/vault/UI Design/04 - Typography Best Practices.md`.

## 1. Variable fonts are the default

One file ships infinite weights along named axes. Smaller payload than 3+ static weights, smoother weight interpolation, runtime control via `font-variation-settings`. ~95% global support.

| Axis | Tag | Range | Use |
|---|---|---|---|
| Weight | `wght` | 1-1000 | Smooth weight transitions; hover/emphasis |
| Width | `wdth` | 75-125 typical | Condensed for tight headlines |
| Italic | `ital` | 0 or 1 | True italic, not slanted roman |
| Slant | `slnt` | -90..90 (deg) | Continuous oblique |
| Optical size | `opsz` | font-defined | Auto-adjust contrast/spacing for size |

```css
:root {
  --font-display: "Geist", system-ui, sans-serif;
  --font-body:    "Geist", system-ui, sans-serif;
  --font-mono:    "Geist Mono", "JetBrains Mono", ui-monospace, monospace;
}

h1 {
  font-family: var(--font-display);
  /* one file, exact weight + larger optical size */
  font-variation-settings: "wght" 720, "opsz" 56;
}

body {
  font-family: var(--font-body);
  font-variation-settings: "wght" 420, "opsz" 14;
}

a:hover {
  font-variation-settings: "wght" 540;
  transition: font-variation-settings 120ms ease-out;
}
```

Reference: [MDN font-variation-settings](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variation-settings).

## 2. Banned and approved typefaces (frontend-design plugin)

Generic typefaces are AI-tells — the equivalent of starting a logo brief with Helvetica.

### Banned by default

| Family | Reason |
|---|---|
| Inter (body) | The most-used AI default. Acceptable only as Inter Display in a header *with* tightened tracking |
| Roboto | Android system font; reads as "framework demo" anywhere else |
| Arial | Generic, no point-of-view |
| Helvetica | Same — overused, no design intent |

### Acceptable defaults

| Family | Use | Why |
|---|---|---|
| Geist / Geist Mono | Body, UI, code | Vercel's neutral display sans, well-paired mono variant |
| Söhne | Body, display | Modern grotesque with character; broad family |
| Inter Display | Header only with custom tracking | Display cut tames the body Inter overuse |
| JetBrains Mono | Code | Strong figures, ligatures, distinct glyphs |
| Berkeley Mono | Code, distinctive UI | Premium proportional mono with strong personality |
| Commit Mono | Code | Free, opinionated, contemporary |
| IBM Plex Sans | Technical apps | Engineered feel, multi-script |
| Instrument Serif | Display | Editorial serif with high contrast |
| Fraunces | Display | Variable axes for `opsz`, `SOFT`, `WONK` |
| Recoleta | Display | Distinctive humanist serif |
| Söhne Mono / Söhne Schmal | Distinctive UI | Condensed/mono variants of Söhne |
| Migra | Display | Strong serif point-of-view |
| GT America / GT Walsheim / GT Sectra / GT Pressura | Custom direction | Grilli Type families with designerly voice |

Pair *one* display + *one* body. Optionally one mono. Three families is the absolute ceiling.

## 3. Type scale

Pick a modular ratio. Multiply base size by the ratio to step up; divide to step down. Higher ratios shout, lower ratios converse.

| Ratio | Name | Best for |
|---|---|---|
| 1.125 | Major Second | Dense data UIs, editor surfaces |
| 1.2 | Minor Third | Small UI, dashboards, settings panels |
| 1.25 | Major Third | General-purpose product UI |
| 1.333 | Perfect Fourth | Marketing pages, balanced content |
| 1.414 | Augmented Fourth | Strong headline systems |
| 1.5 | Perfect Fifth | Editorial, long-form, hero surfaces |

```css
/* Minor third (1.2) starting at 16px */
--text-xs:  0.694rem;   /* 11.1px */
--text-sm:  0.833rem;   /* 13.3px */
--text-base: 1rem;      /* 16px */
--text-lg:  1.2rem;     /* 19.2px */
--text-xl:  1.44rem;    /* 23px */
--text-2xl: 1.728rem;   /* 27.6px */
--text-3xl: 2.074rem;   /* 33.2px */
--text-4xl: 2.488rem;   /* 39.8px */
--text-5xl: 2.986rem;   /* 47.8px */
```

Tooling: [Type Scale](https://typescale.com), [Utopia](https://utopia.fyi).

## 4. Fluid typography with `clamp()`

`clamp(min, preferred, max)` interpolates linearly between viewport breakpoints with no media queries and no resize listener.

Formula: `preferred = baseRem + slope * 1vw`
- `slope = (maxPx - minPx) / (maxViewportPx - minViewportPx) * 100`
- `baseRem = (minPx - slopeAtMinViewport) / 16`

```css
/* 16px @ 360vw -> 20px @ 1280vw */
--step-0: clamp(1rem, 0.875rem + 0.625vw, 1.25rem);     /* body */
--step-1: clamp(1.2rem, 1rem + 1vw, 1.5rem);            /* h4 */
--step-2: clamp(1.44rem, 1.1rem + 1.7vw, 1.875rem);     /* h3 */
--step-3: clamp(1.728rem, 1.2rem + 2.6vw, 2.34rem);     /* h2 */
--step-4: clamp(2.074rem, 1.3rem + 3.9vw, 2.93rem);     /* h1 */
--step-5: clamp(2.488rem, 1.5rem + 4.9vw, 3.66rem);     /* hero */

h1 { font-size: var(--step-4); }
```

WCAG note: ensure max isn't capped so low that 200% browser zoom can't scale text proportionally ([WCAG 2.2 1.4.4](https://www.w3.org/TR/WCAG22/#resize-text)).

Reference: [web.dev min-max-clamp](https://web.dev/articles/min-max-clamp).

## 5. Optical sizing axis

`opsz` adjusts contrast, x-height, and spacing for the rendered size. Display cuts have higher contrast (thin thins, sharp serifs); text cuts have lower contrast and slightly looser spacing for legibility at body size.

| Use | `opsz` value (typical) |
|---|---|
| Hero `<h1>` 56px+ | `opsz` 48-72 (display cut) |
| Section heading 32-44px | `opsz` 28-36 |
| Card title 18-24px | `opsz` 18-24 |
| Body 14-18px | `opsz` 14 (text cut) |
| Caption 11-13px | `opsz` 11 |

```css
/* Auto mode lets the browser pick opsz based on font-size */
:root { font-optical-sizing: auto; }

/* Manual override on a hero */
.hero-title {
  font-size: 4rem;
  font-variation-settings: "wght" 700, "opsz" 72;
}
```

Fonts with strong `opsz`: Fraunces, Roboto Flex, Source Serif 4, Recursive, Inter Display.

## 6. Web font loading

| Strategy | Use when | Tradeoff |
|---|---|---|
| `font-display: swap` | Default | Visible FOUT; LCP unaffected |
| `font-display: optional` | Best CWV | Some users never see the web font |
| `<link rel="preload" as="font" type="font/woff2" crossorigin>` | 1-2 critical fonts | More than 2 wastes bandwidth |
| `next/font` (Next.js) | Next.js app | Self-hosts, subsets, generates `@font-face`, sets `size-adjust` |
| `@font-face` with `size-adjust` and `ascent-override` | Match system font metrics | Eliminates layout shift on swap |
| Self-host on same origin | Always (over Google CDN) | Enables HTTP/3, Cache-Control, no third-party DNS |
| WOFF2 only | Always | 30% smaller than WOFF; >99% support; drop TTF/EOT |
| Subset by `unicode-range` | Multilingual sites | Latin-only is ~30 KB vs ~150 KB full |

```css
@font-face {
  font-family: "Geist";
  src: url("/fonts/geist-variable.woff2") format("woff2-variations");
  font-weight: 100 900;
  font-display: swap;
  font-style: normal;
  size-adjust: 100.06%;
  ascent-override: 92%;
  descent-override: 24%;
  line-gap-override: 0%;
}
```

```html
<!-- Preload only critical above-the-fold fonts -->
<link rel="preload" as="font" type="font/woff2"
      href="/fonts/geist-variable.woff2" crossorigin>
```

Tools: [Fontaine](https://github.com/unjs/fontaine) for fallback metrics, [glyphhanger](https://github.com/zachleat/glyphhanger) for subsetting.

## 7. Microtypography

| Property | Use | Browser |
|---|---|---|
| `text-wrap: balance` | Headlines (multi-line `<h1>`/`<h2>`) — equalize line lengths | Chrome 114+, Safari 17.5+, Firefox 121+ |
| `text-wrap: pretty` | Paragraphs — prevents orphans, balances last 3-4 lines | Chrome 117+, Firefox 124+ |
| `hyphens: auto` + `lang="..."` | Justified or narrow body — prevents river/jagged | All evergreen |
| `hanging-punctuation: first last` | Pull quotes/headings | Safari 18+, Chrome 132+ |
| `text-spacing-trim: trim-start` | CJK content — removes opening punctuation indent | Chrome 123+ |
| `font-feature-settings: "kern"` | Already on by default | All evergreen |

```css
h1, h2, h3 { text-wrap: balance; }
p          { text-wrap: pretty; hyphens: auto; }
blockquote { hanging-punctuation: first last; }
```

References: [MDN text-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/text-wrap), [MDN hyphens](https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens), [MDN hanging-punctuation](https://developer.mozilla.org/en-US/docs/Web/CSS/hanging-punctuation).

## 8. Numerals

Switch numeral styles via OpenType features.

| Feature | CSS | Use |
|---|---|---|
| Tabular | `font-variant-numeric: tabular-nums` | Tables, prices, timers, dashboards |
| Proportional | `font-variant-numeric: proportional-nums` | Body prose |
| Lining | `font-variant-numeric: lining-nums` | Headings, all-caps |
| Old-style | `font-variant-numeric: oldstyle-nums` | Body prose with serif |
| Slashed zero | `font-variant-numeric: slashed-zero` | Code, IDs, monospace UIs |
| Stacked fractions | `font-variant-numeric: stacked-fractions` | Recipe / measurement contexts |

```css
/* Data table — perfectly aligned columns */
.data-cell { font-variant-numeric: tabular-nums lining-nums; }

/* Editorial body — old-style figures, sit on baseline like lowercase */
.prose      { font-variant-numeric: oldstyle-nums proportional-nums; }

/* Always slash the zero in code */
code, pre   { font-variant-numeric: slashed-zero tabular-nums; }
```

Reference: [MDN font-variant-numeric](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric).

## 9. Anti-patterns

| Anti-pattern | Why | Replace with |
|---|---|---|
| `font-size: 16px` everywhere | Fixed px breaks browser zoom for vision-impaired users | `rem` units + a modular scale |
| `letter-spacing: 0.5px` on body | Body text already shipped at the foundry's optimal tracking | Tracking only on display sizes; tighten as size grows |
| Shipping 5 weights | Bloats LCP, payload, render time | Variable font with the 2-3 weights you actually use, animate via `wght` |
| Inter on a generic SaaS landing | Maximum AI tell in 2026 | Geist, Söhne, GT Walsheim, or pair Instrument Serif + Söhne |
| Loading the full Latin Extended subset | Most apps need only Latin Basic | Subset with `unicode-range`, drop ~70% of glyphs |
| `line-height: 1` on body | Crowds descenders, illegible | `1.5` for body, `1.2-1.3` for headings |
| Justifying text without `hyphens: auto` | Creates rivers of whitespace | Either left-align or hyphenate |
| Line-length 100+ characters | Eye loses its place between lines | Cap at 65-75ch with `max-inline-size: 65ch` |
| Mixing fonts of different x-heights | Visual disharmony — paired fonts look mismatched | Pair fonts with similar x-heights (Geist + Geist Mono, Söhne + Söhne Mono) |
| Using `Inter` and `IBM Plex` together | Both neutral grotesques — no contrast | Pair geometric sans + humanist serif, or display + body cuts of the same family |

(`~/Claude/vault/UI Design/04 - Typography Best Practices.md` for fuller treatment + tooling links.)
