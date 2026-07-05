---
topic: aesthetic
role: reference
scope: anti-ai-tells
audience: ui-designer
---

# Anti-AI Tells: The Visible Patterns That Scream "AI Generated This"

The flagship reference. Every entry follows the same shape: pattern -> why AI defaults to it -> concrete remediation. Read this before you ship anything; audit against it before you call work done. Cross-refs at the end.

## What AI-Generated UI Looks Like in 2026

AI design output converges on the same visible shape because risk-minimizing training pulls toward the median of its corpus -- which is Tailwind documentation, shadcn/ui demos, Vercel templates, and SaaS landing pages. The result is a recognizable house style: rounded cards, gradient hero, generic icon-in-circle features, Inter or Geist on every surface, hashtag purple-and-teal palette, default shadcn unmodified, lorem-ipsum-shaped copy, frosted nav, "Most Popular" pricing tier. None of these elements are wrong individually. The tell is when they all show up together with no point-of-view binding them.

The user's verbatim rejection criteria from prior sessions: "rounded cards, Newsreader+JetBrains Mono pairing, colored-left-border sections, emoji indicators, large decorative serif numerals." Treat these as banned by default; require explicit justification to use any of them.

Real human design teams diverge from defaults intentionally. Linear didn't ship with Inter -- they built a custom display variant. Stripe doesn't use indigo -- they have a restrained brand-specific palette. Things app doesn't have a bento grid -- it has generous whitespace and one perfect yellow. The point is not to be weird; the point is to make decisions that someone -- not an averaging algorithm -- can defend.

The "logo swap test": if you can swap any SaaS logo onto the page and it still makes sense, the design lacks identity. AI fails this test on every project. Your job is to pass it.

## A note on specificity

Generic advice ("don't use generic colors") wastes the time of every reader. Every entry below is specific: a concrete pattern with a concrete replacement. Where a hex is named, an OKLCH replacement is named. Where a font is rejected, three alternatives are named. Where a layout is criticized, an alternative composition is described. If you find yourself reading something vague, treat it as a bug in this file and propose a fix.

## 1. Color Tells (17)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| C1 | Pure black background `#000000` | Tailwind `bg-black`, easiest "dark" | Slightly warm or cool dark: `oklch(0.12 0.005 280)` cool-graphite, `oklch(0.14 0.008 60)` warm-coal. Pure black halates on OLED and reads as cheap |
| C2 | Pure white surface `#ffffff` | Tailwind `bg-white` default | Off-whites: `oklch(0.985 0.005 90)` warm paper, `oklch(0.98 0.004 250)` cool fog. Pure white burns the eye and flattens the type |
| C3 | Hashtag indigo `#6366f1` as primary | Tailwind `indigo-500` is the demo color in every starter | Pick a non-default hue: `oklch(0.55 0.18 285)` muted indigo, `oklch(0.62 0.15 25)` warm terra, `oklch(0.65 0.14 200)` workshop teal. Document why this hue and not another |
| C4 | Teal-as-secondary `#14b8a6` | Tailwind `teal-500` paired with indigo is the "AI palette" | Single accent. If you need a second, derive it via `oklch(from var(--accent) l calc(c * 0.7) calc(h + 30))` so it's mathematically related, not arbitrarily picked |
| C5 | Purple-to-pink gradient hero `from-purple-500 to-pink-500` | Trained on Tailwind landing demos and v0 output | Banned in serious products. If the brand demands gradient: rare combos like `oklch(0.7 0.18 30)` -> `oklch(0.5 0.2 350)` warm-blood, or skip the gradient and use a single bold hue with depth |
| C6 | Blue-to-teal gradient `from-blue-500 to-teal-400` | Linear's hero clones | Linear earned that gradient with the rest of the design. Imitating one component without the system reads as cargo-culting. Use a flat color or a textured solid |
| C7 | Same neutral gray scale on every surface | One token set, no surface-aware tuning | Tune neutral per surface: warm gray on light bg, cool gray on dark bg, slightly desaturated within cards. `oklch(0.92 0.005 80)` for light cards, `oklch(0.18 0.01 250)` for dark cards |
| C8 | Rainbow accent palette for categories (red, orange, yellow, green, blue, purple) | Default category color picker in dashboards | Pick 2-3 with intent. Prefer monochromatic ramp (`oklch(l from c30 to 75)`) or analogous trio (`+30deg`, `-30deg` from accent). Six category colors is bad data viz |
| C9 | Glassmorphism translucent white over generic gradient | "Modern" cargo-cult since 2020 | Use sparingly. If you must, layer over real photography or a noise texture, not a gradient. Backdrop-blur is expensive and obscures content |
| C10 | Saturated success-green / error-red / warning-yellow without contrast tuning | Tailwind default semantic colors | Tune for APCA contrast against actual surface, not nominal value. Status colors should desaturate when over busy backgrounds. `oklch(0.65 0.15 145)` over light, `oklch(0.75 0.18 145)` over dark |
| C11 | Decorative blur blobs `bg-purple-300 rounded-full blur-3xl opacity-20` | v0 template default for "visual interest" | Solid blocks, real photography, or noise texture. Decorative blobs add no information and read as filler |
| C12 | Identical 0.1-opacity shadow on every component | `shadow-sm` on every Card | Shadow communicates elevation. Most surfaces should be flat. Reserve shadow for actively-elevated states (modals, dragging, picked items). Vary shadow intensity with elevation |
| C13 | Black `rgba(0,0,0,0.5)` modal overlay | shadcn default `Dialog` overlay | Tinted overlay matched to brand: `oklch(0.05 0.01 280 / 0.6)` cool-deep, `oklch(0.08 0.015 30 / 0.5)` warm-deep. Adds atmospheric depth |
| C14 | Color used as the only signal of meaning (red border = error) | Visual shorthand AI inherits from training | WCAG 1.4.1 violation. Pair color with icon, label, or texture. Never make red-vs-green the only differentiator |
| C15 | Six-digit hex throughout `#3b82f6` | Trained on pre-OKLCH CSS | OKLCH everywhere: `oklch(0.62 0.2 250)` is perceptually uniform, supports P3 wide gamut, enables `oklch(from var(--c) ...)` derivation, and matches modern design tools |
| C16 | "Light mode = white, dark mode = black" mirror inversion | Lazy theme generation | Dark mode is its own design. Linear's dark is not the inverse of any light mode -- the saturation, hue shifts, and component elevations are independently authored. Use `light-dark()` CSS function and tune both |
| C17 | One-color-fits-all primary in marketing AND product | Single brand color used for CTAs, links, charts, and notifications | Marketing brand color is a beacon; product accent should be quieter. Linear's purple in marketing -> almost no purple in the product UI |

