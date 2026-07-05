---
topic: aesthetic
role: reference
scope: taste-checklist
audience: ui-designer
---

# Taste Checklist: Pre-Ship Audit

The single dense checklist for pre-ship taste audit. Run it before any UI ships. Every item: question + how to verify + severity if failed. CRITICAL items block ship; HIGH items block launch; MEDIUM items are taste improvements; LOW items depend on context.

## How to Run This

| Step | Action |
|------|--------|
| 1 | Open the URL or local dev server in browser. Walk every page, every modal, every empty/error/loading state |
| 2 | For each section below, answer every question. Note severity of any failure |
| 3 | Tally: any CRITICAL = block ship. Any HIGH = block launch. Show user the CRITICAL/HIGH list before merging |
| 4 | The "Smell Tests" at the end are the final pass. If any of those fail, the design needs more iteration |
| 5 | Document fixed items in commit message; document deferred items in `design/known-debt.md` with severity |

## 1. Color Audit (8)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 1.1 | All colors expressed in OKLCH? | `grep -r "#[0-9a-fA-F]\{3,8\}" src/` returns no hex in component tokens. `oklch(...)` everywhere | CRITICAL |
| 1.2 | Background not pure black (`#000`) or pure white (`#fff`)? | Inspect root background tokens. Should be `oklch(0.13 ...)` warm-graphite or `oklch(0.985 ...)` warm-paper, never `0` or `1` lightness | HIGH |
| 1.3 | Accent color is NOT default Tailwind (`indigo-500`, `purple-500`, `teal-500`, `blue-500`)? | Check `--accent` token. If hue is 250-285 (indigo/purple) or ~180 (teal), question whether it's deliberate or a default | HIGH |
| 1.4 | APCA contrast 75+ on body text (60+ on small UI text)? | Use APCA Lightness Contrast tool on body text against background. WCAG AA is the minimum bar; APCA 75+ is the modern bar | CRITICAL (a11y) |
| 1.5 | Each surface has its own neutral tuned to its background lightness? | Cards on dark bg should not have the same gray as cards on light bg. Surface-1, surface-2 each should be derived from base | MEDIUM |
| 1.6 | Dark mode uses `light-dark()` CSS function or `@media (prefers-color-scheme)`, not a class-toggle hack? | Inspect CSS. If you see `.dark { ... }` class scoping everywhere, it's the lazy approach. `light-dark(var(--light), var(--dark))` is modern | LOW |
| 1.7 | No more than 2 accent colors on a screen? | Walk the page. Count distinct accent uses (excluding semantic green/red/yellow). If >2, you have palette drift | HIGH |
| 1.8 | Destructive color distinct from warning color? | Destructive should be saturated red/blood (`oklch(0.62 0.22 25)`), warning should be amber/honey (`oklch(0.75 0.16 70)`) -- not both red-ish | MEDIUM |

## 2. Typography Audit (8)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 2.1 | Primary font is NOT Inter, Roboto, Helvetica, or Arial? | Check `font-family` token. Acceptable: any font with personality (Söhne, Author, Switzer, Berkeley Mono, GT America, etc.). Inter is OK only as utility fallback | CRITICAL |
| 2.2 | Variable font with multiple weight stops in use? | Check `font-variation-settings` or `wght` axis. If only `400` and `700`, you're not using the variable axis | MEDIUM |
| 2.3 | Type scale is a modular ratio (1.2, 1.25, 1.333, 1.5)? | Check tokens. 12 -> 14 -> 17 -> 20 (1.2) or 16 -> 20 -> 25 -> 31 (1.25). Random sizes (13, 18, 22) are a tell | HIGH |
| 2.4 | Display headlines have tuned `letter-spacing` (negative on large, positive on small caps)? | Inspect headline CSS. `letter-spacing: 0` on a 64px headline is untuned. Should be `-0.02em` to `-0.04em` | MEDIUM |
| 2.5 | Body text uses optical sizing where the font supports it? | Check `font-optical-sizing: auto;` on body or specific sizes for `opsz` axis | LOW |
| 2.6 | Tabular nums (`font-feature-settings: 'tnum'`) on tables and money? | Inspect any table with numbers. Numerals should align by column | HIGH |
| 2.7 | `text-wrap: balance` on headlines? | Inspect h1, h2 CSS. Should have `text-wrap: balance` to avoid orphan words | LOW |
| 2.8 | No all-caps tracking-wide subtitles unless intentional? | Search for `text-transform: uppercase` or `tracking-widest`. If used, is it a deliberate small-caps design choice? Or default Tailwind reflex? | MEDIUM |

