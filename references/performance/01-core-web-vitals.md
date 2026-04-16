---
topic: performance
role: reference
scope: cwv
audience: ui-engineer
---

# Core Web Vitals — Thresholds, Budgets, Playbooks

Vault anchors: `UI Design/10 - Performance-Aware Design.md`, `SEO/05 - Core Web Vitals and Performance.md`, `Projects/TypeScript Dev Plugin/08 - Frontend Performance and Load-Time Doctrine.md`. Google ranks on p75 field data (CrUX). Lab runs are debugging aids only.

## The 2026 thresholds (p75 field data)

| Metric | Good | Needs Improvement | Poor | Measures |
|--------|------|-------------------|------|----------|
| LCP (Largest Contentful Paint) | <= 2.5s | 2.5-4.0s | > 4.0s | Render time of largest visible element |
| INP (Interaction to Next Paint) | <= 200ms | 200-500ms | > 500ms | Delay from input to next paint (replaced FID Mar 2024) |
| CLS (Cumulative Layout Shift) | <= 0.1 | 0.1-0.25 | > 0.25 | Unexpected layout movement during session |
| TTFB (foundational) | <= 0.8s | 0.8-1.8s | > 1.8s | Server response / HTML start — prerequisite for LCP |

TTFB is not a Core Web Vital but acts as a hard ceiling on LCP; if TTFB > 800ms the 2.5s LCP target is effectively unreachable.

## Field vs lab

| Axis | Field (CrUX) | Lab (Lighthouse / PSI lab / WebPageTest) |
|------|-------------|------------------------------------------|
| Source | Anonymised Chrome users meeting opt-in criteria | Emulated device, throttled network, single run |
| Used for ranking | Yes | No |
| Captures INP | Yes (all interactions across the session) | No — lab cannot simulate real user input |
| Use when | Monitoring regressions, release gate, ranking signal | Reproducing, root-causing, regression bisection |

Lighthouse 100 with bad CrUX = ranked badly. A page can score 100 in the lab and fail CWV in the field because lab runs one cold load, one scripted interaction, on emulated hardware. Field aggregates every navigation type, every device, every third-party, every auth state. Trust CrUX.

## Pass rates (2025-2026 Web Almanac)

| Metric | Mobile pass | Desktop pass |
|--------|-------------|--------------|
| LCP | ~62% | higher |
| INP | ~90% | ~95% |
| CLS | ~85% | ~90% |
| All three | ~47-48% | ~55% |

LCP is the hardest to pass and the single highest-leverage metric to fix. If you have one week to spend, spend it on LCP.

## Performance budgets to declare

Commit these to the repo as `performance-budget.json` and gate CI on them. Values reflect the 2026 doctrine.

```
LCP_TARGET=2200ms
INP_TARGET=180ms
CLS_TARGET=0.05
JS_BUDGET=300KB compressed
CSS_BUDGET=80KB
IMG_ABOVE_FOLD=400KB
FONT_BUDGET=80KB  (2 weights, 1 family)
THIRD_PARTY_BUDGET=150KB
TTFB_TARGET=600ms
HTML_REQUESTS=<=50 initial load
```

Targets sit inside the "Good" thresholds to leave margin for noise and field variance. Break the build when the budget is exceeded rather than warning.

### LCP budget breakdown

```
TTFB:           <= 600ms    (server + CDN + edge cache)
Resource load:  <= 800ms    (LCP image download)
Render delay:   <= 500ms    (CSS parse, layout, paint)
Margin:         <= 300ms    (noise, field variance)
────────────────────────────
Total:           2200ms      (below 2.5s "Good" threshold)
```

Every ms you borrow from one bucket is a ms you must cut from another.

## LCP optimization playbook