### Concrete OKLCH replacements for common AI defaults

When you find a hashtag-purple in a codebase, replace it with one of these instead. Each pairing has a deliberate reason for existing.

| AI default (banned) | Replacement A (warm) | Replacement B (cool) | Replacement C (neutral-bold) |
|---------------------|----------------------|----------------------|-----------------------------|
| `#6366f1` indigo-500 | `oklch(0.62 0.15 25)` warm terra | `oklch(0.55 0.15 250)` ink-blue | `oklch(0.55 0.18 285)` muted indigo (deliberate) |
| `#14b8a6` teal-500 | `oklch(0.65 0.13 60)` honey | `oklch(0.62 0.12 200)` workshop teal | `oklch(0.55 0.13 145)` workshop moss |
| `#8b5cf6` violet-500 | `oklch(0.55 0.18 25)` brick | `oklch(0.55 0.15 280)` deep-iris | `oklch(0.4 0.05 280)` graphite |
| `#ec4899` pink-500 | `oklch(0.7 0.18 15)` coral | `oklch(0.6 0.18 350)` magenta-deep | `oklch(0.6 0.16 5)` russet |
| `#3b82f6` blue-500 | `oklch(0.55 0.13 240)` slate-blue | `oklch(0.45 0.18 250)` deep-prussian | `oklch(0.6 0.04 250)` dust-blue |
| `#10b981` emerald-500 | `oklch(0.55 0.13 145)` workshop moss | `oklch(0.65 0.13 165)` mint-cool | `oklch(0.5 0.08 145)` sage-deep |
| `#f59e0b` amber-500 | `oklch(0.7 0.18 60)` amber-signal | `oklch(0.75 0.16 70)` honey-warm | `oklch(0.6 0.16 50)` ochre-deep |

### How to derive a coherent palette without picking colors

Pick one base accent. Derive everything else with `oklch(from var(--accent) ...)`. The math handles consistency.

```css
:root {
  --accent: oklch(0.62 0.15 25);
  --accent-hover: oklch(from var(--accent) calc(l + 0.05) c h);
  --accent-pressed: oklch(from var(--accent) calc(l - 0.05) c h);
  --accent-soft: oklch(from var(--accent) 0.92 calc(c * 0.3) h);
  --accent-border: oklch(from var(--accent) calc(l - 0.1) calc(c * 0.6) h);
  --accent-secondary: oklch(from var(--accent) l c calc(h + 30));
}
```

If the team can defend why each derived value differs, the system has integrity. If the values were picked by eye, drift will appear within 3 sprints.

