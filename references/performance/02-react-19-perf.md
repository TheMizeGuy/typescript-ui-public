---
topic: performance
role: reference
scope: react-19
audience: ui-engineer
---

# React 19 Performance — Compiler, RSC, Suspense, Concurrent Hooks

Vault anchors: `TypeScript/11 - React with TypeScript.md`, `Projects/TypeScript Dev Plugin/08 - Frontend Performance and Load-Time Doctrine.md`. React 19 is stable as of Dec 2024; React Compiler stable with 19.0+. Install `@types/react@^19` + `@types/react-dom@^19`.

## React Compiler (stable with 19.0+)

| Aspect | Detail |
|--------|--------|
| What it does | Auto-memoizes components, hooks, and derived values. Generates the equivalent of `React.memo` / `useMemo` / `useCallback` at compile time, guided by React's Rules of React |
| Why it exists | Manual memoisation is error-prone (missing deps, stale closures, over-memoising). Compiler is correct by construction and typically outperforms hand-tuned memos |
| Opt-in | Babel plugin `babel-plugin-react-compiler` (Next.js 15+: `experimental.reactCompiler: true` in `next.config.js`; Vite: plugin from `babel-plugin-react-compiler`) |
| Disable a component | `"use no memo"` directive at the top of the function body. Reserved for components with impure code or unusual patterns |
| Healthcheck | Run the ESLint plugin `eslint-plugin-react-compiler` in CI — it reports any component the compiler refused to compile, with the reason |
| When to disable manually | Rare. Only when profiling shows the compiler-memoised component is slower than a hand-tuned variant (effectively never in app code; can happen in hot virtualised-list inner components) |

`babel.config.js`:

```js
module.exports = {
  plugins: [
    ["babel-plugin-react-compiler", {
      // Strictest mode: fail the build on rule violations
      panicThreshold: "all_errors",
      // target: "19" (default)
    }],
    "@babel/plugin-transform-react-jsx",
  ],
};
```

Rules of React enforcement (`eslint-plugin-react-compiler` + `eslint-plugin-react-hooks` v5+):

| Rule | Violation means |
|------|-----------------|
| Components and hooks must be pure | No mutation of captured variables during render |
| Props / state / context are immutable | Don't write to them |
| Setting state during render is only allowed for derived-state pattern | Otherwise a side-effect — move to effect or event |
| Hooks called at the top level | No conditional hook calls |
| Refs mutated only inside effects or event handlers | Never during render |

Compiler output preserves existing `React.memo` / `useMemo` / `useCallback` — safe to leave or progressively remove. The Compiler skips files with `"use no memo"` at the top.

Source: https://react.dev/learn/react-compiler.

## Server Components vs Client Components

RSC is the default in Next.js App Router. A Server Component:

| Dimension | Server Component | Client Component |
|-----------|------------------|------------------|
| Where it runs | Server (once, per request or per build) | Server (SSR) + client (hydration + updates) |
| JS shipped to client | None for its own code | Its code + React runtime + deps |
| Async allowed | `async function Component()` | No (use `use()` hook for promises) |
| State / effects / refs | No | Yes |
| Event handlers | No (`onClick` is a client boundary) | Yes |
| Browser APIs (`window`, `localStorage`) | No | Yes (guard for SSR) |
| Directive | Default (no marker needed in App Router) | `"use client"` at top of file |
| Prop restrictions | Can pass serializable values + Client Components | — |

Decision tree:

```
Does this component need state / effects / browser APIs / event handlers?
├── No → Server Component (default). Ship zero JS.
└── Yes → Client Component. Push the `"use client"` boundary as low as possible.
          Let server components render the heavy static surrounding tree.
```

RSC payload size matters for INP: every byte of RSC payload is parsed before hydration can begin. Prefer plain HTML where interactivity isn't needed. Heavy third-party UI kits imported into an RSC tree still ship if any client descendant re-exports them.

Anti-pattern:

```tsx
// BAD: boundary at the route root forces the whole subtree to the client
"use client";
export default function Page() { /* 50 components */ }
```

Correct:

```tsx
// page.tsx (Server Component)
import { ServerHeavyChart } from "./server-heavy-chart";
import { InteractiveFilters } from "./interactive-filters"; // client component, small

export default async function Page() {
  const data = await db.query.metrics.findMany();
  return (
    <main>
      <h1>Dashboard</h1>
      <ServerHeavyChart data={data} />
      <InteractiveFilters />
    </main>
  );
}
```

Source: https://react.dev/reference/rsc/server-components, https://nextjs.org/docs/app/getting-started/server-and-client-components.

## Suspense boundaries