## 3. Layout Audit (8)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 3.1 | Layout deliberately broken from grid where intended? | Walk the page. Is there asymmetry, content offset, intentional negative space? Or is everything centered in equal-width columns? | HIGH (marketing), MEDIUM (product) |
| 3.2 | Container queries (`@container`) used for components, not viewport queries? | Search for `@container` usage. Components that change layout based on parent width should use `@container`, not `@media` | MEDIUM |
| 3.3 | Logical properties (`padding-inline`, `margin-block`) used for i18n? | Search for `padding-left`, `margin-right`. If used in flow content, consider replacing with logical properties | HIGH (i18n) |
| 3.4 | Spacing on a modular scale (4-8-12-16-24-32-48), not magic numbers? | Search for `padding: 17px`, `margin: 23px`. Magic numbers are a tell | MEDIUM |
| 3.5 | Asymmetric composition where appropriate? | Marketing pages and editorial content should not be all centered. Hero left-aligned, content offset, intentional one-side weight | MEDIUM (marketing) |
| 3.6 | No bento grid unless it's the right answer? | If using bento, verify the data has natural rectilinear chunks of varying importance. If it's just "we wanted a grid", it's wrong | HIGH (if unjustified) |
| 3.7 | No 3-column equal-width feature grid on marketing? | If you have 3 cards in a row with `grid-cols-3`, ask why. Equal columns flatten hierarchy | HIGH (marketing) |
| 3.8 | Section padding varies by content density? | Editorial sections breathe (96px+); functional sections compress (32px). Mechanical `py-24` everywhere is a tell | MEDIUM |

## 4. Component Audit (10)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 4.1 | All shadcn defaults overridden (theme tokens, radius, font, icons, border colors)? | Compare against shadcn unmodified. If `Card`, `Button`, `Input`, `Dialog` look like the shadcn demo, ship is blocked | CRITICAL |
| 4.2 | Lucide icons NOT the only icon set? | Check icon imports. If `from 'lucide-react'` is the only source, replace some with Phosphor / Tabler / custom, or commission a custom set | HIGH |
| 4.3 | Every Button has product-specific copy (not "Button", "Submit", "Click")? | Walk every button. Real verbs: "Save changes", "Send invite", "Start trial". "Submit" is a CRITICAL fail | CRITICAL |
| 4.4 | Form fields have visible labels (not placeholder-as-label)? | Inspect every form. Label above input, placeholder for example/format. Floating-label is also OK if implemented correctly | HIGH (a11y) |
| 4.5 | `:focus-visible` is distinct from `:focus`? | Tab through the page. Keyboard focus should show ring; mouse-click should not | CRITICAL (a11y) |
| 4.6 | Hover states are functional (not decoration)? | Hover every card/button. Hover should communicate affordance change, not be decorative scale-up | MEDIUM |
| 4.7 | Loading state has context label (not just spinner)? | Trigger every loading state. "Loading invoices..." not just a spinning circle | HIGH |
| 4.8 | Empty states are designed (not "No data" or blank page)? | Trigger every empty state. Should have illustration or specific copy + action ("No invoices yet. Send your first one") | MEDIUM |
| 4.9 | Error states have actionable next steps? | Trigger every error. Should explain what failed AND what the user can do. "Network error" is a fail; "Couldn't load. [Retry]" is a pass | HIGH |
| 4.10 | No skeleton loaders on content that loads in <300ms? | Throttle network and time the loads. If skeleton flashes for <300ms, drop it -- adds perceived latency | MEDIUM |