## 2. Typography Tells (12)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| T1 | Inter as primary font | The most-trained font in the LLM corpus, free, ships with shadcn, Vercel uses it | Banned as primary in distinctive design. Use only as utility/UI label fallback. Pick a font with personality: GT America, Söhne, ABC Diatype, Author, Switzer, Inter Display Variable for technical character, IBM Plex Sans for editorial-warm |
| T2 | Roboto / Helvetica / Arial as primary | Default system fonts, zero risk in training | Banned. Same reasoning as Inter. If you must use a system font, use it deliberately as an aesthetic statement (Notion-style "system as honesty"), not as a fallback |
| T3 | Newsreader + JetBrains Mono pairing | A recognizable default-AI pairing | Avoid. Pick fresh pairings: Söhne + Söhne Mono, Author + GT America Mono, Inter Display + Berkeley Mono, Editorial New + ABC Monument Grotesk Mono |
| T4 | Geist Sans + Geist Mono unmodified | Vercel's distribution makes this a tell now -- every Vercel-deployed AI project ships with it | Geist is fine if it suits the project; just override the font feature settings, weights, and tracking so it doesn't read as default. Or pick something else |
| T5 | All-caps tracking-wide subtitles `text-xs uppercase tracking-wider` | Overlines on table headers, section labels | Use sparingly and with intent. If used: tighter tracking (`tracking-wide` not `tracking-widest`), small-caps via `font-feature-settings: 'smcp'` instead of `text-transform: uppercase` (better letter-spacing) |
| T6 | Display serif numerals (large `01`, `02`, `03`) | Was distinctive in 2022 (Stripe Press, Linear changelog), now generic | Skip unless integral to the brand. Numbers as section markers read as decoration; use functional headers instead |
| T7 | Two-weight stack (regular + bold only) | AI specifies `font-weight: 400` and `font-weight: 700` | Variable fonts allow fluid range. Use 350 for body, 500 for emphasis, 650 for display. Subtle weight differences feel intentional; bold/regular feels mechanical |
| T8 | Default `letter-spacing: 0` on display headlines | AI never tunes tracking | Display type needs negative tracking: `letter-spacing: -0.02em` at 48px+, `-0.03em` at 72px+. Body type at 16px keeps `0` to `-0.005em`. Caps want positive: `+0.05em` |
| T9 | `<em>` italic for emphasis when small-caps or weight shift would do | Default emphasis pattern | Pick one emphasis system per project. Small-caps for inline labels, weight shift (450 -> 550) for editorial, italic only when typeface has a real italic (not slanted) |
| T10 | Centered hero copy in a narrow column | The Tailwind hero template | Asymmetric hero: left-aligned headline taking 60% of width, right-side meta column with secondary info, or full-bleed single statement. Center-aligned hero copy is the strongest formula tell |
| T11 | Single-line product tagline as one giant headline | "Build better, faster" 96px centered | Real headlines are sentences. Three-line headline at 56px reads more confident than one line at 120px. Or skip the giant headline entirely for editorial layout |
| T12 | Body type at 14px to fit more on screen | Density obsession, dashboard reflex | 16px body minimum for accessibility (browsers respect user font-size when units are `rem`). 15-18px is the comfortable range. Density should come from line-height tuning, not shrinking type |

### Distinctive font stacks ranked by POV match

Use this table when the user asks "what font?" Don't suggest Inter unless the project specifically calls for invisible utility type.

| POV | Display | Body | Mono | Why this stack works |
|-----|---------|------|------|---------------------|
| Tactical Operator | Berkeley Mono / Martian Mono / Söhne Mono | Söhne Buch / Inter Display Variable | Berkeley Mono | Mono throughout signals operator-tool. Inter Display Variable acceptable as body when mono is too aggressive |
| Editorial Magazine | GT Sectra / Editorial New / Tiempos Headline | Source Serif / Charter / Tiempos Text | ABC Diatype Mono | Distinctive serif carries editorial weight; sans-mono only in code blocks |
| Workshop / Crafted | GT Walsheim / Söhne Buch / Untitled Sans | system-ui (well-tuned) / Inter Variable | Söhne Mono | Humanist warm sans pairs with system-honesty body |
| Brutalist-Warm | ABC Monument Grotesk / Pangram Sans / Druk | Söhne Breit / GT Pressura | ABC Diatype Mono | Heavy display + functional body + mono accent |
| Spatial / visionOS | SF Pro Display (native only) | SF Pro Text (native only) | SF Mono | Apple-licensed; web fallback to Inter Display Variable |