Place boundaries near uncached data, not at the route root. Each boundary streams independently, so slow data never blocks fast data.

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react";
import { RevenueCard, OrdersCard, ActivityFeed } from "./cards";
import { Skeleton } from "@/ui/skeleton";

export default function Dashboard() {
  return (
    <main>
      <h1>Dashboard</h1>
      {/* Fast — paints immediately with the shell */}
      <QuickStats />

      {/* Three independent streams */}
      <Suspense fallback={<Skeleton lines={3} />}>
        <RevenueCard />
      </Suspense>
      <Suspense fallback={<Skeleton lines={3} />}>
        <OrdersCard />
      </Suspense>
      <Suspense fallback={<Skeleton lines={8} />}>
        <ActivityFeed />
      </Suspense>
    </main>
  );
}
```

Anti-pattern (one giant Suspense at the route root):

```tsx
// BAD: slowest query blocks every card
<Suspense fallback={<FullPageSkeleton />}>
  <RevenueCard />
  <OrdersCard />
  <ActivityFeed />
</Suspense>
```

Source: https://react.dev/reference/react/Suspense.

## `useDeferredValue`

For derived values that don't need urgent updates. Canonical use: filter a 10k-row list as the user types.

```tsx
"use client";
import { useDeferredValue, useMemo, useState } from "react";

export function SearchableList({ items }: { items: Row[] }) {
  const [query, setQuery] = useState("");
  // Urgent: the input re-renders immediately
  // Deferred: the expensive filter runs when React has bandwidth
  const deferredQuery = useDeferredValue(query);

  const filtered = useMemo(
    () => items.filter((row) => row.name.includes(deferredQuery)),
    [items, deferredQuery],
  );

  const isStale = query !== deferredQuery;
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul style={{ opacity: isStale ? 0.6 : 1 }}>
        {filtered.map((row) => <li key={row.id}>{row.name}</li>)}
      </ul>
    </>
  );
}
```

Source: https://react.dev/reference/react/useDeferredValue.

## `useTransition`

For state updates that don't need to block input. Canonical use: tab switch that triggers a data fetch.

```tsx
"use client";
import { useTransition, useState } from "react";

export function Tabs() {
  const [tab, setTab] = useState<"home" | "reports" | "settings">("home");
  const [isPending, startTransition] = useTransition();

  const select = (next: typeof tab) => {
    startTransition(() => {
      setTab(next); // re-render happens in the background; old tab stays visible
    });
  };

  return (
    <div>
      <nav style={{ opacity: isPending ? 0.5 : 1 }}>
        <button onClick={() => select("home")}>Home</button>
        <button onClick={() => select("reports")}>Reports</button>
        <button onClick={() => select("settings")}>Settings</button>
      </nav>
      <TabPanel tab={tab} />
    </div>
  );
}
```

The key difference from `useDeferredValue`: `useTransition` wraps the *setter*, giving you `isPending`. `useDeferredValue` wraps the *value*, giving you a lagging copy.

Source: https://react.dev/reference/react/useTransition.

## `useOptimistic`

For instant feedback on mutations. Canonical use: like button, save indicator, optimistic add-to-cart.

```tsx
"use client";
import { useOptimistic } from "react";
import { likePost } from "./actions";

export function LikeButton({ post }: { post: Post }) {
  const [optimisticCount, addOptimistic] = useOptimistic(
    post.likes,
    (state, delta: number) => state + delta,
  );

  const onClick = async () => {
    addOptimistic(1);          // UI updates to +1 immediately
    await likePost(post.id);   // Server Action confirms — reverted if it throws
  };

  return <button onClick={onClick}>Like {optimisticCount}</button>;
}
```

Automatically reverts when the surrounding transition rejects. Only legal inside an async action or transition.

Source: https://react.dev/reference/react/useOptimistic.

## `use()` hook

Read promises / context conditionally. Replaces the `useMemo(() => promise, [])` + `Suspense` dance.

```tsx
"use client";
import { use } from "react";

export function Product({ productPromise }: { productPromise: Promise<Product> }) {
  // Suspends until the promise resolves; unlike other hooks, `use` may be called conditionally
  const product = use(productPromise);
  return <h1>{product.name}</h1>;
}
```

The parent renders on the server, starts the fetch, passes the unresolved promise into the client boundary. The client `use()` suspends until it resolves — streaming HTML along the way.

Source: https://react.dev/reference/react/use.

## Form actions

Server Actions + `useFormStatus` + `useActionState`. Less client JS, server-validated by default.

```tsx
// actions.ts (server)
"use server";
import { z } from "zod";

const schema = z.object({ email: z.string().email() });

export async function subscribe(_: unknown, formData: FormData) {
  const parsed = schema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { ok: false, error: "Invalid email" };
  await db.subscribers.create({ data: parsed.data });
  return { ok: true, error: null };
}
```

```tsx
// subscribe-form.tsx (client)
"use client";
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { subscribe } from "./actions";

