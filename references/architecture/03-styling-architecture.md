---
topic: architecture
role: reference
scope: styling-architecture
audience: ui-engineer
---

# Styling Architecture

Token cascade, source-of-truth choice, Tailwind v4 + design tokens, CVA, theming, and the cascade-layer rules that make all of it predictable. Source: `~/Claude/vault/UI Design/09 - Design Systems.md`, `~/Claude/vault/TypeScript/11 - React with TypeScript.md`. Cross-ref: `references/architecture/01-component-patterns.md` § CVA, `references/design/06-shadcn-customization.md`.

## 1. The token cascade

Six layers, each consumes the layer above. Don't skip layers; the value of the cascade is that a rebrand or theme swap touches one layer.

```text
Primitive tokens     (color.blue.500 = oklch(0.62 0.18 245))
       v
Semantic tokens      (color.accent.default = {color.blue.500})
       v
Component tokens     (button.primary.bg = {color.accent.default})
       v
CSS custom props     (--button-primary-bg: oklch(0.62 0.18 245);)
       v
Tailwind utilities   (bg-button-primary, text-on-accent)
       v
Component classes    (cn(button({intent:"primary"}), className))
```

| Layer | Lives in | Who reads it | Rebrand impact |
|---|---|---|---|
| Primitive | Token JSON / Style Dictionary / `@theme` | Only semantic layer | Never touched on rebrand |
| Semantic | Token JSON / `@theme` | Component layer | Single edit per token re-points an entire brand |
| Component | Token JSON / `@theme` | CSS / Tailwind | Override per component if needed |
| CSS custom property | `:root` / `[data-theme]` / `[data-brand]` | Tailwind, raw CSS | Generated from layers above |
| Tailwind utility | `tailwind.config` / `@theme` | JSX className | Generated; do not handwrite |
| Component class | Component file | DOM | The only place name-collision is forgivable |

## 2. Source-of-truth options

Pick by where the design lives and how many platforms ship it.

| Source | When | Pipeline | Cost |
|---|---|---|---|
| **Tailwind v4 `@theme`** | Tailwind-first project, web-only, design lives in code | CSS in the repo, no transformer | Lowest setup; coupled to Tailwind |
| **Style Dictionary** | Multi-platform (web + iOS + Android + Compose) | YAML/JSON in -> emits CSS, SCSS, Swift, Kotlin, JS | Build step; widely battle-tested |
| **Terrazzo** | Want W3C DTCG spec compliance, multi-platform | DTCG JSON in -> CSS/JS/Swift/Compose | Newer; spec-compliant |
| **Tokens Studio (Figma plugin)** | Design-led handoff, designers control the source | Figma -> JSON commit -> Style Dictionary | Designer training; CI sync |
| **DTCG W3C JSON** | Standardize across teams or vendor-neutral | Any compliant tool consumes | Format only; needs a transformer |
| **Figma Variables (alone)** | Pure design exploration, no engineering handoff yet | None | Drift the moment code is written |

W3C DTCG (Design Tokens Community Group) format released 2025.10 — first stable. Cite: `~/Claude/vault/UI Design/09.md` § W3C Design Tokens.

## 3. Three-tier token example

Full code, all three layers, isolated rebrand. Tailwind v4 `@theme` syntax.

```css
/* tokens/primitives.css — raw values, never used by components */
@layer base {
  :root {
    --color-blue-500: oklch(0.62 0.18 245);
    --color-blue-600: oklch(0.55 0.19 245);
    --color-blue-700: oklch(0.48 0.20 245);
    --color-red-500:  oklch(0.62 0.22 25);
    --color-gray-50:  oklch(0.98 0   0);
    --color-gray-900: oklch(0.18 0   0);
    --space-1: 0.25rem;
    --space-2: 0.5rem;
    --space-4: 1rem;
    --radius-sm: 0.25rem;
    --radius-md: 0.5rem;
  }
}

/* tokens/semantic.css — intent layer; rebrand re-points these */
@layer base {
  :root {
    --color-accent-default:  var(--color-blue-500);
    --color-accent-hover:    var(--color-blue-600);
    --color-accent-active:   var(--color-blue-700);
    --color-danger-default:  var(--color-red-500);
    --color-surface:         var(--color-gray-50);
    --color-fg:              var(--color-gray-900);
    --color-on-accent:       var(--color-gray-50);
  }
  [data-theme="dark"] {
    --color-surface: var(--color-gray-900);
    --color-fg:      var(--color-gray-50);
  }
}

/* tokens/component.css — component-scoped bindings */
@layer base {
  :root {
    --button-primary-bg:        var(--color-accent-default);
    --button-primary-bg-hover:  var(--color-accent-hover);
    --button-primary-fg:        var(--color-on-accent);
    --button-radius:            var(--radius-md);
  }
}
```

