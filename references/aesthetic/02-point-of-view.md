---
topic: aesthetic
role: reference
scope: point-of-view
audience: ui-designer
---

# Point of View: The Coherent Set of Decisions That Makes a Product Recognizable

The fix for AI tells is not "better CSS on the same structure" -- it's a coherent set of decisions about color, type, motion, layout, copy, density, and decoration that distinguishes this product from every other product. That coherent set is the point of view (POV). Without one, you ship a hashtag-purple SaaS template with someone's logo on it. With one, the design is inseparable from the brand and content.

## 1. What Design Point of View Means

POV is not a moodboard. It is not a color palette. It is not a font pairing. It is the set of decisions, made deliberately and applied consistently, that someone could describe in one paragraph and predict the rest of the product from. Examples:

| Product | POV (one paragraph) |
|---------|---------------------|
| **Linear** | Keyboard-first dispatch console for engineering work. Dark warm-graphite background, tight density, monospace numerals, custom Inter Display, single restrained brand purple in marketing and almost none in product. Motion is precise and short. Every interaction has a keyboard shortcut. The chrome gets out of the way of the work |
| **Stripe** | Technical-warm financial infrastructure. Restrained palette (Stripe purple as accent, never gradient), generous whitespace, tabular numerals everywhere money appears, mixed sans (Camphor) and serif on press, motion as functional confirmation (never decoration), distinctive checkout that signals trust |
| **Things 3** | Calm, sentence-case task manager. Avenir Next, soft pastels (one perfect Things-yellow), generous spacing, animation as feedback, sentence case throughout (no Title Case), no decorative chrome. The serenity is the brand |
| **Notion** | Document-as-canvas. Calm gray-on-paper, slash-command first, restrained color (one accent per workspace), functional templates, sidebar that breathes. Color and chrome are subordinated to the user's content |
| **Arc Browser** | Playful chrome that earns its presence. Söhne everywhere, command-bar-first navigation, custom color-mix accents, Spaces concept replaces tabs, peek/Boost states feel toy-like in a serious tool. The chrome is the product |

Every one of those POVs determines hundreds of downstream decisions: button radii, dialog dimensions, error tone, empty-state copy, loading patterns, dark-mode hue shifts. The POV does not micromanage these decisions; it lets you derive them.

The POV test: if you removed the logo, would users know which product this is? If no, the POV is too weak.

## 2. How to Find the POV for a Project

POV is found in conversation with the user before any pixels are pushed. The questions below produce a brief that constrains design without dictating decisions.

| # | Question | Why it matters | Example answer |
|---|----------|----------------|----------------|
| 1 | What 3 adjectives describe the product? | Forces concreteness. "Modern" and "clean" are not adjectives, they are filler | "Tactical, restrained, dense" or "Editorial, warm, generous" |
| 2 | Which existing products inspire? Be specific about which aspect | Anchors aesthetic gravity | "Linear's keyboard model + Stripe's typographic warmth" not "modern SaaS" |
| 3 | What to AVOID? Concrete | Calls out the no-go zone | "No purple gradients, no glass cards, no centered hero, no Lucide icons" |
| 4 | Density target: information-dense vs spacious? | Drives typography scale, padding, line-height | "Dense like Linear" or "spacious like Apple's product pages" |
| 5 | Tone of voice: instructional, peer, expert, friendly, terse? | Drives microcopy, error messages, empty states | "Terse-expert, like Stripe API docs" |
| 6 | Brand temperature: warm or cool? | Drives neutral palette tuning, accent hue | "Warm with one cool accent for status" |
| 7 | Marketing vs product: same POV or split? | Drives whether to design two systems | "Marketing is editorial; product is tactical. Split" |
| 8 | Motion philosophy: invisible-functional, expressive, none? | Drives transition tokens, easing, durations | "Invisible-functional except onboarding" |
| 9 | Decoration tolerance: zero / functional-only / expressive? | Drives texture, illustration, gradient, blob policy | "Zero decoration. Information density is the visual" |
| 10 | What should be screenshot-shareable? | Identifies the one component that needs to be the strongest | "The dispatch board itself" or "The pricing comparison" |

The answers form the POV brief. Every component decision later is checked against the brief. If a button needs a gradient and the brief says "zero decoration", the button doesn't get a gradient -- the brief gets revisited.

### Anti-questions (banned)

| Banned question | Why |
|-----------------|-----|
| "What's your favorite color?" | Color flows from POV, not preference |
| "Should we use shadcn?" | Implementation choice, not POV |
| "Light or dark mode?" | Both, designed independently |
| "Should it look modern?" | "Modern" is the AI default. Useless |
| "What competitor should we copy?" | Aesthetic theft is not POV. Steal principles, not pixels |

## 3. Three POV Templates to Start From

These are starting points -- never ship a template-rendered POV. Use them as gravity wells; deviate intentionally. Each carries a complete token set so the downstream coherence is automatic.

### Template A: Tactical Operator

POV: Linear / Things / Arc / Things-Mac. Dense, keyboard-first, restrained accent, no decoration. The chrome serves the work. The user is an operator, not an audience.

