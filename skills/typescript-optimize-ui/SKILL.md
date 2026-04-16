---
name: typescript-optimize-ui
description: |-
  Use this skill when the user asks to optimize UI performance — Core Web Vitals (LCP, INP, CLS), bundle size, rendering performance, font loading, image optimization, or React-specific patterns (compiler, RSC, Suspense, hydration). Triggers: "optimize my UI", "make it faster", "fix LCP", "reduce bundle size", "optimize for Core Web Vitals", "performance audit", "speed up the page", "reduce CLS", "fix INP". Dispatches the typescript-ui:ui-perf-engineer Opus agent which measures first (Lighthouse / tsc / bundle analysis if available), then produces severity-tagged findings with estimated metric impact and concrete code fixes.
argument-hint: '[path | file | "staged" | "diff" | "pr" | "all"]'
allowed-tools: Bash, Read, Grep, Glob, TodoWrite, Agent
---

# Optimize UI

You are coordinating a performance review and optimization of UI code. Your job is to gather context, dispatch the perf engineer, and present actionable results.

## Step 1: Determine scope

Same resolution rules as `typescript-review-ui`:

| Argument | Meaning |
|---|---|
| (empty) or `diff` | Uncommitted + staged, filtered to UI files |
| `<file>` / `<directory>` | Specific target |
| `all` | Entire project |

Include: components, pages, layouts, styles, configs (next.config, tailwind.config, package.json). Exclude: node_modules, dist, build, .next.

## Step 2: Pre-flight context (parallel)

1. **package.json** — framework, React version, bundler (webpack / Vite / Turbopack).
2. **next.config.ts / vite.config.ts** — any perf-relevant config (images, fonts, experimental flags).
3. **tsconfig.json** — strict flags, target, module.
4. **Existing perf tooling** — Glob for `lighthouserc.*`, `.lighthouseci/`, `web-vitals`, `@vercel/analytics`.
5. **Bundle analysis** — check if `@next/bundle-analyzer` or `rollup-plugin-visualizer` is installed.

## Step 3: Construct the agent prompt

```
SCOPE — review these files for performance:
<absolute path list>

PROJECT CONTEXT:
- Root: <absolute path>
- Framework: <react/next/vite/remix/astro>
- React version: <version>
- Bundler: <webpack/vite/turbopack>
- Tailwind: <v3/v4/none>
- Image optimization: <next/image / manual / none>
- Font loading: <next/font / manual preload / Google Fonts CDN / none>
- Perf tooling installed: <web-vitals / vercel-analytics / lighthouse-ci / none>
- Bundle analyzer: <available / not installed>

PLUGIN REFERENCES: ${CLAUDE_PLUGIN_ROOT}/references/performance/ (5 files). Read them BEFORE reviewing.
Also read: ${CLAUDE_PLUGIN_ROOT}/references/design/05-tailwind-v4.md for Tailwind v4 perf.

TASK:
1. Read all files in scope.
2. Read the 5 performance reference files.
3. If available: run `tsc --noEmit`, build command, and Lighthouse.
4. Identify the likely LCP element per page.
5. Review all 12 perf angles per your system prompt.
6. Quantify estimated impact for each finding ("+800ms LCP", "+0.15 CLS", "+120KB JS").
7. Output in strict finding format with impact line.
8. Severity: CRITICAL (CWV threshold breach), HIGH (significant cost), MEDIUM (material), LOW (minor), NIT (micro).
9. End with perf budget check table + raw tooling output.

HARD RULES:
- Measure first. Run tooling if available.
- Quantify impact. "This is slow" → "This adds ~Xms to Y."
- Show the fix. Current → reworked code.
- Cite references.
- No AI slop.
```

## Step 4: Dispatch

- `subagent_type`: `"typescript-ui:ui-perf-engineer"`
- `description`: `"Perf review of N files"`
- Foreground

## Step 5: Present results

1. Show verbatim.
2. Prompt:
   ```
   Apply any of these optimizations? Tell me which:
   - "all CRITICAL" / "all CRITICAL and HIGH"
   - "finding 3 and 7"
   - "everything in <filename>"
   - "skip"
   ```
3. If user picks, apply with Edit/Write. After applying:
   - Run `tsc --noEmit` to verify fixes compile
   - Run build if possible to check bundle size delta
   - Offer to re-run the perf review to verify improvement
   - Offer to run `typescript-review-ui` if design quality wasn't checked yet

## Anti-patterns

- Don't skip tooling — if build/Lighthouse is available, run it.
- Don't guess metric impact — measure or cite the reference's documented impact.
- Don't summarize agent output.
- Don't auto-apply.
