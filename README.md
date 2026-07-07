# typescript-ui

TypeScript 6/7 UI engineering team for [Claude Code](https://claude.com/claude-code). Six specialists -- running on the session model, always the strongest available Claude -- that design, review, optimize, and improve UI code so it looks like a senior human design team shipped it, not AI. Type safety is verified with the tsgo (TypeScript 7) gate.

## Table of Contents

- [What it does](#what-it-does)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Skills reference](#skills-reference)
- [Agents reference](#agents-reference)
- [Knowledge base](#knowledge-base)
- [Scope syntax](#scope-syntax)
- [Finding format](#finding-format)
- [Configuration](#configuration)
- [Workflows](#workflows)
- [Troubleshooting](#troubleshooting)
- [Architecture](#architecture)

## What it does

Four operations on UI code, each backed by specialized agents running on the session model:

| Operation | Skill | What happens |
|---|---|---|
| **Design** | `/typescript-ui:typescript-design-ui` | Commits to a distinctive aesthetic point-of-view, generates an OKLCH token system, produces production-grade TS + React + Tailwind v4 code |
| **Review** | `/typescript-ui:typescript-review-ui` | Dispatches 2-3 specialists in parallel for design quality, anti-AI aesthetic, accessibility, and TS type safety |
| **Optimize** | `/typescript-ui:typescript-optimize-ui` | Audits Core Web Vitals (LCP/INP/CLS), bundle size, rendering, with quantified metric impact per finding |
| **Improve** | `/typescript-ui:typescript-improve-ui` | Full 4-specialist sweep (design + anti-AI + perf + TS), deduplicated unified report ordered by impact |

All reviews are read-only. Findings are advisory. You pick which to apply.

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/typescript-ui-public.git

# 2. Install the plugin
claude plugin install typescript-ui@typescript-ui-public

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list`. Updates ship through the same channel: when a new release lands, run `claude plugin marketplace update typescript-ui-public` then `claude plugin update typescript-ui@typescript-ui-public`, or accept the update prompt in `/plugin`.

Manual alternative: `git clone https://github.com/TheMizeGuy/typescript-ui-public.git` and load with `claude --plugin-dir <path>`.

## Quick start

### Design a new UI

```
design a settings page for a developer tool -- dark theme, dense, keyboard-first
```

The plugin triggers `typescript-design-ui`, dispatches the design architect, and produces:
- A POV statement (e.g. "Tactical Operator -- monospace, functional, single accent")
- A full OKLCH token system (`@theme` block)
- Production-grade TS + React components
- Taste audit results

### Review existing UI

```
/typescript-ui:typescript-review-ui src/components/
```

Dispatches the design reviewer and anti-slop auditor in parallel. Returns severity-tagged findings:

```
### [CRITICAL] Component: default shadcn tokens untouched
File: src/app/globals.css:1-15
Issue: --background and --foreground use shadcn default HSL values
...
```

### Optimize for performance

```
/typescript-ui:typescript-optimize-ui all
```

Dispatches the perf engineer. Each finding has quantified impact:

```
### [CRITICAL] LCP: hero image lazy-loaded
File: src/app/page.tsx:42
Impact: LCP +1200ms estimated
...
```

### Full improvement pass

```
/typescript-ui:typescript-improve-ui src/
```

Dispatches the team lead, who coordinates all 4 specialists, deduplicates findings across agents, and presents a unified plan:
- Quick wins (under 30 min each)
- Design pass (requires creative decisions)
- Performance pass (measurement needed)
- Type safety pass (mechanical)

## Skills reference

### typescript-design-ui

```
/typescript-ui:typescript-design-ui <brief>
```

**Triggers:** "design a [thing]", "build me a [screen/page/flow]", "create the UI for", "make this look distinctive/professional/not AI"

**Process:**
1. Gathers project context (`package.json`, existing tokens, component primitives)
2. Dispatches `ui-design-architect` (session model)
3. The architect reads the aesthetic, design, and accessibility references
4. Commits to a POV from 3 templates (Tactical Operator / Editorial Magazine / Workshop)
5. Generates a full OKLCH token system, type stack, spacing scale, and motion config
6. Designs each component with TS6 strict typing
7. Runs the 54-item taste checklist
8. Returns: POV, tokens, components, composition, taste audit, open questions

**After output:** you choose what to apply (all, tokens only, specific components, revisions).

### typescript-review-ui

```
/typescript-ui:typescript-review-ui [scope]
```

**Triggers:** "review my UI", "check the design quality", "is this accessible?", "does this look AI-generated?", "audit the UI", "a11y check"

**Specialists dispatched:**
- `ui-design-reviewer` -- 18 lenses (POV, color, typography, spacing, motion, composition, affordances, feedback, keyboard, screen reader, contrast, WCAG 2.2, density, copy, rhythm, distinctiveness)
- `ui-anti-slop-auditor` -- 108-tell AI aesthetic catalogue
- `ui-typescript-engineer` (if the scope includes `.ts`/`.tsx`) -- 10 TS6/7 angles incl. the tsgo gate

**Output:** merged, deduplicated findings sorted CRITICAL > HIGH > MEDIUM > LOW > NIT.

### typescript-optimize-ui

```
/typescript-ui:typescript-optimize-ui [scope]
```

**Triggers:** "optimize my UI", "make it faster", "fix LCP", "reduce bundle size", "optimize for Core Web Vitals", "speed up the page"

**Specialist:** `ui-perf-engineer` -- 12 angles (LCP, INP, CLS, bundle, images, fonts, rendering, React patterns, loading strategy, bfcache, third-party, measurement)

**Runs tooling when available:** the typecheck gate (`tsgo --noEmit`; `tsc --noEmit` where tsgo is absent), the project's build command, Lighthouse, bundle analysis.

**Each finding has quantified impact:** "+800ms LCP", "+0.15 CLS", "+120KB JS".

### typescript-improve-ui

```
/typescript-ui:typescript-improve-ui [scope]
```

**Triggers:** "improve this UI", "make this better", "make this god tier", "polish these components", "level up the design", "full UI pass"

**Dispatches:** `ui-team-lead`, which coordinates all 4 reviewers in parallel, then merges into one report.

**Warning:** more than 100 files triggers a scope confirmation prompt (4 agents running in parallel is significant compute).

## Agents reference

| Agent | Color | Focus |
|---|---|---|
| `ui-design-architect` | Cyan | Greenfield design, POV commitment, token systems, production code |
| `ui-design-reviewer` | Blue | Visual quality, UX, accessibility, POV coherence (18 lenses) |
| `ui-perf-engineer` | Yellow | Core Web Vitals, bundle, rendering, React perf (12 angles) |
| `ui-typescript-engineer` | Magenta | TS6/7 strictness, tsgo gate, component typing, state safety (10 angles) |
| `ui-anti-slop-auditor` | Red | 108 AI-generated aesthetic tell detection |
| `ui-team-lead` | Green | Multi-agent orchestration, finding dedup, unified report |

All agents inherit **the session model** -- always the strongest available Claude. All review agents are **read-only** (no Edit/Write). Only the team lead has the Agent tool (to dispatch sub-agents).

## Knowledge base

26 self-contained reference files (8,600+ lines) organized into 6 domains. Agents read these before every review -- no external dependencies required.

| Domain | Files | Covers |
|---|---|---|
| `typescript/` | 4 | TS6/7 essentials + tsgo gate, component typing, state typing, branded primitives |
| `design/` | 6 | OKLCH color, typography, spacing, motion, Tailwind v4, shadcn customization |
| `aesthetic/` | 4 | 108 anti-AI tells, POV discovery, 12 distinctive-system case studies, taste checklist |
| `performance/` | 5 | Core Web Vitals, React 19 perf, CSS perf, bundle/loading, measurement |
| `accessibility/` | 4 | WCAG 2.2, keyboard/focus, reduced motion, screen reader |
| `architecture/` | 3 | Component patterns, state architecture, styling architecture |

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full file tree and the reference-to-agent reading assignments.

## Scope syntax

All review/optimize/improve skills accept the same scope argument:

| Argument | Resolves to |
|---|---|
| (empty) or `diff` | Uncommitted + staged changes |
| `staged` | Only staged files |
| `pr` | Diff vs main/master |
| `<file>` | Single file |
| `<directory>` | All UI files in the directory |
| `all` | Entire project |

Files filtered to: `.tsx`, `.jsx`, `.css`, `.scss`. Excludes `node_modules`, `dist`, `build`, `.next`, `out`, `coverage`.

## Finding format

Every agent produces findings in the same structure:

```
### [SEVERITY] [Category]: one-line title

File: path/to/file.tsx:42-58
Issue: plain-English explanation
Why it matters: concrete consequence

Current code:
  (5-15 line extract)

Suggested rework:
  (concrete rewrite, applicable verbatim)

Reference: references/<domain>/<file>.md #section
```

### Severity scale

| Tag | Meaning |
|---|---|
| **CRITICAL** | Blocks ship -- a11y violation, shipped AI-default aesthetic, CWV threshold breach, type unsoundness |
| **HIGH** | Fix before launch -- failing contrast, broken keyboard nav, significant perf cost |
| **MEDIUM** | Quality cost -- weak composition, missing exhaustiveness, suboptimal motion |
| **LOW** | Polish -- missing JSDoc, could use a newer feature |
| **NIT** | Preference -- include sparingly |

### Applying findings

After every review, you're prompted:

```
Apply any of these findings? Tell me which:
- "all CRITICAL" / "all CRITICAL and HIGH"
- "finding 3 and 7"
- "everything in <filename>"
- "skip" to handle yourself
```

The orchestrator (not the reviewer agent) applies your selections using Edit/Write.

## Configuration

The plugin works standalone with its 26 reference files. These integrations are optional and enhance it further.

### GoodMem (optional)

If you run a [GoodMem](https://github.com/goodmemai/goodmem) instance, agents search it for prior design decisions and gotchas before reviewing. Fill in your own space and reranker IDs below:

| Space | ID | Purpose |
|---|---|---|
| Learnings | `<your-goodmem-learnings-space-id>` | Prior debugging findings, API quirks, library gotchas |
| UserContext | `<your-goodmem-usercontext-space-id>` | User preferences (e.g. preferred fonts, color palettes) |

Reranker: pass the Voyage `rerank-2.5` (or equivalent) post-processor block, `<your-goodmem-reranker-id>`. Without GoodMem, agents still function using the self-contained reference files.

### Obsidian vault (optional)

Agents can supplement the reference files with depth from a vault, if one is configured, e.g.:
- Your own TypeScript reference vault, if you keep one
- Your own UI design reference vault, if you keep one
- Your own SEO reference vault, if you keep one (Core Web Vitals deep reference)

Without a vault, agents rely entirely on the 26 plugin reference files. Adjust the paths to match your own notes.

### Context7 (optional)

Agents use Context7 for live library docs (Tailwind v4, React 19, Motion, TanStack Query, etc.). If Context7 is unavailable or rate-limited, agents fall back to reference files (and the vault, if configured).

## Workflows

### New project setup

```
1. /typescript-ui:typescript-design-ui "dashboard for [product] -- [adjectives]"
   -> Generates POV, tokens, components
2. Apply the token system and components
3. /typescript-ui:typescript-review-ui all
   -> Catches any remaining AI tells or a11y gaps
4. Fix findings
5. /typescript-ui:typescript-optimize-ui all
   -> Verifies CWV targets met
```

### Pre-launch audit

```
/typescript-ui:typescript-improve-ui all
```

Single command dispatches all 4 specialists. Fix CRITICAL and HIGH before launch.

### Post-feature review

```
/typescript-ui:typescript-review-ui diff
```

Reviews only your uncommitted changes. Fast, focused.

### Performance regression diagnosis

```
/typescript-ui:typescript-optimize-ui src/app/page.tsx
```

Single-file perf audit. Traces the LCP element, checks loading strategy, quantifies impact.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Skills not appearing | Plugin not in cache, or not enabled | Verify the cache path exists (`ls ~/.claude/plugins/cache/typescript-ui/<version>/`), verify `enabledPlugins` in `~/.claude/settings.json`, then restart Claude Code -- a new session is required after settings changes |
| Agent dispatch fails | Rarely a model issue -- agents inherit the session model, not a fixed one | Confirm the session itself is healthy; if the team lead's parallel dispatch hits provider rate limits, reduce scope (fewer files, narrower `<directory>`) |
| Context7 quota exceeded | Third-party API rate limit | No action needed -- agents fall back to the plugin's reference files (and the vault, if configured); quality stays high since Context7 is supplementary |
| Empty scope | No files match the scope filter | The skill reports this and suggests alternatives; confirm your working directory has `.tsx`/`.jsx`/`.css`/`.scss` files |
| Findings reference missing files | Reference paths resolve via `${CLAUDE_PLUGIN_ROOT}/references/...` at runtime; the cache copy is stale or incomplete | Re-copy from source: `cp -R /tmp/typescript-ui-public/references/ ~/.claude/plugins/cache/typescript-ui/<version>/references/` (adjust the source path to wherever you cloned the repo) |
| `typescript-improve-ui` refuses to run on a huge tree | More than 100 files triggers a scope-confirmation prompt by design | Confirm the prompt to proceed, or narrow the scope to a subdirectory |

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the internal map, including:
- Agent-to-skill dispatch mapping
- Reference-to-agent reading assignments
- Shared severity scale
- Cross-reference format
- Hard rules baked into every agent