| Token | Value |
|-------|-------|
| Background base | `oklch(0.13 0.005 270)` warm-graphite (not pure black) |
| Surface 1 | `oklch(0.16 0.006 270)` |
| Surface 2 (elevated) | `oklch(0.19 0.007 270)` |
| Border | `oklch(0.25 0.008 270)` hairline 1px |
| Text primary | `oklch(0.95 0.005 90)` warm-paper |
| Text secondary | `oklch(0.65 0.01 90)` |
| Text tertiary | `oklch(0.45 0.01 90)` |
| Accent | `oklch(0.7 0.18 60)` amber-signal -- single accent, used sparingly for active state and primary action |
| Status alert | `oklch(0.62 0.22 25)` blood-red, used only for destructive confirms and outage |
| Status ok | `oklch(0.7 0.13 145)` muted forest -- desaturated to not compete |
| Display font | Söhne Mono Variable / Berkeley Mono / Martian Mono (monospace as the brand statement) |
| Body font | Söhne Variable / Inter Display Variable (450-550 range, never 400/700) |
| Numeric font | Same as body with `font-feature-settings: 'tnum'` |
| Modular scale | 1.2 (12 -> 14 -> 17 -> 20 -> 24 -> 29 -> 35) |
| Spacing scale | 4-8-12-16-24-32-48-72 (8px grid with 4 for tight inline) |
| Radius | `0` (sharp) or `2px` (precise). Never `rounded-md` |
| Motion | 100-180ms, ease-out (instantaneous feel). No bounce. Spring only on drag |
| Density | Tight: 32px row height in tables, 12-14px gap between sections |
| Decoration | None. No gradients, no blobs, no decorative icons. Hairline borders and color-as-status only |

Component recompositions:
- **Button**: borderless filled, sharp corners, `font-feature-settings: 'tnum'`, no shadow, hover = brightness shift only
- **Card**: borderless with hairline divider on top, no rounded corners, no shadow
- **Input**: bottom-border only (1px), focus expands to 2px accent. No default border-input
- **Modal**: full-bleed slide-from-right or centered with hairline border, no rounded corners, no decorative overlay tint

### Template B: Editorial Magazine

POV: Stripe Press / Vercel marketing / Apple product pages / Robin Sloan personal site. Generous whitespace, mixed serif/sans, restrained color, large type, asymmetric composition. The reader is the audience; the page is a publication.

| Token | Value |
|-------|-------|
| Background base | `oklch(0.985 0.005 90)` warm-paper |
| Surface 1 (no elevation needed in editorial) | same as base |
| Border | `oklch(0.85 0.005 90)` hairline only at column edges |
| Text primary | `oklch(0.18 0.01 280)` ink-cool (not pure black) |
| Text secondary | `oklch(0.42 0.01 280)` |
| Text tertiary | `oklch(0.58 0.01 280)` |
| Accent | `oklch(0.55 0.18 25)` warm terracotta -- used as one-color punctuation, not a primary CTA color |
| Display serif | GT Sectra / Editorial New / Source Serif / Söhne Schmal -- distinctive serif with personality |
| Body serif (for long-form) | Source Serif Pro / Charter / Tiempos Text |
| Body sans (for UI) | GT America / ABC Diatype / Söhne Buch |
| Caps / overline | `font-feature-settings: 'smcp'` small-caps, never `text-transform: uppercase` |
| Modular scale | 1.333 (perfect fourth -- editorial standard) |
| Spacing scale | Generous: 16-24-40-64-96-144 (margins generally 80px+ on section breaks) |
| Radius | `0` for editorial, or `4px` for non-content surfaces (cards if needed). Never `rounded-md` for editorial content |
| Motion | View Transitions API for page transitions, 200-300ms ease-in-out. No element-level motion in body |
| Density | Spacious: 1.6 line-height on body, 1.1 on display, 65ch max content width |
| Decoration | One distinctive element used sparingly: a custom drop-cap, a colored sidebar margin, a page-number marker |

Component recompositions:
- **Hero**: left-aligned, three-line headline at 56-72px, subhead at 24px, no buttons (CTAs come later in the page)
- **Article body**: 65ch column width, drop-cap on first paragraph, pull-quotes with em-dash attribution
- **Footer**: minimal. Date, attribution, single nav row. No four-column legal grid

### Template C: Workshop / Crafted

POV: Figma / Linear settings / Notion calendar / Bear Notes / Things Mac. Warm neutrals, small functional details, hand-tuned density, system fonts where they fit, tactile interactions. The user is a craftsperson; the tool feels touchable.

