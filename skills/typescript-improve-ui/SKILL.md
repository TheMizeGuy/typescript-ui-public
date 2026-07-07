---
name: typescript-improve-ui
description: |-
  Use this skill when the user asks to comprehensively improve existing UI — design + performance + type safety + anti-AI aesthetics in one pass. Triggers: "improve this UI", "make this better", "make this god tier", "polish these components", "refactor the UI", "level up the design", "make this look like a human team built it", "full UI pass", "improve everything". Dispatches the typescript-ui:ui-team-lead orchestrator (runs on the session model) which coordinates 4 specialist agents in parallel (design reviewer, anti-slop auditor, perf engineer, TS engineer), deduplicates findings, and produces a unified improvement plan ordered by impact. This is the heavy-hitter — use when the user wants the full treatment.
argument-hint: '[path | file | "staged" | "diff" | "pr" | "all"]'
allowed-tools: Bash, Read, Grep, Glob, TodoWrite, Agent
---

# Improve UI

You are coordinating a comprehensive multi-specialist UI improvement pass. This is the plugin's flagship skill — it dispatches the team lead who orchestrates 4 specialist agents.

## Execution mode

Agents inherit the session model — always the strongest Claude available to this session. If the session model is already the strongest tier and the improvement pass is important or complex, run the pass inline in the main context (foreground) instead of dispatching, following the same 4-specialist process the team lead uses. Never block on, or call out to, an unavailable model. The 4 specialist reviewers stay read-only regardless of dispatch mode.

## Step 1: Determine scope

Same resolution rules as `typescript-review-ui` / `typescript-optimize-ui`:

| Argument | Meaning |
|---|---|
| (empty) or `diff` | Uncommitted + staged |
| `<file>` / `<directory>` | Specific target |
| `all` | Entire project |

Include all UI-relevant files: components, pages, layouts, styles, theme config, tsconfig. Exclude standard patterns.

Warning thresholds:
- >100 files: "This is a large scope. The team lead will dispatch 4 agents in parallel — this will take several minutes. Continue or narrow the scope?"
- 0 files: tell user and suggest alternatives.

## Step 2: Pre-flight context (parallel, comprehensive)

This is the full treatment — gather everything:

1. **package.json** — framework, React version, Tailwind version, ALL relevant deps.
2. **tsconfig.json** — ALL strictness flags.
3. **Token system** — full read of `globals.css`, `tailwind.config.*`, `:root` blocks.
4. **Linter config** — eslint / biome config.
5. **Component structure** — Glob `**/components/**/*.tsx` and count + list.
6. **Page structure** — Glob `**/app/**/page.tsx` or `**/pages/**/*.tsx`.
7. **Font usage** — Grep for font-family, @font-face, next/font, Google Fonts.
8. **Icon usage** — Grep for lucide-react, @heroicons, @tabler/icons, @phosphor-icons.
9. **Image usage** — Grep for `<img`, `<Image`, `next/image`.
10. **Perf tooling** — Glob for lighthouse, web-vitals, analytics configs.
11. **Workspace root** — `git rev-parse --show-toplevel`.

## Step 3: Construct the team lead prompt

The team lead needs FULL context because it dispatches 4 sub-agents, each of which needs a complete briefing.

