# typescript-ui -- Architecture

Internal map of files, responsibilities, and cross-references. Not loaded by Claude -- for human reading and for build-time agent coordination.

## Mission

A team of Opus specialists that designs, reviews, optimizes, and improves TypeScript UI code so the result looks like a senior human design team shipped it. No generic AI tells. Strict TS6 type safety. Sub-2.5s LCP, sub-200ms INP, sub-0.1 CLS. WCAG 2.2 + APCA contrast. Distinctive aesthetic point-of-view per project, not a recycled template.

## Layout

```
typescript-ui/
├── .claude-plugin/plugin.json
├── README.md
├── ARCHITECTURE.md          (this file)
├── skills/                  (4 user-invoked entry points)
│   ├── typescript-design-ui/SKILL.md   "design a [thing]"
│   ├── typescript-review-ui/SKILL.md   "review my UI / check this component"
│   ├── typescript-optimize-ui/SKILL.md "optimize for Core Web Vitals / faster LCP"
│   └── typescript-improve-ui/SKILL.md  "improve / refactor / make this better"
├── agents/                  (6 Opus specialists)
│   ├── ui-design-architect.md       creative direction + greenfield design
│   ├── ui-design-reviewer.md        visual / UX / a11y audit
│   ├── ui-perf-engineer.md          CWV + bundle + render perf
│   ├── ui-typescript-engineer.md    TS6 strictness + component-typing
│   ├── ui-anti-slop-auditor.md      AI-tell catcher
│   └── ui-team-lead.md              orchestrator for typescript-improve-ui multi-pass
└── references/              (self-contained knowledge base, agents read these)
    ├── typescript/
    │   ├── 01-ts6-essentials.md           TS6 defaults, strictness, what changed
    │   ├── 02-component-typing.md         Polymorphic, compound, slot, ref forwarding
    │   ├── 03-state-typing.md             Discriminated unions for UI state
    │   └── 04-branded-primitives.md       UserId, Pixels, Hex -- nominal in UI
    ├── design/
    │   ├── 01-color-oklch.md              OKLCH + Display P3 + light-dark()
    │   ├── 02-typography.md               Variable fonts, optical sizing, scales
    │   ├── 03-spacing-rhythm.md           Modular scale, fluid, logical properties
    │   ├── 04-motion.md                   Spring physics, View Transitions, scroll
    │   ├── 05-tailwind-v4.md              CSS-first config, OKLCH defaults
    │   └── 06-shadcn-customization.md     How to make shadcn not look like shadcn
    ├── aesthetic/
    │   ├── 01-anti-ai-tells.md            What AI ships by default + remediation
    │   ├── 02-point-of-view.md            How to commit to a direction
    │   ├── 03-distinctive-systems.md      Reference systems (Linear, Vercel, Stripe, Apple, Things, Arc, Figma)
    │   └── 04-taste-checklist.md          Pre-ship taste audit
    ├── performance/
    │   ├── 01-core-web-vitals.md          LCP / INP / CLS thresholds + budgets
    │   ├── 02-react-19-perf.md            Compiler, RSC, Suspense, useDeferredValue
    │   ├── 03-css-perf.md                 content-visibility, contain, scroll-driven, GPU
    │   ├── 04-bundle-loading.md           Code split, fonts, images, prefetch, bfcache
    │   └── 05-measurement.md              CrUX, web-vitals JS, INP attribution
    ├── accessibility/
    │   ├── 01-wcag-2-2.md                 New criteria in 2.2 + APCA shift
    │   ├── 02-keyboard-focus.md           Focus rings, tab order, skip links, traps
    │   ├── 03-motion-reduce.md            prefers-reduced-motion + safe alternatives
    │   └── 04-screen-reader.md            ARIA, semantic HTML, live regions
    └── architecture/
        ├── 01-component-patterns.md       Compound, slot, render props, polymorphic
        ├── 02-state-architecture.md       URL state, server state, client state, form state
        └── 03-styling-architecture.md     Tokens → CSS vars → Tailwind → CVA
```

## Agent <-> skill mapping

| Skill | Agents dispatched | Mode |
|---|---|---|
| `typescript-design-ui` | `ui-design-architect` (then `ui-typescript-engineer` for the type surface) | Sequential |
| `typescript-review-ui` | `ui-design-reviewer` + `ui-anti-slop-auditor` + `ui-typescript-engineer` | Parallel, then orchestrator merges |
| `typescript-optimize-ui` | `ui-perf-engineer` (with optional `ui-typescript-engineer` for component-level perf) | Sequential |
| `typescript-improve-ui` | `ui-team-lead` orchestrates all 4 specialist reviewers in 3 phases (audit -> plan -> apply per user approval) | Multi-pass, team-lead owns dispatch |

## Reference <-> agent mapping

| Agent | Reads (must) |
|---|---|
| `ui-design-architect` | aesthetic/* (all), design/* (all), accessibility/01,02,03 |
| `ui-design-reviewer` | aesthetic/*, design/*, accessibility/* |
| `ui-perf-engineer` | performance/* (all), design/05 (Tailwind perf) |
| `ui-typescript-engineer` | typescript/* (all), architecture/* (all) |
| `ui-anti-slop-auditor` | aesthetic/01 (primary), aesthetic/04, design/06 |
| `ui-team-lead` | All references (it routes; doesn't do deep work itself) |

## Hard rules baked into every agent

1. **Read-only review by default.** Findings are advisory. Orchestrator applies on explicit user OK.
2. **Cite references.** Every finding points to a file:section. No hand-waving.
3. **Show code.** Concrete current -> suggested rework. Verbatim-applicable.
4. **Severity tags.** CRITICAL / HIGH / MEDIUM / LOW / NIT.
5. **No AI slop.** No "Great work!", "Just a minor suggestion", "Hope this helps", emojis, hedging, trailing summaries.
6. **No defensive lower bounds.** Real values, not "at least N".
7. **Respect existing project patterns.** Read repo conventions before suggesting; don't impose a different design system on top of a coherent one.
8. **Anti-AI tells are CRITICAL when shipped fresh.** Generic gradients, default shadcn untouched, lazy emoji icons, hashtag palette = blocking findings on greenfield work.

## Severity scale (shared by all agents)

| Tag | Meaning |
|---|---|
| CRITICAL | Will break production / a11y violation that excludes users / data-handling bug / shipped AI-default aesthetic |
| HIGH | Visible degradation: failing CWV target, unhandled async error, type unsoundness in UI state, broken keyboard nav |
| MEDIUM | Quality cost: weak component API, missing exhaustiveness, suboptimal motion, suboptimal token use |
| LOW | Polish: missing JSDoc, could use newer TS feature, minor visual rhythm issue |
| NIT | Style preference -- include sparingly |

## Cross-references

Whenever a reference doc cites another reference, use this exact format so agents can resolve them:

```
See `references/<topic>/<file>.md#<heading-slug>`
```