| Token | Value |
|-------|-------|
| Background base | `oklch(0.97 0.008 80)` warm-cream |
| Surface 1 | `oklch(0.99 0.005 80)` paper |
| Surface 2 | `oklch(0.94 0.01 80)` recess |
| Border | `oklch(0.88 0.008 80)` 1px |
| Text primary | `oklch(0.22 0.01 60)` warm-ink |
| Text secondary | `oklch(0.45 0.01 60)` |
| Accent (moss, not blue/purple) | `oklch(0.5 0.13 145)` workshop-moss |
| Accent secondary (warm) | `oklch(0.65 0.16 50)` honey |
| Display font | Söhne Buch / GT Walsheim / Untitled Sans (warm humanist sans) |
| Body font | System UI stack (`-apple-system, system-ui`) tuned for the OS |
| Mono (when used) | Söhne Mono / Berkeley Mono |
| Modular scale | 1.25 (major third) |
| Spacing scale | 4-6-8-12-16-24-32-48 (slightly tighter than tactical) |
| Radius | `6px` consistent (not 4, not 8 -- the in-between) |
| Motion | Spring physics on drag and toggle (200ms tension 170 friction 26). Instant on click |
| Density | Medium: 36px row height, 16-24px section gaps |
| Decoration | Functional details only: subtle inner-shadow on inputs, hairline divider gradients, hand-tuned focus rings |

Component recompositions:
- **Button**: subtle drop-shadow on hover (1px translate-y), warm border
- **Toggle**: spring-animated thumb, warm color, slight inner shadow on track
- **Card**: surface-2 background (recessed feel), 6px radius, hairline border
- **Input**: paper bg with inner shadow on focus, accent border at 2px

## 4. Committing to the POV

Once chosen, every component reflects it. Buttons, dialogs, forms, tables, errors, toasts, empty states, loading, settings panels -- all carry the same DNA. The taste audit (file 04) verifies coherence.

| Discipline | Detail |
|------------|--------|
| Token-driven | All colors, spacing, type as tokens. Never hardcode `#3b82f6` or `padding: 17px` |
| Component-recomposition | Default shadcn primitives are starting points only. Recompose `Button`, `Card`, `Dialog`, `Input` to carry the POV |
| Pattern propagation | The hover state on cards must match the hover state on buttons. The radius on inputs must match the radius on buttons. Coherence is the entire game |
| Copy carries POV too | A tactical operator product writes "Save and continue", an editorial product writes "Publish article", a workshop product writes "Save changes" |
| Empty / error / loading | These are 50% of POV proof. A real POV designs them; AI defaults skip them |

## 5. When to Break Your Own POV

POV is a default, not a prison. Rare, justified breaks reinforce the system. Document the exception explicitly in the design notes.

| Justified break | Why |
|-----------------|-----|
| Destructive confirm | Higher contrast, larger touch target, more space than usual to slow the user down |
| Marketing pages | May warrant editorial POV even in a tactical product (Stripe homepage vs Stripe dashboard) |
| Onboarding | Friendlier tone, more decoration, more whitespace to reduce intimidation |
| Error pages | Personality and warmth even in restrained products. The 404 is a chance to be human |
| First-run empty state | Generous illustration or copy that wouldn't fit in the working state |
| Marketing modal in product | A "What's new" announcement may visually sit slightly outside the product POV |

Unjustified breaks: a "modern" gradient because it's trendy. A single flashy card to "stand out". Color theming for a holiday. These dilute the POV.

## 6. POV Evolution

POV is not version 1.0. It deepens through iteration as the team learns what the product actually is.

| Stage | Pattern |
|-------|---------|
| v1 | Pick the anchor decisions (dark vs light, density, accent hue, type system). Ship something coherent |
| v2-v5 | Discover edge cases (empty states, errors, undocumented contexts). Each one teaches the POV |
| v6+ | The POV starts derivating itself: new components written by the team naturally carry the DNA without explicit reference |
| Migration | Major shifts (Linear off-white -> deep dark, Notion's gradual warming) happen across multiple versions, never in one sprint |

The anchor decisions are the foundation. Once set, they should rarely change. The expressions of those decisions (specific tokens, exact components) refine continuously.

## 7. POV Worksheet (Use Before Designing)

Fill this before opening Figma / writing JSX. Save it in the project as `design/POV.md`. Reference it in every PR.

```markdown
# Project POV

## Three adjectives
[ , , ]

## Aesthetic anchors (specific products + which aspect)
- [Product]: [aspect we're stealing]
- [Product]: [aspect we're stealing]

## Banned (concrete)
- [no X]
- [no Y]

## Density
[information-dense | medium | spacious]

## Tone
[terse | warm | expert | peer]

## Marketing vs Product
[same POV | split — marketing X, product Y]

## Motion
[invisible-functional | expressive | none]

## Decoration
[zero | functional-only | expressive]

## Screenshot-shareable hero
[the one component that must be the strongest]

## Anchor decisions (the ones we won't change without re-deciding the whole system)
- [decision]
- [decision]
- [decision]
```

## 8. Cross-References

| Need | File |
|------|------|
| The catalog of patterns to avoid | `references/aesthetic/01-anti-ai-tells.md` |
| Distinctive systems to study (which POV they exemplify) | `references/aesthetic/03-distinctive-systems.md` |
| Pre-ship audit | `references/aesthetic/04-taste-checklist.md` |
| Token architecture (OKLCH, light-dark, container queries) | `references/ui-design/` (project tokens reference) |
