---
topic: design
role: reference
scope: shadcn-custom
audience: ui-designer
---

# shadcn/ui Customization

Replacing the AI-tell defaults — tokens, typography, recompositions, dropping to Radix, distinctive icons, custom motion, and a taste audit. Reference for any project built on shadcn primitives.

Source vault: `~/Claude/vault/UI Design/12 - Emerging Trends 2025-2026.md`. shadcn docs: [ui.shadcn.com](https://ui.shadcn.com).

## 1. shadcn defaults are AI-tells

Every AI-built site converges on the same shadcn shell. Pattern-recognizable in seconds:

| Default | Tell |
|---|---|
| `--radius: 0.5rem` everywhere | `rounded-md` on cards, buttons, inputs, dialogs — uniform medium radius |
| `bg-background text-foreground` | Generic semantic tokens with no point-of-view |
| Same neutral gray ramp | The default zinc/slate. Identical across thousands of v0/Lovable sites |
| Lucide icon set | One specific stroke weight, one specific style. Instant tell |
| Default fonts (`Inter` or system stack) | No designer would have shipped this |
| Card with `rounded-lg border bg-card text-card-foreground shadow-sm` | The most-cloned component in the index |
| Default `Button` with no transition | Static feel, no character |
| `Tabs`, `Select`, `DropdownMenu` with default chevrons + spacing | Same composition as every demo |
| Hero -> CTA -> 3-feature grid -> testimonials -> footer | The default landing structure |

**Goal of this reference**: replace every default with a project-specific decision. Below, in order of impact.

## 2. Replace the theme tokens

shadcn ships a token set in `app/globals.css`. Replace it with a project-specific OKLCH ramp. See `~/Claude/plugins/typescript-ui/references/design/01-color-oklch.md` for ramp generation.

```css
/* Replace shadcn's default :root + .dark tokens entirely */
@import "tailwindcss";

@theme {
  /* Primitive ramp — derive from one origin */
  --color-brand-50:   oklch(0.97 0.02 250);
  --color-brand-100:  oklch(0.93 0.04 250);
  --color-brand-500:  oklch(0.58 0.20 250);
  --color-brand-600:  oklch(0.48 0.20 250);
  --color-brand-900:  oklch(0.20 0.10 250);

  --color-neutral-50:  oklch(0.98 0.005 250);  /* tinted toward brand hue */
  --color-neutral-200: oklch(0.92 0.008 250);
  --color-neutral-500: oklch(0.55 0.010 250);
  --color-neutral-900: oklch(0.16 0.012 250);

  --color-feedback-error:   oklch(0.62 0.24 25);
  --color-feedback-success: oklch(0.65 0.18 145);
  --color-feedback-warn:    oklch(0.78 0.18 80);

  /* Map to shadcn's semantic names — keep the API shape */
  --background:        light-dark(var(--color-neutral-50),  var(--color-neutral-900));
  --foreground:        light-dark(var(--color-neutral-900), var(--color-neutral-50));
  --card:              light-dark(oklch(1 0 0),             oklch(0.21 0 0));
  --card-foreground:   var(--foreground);
  --popover:           var(--card);
  --popover-foreground:var(--foreground);
  --primary:           light-dark(var(--color-brand-500), var(--color-brand-300));
  --primary-foreground:light-dark(oklch(1 0 0),          var(--color-neutral-900));
  --secondary:         light-dark(var(--color-neutral-200), oklch(0.30 0 0));
  --secondary-foreground: var(--foreground);
  --muted:             var(--secondary);
  --muted-foreground:  light-dark(oklch(0.45 0 0), oklch(0.66 0 0));
  --accent:            light-dark(var(--color-brand-500), var(--color-brand-300));
  --accent-foreground: var(--primary-foreground);
  --destructive:       var(--color-feedback-error);
  --destructive-foreground: oklch(1 0 0);
  --border:            light-dark(oklch(0.90 0 0), oklch(0.28 0 0));
  --input:             var(--border);
  --ring:              var(--accent);

  /* The single most powerful shadcn override — kill the uniform 0.5rem feel */
  --radius: 0;             /* sharp, editorial */
  /* OR */
  --radius: 0.125rem;      /* subtle softening, still architectural */
  /* OR */
  --radius: 1rem;          /* friendly, marketing-forward */
}

:root { color-scheme: light dark; }
```

**Pick exactly one radius scale per project** and commit to it. Mixing `rounded-sm` on inputs, `rounded-md` on buttons, `rounded-lg` on cards is the default — and the tell.

## 3. Replace the typography stack

shadcn's default `--font-sans` is system stack or Inter. Replace it with a paired typeface set with intent. See `~/Claude/plugins/typescript-ui/references/design/02-typography.md` for the banned/approved list.

```css
@theme {
  --font-sans:    "Geist", system-ui, sans-serif;
  --font-display: "Söhne", "Geist", system-ui, sans-serif;
  --font-mono:    "Berkeley Mono", "Geist Mono", ui-monospace, monospace;
  --font-serif:   "Instrument Serif", Georgia, serif;
}
```

Pairing rules:
| Goal | Pair |
|---|---|
| Editorial product | Instrument Serif (display) + Geist (body) + Berkeley Mono (code/UI accents) |
| Tactical / data app | Berkeley Mono or Söhne Mono throughout (functional headers, data tables, body) |
| Friendly SaaS | GT Walsheim (display + body) + Commit Mono (code) |
| Technical platform | IBM Plex Sans (body) + IBM Plex Mono (code) + Migra (display) |

Tracking:
```css
.h-display {
  font-family: var(--font-display);
  font-weight: 540;
  letter-spacing: -0.02em;     /* tighten display headlines */
  text-wrap: balance;
}
body { letter-spacing: 0; }    /* never track body */
.eyebrow {
  font-family: var(--font-mono);
  text-transform: uppercase;
  font-size: 0.6875rem;
  letter-spacing: 0.08em;      /* loosen all-caps labels */
}
```

## 4. Recompose components, don't accept defaults

Three before/after examples. The pattern: identify what's generic, pick one element to over-invest in.

### Card

Before (shadcn default):
```tsx
<Card>
  <CardHeader>
    <CardTitle>Revenue</CardTitle>
    <CardDescription>Last 30 days</CardDescription>
  </CardHeader>
  <CardContent>
    <div className="text-2xl font-bold">$12,430</div>
  </CardContent>
</Card>
```

After (no border, custom shadow, asymmetric padding, monospace metric):
```tsx
<article className="bg-(--card) p-6 pb-8
                    shadow-[0_1px_0_var(--border),0_24px_48px_-32px_oklch(0_0_0_/_0.18)]">
  <header className="flex items-baseline justify-between mb-4">
    <span className="font-mono text-[0.6875rem] uppercase tracking-[0.08em]
                     text-(--muted-foreground)">Revenue · 30d</span>
    <DeltaPill value={+0.12} />
  </header>
  <div className="font-mono text-4xl font-medium tabular-nums tracking-tight">
    $12,430
  </div>
</article>
```

### Button

Before:
```tsx
<Button variant="default">Save</Button>
```

After (custom transition, animated icon, inset press shadow):
```tsx
<button className="group inline-flex items-center gap-2 h-10 px-4
                   bg-(--primary) text-(--primary-foreground)
                   transition-[transform,box-shadow,background-color] duration-120 ease-out
                   hover:bg-(--color-brand-600)
                   active:scale-[0.97]
                   active:shadow-[inset_0_1px_2px_oklch(0_0_0_/_0.25)]
                   focus-visible:outline-2 focus-visible:outline-offset-2
                   focus-visible:outline-(--ring)">
  Save
  <ArrowUpRight className="size-4 transition-transform duration-150
                          group-hover:translate-x-0.5 group-hover:-translate-y-0.5"/>
</button>
```

### Dialog

Before: shadcn's default centered dialog with overlay fade.

After (slide from edge, named view-transition for morph):
```tsx
<Dialog.Root>
  <Dialog.Trigger asChild><Button>Open</Button></Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="fixed inset-0 bg-(--background)/70
                               backdrop-blur-sm
                               data-[state=open]:animate-in
                               data-[state=open]:fade-in-0
                               data-[state=closed]:animate-out
                               data-[state=closed]:fade-out-0" />
    <Dialog.Content className="fixed inset-y-0 right-0 w-full max-w-md
                               bg-(--card) p-8 shadow-2xl
                               border-l border-(--border)
                               data-[state=open]:animate-in
                               data-[state=open]:slide-in-from-right
                               data-[state=closed]:animate-out
                               data-[state=closed]:slide-out-to-right
                               duration-220 ease-[cubic-bezier(0.25,1,0.5,1)]">
      <Dialog.Title className="font-display text-2xl mb-2">Settings</Dialog.Title>
      <Dialog.Description className="text-(--muted-foreground) mb-6">
        Configure preferences.
      </Dialog.Description>
      {/* … */}
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

## 5. Reach for Radix primitives directly

When shadcn's wrapper closes off the prop you need, drop down. shadcn ships generated Radix wrappers — they are not blessed black boxes.

Pattern: re-export typed Radix primitives, build the styled layer yourself.

```tsx
// components/ui/dialog.tsx — fully custom on Radix
import * as DialogPrimitive from "@radix-ui/react-dialog";
import { forwardRef, type ComponentPropsWithoutRef, type ElementRef } from "react";
import { cn } from "@/lib/utils";

export const Dialog        = DialogPrimitive.Root;
export const DialogTrigger = DialogPrimitive.Trigger;
export const DialogPortal  = DialogPrimitive.Portal;
export const DialogClose   = DialogPrimitive.Close;

export const DialogOverlay = forwardRef<
  ElementRef<typeof DialogPrimitive.Overlay>,
  ComponentPropsWithoutRef<typeof DialogPrimitive.Overlay>
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Overlay
    ref={ref}
    className={cn(
      "fixed inset-0 z-50 bg-(--background)/70 backdrop-blur-sm",
      "data-[state=open]:animate-in data-[state=open]:fade-in-0",
      "data-[state=closed]:animate-out data-[state=closed]:fade-out-0",
      className
    )}
    {...props}
  />
));
DialogOverlay.displayName = "DialogOverlay";