Banned-as-primary list (with reason): Inter (training-corpus median), Roboto (Google's invisible default), Helvetica (cliché since 2008), Arial (system fallback only), Open Sans (over-used SaaS sans), Poppins (Canva-tier ubiquity), Newsreader (when paired with JetBrains Mono — verbatim AI fingerprint), Lora (over-used "warm serif"), Playfair Display (over-used "luxury display"), Montserrat (Bootstrap reflex), Source Sans (Adobe ubiquity), DM Sans (every shadcn dashboard).

## 3. Layout Tells (12)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| L1 | Generic SaaS scaffold: hero -> CTA -> 3-col features -> testimonials -> CTA -> footer | The most-trained landing page sequence | Let the content drive the structure. A portfolio leads with work. A restaurant leads with the menu. A dev tool leads with code. The page sequence should match the user's intent, not a template |
| L2 | Centered everything (text, buttons, sections) | Easiest to compose, looks "balanced" | Asymmetry. Left-aligned text in editorial, content offset to one side, intentional negative space on the opposite. Center is the lazy answer to composition |
| L3 | Bento grid as the only layout | Hot since 2024, AI overuses it | Use bento when the data has natural rectilinear chunks of varying importance (Apple-style product spec page). Don't use it because "bento looks modern." Mixed sizes need real reasoning |
| L4 | Equal-width 3-col or 4-col feature grid | The Tailwind `grid-cols-3` reflex | Vary widths: golden-ratio split (1:1.618), asymmetric (2:1:1), or no grid (long-form alternation). Equal columns flatten the hierarchy |
| L5 | Cards everywhere -- every container is `Card` with `rounded-lg border` | shadcn `<Card>` is the default container | Recompose. Sections separated by whitespace, color, or subtle rule. Cards should be reserved for genuinely interactive grouped content. Don't card-wrap the entire page |
| L6 | Floating frosted nav `bg-white/80 backdrop-blur-md sticky` | Strongest single-component AI fingerprint | Vary nav height. Skip the blur if it doesn't serve the design. Sidebar nav, mega-menu, or static header all valid. Not every site needs a sticky nav |
| L7 | Section padding overload (`py-24` on every section) | The default section spacing | Vary section padding by content density. Editorial sections breathe with `py-32`; functional sections compress to `py-12`. Mechanical regularity is a tell |
| L8 | Marquee scrolling logo wall | "Trusted by" social proof reflex | Skip unless you actually have logos worth showing. If you do: static grid with subtle hover, not perpetual motion. Static respects the user's attention |
| L9 | Stat counters animating from 0 | "Engagement" decoration | Static stats with sparkline context if the trend matters. Animation steals attention from the number itself |
| L10 | Generic dashboard: sidebar (`w-64`) + topbar + 4 KPI cards + chart + table | shadcn dashboard demo verbatim | Custom sidebar width based on nav depth. Stats relevant to actual domain. Real empty states. Custom chart styling matching brand. Read your dashboard against the shadcn demo -- if it's the same, restart |
| L11 | Alternating left-right feature showcase (text-image, image-text, repeating) | Mechanical alternation pattern | Vary section layouts. Some features deserve full-width demos, some only text, some grids. Let content choose, not pattern |
| L12 | SVG wave / curved section dividers | Decorative separator on every section | Use background color shifts, generous whitespace, or a thin rule. If sections are well-designed they don't need decorative dividers |

### Layout compositions to consider instead of the SaaS scaffold

When the brief says "build a landing page", the AI default is the eight-section vertical scroll: nav, hero, logo bar, features, alternating L-R, testimonials, pricing, footer. Resist. Pick a composition that matches the product, not the template.

| Composition | When to use | Reference |
|-------------|-------------|-----------|
| Editorial single-column | Long-form story, manifesto, founder letter | Stripe Press, Robin Sloan, Paul Graham essays |
| Documentary horizontal scroll | Visual narrative (case study, product story) | Apple product pages, Pitch presentation |
| Dispatch board / dashboard-as-marketing | Product itself is the demo | Linear (issue board screenshot is the hero) |
| Product-spec vertical bento | Hardware-style detail walkthrough | Apple iPhone product page |
| Asymmetric magazine grid | News-publication or portfolio | Vercel Conf, FT.com, Apple Newsroom |
| Single-frame interactive | Tool that works in one viewport | tldraw, Excalidraw, Figma marketing |
| Two-column documentation | Reference content with permanent nav | Stripe API docs, Tailwind docs |
| Calendar / time-rhythm | Time-series product | Cron, Linear roadmap |

If the brief doesn't fit any of these, design the composition before designing the components. The composition decision is upstream of every other layout decision.

## 4. Component / shadcn Tells (12)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| S1 | Default `border-input` color | shadcn default token, never overridden | Override `--input` to brand-specific. `oklch(0.92 0.005 250)` light or `oklch(0.25 0.01 280)` dark. Borders are 30% of the visual texture; tune them |
| S2 | `rounded-md` everywhere | shadcn default radius | Pick a radius identity and commit: `rounded-none` (tactical/technical), `rounded-sm` 2px (precise), `rounded-2xl` 16px (soft/playful). Mixed radii within one component reads as accident |
| S3 | Default `<Card>` with `border + p-6 + shadow-sm` | shadcn primitive unmodified | Recompose: borderless with a subtle bg shift, full-bleed within section, or borderless with hairline divider. Cards are not the only grouping device |
| S4 | Lucide icons unchanged across the entire app | shadcn ships with Lucide, AI never replaces | Vary or replace. Phosphor (variable weight), Tabler, Carbon, Iconoir, Heroicons all distinctive. Or commission a custom 20-icon set for the product. Lucide is fine for utility; don't let it carry the brand |
| S5 | Button labeled "Button" or "Submit" or "Click" | Placeholder copy from demos | Real product copy: "Save changes", "Send invite", "Start trial". Verb + noun. Action-oriented. Never ship "Submit" as the literal label |
| S6 | Floating-label input or placeholder-as-label | Material-Design holdover from 2017 | Visible label above input, placeholder for example value or format hint, optional helper text below. Placeholder-as-label fails accessibility and disappears on focus |
| S7 | Default toast styling (Sonner/shadcn) | `<Toaster />` mounted with no theme override | Restyle to brand: position (bottom-center vs top-right based on app density), animation (slide vs fade), surface treatment (elevated vs inline). Default toast is a recognizable shadcn fingerprint |
| S8 | Default browser scrollbar | No scrollbar styling | Custom scrollbar via `scrollbar-color` and `scrollbar-width: thin` in CSS. Match scrollbar to surface tone. Browser default reads as unfinished |
| S9 | DropdownMenu with default arrow indicator | shadcn primitive default | Match indicator to the rest of the icon system (Phosphor caret if Phosphor, custom chevron if branded). Or remove entirely and let position cue the dropdown |
| S10 | Skeleton loader on content that loads in <300ms | Loading-state reflex | Skeletons are for content that takes 300ms+ to arrive. Sub-300ms loads should use direct render or a brief opacity-fade. Skeletons on fast loads add perceived latency |
| S11 | "Most Popular" floating badge on middle pricing tier | Pricing-card formula | Vary tier emphasis: lead with the recommended plan at full size, deemphasize others. Or use comparison table layout. The floating badge is a top-3 AI tell |
| S12 | Stats card with `$45,231.89 +20.1% from last month` | Verbatim from shadcn dashboard demo | Real stats. Real numbers. Real comparison context (vs last week, vs target, vs benchmark). The exact value `$45,231.89` is in the model's training data as a fingerprint |

### shadcn defaults that MUST be overridden

The shadcn/ui community joke is that every site looks the same because nobody overrides the defaults. The list below is what you must change for shadcn to stop reading as shadcn.

| shadcn default | What to change |
|----------------|----------------|
| `--background: 0 0% 100%` (HSL `#fff`) | OKLCH off-white per surface (see C2) |
| `--foreground: 222.2 84% 4.9%` (near-black ink) | Tuned ink-tone matching the warm/cool brand temperature |
| `--primary: 222.2 47.4% 11.2%` | Brand accent — never the navy default |
| `--radius: 0.5rem` (8px = `rounded-md`) | Radius identity per POV (see S2) |
| Inter as `--font-sans` | POV-appropriate stack (see typography section) |
| Lucide as the icon library | Phosphor / Tabler / Carbon / custom (see S4) |
| `<Card>` default `border + bg-card + shadow-sm + p-6` | Recompose to surface-aware variant (see S3) |
| `<Button>` default size `h-10 px-4 py-2 rounded-md` | POV radius + tuned padding for content density |
| `<Input>` default `border-input bg-background px-3 py-2 rounded-md` | Hairline bottom-border or recessed surface variant |
| `<Dialog>` default `bg-background border rounded-lg shadow-lg` | Position, sizing, and overlay tint per POV |

If a project ships shadcn without overriding any of these, the design-quality bar has not been met. The frontend-design plugin's whole pitch is "avoid generic AI aesthetics" — don't undermine it by leaving shadcn at default.

## 5. Copy + Content Tells (10)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| W1 | Lorem ipsum or "the future of X" tagline | Placeholder content shipped | All content real or marked `[PLACEHOLDER]`. Lorem ipsum in production is a CRITICAL bug |
| W2 | Vague benefit copy: "Streamline your workflow", "Unlock the power of..." | SaaS-speak from training corpus | Specific verbs and objects. "Cut deploy time from 12 minutes to 90 seconds" beats "streamline deploys" |
| W3 | "Built for teams of every size" | Trained-in inclusivity hedge | Pick the team size you serve. "For teams of 5-50 engineers" is honest and disqualifies wrong leads |
| W4 | "Powered by AI" everywhere | The 2024 SaaS reflex | Show what the AI does, not that it exists. "Categorizes invoices automatically" beats "AI-powered invoicing" |
| W5 | Generic SaaS-speak ("collaborate seamlessly", "leverage", "supercharge") | Median tone of marketing corpus | Plain, specific verbs. "Edit the same doc together" beats "collaborate seamlessly" |
| W6 | Three-bullet feature description: title -> verb-the-noun | Template copy structure | One sharp sentence each. Vary the structure -- some bullets short, some long, some skip the bullet entirely |
| W7 | "Trusted by [logos]" with no real logos | Social proof reflex | Show real logos with permission, with quote, with metric. Or skip entirely. Fake logos are a credibility kill |
| W8 | Fake testimonials with stock-photo headshots and "CEO at TechCorp" | Pulled from training data | Real testimonials with names, real photos, real titles, real company. Or skip. "Sarah Johnson, CEO at TechCorp" is a known AI fingerprint |
| W9 | "Get started in seconds" with no actual onboarding | Speed reflex | Be honest about time cost. "5-minute setup, no credit card" with the actual signup form one click away |
| W10 | "Welcome back!" on every dashboard | Generic personalization | Show the user something only they would see. "Your last build deployed 3 hours ago" or "2 PRs await review". Recognition is data, not a greeting |

## 6. Motion Tells (7)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| M1 | Every element fades-in on scroll | Framer Motion + Intersection Observer reflex | Reserve scroll-triggered motion for content that benefits from progressive reveal (case studies, long-form). Default to no motion |
| M2 | Spring-bounce on serious confirms ("Are you sure you want to delete?") | Single global motion preset | Match motion energy to message gravity. Destructive actions: minimal motion, slight scale-up, no bounce. Playful actions: more energy. One-size motion is a tell |
| M3 | Page transitions 600ms+ | "Cinematic" reflex | Snappy transitions: 150-300ms for page changes, 100-200ms for component state, 50-100ms for hover. Over 400ms feels broken |
| M4 | Marquee that scrolls forever | Logo wall, banner news, "what's new" bar | Scroll on demand or pause on hover. Perpetual motion is a battery + attention tax |
| M5 | Hover scale-1.05 on every card | Default Framer Motion preset | Hover should communicate affordance. Scale on cards that lift to a new state; underline-grow on links; brightness shift on buttons. Same hover everywhere is a tell |
| M6 | All animations ignore `prefers-reduced-motion` | Default Motion config | Wrap every decorative animation in `@media (prefers-reduced-motion: reduce)`. Functional motion (loading, drag) can degrade to instant; decorative motion must disable cleanly |
| M7 | Animating layout-triggering properties (`width`, `height`, `top`, `left`) | AI doesn't know the GPU pipeline | Animate `transform`, `opacity`, `filter` only on hot paths. Layout-triggering animations cause jank at 60fps, especially on mobile |

## 7. Image / Asset Tells (6)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| I1 | Stock photography (smiling team in office) | Free/cheap content | Real product screenshots, custom photography, illustration, or skip the image entirely. Stock kills credibility |
| I2 | Generic 3D blob shapes from Spline/Blender | "Modern 3D" reflex | Skip unless the product is genuinely about 3D (visualization, design tool). 3D blobs are 2024's gradient-blobs |
| I3 | Hand-drawn arrows pointing at CTA | "Conversion optimization" gimmick | Trust the design. If a CTA needs an arrow to be found, the layout is broken |
| I4 | Unsplash gradient photos as section backgrounds | Free, looks "premium" | Custom photography, brand textures, solid color, or generative art. Unsplash gradients are recognizable |
| I5 | Placeholder avatars: initials in a colored circle | Default fallback | Real photos when available; abstract identicons (Boring Avatars) or geometric monograms when not. Initials-in-circle is fine in product UI but a tell on marketing |
| I6 | "AI-generated" looking imagery (uncanny faces, six-fingered hands, melted text) | Lazy fal.ai or Midjourney use | Hand-curate every generated asset. Reject the first three results. Edit visible defects. Or use real photography |

## 8. Micro Tells (12)

| # | Pattern | Why AI does it | What humans do |
|---|---------|----------------|----------------|
| U1 | Trailing colons on every form label ("Email:") | 1990s convention from training corpus | Drop the colon. "Email" is enough. Colons add visual noise and have no semantic value |
| U2 | "Enter your email" placeholder | Stock placeholder copy | "you@company.com" gives format hint without redundant label-restate. Or skip placeholder entirely if label is clear |
| U3 | Default focus ring (browser blue or `outline: none`) | AI removes focus ring for "aesthetics" | Custom `:focus-visible` ring matched to brand: 2px solid `oklch(0.65 0.18 250)` with 2px offset. Distinct from `:focus` (mouse) and `:hover`. WCAG 2.4.7 |
| U4 | No `:focus-visible` distinction from `:focus` | AI conflates the two | Mouse-focus: minimal or none. Keyboard-focus: prominent ring. `:focus-visible` is the modern selector |
| U5 | `cursor: pointer` on non-interactive elements | Trained-in habit | Cursor signals interaction. Static text, decorative icons, and disabled controls should not get pointer cursor |
| U6 | Tooltip on hover-only (no keyboard equivalent) | Default Radix Tooltip behavior | Tooltips must show on `:focus-visible` too. Use `<button aria-describedby>` pattern, not `title=` attribute |
| U7 | Modal with X in top-right AND "Close" button in bottom-right | Pattern-stacking from multiple training sources | Pick one close affordance. Either chrome X (with `aria-label="Close"`) or content-area button. Two close buttons is decision-paralysis design |
| U8 | Loading spinner without context label | Default `<Spinner />` | Always paired with text: "Loading invoices...", "Saving changes...". Spinner alone fails screen readers and creates anxiety |
| U9 | Error and success messages styled identically except color | Color-only-meaning failure | Different icon (alert vs check), different bg tint, different border weight, different copy structure. Red-vs-green is not enough |
| U10 | Date format ambiguity (`1/2/26` -- is that Jan 2 or Feb 1?) | Locale-naive default | Explicit format: `Jan 2, 2026` or `2026-01-02` or `2 days ago`. Never numeric-ambiguous |
| U11 | Numeric ranges without thousands separator (`12345`) | Raw `Number.toString()` | `Intl.NumberFormat` with locale: `12,345` or `12 345` based on user locale. Tabular nums for alignment in tables |
| U12 | Singular/plural unhandled ("1 items remaining") | Naive interpolation | `Intl.PluralRules`: "1 item", "2 items", "0 items" or "Nothing remaining". Pluralization bugs are a credibility kill |

## 9. The Strongest 10 AI Fingerprints (Ranked)

The patterns most reliably marking output as AI-generated. None are wrong individually; together with no variation they create the "AI house style":

| Rank | Fingerprint |
|------|-------------|
| 1 | `rounded-xl shadow-sm border` on every Card |
| 2 | `bg-white/80 backdrop-blur-md border-b sticky top-0 z-50` frosted nav |
| 3 | `w-64` dark sidebar + `h-16` header dashboard |
| 4 | `text-xs uppercase tracking-wider` table headers and overlines |
| 5 | "Most Popular" badge floating above middle pricing tier |
| 6 | `bg-primary/10 w-12 h-12 rounded-lg` icon-in-tinted-square in feature grid |
| 7 | `animate-pulse bg-gray-200 w-3/4` skeleton with fractional widths |
| 8 | `bg-black/50` modal overlay with `max-w-md p-6 rounded-xl` dialog |
| 9 | `text-4xl sm:text-5xl lg:text-6xl font-bold tracking-tight` hero hero scale |
| 10 | 4-column footer with "Product / Company / Legal" headers and "All rights reserved" |

## 9b. Deep Cuts: Subtle Tells That Catch Out Even Careful Teams

The catalog above covers the obvious. The list below catches teams that overrode the defaults but still left fingerprints in the second-order details.

| # | Pattern | Why it reads as AI |
|---|---------|--------------------|
| D1 | Border-radius `8px` on every interactive control | The shadcn `--radius: 0.5rem` default. Pick `0`, `2`, `6`, or `12` deliberately |
| D2 | Hover transitions all `transition-all duration-200` | Animates everything including padding/border, causing layout twitches. Specify properties: `transition: background-color 150ms, transform 100ms` |
| D3 | Shadow scale `shadow-sm`, `shadow-md`, `shadow-lg` used in arithmetic progression | Tailwind defaults. Tune your own elevation scale matched to surface tone |
| D4 | Every divider is `border-t border-border` (1px solid) | Vary: hairline gradient, dotted, or whitespace-only |
| D5 | Every `<Avatar>` is a perfect circle with initials in `bg-muted` | Square with rounded-corner (4-6px), or hexagon, or asymmetric crop, or actual photo |
| D6 | Tooltip transition fades in over 200ms with no offset | Tooltips should snap (50-100ms) or rise on a slight Y-translate. Default fade reads as Radix-default |
| D7 | All cards have the same hover treatment (slight shadow lift + scale) | Cards that lead to different actions should have different hover states |
| D8 | Form errors all appear below input in red text | Vary: inline icon + tooltip for inline errors, panel for form-level errors, modal for blocking errors |
| D9 | Loading spinner is the Lucide `Loader2` with `animate-spin` | Custom branded loader, or shimmer skeleton, or progressive content reveal. The Loader2 spinner is a top-20 fingerprint |
| D10 | All section h2 are the same size with the same `mb-4` to subhead | Vary section openings — some lead with quote, some with image, some with stat. Mechanical "h2 + subhead + content" is a tell |
| D11 | Content max-width is `max-w-7xl` (the Tailwind constant) | Custom max-width matched to type scale. Editorial: `65ch`. Dashboard: full-bleed minus sidebar. Marketing: `1240px` or whatever your hero design needs |
| D12 | Container padding is always `px-4 sm:px-6 lg:px-8` | Tailwind responsive container reflex. Pick a single padding value and use container queries for component-level adjustments |
| D13 | Every page section ends with a centered CTA button | Marketing reflex from training data. Some sections lead to action; some lead to the next section. Vary |
| D14 | Hero subhead is `text-xl text-muted-foreground max-w-2xl mx-auto` | Centered narrow-column subhead is the strongest hero formula tell. Left-align it; let it break the column width |
| D15 | Footer has 4 columns with `Product / Company / Resources / Legal` headers | This footer structure appears in 80%+ of AI sites. Match footer to the actual sitemap |
| D16 | Navigation logo on left, links centered, CTA on right | The Tailwind nav template. Left-aligned logo + left-aligned links + right CTA is one valid alternative |
| D17 | Search input with `<Search />` icon on the left, placeholder "Search..." | Vary placeholder ("Find an issue", "Search invoices") and consider keyboard-shortcut hint (`Cmd K`) instead of a permanent input |
| D18 | Settings page is a `Tabs` component with sections inside | Sidebar nav with detail-pane is denser; vertical scroll with section anchors is more scannable. Pick what fits |
| D19 | Date picker is a calendar grid in a popover | If users always pick "today" or "next week", offer those as buttons before showing the calendar |
| D20 | Empty state is `<EmptyState icon={...} title="No X yet" description="..." action={<Button>Create X</Button>} />` | Empty state should have voice. Tactical: minimal. Editorial: copy. Workshop: illustration. Default-EmptyState component is a tell |

## 10. Severity Classification

Used by the taste-checklist (file 04) and pre-ship audit. Anything CRITICAL blocks ship; anything HIGH blocks launch.

| Severity | Tells |
|----------|-------|
| **CRITICAL (block ship)** | C15 hex-not-OKLCH, T1/T2/T3 Inter/Roboto/Newsreader+JBM as primary, S1/S2/S3 default shadcn unmodified, S5 "Button" copy, W1 lorem ipsum, S4 Lucide everywhere, U3 default focus ring, M6 ignored prefers-reduced-motion, U8 spinner without label, L1 generic SaaS scaffold, the top 3 of the strongest-10 fingerprints unmodified |
| **HIGH (fix before launch)** | C1/C2 pure black/white, C3/C4 hashtag indigo+teal, C5/C6 default gradients, T4 Geist unmodified, T8 untuned letter-spacing, T10 centered hero, L2 centered everything, L4 equal-column grid, L6 frosted nav verbatim, L10 generic dashboard, S6 placeholder-as-label, S11 "Most Popular" badge, W2/W5 SaaS-speak, U7 double-close-button, the rest of the strongest-10 |
| **MEDIUM (taste improvements)** | C7 untuned neutrals, C9 glassmorphism, C12 uniform shadow, C16 mirror dark mode, T5 all-caps overlines, T6 display numerals, T7 two-weight stack, T11 single-line giant headline, T12 14px body, L3 bento-as-default, L7 padding overload, L11 alternating L-R features, M1 fade-on-scroll, M2 wrong-energy motion, S7 default toast, U10 ambiguous dates |
| **LOW (depends on context)** | C11 blur blobs, C13 black overlay, L8 logo marquee, L9 stat counters, L12 wave dividers, M3 slow page transitions, M5 hover-scale-everywhere, S10 skeleton-on-fast-load, S12 demo stats, U1 trailing colons, I1-I6 image tells (heavily project-dependent) |

## 11. How AI Design Fails Slip Past Taste Audits

Even teams that read this file ship work with visible AI tells. The failure modes are predictable.

| Failure mode | What happens | Counter |
|--------------|--------------|---------|
| Override-by-checkbox | Team overrides every shadcn token but picks values close enough to the defaults that the result is indistinguishable. `--primary` set to `oklch(0.55 0.18 280)` is still hashtag-purple | Compare your tokens against the OKLCH replacement table above. Be at least 30 hue degrees from the AI default |
| POV inheritance from the prompt | "Make it modern and clean" — the AI's interpretation is the median of its corpus = AI default. The team accepts the first output | Write the POV brief BEFORE prompting. Constrain hard ("no purple, no Inter, dense like Linear") |
| Accept-then-tweak | First pass is shadcn-default. Team tweaks colors and ships. The structure (composition, copy formulas, component arrangement) is still AI default | Restructure the page composition first. Don't tweak — recompose |
| Component-by-component aesthetic | Each component looks fine on its own. Together they don't share DNA | Coherence audit: print all components on one sheet. Do they look like a system? |
| Marketing matches product POV but flatter | A Tactical Operator product gets a Tactical Operator marketing site. Marketing usually wants more breath | Marketing/product split. Two POVs in one company is normal (Stripe, Vercel) |
| Copy gets a final pass that adds SaaS-speak | Designer ships, marketer "improves" the copy, "leverage" appears | Lock copy in the design phase. Revisions go through the designer |
| Lucide stays because "we don't have time to swap icons" | The fastest single-component AI fingerprint to fix | Replacing 20 Lucide icons with Phosphor takes 30 minutes. Always worth it |
| Designer accepts AI output verbatim because "it's good enough" | The "good enough" bar is the problem | Ship "different from AI default" not "good enough" |
| Reviewer doesn't know the tells | Code review approves visible tells because the reviewer hasn't read this file | Make this file required reading for any UI PR review |

## 12. The "Show This to a Designer" Test

The final filter before shipping. Find a working designer (not a developer) who has built brand-shaping work for a real product. Show them three screenshots of your design with no context. Ask:

| Question | What a pass looks like |
|----------|------------------------|
| "What is this product?" | They guess the category correctly from visual signals |
| "Who is this for?" | They infer audience density, expertise level, tone |
| "What three words describe the design?" | They pick adjectives, not "modern" or "clean" |
| "Anything that looks AI-generated?" | They name nothing, or name only deliberate choices you can defend |
| "What's the one thing you'd change?" | Their answer reveals what's weakest in the POV |

If the designer can't answer most of these confidently, the design hasn't earned identity. Restart from the POV worksheet (file 02) and rebuild from the brief.

## Cross-References

| Need | File |
|------|------|
| How to fix the underlying problem (commit to a point-of-view) | `references/aesthetic/02-point-of-view.md` |
| Distinctive systems to study (and not over-imitate) | `references/aesthetic/03-distinctive-systems.md` |
| Pre-ship audit checklist | `references/aesthetic/04-taste-checklist.md` |
| Code-level anti-patterns (React, CSS, perf) | `~/.claude/plugins/cache/anti-slop/anti-slop/<version>/skills/anti-slop/references/frontend-patterns.md` |
| The component-fingerprint catalog | `~/.claude/plugins/cache/anti-slop/anti-slop/<version>/skills/anti-slop/references/design-patterns.md` |

Resolve `<version>` at read time — version-pinned cache paths rot on every anti-slop release:

```bash
ls ~/.claude/plugins/cache/anti-slop/anti-slop/ | sort -V | tail -1
```