```tsx
// Component reads via Tailwind utility (preferred) or var()
<button className="bg-accent text-on-accent rounded-md hover:bg-accent-hover">
  Save
</button>
```

Why isolate: change `--color-accent-default` once; every primary button, accent link, focus ring, and brand chip updates. No grep-and-replace through the codebase.

## 4. Tailwind v4 + design tokens

v4 (released late 2024) reads `@theme` directly in CSS — no JS config. Tokens defined in `@theme` become both CSS custom properties **and** Tailwind utilities.

```css
/* app.css */
@import "tailwindcss";

@theme {
  /* Colors -> bg-accent / text-accent / border-accent / ring-accent */
  --color-accent:        oklch(0.62 0.18 245);
  --color-accent-hover:  oklch(0.55 0.19 245);
  --color-on-accent:     oklch(0.98 0   0);
  --color-surface:       oklch(0.98 0   0);
  --color-subtle:        oklch(0.95 0   0);
  --color-fg:            oklch(0.18 0   0);
  --color-border:        oklch(0.88 0   0);
  --color-danger:        oklch(0.62 0.22 25);

  /* Spacing -> p-1 / m-2 / gap-4 */
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-4: 1rem;

  /* Radius -> rounded-sm / rounded-md */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;

  /* Typography -> font-sans / text-base */
  --font-sans: "Inter", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, monospace;
}
```

Best practice: define **semantic** tokens in `@theme` (so `bg-accent` reads as intent, not `bg-blue-500`); reference primitives only inside `@theme` itself. Components touch the semantic utility class. This means a designer renaming the brand color in Figma maps to one CSS edit.

| Pattern | Do | Don't |
|---|---|---|
| Component className | `bg-accent` | `bg-blue-600` |
| `@theme` value | `--color-accent: oklch(0.62 0.18 245)` | `--color-accent: var(--color-blue-500)` (extra indirection only useful with separate primitive layer) |
| Theme override | `[data-theme="dark"] { --color-accent: ... }` outside `@theme` | Branch inside `@theme` (Tailwind doesn't compile selectors there) |

If you keep a separate primitive layer (recommended for multi-brand), define primitives in regular `:root {}` and reference them in `@theme`.

## 5. CVA for variant management

CVA when a component has 3+ variant axes; raw Tailwind utilities for one-offs.

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/cn";

const card = cva(
  "rounded-lg ring-1 ring-border bg-surface text-fg",
  {
    variants: {
      padding:  { none: "p-0", sm: "p-3", md: "p-5", lg: "p-8" },
      tone:     { default: "", subtle: "bg-subtle", accent: "bg-accent text-on-accent ring-accent" },
      interactive: { false: "", true: "transition-shadow hover:shadow-md cursor-pointer focus-visible:outline-2 focus-visible:outline-offset-2" },
    },
    compoundVariants: [
      // accent + interactive needs higher contrast hover
      { tone: "accent", interactive: true, class: "hover:bg-accent-hover" },
    ],
    defaultVariants: { padding: "md", tone: "default", interactive: false },
  },
);

interface CardProps extends React.HTMLAttributes<HTMLDivElement>, VariantProps<typeof card> {}

function Card({ padding, tone, interactive, className, ...rest }: CardProps) {
  return <div className={cn(card({ padding, tone, interactive }), className)} {...rest} />;
}
```

| When | Use |
|---|---|
| 1 variant axis, <4 cases | Tailwind directly; conditional `cn()` is fine |
| 2-3 axes | CVA for typed defaults and prop autocomplete |
| 3+ axes or compound interactions | CVA — required; compound variants are otherwise unmaintainable |
| Slot-based composition (`Tabs.Root` styles `Tabs.Trigger` differently when active) | `tailwind-variants` (slots support) |

Cite: `~/Claude/vault/TypeScript/11.md` § Tailwind + TS.

## 6. CSS Modules vs Tailwind

| Concern | Tailwind | CSS Modules |
|---|---|---|
| Design-system enforcement | Strong — utilities only exist if defined | Weak — anyone can add any property |
| Bundle | Single CSS, purged | Per-route CSS, code-split |
| AI generation quality | Excellent — class names map 1:1 to design intent | Poor — invents class names that don't exist |
| Animations / keyframes | OK via `@keyframes` in CSS layer | Strong — co-located with the component |
| Complex state-driven styles | Verbose (`data-*` selectors + `data-[state=open]:bg-accent`) | Strong — full CSS at hand |
| Brand sites with bespoke layouts | OK with custom utilities; can feel fighty | Strong — write the CSS you mean |
| Refactor a token | One `@theme` edit | Find/replace across `.module.css` |
| Type safety on class names | Strings, but variants typed via CVA | `typed-css-modules` plugin or `experimental.typedCSSModules` |

Heuristic: design system + product app -> Tailwind + CVA. Marketing site with one-off art direction -> CSS Modules (or vanilla-extract if you want zero-runtime + types). Cite: `~/Claude/vault/TypeScript/11.md` § Styling Types.

## 7. `tailwind-merge` + `cn` helper

Tailwind utilities apply in source order; the **last** wins. When you concatenate strings naively, conflicting utilities cohabit and the result is whichever one wins the cascade — usually whatever comes later in the generated CSS, not what you wrote last.

```tsx
// Bad
<button className={`px-2 ${condition && "px-4"}`} /> // Both utilities present; cascade decides

// Good — tailwind-merge dedupes; last wins predictably
<button className={cn("px-2", condition && "px-4")} />
// -> "px-4"
```

The shadcn `cn` helper (use it everywhere):

```tsx
// lib/cn.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs));
}
```

`clsx` joins conditionals (`{"a": true, "b": false}`, `["a", null, "b"]`); `twMerge` then dedupes Tailwind conflict groups (padding, margin, color, font-size, etc.). Configure `extendTailwindMerge` if you've added custom utility prefixes that should also dedupe.

## 8. Cascade layers (`@layer`)

CSS `@layer` gives you a deterministic priority order; later layers beat earlier layers regardless of selector specificity. Tailwind v4 already organizes itself into `theme`, `base`, `components`, `utilities`. Add custom layers safely:

```css
@import "tailwindcss";          /* registers theme, base, components, utilities */