export const DialogContent = forwardRef<
  ElementRef<typeof DialogPrimitive.Content>,
  ComponentPropsWithoutRef<typeof DialogPrimitive.Content> & { side?: "right" | "center" }
>(({ className, side = "right", children, ...props }, ref) => (
  <DialogPortal>
    <DialogOverlay />
    <DialogPrimitive.Content
      ref={ref}
      className={cn(
        "fixed z-50 grid gap-4 bg-(--card) p-6 shadow-2xl",
        side === "right" && "inset-y-0 right-0 w-full max-w-md border-l border-(--border)",
        side === "center" && "left-[50%] top-[50%] -translate-x-1/2 -translate-y-1/2 max-w-lg",
        className
      )}
      {...props}
    >
      {children}
    </DialogPrimitive.Content>
  </DialogPortal>
));
DialogContent.displayName = "DialogContent";
```

Other Radix primitives worth bypassing shadcn for: `Popover`, `Tooltip`, `Tabs`, `Toast`, `NavigationMenu`. Shadcn's wrappers are opinionated about className composition that bites once the design diverges.

## 6. Replace Lucide with a distinctive icon set

Lucide is the AI-default. Pick a different set per project — even a different default *stroke* weight reads differently within the first 100ms.

| Set | Variants | Personality | When |
|---|---|---|---|
| Lucide | one weight | Generic, ubiquitous | Default — therefore the tell |
| [Tabler](https://tabler.io/icons) | filled, outline, two-tone | Clean, broad coverage (4500+) | When you want shadcn-adjacent but not shadcn |
| [Phosphor](https://phosphoricons.com) | thin, light, regular, bold, fill, duotone | Six weights | When the design has rhythm — pair `regular` with `bold` for emphasis |
| [Iconoir](https://iconoir.com) | regular, solid | Friendly, slightly geometric | Marketing, consumer apps |
| [Untitled UI Icons](https://www.untitledui.com/icons) | line + duotone | Premium UI library | When matching the broader Untitled UI visual language |
| [Heroicons](https://heroicons.com) | 24-outline, 24-solid, 20-mini | Vercel/Tailwind-adjacent | Default when the rest is Vercel stack — but a tell in that combo |
| [Lineicons](https://lineicons.com) | line, solid | Distinctive thin strokes | Editorial / SaaS with serif headers |
| Custom drawn (Figma -> SVG -> sprite) | one set, your stroke | Strongest brand differentiator | When >20 icons need to match a specific tone |

Standardize one stroke weight, one corner radius for line-style icons, one set throughout. Ship as a sprite or via [Iconify](https://iconify.design) for tree-shaken on-demand loading.

## 7. Custom motion on shadcn primitives

shadcn defaults animate via `tailwindcss-animate`. Replace with explicit transitions or Motion springs for components that touch the user's gesture surface. See `~/Claude/plugins/typescript-ui/references/design/04-motion.md`.

```tsx
// Replace the default Tabs underline with a morphing pill via View Transitions
function MotionTabs({ tabs, value, onChange }: TabsProps) {
  const handle = (next: string) => {
    if (!document.startViewTransition) return onChange(next);
    document.startViewTransition(() => onChange(next));
  };

  return (
    <Tabs.Root value={value} onValueChange={handle}>
      <Tabs.List className="relative inline-flex gap-1 p-1 bg-(--secondary)
                            rounded-(--radius)">
        {tabs.map((t) => (
          <Tabs.Trigger
            key={t.id}
            value={t.id}
            className="relative z-10 px-3 py-1.5 text-sm
                       data-[state=active]:text-(--primary-foreground)
                       transition-colors duration-150"
          >
            {value === t.id && (
              <span
                className="absolute inset-0 -z-10 bg-(--primary) rounded-[inherit]"
                style={{ viewTransitionName: "tab-pill" }}
              />
            )}
            {t.label}
          </Tabs.Trigger>
        ))}
      </Tabs.List>
    </Tabs.Root>
  );
}
```

```css
::view-transition-old(tab-pill),
::view-transition-new(tab-pill) {
  animation-duration: 220ms;
  animation-timing-function: cubic-bezier(0.25, 1, 0.5, 1);
}
```

For springs (gesture-driven sliders, drag-to-dismiss sheets), wrap with Motion's `<motion.div>` and pass `transition={{ type: "spring", stiffness: 320, damping: 30 }}`.

## 8. Distinctive component patterns to add

Recipes that signal "designed", not "scaffolded":

| Pattern | Implementation sketch |
|---|---|
| Search bar with inline shortcut hint | `<input>` plus an `<kbd className="hidden md:inline-flex">⌘K</kbd>` absolutely-positioned in the input's right padding |
| Button with subtle press inset shadow | `active:shadow-[inset_0_1px_2px_oklch(0_0_0_/_0.25)]` + `active:scale-[0.97]` |
| Card that lifts AND shifts inner content on hover | `.card:hover { transform: translateY(-2px) }` + `.card:hover .card-title { transform: translateY(-1px) }` with a 30ms delay for the title |
| Navigation pill with morphing background | `view-transition-name` on the pill background + `document.startViewTransition` on tab change (see §7) |
| Data table with sticky column + row-hover spotlight | `position: sticky; left: 0` for the first column + `tr:hover { background: color-mix(in oklch, var(--accent) 6%, transparent) }` |
| Empty state with an actionable next step | Replace the generic illustration; ship a button labeled with a verb specific to the page |
| Toast that morphs from the originating button | Apply `view-transition-name: toast-{id}` to both the button and the eventual toast |
| Checkbox with stroke-draw check | `<svg>` with `stroke-dasharray` + `stroke-dashoffset` animation on `data-[state=checked]` |
| Scroll-driven section progress | A 2px bar at the top of the section bound to `animation-timeline: view()` |
| Number that counts up on enter | Motion `animate(count, target, { duration: 0.6 })` triggered by IntersectionObserver |

## 9. Anti-pattern checklist (taste audit)

Run this list before shipping. Each line that's true = one AI-tell to fix. The bar: a design-literate reviewer rejects work that looks "bolted together" and recognizes generic AI aesthetics immediately.

| Tell | Fix |
|---|---|
| Default shadcn left untouched (`rounded-md`, default Inter, default zinc/slate, `bg-background text-foreground` everywhere) | Replace tokens, fonts, and radius. See §2 + §3 |
| Lucide icons everywhere | Pick a different set and a different weight. See §6 |
| Gray-on-gray cards stacked on a gray page | Tint each surface; introduce one accent surface per page; use elevation via shadow not color sameness |
| 6 weights of Inter loaded | One variable font family; remove all Inter usage if other typefaces ship |
| Default border-radius applied uniformly to every primitive | Pick one radius (often 0 or `0.125rem` or `1rem`) and commit, OR vary deliberately by primitive class |
| Generic hero -> CTA -> 3-feature grid -> testimonials -> footer landing | Lead with the product surface (an actual screenshot or live demo). Skip the rote feature grid |
| Newsreader + JetBrains Mono pairing | Specifically rejected — too common in AI output. Pick one of: Geist + Berkeley Mono, Söhne + Söhne Mono, Instrument Serif + Geist + Berkeley Mono |
| Rounded cards | If the design reads as "polished AI default", flatten. Sharp corners (`--radius: 0`) signal intent |
| Colored-left-border sections (`border-l-4 border-blue-500 pl-4`) | Replace with typographic hierarchy or a true sidebar. The colored left border is a 2023 Notion clone tell |
| Emoji indicators in product UI (status, callouts, headers) | Use icons from your chosen set, or color/typographic differentiation. No emojis in the product surface |
| Large decorative serif numerals (`01 / 02 / 03` at 72-96px) | Specifically rejected. If numbering matters, use functional headers with monospaced figures and small caps |
| Editorial newspaper / "The X Log" masthead aesthetic | Specifically rejected as a default direction. Only ship if the brand is genuinely editorial |
| Per-category color borders | Specifically rejected. Use one accent color and one feedback color; categorize via typography or layout |
| Expand/collapse cards as the primary disclosure pattern | Specifically rejected. Prefer single-focus mode with sidebar navigation OR a dedicated detail page |
| Skeleton screens for sub-300ms loads | Adds perceived latency; remove and just render the data |
| Default Vercel/v0 visual language (Geist + Vercel-blue + black + dotted-grid bg) | Recognizable in seconds. Pick a different palette + a different background treatment |
| Soft gradients everywhere as background filler | Remove. One bold surface beats five gradient washes |
| 3 boxes side-by-side as the dominant page rhythm | Vary cadence — full-bleed feature, asymmetric split, dense list, then a hero |

(Approved direction examples per the user's validated tactical-dispatch-board memory: monospace throughout, functional headers as data not decoration, one accent color used sparingly, single-report focus with sidebar navigation, warm-dark backgrounds.)