| Lever | Impact | How |
|-------|--------|-----|
| Preload LCP image | CRITICAL | `<link rel="preload" as="image" href="hero.avif" fetchpriority="high">` in `<head>` |
| `fetchpriority="high"` on LCP `<img>` | CRITICAL | `<img fetchpriority="high" src=...>`. One of highest-impact single optimizations |
| Never lazy-load the LCP image | CRITICAL | Remove `loading="lazy"`; explicit `loading="eager"` is fine |
| Inline critical CSS (above-the-fold) | HIGH | 10-20KB of hand-picked rules in `<style>`; load the rest with `rel="preload" as="style" onload=...` |
| SSR or SSG the LCP element | HIGH | Never put likely-LCP content behind client-only rendering or post-hydration DOM assembly |
| Modern image formats | HIGH | AVIF first, WebP fallback, JPG last — AVIF is typically 25-50% smaller than JPEG |
| Compress with Brotli | HIGH | Brotli 4 for dynamic HTML, Brotli 11 for static assets. ~15-20% smaller than gzip |
| TTFB <= 600ms | HIGH | Edge cache HTML, CDN, server tuning, DB denormalisation for hot reads, 103 Early Hints where applicable |
| Remove render-blocking scripts | HIGH | `defer` / `async` on `<script>`; inline only true-critical JS |
| Preconnect to required origins | MEDIUM | `<link rel="preconnect" href="https://cdn...">` — cap at 3 origins to avoid connection contention |
| Minimize redirects | MEDIUM | Each hop is one round-trip |
| Right-size images | MEDIUM | `srcset` + `sizes` per breakpoint; never ship a 3000px image to a 400px viewport |
| Fetch priority on CSS/JS | MEDIUM | `fetchpriority="high"` on critical CSS; `fetchpriority="low"` on analytics |

Sources: https://web.dev/articles/lcp, https://web.dev/articles/fetch-priority, https://web.dev/articles/top-cwv, https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/fetchpriority.

## INP optimization playbook

INP aggregates the slowest interactions across the whole session (p98 of observed interactions, reported at p75 of sessions). Fixing one slow click matters.

| Lever | Impact | How |
|-------|--------|-----|
| Break long tasks (> 50ms) | CRITICAL | `await scheduler.yield()` (Chromium 129+), `scheduler.postTask(fn, {priority: 'user-blocking'})`, or `setTimeout(fn, 0)` as fallback |
| Defer / async non-critical JS | CRITICAL | `<script defer>` / `<script async>`; nothing synchronous in `<head>` |
| Web workers for CPU-heavy work | HIGH | Parsing, sorting, data transforms, crypto. Main thread stays free for input |
| DOM size <= 1500 elements | HIGH | Virtualise lists (TanStack Virtual, react-window); `content-visibility: auto` for offscreen sections |
| Move expensive handlers to `requestIdleCallback` | HIGH | Analytics, logging, non-visible computation |
| Batch DOM reads then writes | HIGH | Read-then-write pattern; never read → write → read → write (layout thrash) |
| Defer hydration of non-critical islands | HIGH | Lazy-hydrate below-fold components; server-only render static islands |
| `useDeferredValue` for typing → filter | MEDIUM | Urgent input update + deferred list render (React 19) |
| Debounce input handlers | MEDIUM | 50-100ms window for typeahead; 150-250ms for server-hit |
| Avoid forced synchronous layout | MEDIUM | `offsetHeight` / `getBoundingClientRect` inside a write loop triggers full layout |

Sources: https://web.dev/articles/inp, https://web.dev/articles/optimize-long-tasks, https://web.dev/articles/off-main-thread, https://web.dev/blog/responsiveness.

## CLS optimization playbook

| Lever | Impact | How |
|-------|--------|-----|
| `width` + `height` on every image | CRITICAL | `<img src=... width="1200" height="600">` — browser reserves the box before the image lands |
| `aspect-ratio` for fluid images | CRITICAL | `img { aspect-ratio: 16 / 9; width: 100%; height: auto; }` |
| Reserve space for ad/embed slots | CRITICAL | `.ad-slot { min-height: 250px; }` regardless of whether the ad loads |
| `font-display: swap` + metric overrides | HIGH | `size-adjust`, `ascent-override`, `descent-override`, `line-gap-override` on `@font-face` to match fallback metrics exactly |
| No layout-shifting transitions on input | HIGH | Accordions that push content below them are fine; banners that push content on click are not |
| Avoid inserting content above viewport | HIGH | No "cookie consent banner pushes page down after 500ms". Overlay, never push |
| Set dimensions on iframes | MEDIUM | Twitter embeds, YouTube embeds, Maps iframes — `width` + `height` or `aspect-ratio` |
| Preload the exact font file | MEDIUM | `<link rel="preload" as="font" type="font/woff2" crossorigin href=...>` — prevents swap flash |

`@font-face` metric override example:

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter.woff2") format("woff2");
  font-display: swap;
  size-adjust: 107%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}