```
SCOPE — comprehensive UI improvement pass on these files:
<absolute path list>

PROJECT CONTEXT:
- Root: <absolute path>
- Framework: <name + version>
- React: <version>
- Tailwind: <v3/v4/none>
- tsconfig: strict=<bool>, noUncheckedIndexedAccess=<bool>, exactOptionalPropertyTypes=<bool>, verbatimModuleSyntax=<bool>
- Linter: <eslint flat / legacy / biome / none>
- Token system: <OKLCH / shadcn default / hex / custom>
- Primary font: <name>
- Icon source: <lucide / heroicons / custom / mixed>
- Component count: <N>
- Page/route count: <N>
- Perf tooling: <web-vitals / vercel-analytics / lighthouse-ci / none>

USER GOAL: <user's words verbatim — "make this god tier", "improve for launch", etc.>

PLUGIN REFERENCES: ${CLAUDE_PLUGIN_ROOT}/references/ — all 26 files.

TASK:
1. Read ARCHITECTURE.md for the reference → agent mapping.
2. Dispatch 4 specialists in parallel:
   a. ui-design-reviewer — visual + UX + accessibility
   b. ui-anti-slop-auditor — AI-tell detection
   c. ui-perf-engineer — CWV + bundle + rendering
   d. ui-typescript-engineer — TS6/7 strictness, tsgo gate, component typing
3. Each specialist gets the full PROJECT CONTEXT above.
4. Wait for all 4.
5. Merge findings: deduplicate, re-rank by impact, number sequentially.
6. Present the unified report in the format from your system prompt.
7. End with the structured improvement plan (quick wins / design pass / perf pass / type safety pass).

HARD RULES:
- Dispatch real agents. Don't simulate their output.
- Agents inherit the session model — never pinned, never Haiku.
- Foreground execution.
- Deduplicate cross-agent findings.
- No AI slop.

ACCEPTANCE CRITERIA (merged report is rejected if any fails):
1. Summary block present with per-dimension status and total finding counts by severity.
2. All 4 specialists accounted for — a report per specialist, or a named failure with its error.
3. Findings deduplicated, sequentially numbered, severity-ordered; every finding keeps file:line + rework + reference.
4. Improvement plan present with all four passes (quick wins / design / perf / type safety).
```

## Step 4: Dispatch the team lead

Dispatch via `general-purpose`, not the plugin namespace — `ui-team-lead` declares the `Agent` tool to dispatch its 4 specialists, and plugin-namespaced dispatch silently strips it at runtime (mem `019d8bcb`; see the RUNTIME DISPATCH NOTE in `agents/ui-team-lead.md`). Read `${CLAUDE_PLUGIN_ROOT}/agents/ui-team-lead.md` and inline its body as the prompt prefix.

- `subagent_type`: `"general-purpose"`
- `description`: `"Full UI improvement: N files"`
- `prompt`: the full body of `agents/ui-team-lead.md`, then the prompt from Step 3
- Foreground

## Step 5: Present results

1. Gate the merged report against the ACCEPTANCE CRITERIA from Step 3. Any failure → ONE re-dispatch naming the failed criterion; a second failure → present flagged.
2. Show the team lead's merged report verbatim.
3. Prompt:
   ```
   This is the full improvement report from 4 specialist agents. Apply changes? Options:
   - "all CRITICAL" / "all CRITICAL and HIGH"
   - "quick wins only"
   - "design pass" / "perf pass" / "type safety pass"
   - "finding 3, 7, 12, 15"
   - "everything in <filename>"
   - "skip"
   ```
4. If the user picks, apply each finding's suggested rework using Edit/Write.
5. After applying a batch, offer:
   - Re-run the full improvement pass to verify the fixes
   - Run individual skills (`typescript-review-ui`, `typescript-optimize-ui`) on specific areas
   - Write a GoodMem learning if a non-obvious pattern came up

## Step 6: Post-application verification

After applying any fixes:
1. Run the typecheck gate (`tsgo --noEmit`; `tsc --noEmit` where tsgo is absent) to verify compilation.
2. Run the project's lint command.
3. If Tailwind: check that `@theme` tokens are valid.
4. Report any breakage with the fix — offer to iterate.

## Anti-patterns

- Don't dispatch the team lead for a single-dimension review. Use `typescript-review-ui` (design), `typescript-optimize-ui` (perf), or the TS senior review plugin.
- Don't dispatch without comprehensive project context — the team lead needs it for all 4 sub-agents.
- Don't summarize the report.
- Don't auto-apply.
- Don't run in background — user wants real-time progress.
