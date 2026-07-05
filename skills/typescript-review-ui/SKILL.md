---
name: typescript-review-ui
description: |-
  Use this skill when the user asks to review existing UI code for design quality, accessibility, or AI-generated aesthetic tells. Triggers: "review my UI", "review this component", "check the design quality", "is this accessible?", "does this look AI-generated?", "audit the UI", "check the components for design issues", "a11y check". Dispatches up to 3 typescript-ui specialists in parallel (ui-design-reviewer + ui-anti-slop-auditor + ui-typescript-engineer) and merges findings into a unified report. Read-only — does not modify files.
argument-hint: '[path | file | "staged" | "diff" | "pr" | "all"]'
allowed-tools: Bash, Read, Grep, Glob, TodoWrite, Agent
---

# Review UI

You are coordinating a UI design + accessibility + aesthetic review. Your job is to determine scope, gather context, dispatch 2-3 specialists, and present a merged report.

## Execution mode

Agents inherit the session model — always the strongest Claude available to this session. If the session model is already the strongest tier and the review is important or complex, run the review inline in the main context (foreground) instead of dispatching, following the same process the specialists use. Never block on, or call out to, an unavailable model. Reviewer agents stay read-only regardless of dispatch mode.

## Step 1: Determine scope

Resolve the argument to a concrete file list (same as other review skills):

| Argument | Meaning |
|---|---|
| (empty) or `diff` | Uncommitted + staged changes, filtered to `*.tsx`/`*.jsx`/`*.css`/`*.scss` |
| `staged` | Only staged files |
| `pr` | Diff vs `main`/`master` |
| `<file>` | Single file |
| `<directory>` | All UI files in dir (Glob `**/*.{tsx,jsx,css,scss}` excluding node_modules/dist/build) |
| `all` | Entire project (excluding standard ignore patterns) |

Filter to UI-relevant files: components, pages, layouts, styles. Skip pure utility/API/model files unless they have JSX.

If empty scope, tell user and suggest alternatives. If >50 files, warn and ask.

## Step 2: Pre-flight context (parallel)

1. **package.json** — framework, React version, Tailwind version, design libraries.
2. **Token system** — Read `globals.css` / `tailwind.config.*`. Note OKLCH vs hex vs shadcn default.
3. **Linter config** — eslint/biome for existing design lint rules.
4. **tsconfig.json** — strict flags (for the TS engineer).

## Step 3: Decide which specialists to dispatch

| Question | Specialist |
|---|---|
| Always | `typescript-ui:ui-design-reviewer` — visual + UX + a11y |
| Always | `typescript-ui:ui-anti-slop-auditor` — AI-tell detection |
| If user mentions "type safety", TS review, or scope includes .ts/.tsx logic | `typescript-ui:ui-typescript-engineer` |

Dispatch 2-3 agents.

## Step 4: Construct prompts

Each agent gets a self-contained prompt with:
- Absolute file paths
- Project context (framework, Tailwind, tsconfig, token system snapshot)
- Instruction to read their plugin references BEFORE reviewing
- Output in strict finding format

Include these ACCEPTANCE CRITERIA in every specialist prompt, and check them on each returned report before merging:
1. Report opens with the specialist's summary block (scope, finding counts by severity, one-line verdict).
2. Every finding carries: severity tag, `file:line`, current code extract, concrete rework, reference citation.
3. Read-only respected — the report claims no file modifications.

A report failing any criterion → ONE re-dispatch naming the failed item; a second failure → include the raw output in the merge, flagged as non-conforming.

## Step 5: Dispatch agents

Dispatch all chosen specialists in parallel using the Agent tool:
- Design Reviewer: `subagent_type: "typescript-ui:ui-design-reviewer"`
- Anti-Slop Auditor: `subagent_type: "typescript-ui:ui-anti-slop-auditor"`
- TS Engineer (if included): `subagent_type: "typescript-ui:ui-typescript-engineer"`
- Foreground execution
- `description`: `"<Agent name>: N files"`

## Step 6: Merge and present

1. Collect findings from all agents.
2. Deduplicate — same file:line range, same issue from multiple agents = keep the most specific, credit the source.
3. Re-number sequentially.
4. Sort: CRITICAL → HIGH → MEDIUM → LOW → NIT, then by file within severity.

Present:

```
## UI Review

**Scope:** <files, count>
**Specialists:** Design Reviewer, Anti-Slop Auditor[, TS Engineer]
**Token system:** <OKLCH / default shadcn / hex>
**Primary font:** <name — PASS/FAIL>
**POV detected:** <description or "none">
**Findings:** N CRITICAL, N HIGH, N MEDIUM, N LOW, N NIT
**Verdict:** <ship as-is / fix HIGH+ / needs design pass / generic, redesign>
```

All findings in strict format.

```
Apply any of these findings? Tell me which:
- "all CRITICAL" / "all CRITICAL and HIGH"
- "finding 3 and 7"
- "everything in <filename>"
- "skip" to handle yourself
```

## Step 7: Apply if requested

If user picks findings, YOU apply them using Edit/Write. Do not re-dispatch a reviewer. After applying:
- Offer a re-review on the same scope
- Offer to run `typescript-optimize-ui` if perf wasn't in this pass

## Anti-patterns

- Don't dispatch without project context.
- Don't summarize findings — show verbatim.
- Don't auto-apply — wait for user pick.
- Don't run in background.