## 5. Motion Audit (5)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 5.1 | All decorative motion respects `prefers-reduced-motion: reduce`? | Set OS to reduce motion. Walk the page. All decorative animations should disable cleanly | CRITICAL (a11y) |
| 5.2 | Only `transform`, `opacity`, `filter` animated in hot paths (scroll, drag, hover)? | DevTools Performance tab while scrolling and dragging. Layout-triggering animations (`width`, `height`, `top`, `left`) cause jank | HIGH (perf) |
| 5.3 | Spring physics used on tactile micro-interactions (toggles, drag, snap)? | Toggle a switch. Drag a card. Should feel physical, not linear | MEDIUM |
| 5.4 | No bounce on serious confirms (delete, payment, sign-out)? | Trigger destructive confirms. Motion should be subtle/serious, not bouncy | MEDIUM |
| 5.5 | Page transitions <400ms (and use View Transitions API where supported)? | Time page transitions. Over 400ms feels broken | MEDIUM |

## 6. Copy Audit (5)

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 6.1 | No lorem ipsum, "Welcome back!", "Get started today!", "Your future, today"? | `grep -ri "lorem ipsum\|welcome back\|get started today\|the future of" src/` -- any hit blocks ship | CRITICAL |
| 6.2 | No SaaS-speak ("seamless", "leverage", "supercharge", "unlock the power", "streamline")? | `grep -ri "seamless\|leverage\|supercharge\|unlock\|streamline\|robust\|cutting-edge" src/`. Each hit is a HIGH | HIGH |
| 6.3 | Dates have explicit format (not ambiguous `1/2/26`)? | Inspect every date display. Should be `Jan 2, 2026`, `2026-01-02`, or relative (`2 days ago`) | MEDIUM |
| 6.4 | Plurals handled correctly (not "1 items" or "0 item")? | Trigger 0, 1, 2, many counts. Use `Intl.PluralRules` | HIGH |
| 6.5 | Empty states have voice (not "Nothing here" or "No data")? | Walk every empty state. Should have personality matching POV: tactical/editorial/workshop tone | MEDIUM |

## 7. Accessibility Audit (5 critical baseline)

This is the floor. The full a11y reference lives elsewhere; these are the must-pass items for taste sign-off.

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 7.1 | Tab through entire page reaches every interactive element in logical order? | Keyboard-only walkthrough. No traps, no skipped elements, no stranded focus | CRITICAL (a11y) |
| 7.2 | Every interactive non-text element has accessible name? | Inspect icon buttons, links. Should have `aria-label` or visually-hidden text | CRITICAL (a11y) |
| 7.3 | Modal traps focus and returns it on close? | Open modal, tab through, close. Focus should return to the trigger element | CRITICAL (a11y) |
| 7.4 | Dynamic content announced via `aria-live` (toasts, errors, status)? | Trigger a toast. Screen reader should announce it | HIGH (a11y) |
| 7.5 | Color is not the only signal of meaning? | Walk error/success/warning states. They should differ in icon, copy, or texture too | CRITICAL (a11y) |

## 8. Performance Audit (5 must-pass)

The full perf reference lives elsewhere; these are the visible-before-ship items.