```

Generate precise values with `https://github.com/seek-oss/capsize` or `next/font` / `@vercel/font` (which do it automatically).

Sources: https://web.dev/articles/optimize-cls, https://web.dev/articles/css-size-adjust.

## INP attribution

`web-vitals` 4.x ships an attribution build that tells you which element and which phase was slow. Use it in production RUM, not just locally.

```ts
import { onINP } from "web-vitals/attribution";

onINP(
  (metric) => {
    const a = metric.attribution;
    fetch("/rum/inp", {
      method: "POST",
      body: JSON.stringify({
        value: metric.value,
        rating: metric.rating,
        // Which element the user interacted with
        target: a.interactionTarget,
        // The first input, scroll, or animation event type
        type: a.interactionType,
        // Phase breakdown (ms)
        inputDelay: a.inputDelay,
        processingDuration: a.processingDuration,
        presentationDelay: a.presentationDelay,
        // Long animation frame data (LoAF)
        loafScripts: a.longAnimationFrameEntries?.flatMap((f) =>
          f.scripts.map((s) => ({
            source: s.sourceURL,
            fn: s.sourceFunctionName,
            duration: s.duration,
          })),
        ),
        // Next-paint navigation type (so bfcache restores don't pollute)
        navigationType: metric.navigationType,
      }),
      keepalive: true,
    });
  },
  { reportAllChanges: false },
);
```

Phase breakdown guide:

| Phase | What it is | Typical culprit |
|-------|-----------|-----------------|
| `inputDelay` | Time from user input to handler start | Long task already running on main thread (hydration, third-party) |
| `processingDuration` | Time inside the event handler | Your JS — profile it |
| `presentationDelay` | Time from handler end to next paint | Style recalc, layout, large DOM, forced sync layout |

Source: https://github.com/GoogleChrome/web-vitals#attribution-build.

## Anti-patterns flagged as CRITICAL by typescript-ui

| Anti-pattern | Why it's critical |
|--------------|-------------------|
| Lazy-loading the LCP image | Single most common 2026-era regression; can add 1000ms+ to LCP |
| Trusting Lighthouse score over CrUX | Google ranks on CrUX; 100 in lab + red in field = ranked red |
| Synchronous third-party scripts in `<head>` | Blocks HTML parser, blocks LCP, blocks hydration |
| `unload` event listeners | Disqualifies the page from bfcache — restore cost becomes full page load |
| Client-only rendering of likely-LCP content | Hydration delay stacks on top of network delay; often doubles LCP |
| Missing `aspect-ratio` on responsive images | Guarantees CLS when the image arrives |
| `Cache-Control: no-store` on main HTML | Also disqualifies from bfcache |
| Carousel above the fold | LCP target is ambiguous, CLS risk, JS overhead — static hero wins |
| Custom fonts with 4+ weights | Each weight is a separate network request; variable font + subsetting is 1 file |
| CSS-in-JS runtime injection | Styles land after React boots; forces extra paint; measurable LCP hit |
| No declared performance budget | You cannot regress below a threshold you never set |

Sources: https://web.dev/articles/bfcache, https://developer.mozilla.org/docs/Glossary/bfcache, https://developer.mozilla.org/en-US/docs/Web/API/NotRestoredReasons.

## Sources (canonical)

| Topic | URL |
|-------|-----|
| LCP guide | https://web.dev/articles/lcp |
| INP guide | https://web.dev/articles/inp |
| CLS guide | https://web.dev/articles/optimize-cls |
| TTFB | https://web.dev/articles/ttfb |
| Responsiveness overview | https://web.dev/blog/responsiveness |
| Top CWV techniques | https://web.dev/articles/top-cwv |
| Long tasks | https://web.dev/articles/optimize-long-tasks |
| Off main thread | https://web.dev/articles/off-main-thread |
| Fetch priority (MDN) | https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/fetchpriority |
| Fetch priority (web.dev) | https://web.dev/articles/fetch-priority |
| bfcache | https://web.dev/articles/bfcache |
| NotRestoredReasons | https://developer.mozilla.org/en-US/docs/Web/API/NotRestoredReasons |
| Performance budgets 101 | https://web.dev/articles/performance-budgets-101 |
| Debug in the field | https://web.dev/articles/debug-performance-in-the-field |
| web-vitals library | https://github.com/GoogleChrome/web-vitals |
