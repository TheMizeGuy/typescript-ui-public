---
topic: typescript
role: reference
scope: ts6-essentials
audience: ui-engineer
---

# TypeScript 6.0 Essentials for UI Engineers

Covers the TS 6.0 baseline (released 2026-03-23, current LTS line as of 2026-04-14), the tsconfig that any new UI project should ship with, the strictness-ladder for legacy projects, the deprecation warnings that become hard failures in TS 7, and the anti-patterns no UI PR should pass review.

## What changed in TS 6.0

| Area | TS 5.9 default | TS 6.0 default | Why this matters for UI |
|---|---|---|---|
| `strict` | `false` | **`true`** | Strict-null + `noImplicitAny` are on by default; component prop bugs surface at edit time |
| `types` | inferred from `node_modules/@types/*` | **`[]`** | No more accidental `jest`/`mocha` globals leaking into a Vite app; opt in explicitly |
| `rootDir` | longest common ancestor of input files | **dir of `tsconfig.json`** | Predictable `outDir` shape; prevents flattened `dist/` surprises in monorepos |
| `noUncheckedSideEffectImports` | `false` | **`true`** | `import "./foo.css"` now errors if `foo.css` (or its `.d.ts` shim) is missing — catches dead style imports |
| `module` default | `commonjs` | modern ESM-oriented (set explicitly) | Apps and libraries align with bundler/runtime ESM |
| `moduleResolution` | `node`/`node10` | legacy; use `bundler` or `nodenext` | `node10` is deprecated and emits warnings |
| `baseUrl` | implicit lookup root | **deprecated** | Use `package.json` `imports` (`#/...`) or bundler aliases instead |
| `target` | `es5` was historical | shifted modern; `es5` discouraged | UI runtimes are evergreen; `es5` blocks modern emit and bloats output |
| Legacy module formats | `amd`, `umd`, `systemjs`, `none`, `outFile`-era | effectively gone | UI bundlers don't use them |

Source: `~/Claude/vault/TypeScript/17 - TypeScript 6 and 7 Migration Playbook.md`. TS 6.0 is the **bridge release** to native TS 7 (`tsgo`); flags removed in TS 6 are hard errors in 7.

## Required tsconfig for UI projects

Apps (Vite, Next.js, Remix, Astro):

```jsonc
{
  "compilerOptions": {
    "target": "es2022",
    "module": "esnext",
    "moduleResolution": "bundler",
    "lib": ["es2023", "dom", "dom.iterable"],
    "jsx": "react-jsx",

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "useUnknownInCatchVariables": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noFallthroughCasesInSwitch": true,

    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "erasableSyntaxOnly": true,
    "noUncheckedSideEffectImports": true,

    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true,

    "noEmit": true,
    "types": []
  },
  "include": ["src"]
}
```

Why each flag earns its place in a UI project:

| Flag | Why for UI |
|---|---|
| `target: es2022` | Class fields with proper `defineProperty` semantics; `at()`, top-level `await`. Minimum baseline for React 19 + modern bundlers |
| `moduleResolution: bundler` | Vite/Next/Rolldown/esbuild resolve via `package.json` `exports`; matches bundler reality |
| `jsx: react-jsx` | Automatic runtime — no manual `import React`; smaller bundle |
| `strict: true` | TS 6.0 default; explicit for clarity |
| `noUncheckedIndexedAccess` | `arr[0]`, `tuple[1]`, `record[key]` all become `T \| undefined`. Catches "first selected item" + "current row" UI bugs at compile time |
| `exactOptionalPropertyTypes` | `prop?: string` rejects `prop: undefined`. Forces components to express "absent" by omitting the key — matches React's diff/render semantics |
| `useUnknownInCatchVariables` | `catch (e)` is `unknown`. UI fetch/parse code must narrow before reading `.message` |
| `noImplicitOverride` | Subclass methods need `override`. Catches stale lifecycle hooks in class components and renamed React parents |
| `noPropertyAccessFromIndexSignature` | `obj.foo` errors when `foo` came only from an index signature; catches typos in i18n dicts and feature-flag lookups |
| `noFallthroughCasesInSwitch` | UI state machines and reducer switches don't silently fall through |
| `isolatedModules` | Each file transpilable in isolation — required by Vite/SWC/esbuild/tsdown |
| `verbatimModuleSyntax` | `import type` stays `import type`; no import elision; required by Node strip-types and modern bundlers |
| `erasableSyntaxOnly` | Bans enum, namespace-with-values, parameter properties, `import =`/`export =` — TS-only runtime syntax. Aligns with Node 24 `--experimental-strip-types` |
| `noUncheckedSideEffectImports` | TS 6 default; `import "./theme.css"` errors if missing. Kills dead CSS imports |
| `skipLibCheck` | 30–60% faster typecheck; trade off `.d.ts` bug detection in deps for build speed |
| `noEmit: true` | Bundler emits JS; tsc only typechecks |
| `types: []` | TS 6 default; opt in explicitly to `node`, `vite/client`, `vitest/globals` |

