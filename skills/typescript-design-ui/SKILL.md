---
name: typescript-design-ui
description: |-
  Use this skill when the user asks to design new UI — a screen, flow, page, component, or full product. Triggers: "design a [thing]", "build me a [screen/page/flow]", "create the UI for", "design the [dashboard/settings/onboarding/landing]", "make this look [distinctive/professional/not AI]". Dispatches the typescript-ui:ui-design-architect agent (runs on the session model) which commits to a distinctive aesthetic POV, generates a token system (OKLCH + variable fonts + modular spacing + spring motion), and produces production-grade TypeScript + React + Tailwind v4 code that does not look AI-generated. Uses the anti-AI-tells catalogue (108 patterns) as a hard floor and the taste checklist as a pre-ship gate.
argument-hint: '<brief description of what to design>'
allowed-tools: Bash, Read, Grep, Glob, TodoWrite, Agent
---

# Design UI

You are coordinating new UI design on the user's behalf. Your job is to gather project context, construct a self-contained prompt, and dispatch the `typescript-ui:ui-design-architect` agent, which inherits the session model.

## Execution mode

Agents inherit the session model — always the strongest Claude available to this session. If the session model is already the strongest tier and the design task is important or complex, design inline in the main context (foreground) instead of dispatching, following the same knowledge sources and process as `ui-design-architect`. Never block on, or call out to, an unavailable model.

## Step 1: Parse the brief

The user passed a description (may be empty). Extract:
- What to design (screen, page, flow, component family, full product)
- Any stated constraints ("dark theme", "minimal", "dense dashboard")
- Any stated POV ("like Linear", "editorial", "tactical")
- Any stated framework (React, Next.js, Svelte, etc.)

If the brief is empty or too vague to act on, ask one focused question: "What screen or flow should I design? Any aesthetic direction?" Do NOT proceed without knowing what to build.

## Step 2: Pre-flight context (run in parallel)

If working in an existing repo:

1. **package.json** — Read it. Note: framework, React version, Tailwind version, existing design libraries.
2. **Token system** — Read `app/globals.css`, `tailwind.config.*`, or any `theme.ts`/`:root` blocks. Note existing tokens, color format (OKLCH vs hex vs HSL), and whether shadcn defaults are in place.
3. **Existing components** — Glob `**/components/ui/*.tsx` or similar. List what primitives exist.
4. **Workspace root** — `git rev-parse --show-toplevel` for absolute paths.

If no repo, greenfield is assumed.

## Step 3: Construct the agent prompt

Build a self-contained prompt for the design architect. It has zero conversation context — everything it needs goes in the prompt.

```
BRIEF:
<what to design, from the user>

CONSTRAINTS:
<aesthetic direction, brand, framework, existing tokens, any stated POV>

PROJECT CONTEXT (if existing repo):
- Root: <absolute path>
- Framework: <react/next/vite/svelte/none>
- React version: <version>
- Tailwind: <v3/v4/none>
- Existing tokens: <OKLCH/shadcn default/hex/none>
- Existing component count: <N>
- Key deps: <zod, tanstack-query, etc.>

PLUGIN REFERENCES: ${CLAUDE_PLUGIN_ROOT}/references/ — read the files listed in your system prompt's reference table BEFORE designing.

TASK:
1. Read the relevant reference files (aesthetic/*, design/*, accessibility/01-03, architecture/01+03).
2. Inspect existing repo components and tokens if applicable.
3. Commit to a POV. Write the 3-4 sentence statement.
4. Generate the full token system (OKLCH color, font stack, spacing, motion, radius).
5. Design each requested component/screen with production-grade TypeScript + JSX.
6. Run the taste audit (references/aesthetic/04-taste-checklist.md) — report PASS/FAIL per section.
7. Output the POV, tokens, components, composition, taste audit, and open questions.

HARD RULES:
- No AI-default aesthetic. 108-tell catalogue is the floor.
- OKLCH colors only (no hex, no HSL in the token system).
- Distinctive font stack (no Inter/Roboto/Helvetica/Arial as primary).
- Production code — compiles under TS6 strict. No // TODO, no placeholders.
- Real product copy — no lorem ipsum, no "Submit", no "Get started in seconds".
- prefers-reduced-motion honored on all decorative animation.
- APCA Lc 75+ on body text.
- No emojis, no AI slop, no trailing summary.

ACCEPTANCE CRITERIA (output is rejected if any fails):
1. POV statement present, 3-4 sentences, names what the design does NOT do.
2. Token block present; every color value is oklch(...); states derived via relative color syntax.
3. Every requested component has a file path + complete code (no elided bodies).
4. At least one full composition example in real JSX.
5. Taste audit table present with PASS/FAIL per checklist section.
6. Zero hits for: #6366f1, #14b8a6, #000000, #ffffff, "lorem", "Get started", "Submit", "TODO".
```

## Step 4: Dispatch the agent

Use the Agent tool:
- `subagent_type`: `"typescript-ui:ui-design-architect"`
- `description`: `"Design <brief summary>"`
- `prompt`: the prompt from Step 3
- Foreground (NOT `run_in_background: true`)

## Step 5: Present results

1. Gate the output against the ACCEPTANCE CRITERIA from Step 3 (all six, mechanically). Any failure → ONE re-dispatch naming the exact failed criterion. A second failure → present anyway, flagging the gap explicitly.
2. Display the agent's output verbatim. Do not summarize or reformat.
3. Prompt:
   ```
   Apply this design to the project? Options:
   - "apply all" — write every component + token file
   - "apply tokens only" — just the @theme / globals.css changes
   - "apply [component name]" — specific components
   - "revise [aspect]" — adjust POV/colors/typography/layout
   - "skip" — take the output and apply yourself
   ```
4. If the user asks to apply, YOU (the orchestrator) create the files using Write/Edit. Do not re-dispatch the agent.
5. After applying, offer to run `typescript-review-ui` on the new components for a second-opinion audit.

## When to skip parts

- **No repo**: Skip pre-flight. Greenfield is fine.
- **User gave strong constraints**: Don't re-ask what they already said. Pass through to the prompt.
- **Simple component request** (e.g. "design a button"): Still dispatch the architect — even a single button should carry the project's POV.

## Anti-patterns to avoid

- Don't dispatch without a brief — ask first.
- Don't dispatch without project context if there IS a repo — the architect needs it.
- Don't summarize the agent output — show it raw.
- Don't auto-apply — wait for explicit user approval.
- Don't run in background — user wants to watch progress.
