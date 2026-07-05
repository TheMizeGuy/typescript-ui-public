---
topic: design
role: reference
scope: tailwind-v4
audience: ui-designer
---

# Tailwind CSS v4

CSS-first config, OKLCH defaults, container queries first-class, native cascade layers, dynamic utilities, subgrid, performance, and migration path. Reference for any Tailwind decision in a 2026 codebase.

Source vault: `~/Claude/vault/UI Design/12 - Emerging Trends 2025-2026.md`. Official: [tailwindcss.com/docs](https://tailwindcss.com/docs).

## 1. Why v4 over v3

| Area | v3 | v4 |
|---|---|---|
| Config | `tailwind.config.{js,ts}` | CSS-first via `@theme { --... }` |
| Color space | RGB defaults | OKLCH defaults, P3 gamut |
| Cascade isolation | Manual `@layer` discipline | Native `@layer theme, base, components, utilities` injected automatically |
| Container queries | Plugin (`@tailwindcss/container-queries`) | First-class — `@container`, `@max-md:`, `@min-2xl:` core |
| Dynamic values | Arbitrary value `[var(--foo)]` | `bg-(--foo)`, `mt-(--space-base)` |
| Custom property animation | Plugin | Native `@property` registration |
| Build engine | PostCSS + Tailwind CLI | Oxide engine — Rust-backed, ~5x faster cold, ~100x incremental |
| Bundle size | ~15-30 KB typical | Often single-digit KB after tree-shake |
| Browser baseline | ES2017, broad | Modern only — Safari 16.4+, Chrome 111+, Firefox 128+ |

Trade: drop legacy browser support, get a smaller, faster, less-config Tailwind. Default for any new 2026 project.

## 2. The `@theme` directive

Replaces `tailwind.config.ts`. Every CSS variable in `@theme` becomes both a runtime CSS variable *and* a generator for utility classes.

```css
@import "tailwindcss";

@theme {
  /* Color tokens — generates bg-brand, text-brand, border-brand, etc. */
  --color-brand:       oklch(0.58 0.20 250);
  --color-brand-50:    oklch(0.97 0.02 250);
  --color-brand-500:   oklch(0.58 0.20 250);
  --color-brand-900:   oklch(0.20 0.10 250);
  --color-surface:     oklch(0.98 0 0);
  --color-text:        oklch(0.18 0 0);

  /* Font families — generates font-display, font-body, font-mono */
  --font-display: "Geist Display", system-ui, sans-serif;
  --font-body:    "Geist", system-ui, sans-serif;
  --font-mono:    "Geist Mono", "JetBrains Mono", ui-monospace, monospace;

  /* Spacing base — every gap-*, p-*, m-* is a multiple of this */
  --spacing: 0.25rem;

  /* Type scale */
  --text-2xs:  0.6875rem;
  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.125rem;
  --text-xl:   1.25rem;
  --text-2xl:  1.5rem;
  --text-3xl:  1.875rem;
  --text-4xl:  2.25rem;
  --text-5xl:  3rem;

  /* Breakpoints — generates sm:, md:, lg:, xl:, 2xl:, 3xl: variants */
  --breakpoint-3xl: 120rem;

  /* Radii */
  --radius-xs:  0.125rem;
  --radius-sm:  0.25rem;
  --radius-md:  0.375rem;
  --radius-lg:  0.5rem;
  --radius-xl:  0.75rem;
  --radius-2xl: 1rem;

  /* Easing tokens — usable as `ease-(--ease-out-quart)` */
  --ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
  --ease-out-quint: cubic-bezier(0.16, 1, 0.3, 1);
}
```

| Theme namespace | Generates |
|---|---|
| `--color-*` | `bg-`, `text-`, `border-`, `outline-`, `ring-`, `from-`, `to-`, `via-`, `divide-`, `accent-`, `caret-`, `decoration-` |
| `--font-*` | `font-` |
| `--text-*` | `text-` (size) |
| `--font-weight-*` | `font-` (weight) |
| `--leading-*` | `leading-` |
| `--tracking-*` | `tracking-` |
| `--breakpoint-*` | `sm:`, `md:`, ... variants |
| `--container-*` | `@sm:`, `@md:`, ... container variants |
| `--spacing` | All `gap-N`, `p-N`, `m-N`, `w-N`, `h-N` (any positive number multiplier) |
| `--radius-*` | `rounded-` |
| `--shadow-*` | `shadow-` |
| `--ease-*` | `ease-` |
| `--blur-*` | `blur-` |
| `--animate-*` | `animate-` |

## 3. OKLCH theme tokens

Authoritative pattern: define a primitive ramp, alias semantic tokens, then optionally specialize per scheme. Tailwind v4 uses OKLCH as the default colorspace; sRGB-only browsers get a fallback automatically.

```css
@import "tailwindcss";

@theme {
  /* Primitives */
  --color-blue-50:  oklch(0.97 0.02 250);
  --color-blue-100: oklch(0.93 0.04 250);
  --color-blue-500: oklch(0.58 0.20 250);
  --color-blue-600: oklch(0.48 0.20 250);
  --color-blue-900: oklch(0.20 0.10 250);

  --color-gray-50:  oklch(0.98 0 0);
  --color-gray-200: oklch(0.92 0 0);
  --color-gray-500: oklch(0.55 0 0);
  --color-gray-900: oklch(0.18 0 0);

  /* Semantics — wired to color-scheme via light-dark() */
  --color-surface:        light-dark(var(--color-gray-50),  var(--color-gray-900));
  --color-surface-2:      light-dark(var(--color-gray-100), oklch(0.22 0 0));
  --color-text:           light-dark(var(--color-gray-900), var(--color-gray-50));
  --color-text-muted:     light-dark(oklch(0.45 0 0),       oklch(0.65 0 0));
  --color-border:         light-dark(var(--color-gray-200), oklch(0.30 0 0));
  --color-accent:         light-dark(var(--color-blue-500), var(--color-blue-300));
  --color-accent-hover:   light-dark(var(--color-blue-600), var(--color-blue-200));
}

:root { color-scheme: light dark; }
.theme-light { color-scheme: light; }
.theme-dark  { color-scheme: dark; }
```

Alternative (when `light-dark()` is too restrictive — e.g. shadows, images):

```css
@theme {
  --color-surface: oklch(0.98 0 0);
  --color-text:    oklch(0.18 0 0);
}

@media (prefers-color-scheme: dark) {
  @theme inline {
    --color-surface: oklch(0.16 0 0);
    --color-text:    oklch(0.92 0 0);
  }
}
```

## 4. Container query utilities

First-class in v4 — no plugin. Mark an element as a container, use `@<size>:` variants on children.

```html
<article class="@container">
  <div class="grid grid-cols-1 gap-4 @md:grid-cols-2 @xl:grid-cols-[12rem_1fr_8rem]">
    <img src="..." class="rounded-lg @md:row-span-2">
    <h2 class="text-lg @md:text-xl @xl:text-2xl">Title</h2>
    <p class="text-sm text-(--color-text-muted)">Body…</p>
  </div>
</article>
```

| Syntax | Meaning |
|---|---|
| `@container` | Mark as containment context |
| `@container/foo` | Named container `foo` |
| `@sm:`, `@md:`, `@lg:`, `@xl:`, `@2xl:` | Min-width container variants |
| `@max-md:` | Max-width container variants |
| `@min-[24rem]:` | Arbitrary container breakpoint |
| `@sm/foo:` | Apply only when named container `foo` matches |

Default container breakpoints (override via `--container-*` in `@theme`):
| Name | Default |
|---|---|
| `@xs` | `20rem` (320px) |
| `@sm` | `24rem` (384px) |
| `@md` | `28rem` (448px) |
| `@lg` | `32rem` (512px) |
| `@xl` | `36rem` (576px) |
| `@2xl` | `42rem` (672px) |
| `@3xl` | `48rem` (768px) |

## 5. Dynamic utilities

`bg-(--my-var)` reads a custom property. Replaces v3's `bg-[var(--my-var)]` arbitrary syntax for variable-driven values.

```html
<!-- Reads --color-accent from the cascade (theme, component, inline) -->
<button class="bg-(--color-accent) text-(--color-surface)">Save</button>

<!-- Spacing token -->
<section class="py-(--space-2xl) px-(--space-l)">…</section>

<!-- Per-card accent override via inline custom property -->
<article class="bg-(--card-tint)" style="--card-tint: oklch(0.95 0.04 30)">…</article>

<!-- Arbitrary value still works for one-offs -->
<div class="bg-[oklch(0.7_0.18_140)]">…</div>
```

| Syntax | Use |
|---|---|
| `bg-(--var)` | Read a CSS variable from the cascade |
| `bg-(color:--var)` | Disambiguate when the property accepts multiple types |
| `bg-[oklch(...)]` | Arbitrary value (one-off) |
| `bg-(length:--var)` | Length-typed variable |

## 6. Subgrid

Both axes available as utilities.

```html
<section class="grid grid-cols-[repeat(auto-fill,minmax(18rem,1fr))] gap-6">
  <article class="grid grid-rows-subgrid row-span-4 gap-2">
    <img src="...">
    <h3>Title</h3>
    <p>Body</p>
    <footer>Actions</footer>
  </article>
  <!-- repeat -->
</section>
```

`grid-rows-subgrid` and `grid-cols-subgrid` propagate parent tracks, eliminating the need for content-length JS or hand-tuned heights. Use whenever cards must align across rows.

## 7. Performance tips

| Tip | Detail |
|---|---|
| `@layer components` for repeated patterns | Component classes that do appear in markup go into `@layer components`. Lower precedence than utilities so a one-off `mt-4` still wins |
| Avoid `@apply` in components | Defeats tree-shake — utility CSS is duplicated per component instead of reused. Use CSS variables or component classes via `@layer` |
| Use [CVA](https://cva.style) or [tailwind-variants](https://www.tailwind-variants.org) | Variant logic in TS; ships only the classes you use |
| `cn()` helper | Standard pattern: `import { clsx } from "clsx"; import { twMerge } from "tailwind-merge";` then `cn(...inputs) = twMerge(clsx(inputs))`. Resolves conflicting Tailwind classes deterministically |
| Group selectors over many utilities | `[&:has(input:checked)>span]:bg-accent` keeps logic in one rule |
| `not-prose` to escape Typography plugin scope | When a `<div>` inside `prose` contains a chart or app UI |
| Audit utility usage | `npx tailwindcss --content "..." -o out.css --minify` then check size; the Oxide engine emits ~10x less output than v3 for the same markup |

```ts
// cn helper — ship in lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs));
}
```

```ts
// Variant logic with tailwind-variants
import { tv } from "tailwind-variants";

export const button = tv({
  base: "inline-flex items-center justify-center font-medium rounded-md " +
        "transition-transform duration-150 ease-out " +
        "focus-visible:outline-2 focus-visible:outline-offset-2 " +
        "focus-visible:outline-(--color-accent) " +
        "active:scale-[0.97] disabled:opacity-55 disabled:pointer-events-none",
  variants: {
    intent: {
      primary:  "bg-(--color-accent) text-(--color-surface) hover:bg-(--color-accent-hover)",
      neutral:  "bg-(--color-surface-2) text-(--color-text) hover:bg-(--color-border)",
      ghost:    "bg-transparent text-(--color-text) hover:bg-(--color-surface-2)",
      danger:   "bg-(--color-feedback-error) text-(--color-surface)",
    },
    size: {
      sm: "text-sm h-8 px-3 gap-1.5",
      md: "text-sm h-10 px-4 gap-2",
      lg: "text-base h-12 px-6 gap-2",
    },
  },
  defaultVariants: { intent: "primary", size: "md" },
});
```

## 8. Migration from v3

Run `npx @tailwindcss/upgrade` for a mostly-mechanical migration. Manual checklist for the remaining work:

| Change | v3 | v4 |
|---|---|---|
| Config file | `tailwind.config.ts` | `@theme` block in CSS |
| Import | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| PostCSS plugin | `tailwindcss` + `autoprefixer` | `@tailwindcss/postcss` (no separate autoprefixer) |
| `bg-opacity-50` style | `bg-blue-500/50` already encouraged | Old `bg-opacity-*` removed |
| `content` paths | `content: ["./src/**"]` in config | Auto-discovered via `@source` directive in CSS or default heuristics |
| `theme.extend` | Inside config | Just declare `@theme` — every var is an extension; remove with `--color-foo: initial` |
| `safelist` | `safelist: [...]` in config | `@source inline("class-1 class-2")` |
| `darkMode: "class"` | Config | `@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));` |
| Plugin API | JS | Some plugins not yet ported; check before upgrading |
| Browser support | IE11 in v2; ES2017 in v3 | Modern only (Safari 16.4+) |

```css
/* v4 starter */
@import "tailwindcss";

@source "./src/**/*.{ts,tsx,html}";   /* if outside auto-discovered roots */

@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));

@theme {
  --color-brand: oklch(0.58 0.20 250);
}
```

`cn()` and CVA/tailwind-variants work unchanged. ESLint plugin `eslint-plugin-tailwindcss` v4-aware as of v3.18+.

## 9. Anti-patterns

| Anti-pattern | Why | Replace with |
|---|---|---|
| Liberal `@apply` in component CSS | Ships duplicated bytes; defeats v4 tree-shake | Inline utilities in markup, or use `@layer components` |
| Design tokens defined in TS / JS theme object | Doesn't ship to CSS; can't be used outside React; forks design system | Declare in `@theme` so tokens are CSS variables and utility generators |
| `sm:`, `md:`, `lg:` viewport variants on a portable component | Couples to viewport, not to its container | `@container` + `@sm:`, `@md:` container variants |
| Hex literals in markup | Bypasses theming, dark mode, multi-brand | `bg-brand`, `bg-(--color-brand)`, or `bg-[oklch(...)]` for one-offs only |
| Ignoring container queries | Component layout breaks when reused in narrow contexts | `@container` host + `@<size>:` variants |
| `text-[16px]` everywhere | Bypasses scale; inconsistent | `text-base`, `text-lg`, ... from `--text-*` |
| Custom shadow per component as arbitrary value | Drift across the app | Define `--shadow-*` in `@theme`, use `shadow-md` |
| Plugin sprawl in v4 | v4's core swallows old plugins (forms, typography stays useful) | Remove `@tailwindcss/container-queries`, autoprefixer, nesting plugins |
| `transition-all duration-300` | Animates layout/colors during theme switch (jank) | List explicit properties: `transition-[transform,background-color]` |
| Class string >300 chars on a single element | Unreadable; impossible to diff | Extract to a `tv()` variant or component class |
| Forgetting `darkMode` strategy in v4 | Default is `prefers-color-scheme`; class-based needs custom variant | Define `@custom-variant dark` once |

(`~/Claude/vault/UI Design/12 - Emerging Trends 2025-2026.md` for the framework-landscape table.)