@layer reset, app-base, components, utilities, overrides;
/* anything in `overrides` beats utilities, beats components, etc. */

@layer app-base {
  body { background: var(--color-surface); color: var(--color-fg); }
}

@layer overrides {
  /* Surgical opt-out for one widget that must beat utilities */
  .legacy-widget * { all: revert-layer; }
}
```

| Layer | What lives there | Beats |
|---|---|---|
| `reset` | normalize / preflight | nothing |
| `theme` | token defs (Tailwind v4 puts `@theme` here) | reset |
| `base` | element selectors (`h1`, `body`) | theme |
| `components` | reusable component classes (CVA-generated, `.btn`, `.card`) | base |
| `utilities` | Tailwind utility classes | components |
| `overrides` (custom) | one-off escape hatches | utilities |

Rule: **never** use `!important` in 2026. Layers solve every case `!important` was reaching for, without polluting specificity globally.

## 9. Light / dark mode strategies

Three options, picked by whether the user can override the system preference.

| Option | Mechanism | User toggle? | FOUC risk | Browser support (2026) |
|---|---|---|---|---|
| `light-dark()` CSS function | `color: light-dark(black, white);` driven by `color-scheme: light dark` | No (system only) | Zero — pure CSS | Chrome 123+, Firefox 120+, Safari 17.5+ — universal |
| `prefers-color-scheme` `@media` | `@media (prefers-color-scheme: dark) { :root { --fg: white; } }` | No | Zero | Universal since 2020 |
| `data-theme` / class on `<html>` | `<html data-theme="dark">`; CSS targets `[data-theme="dark"]` | Yes | Yes — needs inline pre-script | Universal |

Recommended default: **`data-theme` attribute** on `<html>`, set by an inline script before hydration (avoid FOUC), state stored in `localStorage`. Pair with `color-scheme: dark` for native UI (scrollbars, form controls).

```html
<!-- inline pre-hydration script, blocking, in <head> -->
<script>
  (() => {
    const stored = localStorage.getItem("theme");
    const theme = stored ?? (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light");
    document.documentElement.dataset.theme = theme;
    document.documentElement.style.colorScheme = theme;
  })();
</script>
```

```css
:root { --color-surface: oklch(0.98 0 0); --color-fg: oklch(0.18 0 0); }
[data-theme="dark"] { --color-surface: oklch(0.18 0 0); --color-fg: oklch(0.98 0 0); }
```

```tsx
// Toggle from a Zustand slice (see references/architecture/02-state-architecture.md § 6)
function ThemeToggle() {
  const theme = useAppStore(s => s.theme);
  const setTheme = useAppStore(s => s.setTheme);
  return (
    <button onClick={() => {
      const next = theme === "dark" ? "light" : "dark";
      document.documentElement.dataset.theme = next;
      document.documentElement.style.colorScheme = next;
      setTheme(next);
    }}>
      {theme === "dark" ? "Light" : "Dark"}
    </button>
  );
}
```

`light-dark()` is great when you have **no** override (marketing pages), but it currently has no manual override mechanism — Chrome/Firefox/Safari all still tie it to `color-scheme`. If your product needs a "force dark" button, use `data-theme`.

## 10. Multi-brand theming

Brand = a complete swap of semantic tokens. Component tokens inherit; primitive layer is shared (or per-brand if brands use entirely different palettes).

```css
/* tokens/brands.css */
:root[data-brand="acme"] {
  --color-accent: oklch(0.62 0.18 245);   /* Acme blue */
  --color-accent-hover: oklch(0.55 0.19 245);
  --font-sans: "Inter", sans-serif;
  --radius-md: 0.5rem;
}
:root[data-brand="contoso"] {
  --color-accent: oklch(0.55 0.20 145);   /* Contoso green */
  --color-accent-hover: oklch(0.48 0.21 145);
  --font-sans: "Manrope", sans-serif;
  --radius-md: 0.25rem;                    /* sharper corners */
}
```

```tsx
// Set once at app root, swap by route / subdomain / user setting
<html data-brand={brand} data-theme={theme}>...</html>
```

| Strategy | Pros | Cons |
|---|---|---|
| Selector overrides on `:root[data-brand]` | One CSS bundle; instant runtime swap; no rebuild | All brand CSS shipped to all users |
| Build-time per-brand bundle | Smaller per-brand bundle | Build matrix multiplies; CDN cache fragmentation |
| Token JSON + Style Dictionary per brand | Multi-platform parity (web + native) | More tooling |

Combine `[data-brand]` x `[data-theme]` for `brand x theme` matrices: `:root[data-brand="acme"][data-theme="dark"] { --color-accent: ... }`. Avoid stacking more than two axes — at three you've reinvented Sass mixins poorly.

W3C DTCG `$extensions` field supports brand inheritance (`com.tokens.extends: "Brand A"`) — useful when one brand is a tweak of another. Cite: `~/Claude/vault/UI Design/09.md` § Multi-Brand Theming.

## 11. Anti-patterns

| Anti-pattern | Fix |
|---|---|
| Hardcoded hex in components (`color: "#3b82f6"`) | Use a token (`var(--color-accent)` or `bg-accent`) |
| Inline `style={{ color: "..." }}` | Use `className`; reserve `style` for dynamic computed values (positioning, transforms) |
| One giant `theme.ts` JS object consumed via context | CSS custom properties cost zero at runtime and drive both CSS and JS |
| Tailwind v3 `tailwind.config.js` dust on a v4 project | Migrate to `@theme` in CSS; v4 reads CSS, not JS config |
| `!important` to win cascade fights | Use `@layer overrides` instead; `!important` poisons every consumer |
| Primitive tokens used directly in components (`bg-blue-500`) | Add a semantic token (`bg-accent`) and reference that |
| Token names encoding visual values (`color-blue-500` referenced as the brand) | Rename to intent (`color-accent`); rebranding doesn't require rename then |
| Sequence numbers / dates in token names (`color-v2-2025`) | Versioning lives in package version, not token names |
| shadcn defaults shipped to production (the default neutral / accent / radius palette) | Override the theme tokens; cross-ref `references/design/06-shadcn-customization.md` |
| `style.setProperty("--token", value)` for theming | Theme via `[data-theme]` selector; `setProperty` is for genuinely dynamic per-instance values (a user-picked accent on a card) |
| Two sources of truth: tokens in Figma AND tokens hand-written in CSS | Pick one source; sync the other via Tokens Studio or similar |
| Documentation separate from component code | Storybook autodocs + MDX co-located; design docs in Zeroheight reference the same tokens |

Cite: `~/Claude/vault/UI Design/09.md` § Anti-Patterns.

## References

- `~/Claude/vault/UI Design/09 - Design Systems.md` — W3C tokens, three-tier hierarchy, multi-brand, naming
- `~/Claude/vault/TypeScript/11 - React with TypeScript.md` § Tailwind + TS — CVA, vanilla-extract, tailwind-merge
- W3C Design Tokens Specification 2025.10 — `https://design-tokens.github.io/community-group/format/`
- Tailwind v4 docs — `https://tailwindcss.com/docs` (CSS-first config)
- `class-variance-authority` 0.7+ — `https://cva.style`
- `tailwind-merge` — `https://github.com/dcastil/tailwind-merge`
- Style Dictionary — `https://amzn.github.io/style-dictionary/`
- Terrazzo — `https://terrazzo.dev`
- shadcn `cn` helper — `https://ui.shadcn.com/docs/installation`
- `references/architecture/01-component-patterns.md` — CVA in components
- `references/architecture/02-state-architecture.md` — theme-state Zustand slice
- `references/design/06-shadcn-customization.md` — overriding shadcn defaults