| # | Question | Verify | Severity if failed |
|---|----------|--------|--------------------|
| 8.1 | LCP image has explicit `width`/`height` and `fetchpriority="high"`? | Inspect hero / above-fold image. Lighthouse will flag missing dimensions | HIGH (perf) |
| 8.2 | No layout shift (CLS) on font-load, image-load, lazy components? | Lighthouse run. CLS should be <0.1. Use `font-display: swap` with `size-adjust` matching | HIGH (perf) |
| 8.3 | Images are WebP/AVIF, not PNG/JPG, where applicable? | Network tab. PNG/JPG fallback only for transparency/photos | MEDIUM (perf) |
| 8.4 | `<Image>` (Next.js) or `loading="lazy"` on below-fold images? | Inspect img tags below fold. Should lazy-load | MEDIUM (perf) |
| 8.5 | Critical fonts preloaded with `<link rel="preload" as="font" crossorigin>`? | Inspect `<head>`. Custom fonts should be preloaded to avoid FOUT | LOW |

## 9. The Smell Tests (Final Pass)

If any of these fail, the design needs more iteration before ship. These are the gestalt-level questions that catch when the parts are right but the whole still reads as AI.

| # | Smell test | Failure means |
|---|------------|---------------|
| 9.1 | Could a Vercel template have shipped this? | Ship has converged to template. Restart with stronger POV |
| 9.2 | Does this look like every other AI-generated SaaS landing? | The strongest 10 fingerprints (file 01) are all present. Audit each, replace half |
| 9.3 | Is there a single component that someone would screenshot and share? | Design lacks a hero -- one component must be visually exceptional, the rest can be quiet |
| 9.4 | If you removed the logo, would users know which product this is? | POV is too weak. Revisit file 02 worksheet, deepen brand-specific decisions |
| 9.5 | Does the design have a clear point-of-view that one paragraph could describe? | If you can't write that paragraph, the team cannot build coherently. Stop, define POV |
| 9.6 | Would a designer at the reference product (Linear, Stripe, etc.) approve this without changes? | If no, identify the specific component that fails the bar and rebuild it |
| 9.7 | Is there a single visible AI tell from the strongest-10 list (file 01) that we did not deliberately choose to keep? | If yes, fix or document the deliberate exception |
| 9.8 | Have all empty / error / loading states been designed, or just the populated view? | "Happy path only" is the AI default. Production needs all four states for every screen |

## 10. Sign-Off Template

Use at the end of the audit. Save in `design/audit-YYYY-MM-DD.md`.

```markdown
# Pre-ship Taste Audit — [Project] — [Date]

## Critical (block ship)
- [ ] [item] — [status: fixed | deferred with reason]

## High (block launch)
- [ ] [item] — [status]

## Medium (taste improvements)
- [ ] [item]

## Low (context-dependent, accepted)
- [item] — accepted because [reason]

## Smell tests
- 9.1 Vercel template? [pass | fail]
- 9.2 Generic SaaS? [pass | fail]
- 9.3 Screenshot-shareable? [pass | fail]
- 9.4 Logo-swap test? [pass | fail]
- 9.5 POV paragraph? [pass | fail]
- 9.6 Reference designer would approve? [pass | fail]
- 9.7 Strongest-10 fingerprints? [count present, deliberately kept: ...]
- 9.8 All four states designed? [pass | fail]

## Sign-off
Designer: [name]
Date: [date]
Verdict: [SHIP | NEEDS WORK]
```

## Cross-References

| Need | File |
|------|------|
| The catalog of AI tells (referenced by every audit item) | `references/aesthetic/01-anti-ai-tells.md` |
| How to set the POV that the audit is checking against | `references/aesthetic/02-point-of-view.md` |
| Distinctive systems to compare against (smell test 9.6) | `references/aesthetic/03-distinctive-systems.md` |
| Code-level review checklist | `~/.claude/plugins/cache/anti-slop/anti-slop/<version>/skills/anti-slop/references/frontend-patterns.md` |
| Component-fingerprint catalog | `~/.claude/plugins/cache/anti-slop/anti-slop/<version>/skills/anti-slop/references/design-patterns.md` |

Resolve `<version>` at read time — version-pinned cache paths rot on every anti-slop release:

```bash
ls ~/.claude/plugins/cache/anti-slop/anti-slop/ | sort -V | tail -1
```
