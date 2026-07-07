# Changelog

## 0.3.1

- DEPRECATED: superseded by the consolidated ui-craft plugin. No further releases.

## 0.3.0

- Adopted the tsgo / TypeScript 7 standard: tsgo (`@typescript/native-preview`) is the primary and only typecheck gate across the plugin's guidance. `references/typescript/01-ts6-essentials.md` retitled to "TypeScript 6/7 Essentials", its "native preview status / run side by side, never replace" section rewritten as "tsgo -- the typecheck gate", the CI script block now maps `typecheck` to `tsgo --noEmit` with `typecheck:ts6` as the legacy lane, and the verify-config tables put `tsgo --noEmit` as the canonical CI gate.
- `ui-typescript-engineer`: runs `tsgo --noEmit` as the gate (tsc fallback with a mandatory adoption finding), angle 1 now checks that `npm run typecheck` is the tsgo gate, description updated to TS6/7.
- Skills (`typescript-optimize-ui`, `typescript-improve-ui`): post-application verification and tooling steps run the tsgo gate instead of bare `tsc --noEmit`.
- README / ARCHITECTURE / plugin.json refreshed to TS6/7 + tsgo-gate wording; `tsgo` keyword added.

## 0.2.2

- Executor model selection v2 (tracks the private source): specialist executor dispatches are conductor-selected -- Sonnet 5 at `xhigh` or Opus 4.8, picked by the work's judgment depth. Conductor-gated; never below `xhigh` for Sonnet; never Haiku.

## 0.2.1

- Model policy: removed every `model: fable` pin from agent frontmatter. Agents now inherit the session model (always the strongest available Claude) instead of pinning a specific one; operative prose ("Backed by Fable 5", "Fable 5 specialists") rewritten to "the session model -- always the strongest available Claude". Skills gained an "Execution mode" note allowing the orchestrator to run a workflow inline when the session model is already top-tier and the task warrants it.
- Optimization pass: `ui-anti-slop-auditor` walks every tell category off the reference file's own headings instead of a memorized count (the catalogue is the single source of truth); `ui-design-architect` gained a POV selection tree, a fixed token-derivation order with a worked decision-log example, and a mechanical pre-ship self-check table (bans hardcoded Tailwind-default colors, pure black/white, placeholder copy, unguarded motion); `ui-team-lead` validates each specialist report against acceptance criteria before merging. All four skills gained explicit acceptance-criteria blocks with a bounded one-retry-then-flag gate.
- `references/aesthetic/01-anti-ai-tells.md` and `04-taste-checklist.md` no longer hardcode the anti-slop plugin's cache version; both resolve the installed version at read time, so the citation survives version bumps.
- Docs: corrected the agent color table (`ui-typescript-engineer` is Magenta, not Purple -- Claude Code's valid agent-color set does not include purple); `plugin.json` `author` is now `{ "name": "TheMizeGuy", "email": "ben@meipath.com" }`; README rewritten for accuracy (agent roster, reference-file counts, install steps, configuration, troubleshooting table).

## 0.2.0

- Skills renamed with a `typescript-` prefix.

## 0.1.0

- Initial public release: 6 agents (design architect, design reviewer, perf engineer, TypeScript engineer, anti-slop auditor, team lead), 4 skills, 26 reference files across typescript/, design/, aesthetic/, performance/, accessibility/, architecture/.
