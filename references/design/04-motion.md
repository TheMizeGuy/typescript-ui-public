---
topic: design
role: reference
scope: motion
audience: ui-designer
---

# Motion

Spring physics, native CSS easing, View Transitions API, scroll-driven animations, performance budgets, reduced motion, micro-interactions, stagger orchestration. Reference for every animated decision.



## 1. Spring physics over duration-based easing

Springs are described by physical parameters (stiffness, damping, mass), not a duration. They overshoot, settle, and inherit velocity from gestures — they feel alive in a way no cubic-bezier ever will.

The current standard JS library is [Motion](https://motion.dev) (formerly Framer Motion, now framework-agnostic).

```ts
import { animate } from "motion";

// Spring (no duration; physics decides)
animate("#card", { x: 100, opacity: 1 }, {
  type: "spring",
  stiffness: 300,   // higher = stiffer/snappier
  damping: 30,      // higher = less oscillation
  mass: 1,          // higher = more inertia
});

// React: framer-motion / motion/react
import { motion } from "motion/react";
<motion.div
  initial={{ y: 20, opacity: 0 }}
  animate={{ y: 0, opacity: 1 }}
  transition={{ type: "spring", stiffness: 320, damping: 28 }}
/>
```

| Easing model | Strength | Weakness |
|---|---|---|
| `cubic-bezier(...)` | Deterministic, 1-line CSS, perfect for predictable UI transitions | Always feels mechanical past ~300ms |
| `linear()` (CSS) | Multi-stop curves in pure CSS — approximate spring without JS | Verbose; no velocity inheritance |
| Spring (Motion / WAAPI) | Natural feel, gesture-velocity continuity, interruptible | JS dependency, harder to reason about timing |
| `spring(...)` (Safari TP / proposal) | Native CSS spring | Not yet baseline |

| Use | Pick |
|---|---|
| Tooltip, popover, dropdown reveal | `cubic-bezier(0.16, 1, 0.3, 1)` (ease-out-quint) at 180ms |
| Button press, hover lift | `cubic-bezier(0.3, 0, 0, 1)` at 120ms |
| Card or panel that follows pointer/gesture | Spring with velocity inheritance |
| Page transition (FLIP/morph) | View Transitions API (see §3) |
| Scroll progress indicator | Scroll-driven animation (see §4) |
| Confirmation (toast), success animation | Spring `stiffness: 200, damping: 18` for satisfying overshoot |
| Destructive confirmation modal | Linear or ease — never bouncy |

## 2. Native CSS easing functions

| Function | Use |
|---|---|
| `ease` | Default — start fast, end slow. Generic |
| `ease-in` | Exits — element leaves the viewport |
| `ease-out` | Entrances — element settles into place (default for most UI) |
| `ease-in-out` | Round trips, looping animations |
| `linear` | Progress bars, spinners, scroll-driven, anything that should not slow at the end |
| `cubic-bezier(x1, y1, x2, y2)` | Custom — pick from [easings.net](https://easings.net) |
| `linear(0, 0.25 30%, 0.75 70%, 1)` | Multi-stop curves; approximate spring shapes |
| `spring(mass stiffness damping initialVelocity)` | Safari TP — native spring with no JS |

```css
/* Recommended baseline tokens */
:root {
  --ease-out-quart:    cubic-bezier(0.25, 1, 0.5, 1);
  --ease-out-quint:    cubic-bezier(0.16, 1, 0.3, 1);
  --ease-out-expo:     cubic-bezier(0.19, 1, 0.22, 1);
  --ease-in-out-quart: cubic-bezier(0.76, 0, 0.24, 1);
  --ease-overshoot:    linear(0, 0.5 40%, 1.04 65%, 0.98 80%, 1);

  --motion-fast:   140ms;
  --motion-base:   220ms;
  --motion-slow:   360ms;
}

button {
  transition: transform var(--motion-fast) var(--ease-out-quart),
              background-color var(--motion-fast) linear;
}
```

CSS suffices when: animation is deterministic, no gesture velocity, no choreography across many elements. Reach for JS (Motion) when you need orchestration, springs from gesture, or animating non-CSS values (canvas, SVG path morphs).

## 3. View Transitions API

Native cross-fade or FLIP-style morphing across same-document state changes (SPA) and across navigations (MPA). GPU composited, off-main-thread. Cross-browser baseline: late 2025 (Chrome 126+, Safari 18+, Firefox 130+ with flag, stable in 2026).

### Same-document (SPA-style)

```ts
// Wrap any DOM update — old + new state are snapshotted, browser cross-fades
if (!document.startViewTransition) {
  applyUpdate();
} else {
  document.startViewTransition(() => applyUpdate());
}
```

### Cross-document (MPA / Astro / Next App Router)

```css
@view-transition { navigation: auto; }
```

That single line opts the entire site into cross-document transitions. The user navigates `/foo` -> `/bar` and the browser cross-fades automatically.

### Named transitions for FLIP morphing

Give matching elements the same `view-transition-name` and the browser morphs position, size, and opacity between them.

```css
/* Persistent header that morphs across pages */
.site-header { view-transition-name: site-header; }

/* Article hero image expands into next page hero */
article-card .hero  { view-transition-name: var(--hero-id); }
article-page .hero  { view-transition-name: var(--hero-id); }

/* Tune the morph */
::view-transition-old(site-header),
::view-transition-new(site-header) {
  animation-duration: 300ms;
  animation-timing-function: var(--ease-out-quint);
}

/* Different motion for back navigation */
:root:active-view-transition-type(backwards) {
  &::view-transition-old(root) { animation-name: slide-out-right; }
  &::view-transition-new(root) { animation-name: slide-in-left; }
}
```

| Pseudo-element | Selects |
|---|---|
| `::view-transition` | Root overlay |
| `::view-transition-group(name)` | Container holding old + new for one named element |
| `::view-transition-image-pair(name)` | Holds the actual snapshots |
| `::view-transition-old(name)` | Outgoing snapshot |
| `::view-transition-new(name)` | Incoming snapshot |

References: [MDN View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API), [Chrome view-transition-types](https://developer.chrome.com/docs/web-platform/view-transitions).

## 4. Scroll-driven animations

Animation timeline driven by scroll position, fully native CSS, runs on the compositor. Zero JS, zero main-thread cost. Chrome 115+, Safari 18+, Firefox 134+ behind flag.

```css
/* Reading-progress bar — scroll() ties to page scroll */
.progress {
  position: fixed;
  inset-block-start: 0;
  inset-inline: 0;
  block-size: 3px;
  background: var(--accent);
  transform-origin: 0 50%;
  animation: progress linear;
  animation-timeline: scroll(root block);
}
@keyframes progress {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

/* Reveal-on-scroll — view() ties to element entering scrollport */
.reveal {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 30%;
}
@keyframes reveal {
  from { opacity: 0; translate: 0 2rem; }
  to   { opacity: 1; translate: 0 0; }
}

/* Parallax hero — slower than scroll */
.hero img {
  animation: parallax linear;
  animation-timeline: view();
  animation-range: cover 0% cover 100%;
}
@keyframes parallax { to { translate: 0 -20%; } }
```

| Timeline function | What it ties to |
|---|---|
| `scroll(<scroller>)` | Position of a scroll container |
| `scroll(root)` | Document scroll |
| `view(<axis> <inset>)` | Element's intersection with its scrollport |
| `view-timeline-name: --foo` (decl) + `animation-timeline: --foo` (consumer) | Named timelines for cross-element coordination |

| Range keyword | Where animation runs |
|---|---|
| `entry` | Element entering scrollport (0% just below, 100% bottom edge crossed top) |
| `exit` | Element leaving scrollport |
| `cover` | Element fully in view |
| `contain` | Scrollport fully inside element (long elements) |

References: [MDN scroll-timeline](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-timeline), [scroll-driven-animations.style](https://scroll-driven-animations.style).

Caveat from : scroll-driven is appropriate for polish, not for correctness-critical UI. Wrap behind reduced-motion.

## 5. Animation performance budget

Stay on the compositor. 16.7ms per frame at 60fps; 8.3ms at 120fps.

| Tier | Properties | Cost |
|---|---|---|
| Compositor only | `transform`, `opacity`, `filter`, `clip-path`, `backdrop-filter` | GPU only — free at scale |
| Paint | `background-color`, `box-shadow`, `border-color`, `color` | Repaint, no layout |
| Layout (avoid) | `width`, `height`, `top`, `left`, `margin`, `padding`, `font-size` | Full layout recalc per frame |

```css
/* DO — composite-only */
.card:hover { transform: translateY(-4px); opacity: 0.96; }

/* DON'T — triggers layout every frame */
.card:hover { margin-top: -4px; padding: 1.05rem; }
```

`will-change` opts an element into compositor promotion *before* the animation starts. Apply briefly:

```css
.card { will-change: auto; }
.card:hover { will-change: transform; }   /* hint just before */
.card:not(:hover) { will-change: auto; }  /* release */
```

Permanent `will-change` allocates GPU memory forever. Apply `will-change: transform` to 50 cards = 50 layers = stutter.

References: [csstriggers.com](https://csstriggers.com), [web.dev animation perf](https://web.dev/articles/animations-overview).

## 6. `prefers-reduced-motion`

Vestibular disorders and motion sensitivity affect 35%+ of users to some degree. Always honor the preference. Default: motion off; opt in for users who accept it.

```css
/* Default-safe — no decorative motion */
.card { opacity: 1; transform: none; }

/* Add motion only for users who didn't opt out */
@media (prefers-reduced-motion: no-preference) {
  .card {
    animation: fade-in 0.3s var(--ease-out-quart) both;
    animation-timeline: view();
  }
}

/* Last-resort kill switch (never the only line of defense) */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

Better than disabling: replace movement with a crossfade. Replace springs with linear opacity. Reference: [MDN prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion).

## 7. Micro-interaction patterns

| Pattern | CSS / behaviour |
|---|---|
| Button press | `transform: scale(0.96)` on `:active`, 80ms ease-out |
| Card hover lift | `transform: translateY(-2px)`, 180ms `--ease-out-quint`, optional shadow grow |
| Focus ring | `outline: 2px solid var(--accent); outline-offset: 2px;` on `:focus-visible` only |
| Loading shimmer | Linear gradient + `background-position` animation; only if data takes >500ms |
| Disabled state | `opacity: 0.55; cursor: not-allowed;` plus `pointer-events: none` if non-interactive |
| Toggle / switch | Spring with `stiffness: 700, damping: 32` — snappy with slight settle |
| Checkbox check-mark draw | `stroke-dashoffset` animation 220ms |
| Toast slide-in | Translate from off-screen, spring `stiffness: 320, damping: 28` |
| Skeleton placeholder | Avoid if data is fast (<300ms). Use a delayed reveal, not a perpetual shimmer |
| Modal entry | Backdrop fade 160ms linear; dialog scale `0.95 -> 1` + opacity, 220ms `--ease-out-quart` |

```css
button {
  transition: transform 80ms cubic-bezier(0.3, 0, 0, 1),
              background-color 120ms linear;
}
button:active { transform: scale(0.96); }

.card {
  transition: transform 180ms var(--ease-out-quint),
              box-shadow 220ms var(--ease-out-quint);
}
.card:hover { transform: translateY(-2px); box-shadow: 0 12px 28px -12px rgb(0 0 0 / 0.18); }

button:focus-visible,
[role="link"]:focus-visible {
  outline: 2px solid color-mix(in oklch, var(--accent), white 12%);
  outline-offset: 2px;
  border-radius: inherit;
  box-shadow: 0 0 0 4px color-mix(in oklch, var(--accent) 22%, transparent);
}
```

## 8. Stagger and orchestration

Staggered entries communicate hierarchy. Cap at 30-50ms per item; >80ms feels slow.

CSS-only stagger via `--i` custom property + `animation-delay`:

```css
.list-item {
  animation: fade-up 240ms var(--ease-out-quart) both;
  animation-delay: calc(var(--i, 0) * 40ms);
}
@keyframes fade-up {
  from { opacity: 0; translate: 0 12px; }
  to   { opacity: 1; translate: 0 0; }
}
```

```html
<li class="list-item" style="--i: 0">…</li>
<li class="list-item" style="--i: 1">…</li>
```

Motion's `stagger()` for choreography across many items, gestures, or springs:

```ts
import { animate, stagger } from "motion";

animate(".list-item", { opacity: 1, y: 0 },
  { delay: stagger(0.04, { from: "first" }), type: "spring", stiffness: 320, damping: 30 }
);
```

| Pick | When |
|---|---|
| CSS `animation-delay` | Static lists, no gesture, no interruption |
| Motion `stagger()` | Gesture-driven lists, springs, or where order may change at runtime |

## 9. Anti-patterns

| Anti-pattern | Why | Replace with |
|---|---|---|
| Animating `width`, `height`, `top`, `left` | Triggers layout every frame | `transform`, `clip-path`, `inset` with compositor properties |
| 800ms transitions on UI | Feels sluggish; 200-400ms is the sweet spot | Cap UI duration at 360ms; reserve longer for narrative motion |
| Bouncy spring on destructive confirm | Trivializes serious action | Linear or ease for confirms; springs on celebratory moments only |
| Animating *everything* | Decoration overload — nothing reads as important | Animate state changes only; static content stays static |
| Parallax that breaks scroll prediction | Vestibular triggers, fights browser scroll | Subtle (max 10-20% offset), wrap in reduced-motion guard |
| Autoplay video without gesture | Bandwidth waste, accessibility violation | Click-to-play or `<video preload="none" muted>` with intent |
| Permanent `will-change` on every card | Eats GPU memory | Apply just before animation, release after |
| No `prefers-reduced-motion` guard | Inaccessible | Default-safe + opt-in motion under `no-preference` |
| Skeleton screens for sub-300ms loads | Adds perceived latency | Just render the data; no shimmer |
| `transition: all` | Animates unintended properties (color, layout) on theme switch | List explicit properties |
| Scroll-jacking carousels | Removes user scroll control | Native scroll-snap + scroll-driven animation |
| Hover-only reveals on touch UI | Touch has no hover | Tap-to-toggle or always-visible on touch via `(hover: hover)` MQ |

( for the full performance tier table and tools.)