function Submit() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "..." : "Subscribe"}</button>;
}

export function SubscribeForm() {
  const [state, action] = useActionState(subscribe, { ok: false, error: null });
  return (
    <form action={action}>
      <input name="email" type="email" required />
      <Submit />
      {state.error && <p role="alert">{state.error}</p>}
    </form>
  );
}
```

No hand-rolled onSubmit, no fetch, no loading state wiring. Validation runs on the server; the form submits without JS as a baseline and progressively enhances.

Sources: https://react.dev/reference/react-dom/hooks/useFormStatus, https://react.dev/reference/react/useActionState.

## Hydration cost reduction

| Technique | Effect |
|-----------|--------|
| Islands architecture (Astro / Qwik / Next RSC) | Only interactive components hydrate; static content is HTML |
| Defer non-critical components below the fold | `React.lazy()` + `<Suspense>` keeps initial JS small |
| `React.lazy()` for heavy widgets | Charts, editors, maps, date pickers load only when rendered |
| Server-render heavy parts | Shift compute from client to server; client just paints |
| Avoid hydrating pure-static subtrees | Wrap in server components; no `"use client"` descendant |
| Keep client boundaries narrow | Push `"use client"` as deep as possible in the tree |

```tsx
// Dynamic import with SSR fallback
import dynamic from "next/dynamic";
const Editor = dynamic(() => import("./markdown-editor"), {
  loading: () => <EditorSkeleton />,
  ssr: false, // editor is client-only; don't pay for SSR pass
});
```

## Render-cost anti-patterns

| Anti-pattern | Why it hurts | Fix |
|--------------|--------------|-----|
| Inline objects / arrays in JSX props | `{{ foo: 1 }}` recreates every render; breaks memo | Hoist to module scope or `useMemo` (or trust the Compiler) |
| Unmemoised context value | `<Ctx.Provider value={{...}}>` every render invalidates all consumers | `useMemo` the value (Compiler does this automatically) |
| Large prop drilling | Every ancestor re-renders when any leaf changes | Lift to context, extract the changing leaf into its own component |
| List keys based on index | Reuses DOM nodes for wrong items; breaks animations, inputs, refs | Use a stable per-item id |
| Derived state in `useState` | Goes stale when inputs change; drift between state and reality | Derive during render: `const filtered = items.filter(...)` |
| `useEffect` for derived values | Extra render cycle + hydration mismatch risk | Compute inline; no effect needed |
| `useEffect` as event handler | `onClick → setState → effect fires` is always an event handler pattern | Move to the event handler |
| Shared state at the route root | Any change in any leaf re-renders the whole subtree | Colocate state with its consumer |
| Creating regex / date / schema per render | Recompiles on every render | Hoist to module scope |

Source: https://react.dev/learn/you-might-not-need-an-effect.

## Component-level perf checklist (per component)

For the `ui-perf-engineer` agent to walk through on every non-trivial component:

1. Is this component server-only? If yes, no `"use client"` and no client-only hooks.
2. Does every list have stable keys (not index)?
3. Are expensive children wrapped in `Suspense` boundaries near their data, not at the route root?
4. Are context provider values stable across renders (memoised or static)?
5. Is heavy computation derived during render (or memoised), not stored in state + effect?
6. Are async children using `use()` or awaited on the server, not `useEffect` + fetch?
7. Are below-fold components lazy-loaded via `dynamic` / `React.lazy`?
8. Are optimistic updates using `useOptimistic`, not manual reducer + rollback logic?

Run `eslint-plugin-react-compiler` + `eslint-plugin-react-hooks` v5 on every PR. Both catch the most common regressions statically.

## Sources (canonical)

| Topic | URL |
|-------|-----|
| React Compiler | https://react.dev/learn/react-compiler |
| Server Components | https://react.dev/reference/rsc/server-components |
| Suspense | https://react.dev/reference/react/Suspense |
| useDeferredValue | https://react.dev/reference/react/useDeferredValue |
| useTransition | https://react.dev/reference/react/useTransition |
| useOptimistic | https://react.dev/reference/react/useOptimistic |
| use() hook | https://react.dev/reference/react/use |
| useActionState | https://react.dev/reference/react/useActionState |
| useFormStatus | https://react.dev/reference/react-dom/hooks/useFormStatus |
| You Might Not Need an Effect | https://react.dev/learn/you-might-not-need-an-effect |
| Next.js RSC | https://nextjs.org/docs/app/getting-started/server-and-client-components |
| Next.js Streaming / loading.tsx | https://nextjs.org/docs/app/api-reference/file-conventions/loading |