Node UI tooling (Vite config, Storybook node side, build scripts):

```jsonc
{
  "compilerOptions": {
    "target": "es2023",
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "lib": ["es2023"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "verbatimModuleSyntax": true,
    "erasableSyntaxOnly": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true,
    "types": ["node"]
  },
  "include": ["vite.config.ts", "scripts"]
}
```

Citations: `~/Claude/vault/TypeScript/02 - Compiler and tsconfig.md` (Strictest preset, Vite preset, module decision matrix); `~/Claude/vault/Projects/TypeScript Dev Plugin/07 - TypeScript 2026 Defaults and Toolchain Decisions.md` (2026 baseline).

## Strictness ladder for existing projects

Adopt incrementally; each step is a separate PR with its own diagnostic wave.

| Step | Flag(s) to enable | What you'll fix |
|---|---|---|
| 1 | `strict: true` (covers `noImplicitAny`, `strictNullChecks`, `useUnknownInCatchVariables`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `alwaysStrict`) | Untyped function params, missing null checks, raw `catch (e)` reads. Largest wave |
| 2 | `noUncheckedIndexedAccess` | `arr[i]` and `record[key]` everywhere — narrow with `if (item)` or `??` |
| 3 | `noImplicitOverride` + `noFallthroughCasesInSwitch` | Add `override` keywords; close switch case fall-throughs in reducers |
| 4 | `noPropertyAccessFromIndexSignature` | Convert dot access on dictionary types to bracket access |
| 5 | `exactOptionalPropertyTypes` | Stop passing `undefined` to optional props; either omit or widen the prop to `T \| undefined`. Largest UI-specific wave; React passes optional props by spreading |

Run each step with `tsc --noEmit` until clean before merging the next.

## TS 6 → 7 migration warnings

TS 6.0 emits deprecation warnings for behaviors that hard-fail in TS 7. Treat warnings as build-blocking now.

| Deprecated in 6, fails in 7 | Replacement |
|---|---|
| `moduleResolution: node` / `node10` | `nodenext` (libs/scripts) or `bundler` (apps) |
| `module: amd` / `umd` / `systemjs` / `none` | `esnext`, `nodenext`, or `preserve` |
| `target: es5` (and `downlevelIteration` to compensate) | `es2022` minimum |
| `baseUrl` as the alias mechanism | `package.json` `imports` (`#shared/*`) or bundler `paths` |
| Import assertions (`assert { type: "json" }`) | Import attributes (`with { type: "json" }`) |
| Floating compiler defaults (`strict`, `module`, `target`, `types`, `rootDir` unset) | Always set these explicitly |
| Numeric `enum`, namespace-with-values, parameter properties, `import =`/`export =` | Forbidden under `erasableSyntaxOnly: true`; rewrite as union + `as const` object |

Citations: `~/Claude/vault/TypeScript/17 - TypeScript 6 and 7 Migration Playbook.md` ("Mandatory 5.9 → 6.0 edits", "Legacy options and patterns to remove").

## tsgo native preview status

`tsgo` is the Go port of the compiler shipped as `@typescript/native-preview` and invoked via `npx tsgo`. Status as of 2026-04-14: **preview only**.

| Use tsgo for | Avoid tsgo for |
|---|---|
| Whole-project type-check smoke lanes (`tsgo --noEmit`) | Anything depending on the `typescript` Compiler API (ts-morph, ts-loader, custom transformers, codemods) |
| CI typecheck speed comparisons against `tsc` | JS/JSDoc-heavy projects (closure-style annotations, `@enum`, `@constructor` are intentionally narrowed in the new JS checker) |
| Large monorepos where `tsc` cold-start dominates | Editor tsserver — tsserver is still TS 6 |
| Side-by-side validation (run both, compare) | Declaration emit for published libraries (incomplete) |

Run side by side, never replace. Keep `typescript@^6.0` as the canonical compiler for emit, declarations, editor, and tooling. Citation: `~/Claude/vault/TypeScript/17 - TypeScript 6 and 7 Migration Playbook.md` ("TS7 evaluation lane").

## Anti-patterns banned in UI code

