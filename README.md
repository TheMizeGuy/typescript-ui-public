# typescript-ui

TypeScript 6 UI engineering team for [Claude Code](https://claude.ai/claude-code). Six Opus specialists that design, review, optimize, and improve UI code so it looks like a senior human design team shipped it -- not AI.

## What it does

Four operations on UI code, each backed by specialized Opus agents:

| Operation | Skill | What happens |
|---|---|---|
| **Design** | `/typescript-ui:typescript-design-ui` | Commits to a distinctive aesthetic POV, generates OKLCH token system, produces production-grade TS + React + Tailwind v4 code |
| **Review** | `/typescript-ui:typescript-review-ui` | Dispatches 2-3 specialists in parallel for design quality, anti-AI aesthetic, accessibility, and TS type safety |
| **Optimize** | `/typescript-ui:typescript-optimize-ui` | Audits Core Web Vitals (LCP/INP/CLS), bundle size, rendering, with quantified metric impact per finding |
| **Improve** | `/typescript-ui:typescript-improve-ui` | Full 4-specialist sweep (design + anti-AI + perf + TS), deduplicated unified report ordered by impact |

All reviews are read-only. Findings are advisory. You pick which to apply.

## Installation

### Method 1: Claude Code plugin directory

```bash
git clone https://github.com/TheMizeGuy/typescript-ui-public.git ~/.claude/plugins/typescript-ui
```

Then add to your Claude Code settings:

```json
{
  "enabledPlugins": {
    "typescript-ui": true
  }
}
```

Restart Claude Code.

### Method 2: Direct plugin directory (single session)

```bash
claude --plugin-dir /path/to/typescript-ui-public
```

### Method 3: Plugin cache

```bash
git clone https://github.com/TheMizeGuy/typescript-ui-public.git /tmp/typescript-ui-public
mkdir -p ~/.claude/plugins/cache/typescript-ui/1.0.0
cp -R /tmp/typescript-ui-public/{.claude-plugin,agents,skills,references,README.md,ARCHITECTURE.md} \
  ~/.claude/plugins/cache/typescript-ui/1.0.0/
```

### Verify installation

After restarting Claude Code, type any of these -- if the skill triggers, it's installed:

```
/typescript-ui:typescript-design-ui
/typescript-ui:typescript-review-ui
/typescript-ui:typescript-optimize-ui
/typescript-ui:typescript-improve-ui
```

## Quick start

### Design a new UI

```
design a settings page for a developer tool -- dark theme, dense, keyboard-first
```

The plugin triggers `typescript-design-ui`, dispatches the design architect, and produces:
- POV statement (e.g. "Tactical Operator -- monospace, functional, single accent")
- Full OKLCH token system (`@theme` block)
- Production-grade TS + React components
- Taste audit results

### Review existing UI

```
/typescript-ui:typescript-review-ui src/components/
```

Dispatches design reviewer + anti-slop auditor in parallel. Returns severity-tagged findings.

### Optimize for performance

```
/typescript-ui:typescript-optimize-ui all
```

Dispatches the perf engineer. Each finding has quantified impact (e.g. "+1200ms LCP").

### Full improvement pass

```
/typescript-ui:typescript-improve-ui src/
```

Dispatches the team lead who coordinates all 4 specialists, deduplicates findings, and presents a unified plan.

## Agents

| Agent | Color | Focus |
|---|---|---|
| `ui-design-architect` | Cyan | Greenfield design, POV commitment, token systems, production code |
| `ui-design-reviewer` | Blue | Visual quality, UX, accessibility, POV coherence (18 lenses) |
| `ui-perf-engineer` | Yellow | Core Web Vitals, bundle, rendering, React perf (12 angles) |
| `ui-typescript-engineer` | Purple | TS6 strictness, component typing, state safety (10 angles) |
| `ui-anti-slop-auditor` | Red | 108 AI-generated aesthetic tell detection |
| `ui-team-lead` | Green | Multi-agent orchestration, finding dedup, unified report |

All agents run on **Opus**. All review agents are **read-only**.

## Knowledge base

26 self-contained reference files (8,600+ lines) organized into 6 domains. Agents read these before every review -- no external dependencies required.

| Domain | Files | Covers |
|---|---|---|
| typescript/ | 4 | TS6 essentials, component typing, state typing, branded primitives |
| design/ | 6 | OKLCH color, typography, spacing, motion, Tailwind v4, shadcn customization |
| aesthetic/ | 4 | 108 anti-AI tells, POV discovery, 12 distinctive system case studies, taste checklist |
| performance/ | 5 | Core Web Vitals, React 19 perf, CSS perf, bundle/loading, measurement |
| accessibility/ | 4 | WCAG 2.2, keyboard/focus, reduced motion, screen reader |
| architecture/ | 3 | Component patterns, state architecture, styling architecture |

## Scope syntax

All review/optimize/improve skills accept the same scope argument:

| Argument | Resolves to |
|---|---|
| (empty) or `diff` | Uncommitted + staged changes |
| `staged` | Only staged files |
| `pr` | Diff vs main/master |
| `<file>` | Single file |
| `<directory>` | All UI files in dir |
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
| **LOW** | Polish -- missing JSDoc, could use newer feature |
| **NIT** | Preference -- included sparingly |

## Optional integrations

The plugin works standalone with its 26 reference files. These optional integrations enhance it:

| Integration | What it adds |
|---|---|
| **Context7** | Live library docs for Tailwind v4, React 19, Motion, TanStack Query |

## Requirements

- Claude Code with Opus model support (agents require Opus)
- For the `typescript-improve-ui` skill: practical concurrency limit is ~4 agents

## License

MIT

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the internal map including agent-to-skill dispatch, reference-to-agent reading assignments, and cross-reference format.