| Anti-pattern | Replacement | Why |
|---|---|---|
| `enum Status { ... }` (numeric or string) | `const STATUS = { idle: "idle", ... } as const` + `type Status = typeof STATUS[keyof typeof STATUS]` | Enums emit runtime, break under `erasableSyntaxOnly`, poor JSON/URL/JSX interop |
| `namespace Foo { ... }` | ES module — separate file or barrel | Pre-ESM legacy; banned under `erasableSyntaxOnly` |
| `const Btn: React.FC<P> = ...` | `function Btn(props: P) { ... }` | `React.FC` blocks generics, hardcodes return type, historically added implicit `children`. React 19 stripped implicit children but the pattern is still discouraged |
| `const x: any` or `(arg: any)` | `unknown` then narrow with type guard, `instanceof`, or schema parse | `any` poisons every downstream caller; `unknown` forces a check |
| `value as unknown as T` (double cast) | Real validation (Zod / valibot) at the boundary; `satisfies T` if you're proving shape | Double cast is a confessed lie — review-blocker |
| `barrel/index.ts` re-exporting an entire feature | Direct imports `import { Button } from "@/ui/button"` | Barrels block tree-shaking, balloon TS program memory in monorepos, slow incremental builds |
| `// @ts-ignore` | `// @ts-expect-error <issue link or short reason>` | `@ts-expect-error` self-deletes when the underlying error is fixed; `@ts-ignore` rots forever |
| `Function`, `Object`, `{}` as types | Specific signatures; `Record<string, unknown>`; `unknown` | All three accept far more than the writer intended |
| `arr[0].name` after `noUncheckedIndexedAccess` | `arr[0]?.name`, or `const first = arr[0]; if (!first) return null;` | TS 6 typing makes `arr[0]` `T \| undefined`; ignoring it crashes the render |
| `prop: undefined` passed to `prop?: T` under `exactOptionalPropertyTypes` | Omit the key, or widen the prop to `prop?: T \| undefined` | Optional-prop spec means "absent", not "explicitly undefined" |
| Class component with new code | Function component + hooks | Class lifecycle is legacy; React 19 ships ref-as-prop, action hooks, server components — none target classes |
| `useRef<HTMLInputElement>()` (zero-arg) | `useRef<HTMLInputElement>(null)` | React 19 + `@types/react@^19` removed the zero-arg form |

Citations: `~/Claude/vault/TypeScript/03 - Best Practices and Idioms.md` ("Anti-Patterns" table, "Enums vs Unions vs `as const`"); `~/Claude/vault/TypeScript/11 - React with TypeScript.md` ("Component Typing", "Common Anti-Patterns"); `~/Claude/vault/TypeScript/13 - Modern TypeScript Features.md` (TS 5.8 `erasableSyntaxOnly`).

## Verifying the config

Three commands every UI engineer should know:

| Command | Purpose |
|---|---|
| `tsc --showConfig` | Print the **resolved** config after `extends` merging. Use first when a flag "isn't taking" |
| `tsc --noEmit` | Typecheck without writing files. The canonical CI gate |
| `npx tsc --traceResolution 2>&1 \| head -50` | Show how each `import` resolved, in order. Use when a `paths`/`exports`/`bundler` resolution surprises you |

Other diagnostics worth knowing:

| Command | Purpose |
|---|---|
| `tsc --listFiles` | Every file pulled into the program (catches accidental `node_modules` inclusion) |
| `tsc --extendedDiagnostics` | Type-check timings + memory; find slow `@types/*` |
| `tsc --generateTrace ./trace` | Chromium-trace-format profile (open with `chrome://tracing`) |
| `npx tsgo --noEmit` | Native preview type-check; compare timings against `tsc --noEmit` |

CI script:

```jsonc
// package.json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "typecheck:trace": "tsc --noEmit --extendedDiagnostics",
    "typecheck:fast": "tsgo --noEmit"
  }
}
```

Citations: `~/Claude/vault/TypeScript/02 - Compiler and tsconfig.md` (Performance flags, Emit flags).

## Cross-references

| File | Covers |
|---|---|
| `references/typescript/02-component-typing.md` | Function components, polymorphic `as` prop, compound components, slots, render props, refs in React 19, CVA variants, banned HOC patterns |
| `references/typescript/03-state-typing.md` | Discriminated unions for async/form/modal state, reducer typing, URL vs server vs client state split, Zod at the boundary |
| `references/typescript/04-branded-primitives.md` | Brand pattern (intersection / unique symbol / Tagged), smart constructors, UI primitive brand catalogue, Zod brand parsing |
| `references/architecture/01-*.md` | Project layout, monorepo references, Vite/Next/tsdown wiring |
