# JavaScript + Next.js Engineer Quickstart (Code-Reading Edition)

Last updated: 2025-08-21 22:38

Purpose: Give you a fast, practical reference so you can read and understand modern JavaScript and
Next.js codebases quickly. Skim sections as needed.

---

## 1) JavaScript Essentials (Modern JS)

- Runtime model
    - Single-threaded event loop with task queues (macrotasks) and microtasks (
      Promises/MutationObserver).
    - Call stack -> executes sync code; async work schedules callbacks.
    - Microtasks run before rendering and before the next macrotask.

- Types and values
    - Primitive types: string, number (double), bigint, boolean, null, undefined, symbol.
    - Objects: arrays, functions, dates, regex, Map/Set/WeakMap/WeakSet.
    - Equality:
        - === strict equality (no coercion), !==
        - == loose equality (coercion). Avoid in most code.
    - Truthy/falsy: falsy -> 0, -0, 0n, "", null, undefined, NaN, false.

- Variables and scope
    - let/const are block-scoped; var is function-scoped (avoid var).
    - const means binding cannot be reassigned; the object can still be mutated.
    - Hoisting: declarations hoisted; TDZ (temporal dead zone) for let/const.

- Functions and this
    - this depends on call site: obj.method(), standalone call (undefined in strict mode, window in
      sloppy), bound via call/apply/bind.
    - Arrow functions: lexical this, no arguments object, not constructible. Great for callbacks.
    - Closures: functions capture variables from outer scopes.

- Objects, classes, prototypes
    - Prototypal inheritance: obj.__proto__ (conceptually) -> prototype chain.
    - class syntax sugar over prototypes; use extends and super.
    - Destructuring, spread/rest:
        - const {a, b: alias, ...rest} = obj
        - const [x, y, ...others] = arr
        - const merged = {...o1, ...o2}

- Modules
    - ESM (import/export): import x from "./x.js"; export default foo; export const a = 1.
    - CommonJS (Node legacy): const x = require("x"); module.exports = ...
    - Next.js app router prefers ESM.

- Async programming
    - Promises: then/catch/finally. Common combinators: Promise.all, allSettled, any, race.
    - async/await: syntactic sugar over Promises; use try/catch; run parallel tasks with
      Promise.all.
    - Event loop ordering example:
        - queueMicrotask / Promise.then before setTimeout(fn, 0).

- Fetch and HTTP
    - fetch(url, { method, headers, body, cache, next }) returns a Response; await res.json() or
      res.text().
    - Handle errors by checking res.ok; throw with context.

- Arrays and objects
    - Common methods: map/filter/reduce/find/some/every/flat/flatMap/sort (note: sort mutates!).
    - Immutability patterns: return new arrays/objects instead of mutating when using React.

- Iterators and generators
    - for...of iterates over iterable; for...in over enumerable keys of object (not recommended for
      arrays).
    - Generators: function* g() { yield ... } produce iterators.

- Dates, Intl, and i18n
    - Date is mutable; use libraries if needed. Intl.DateTimeFormat, Intl.NumberFormat.

- Errors
    - Throw Error with message and cause: new Error("msg", { cause: original }).
    - Try/catch around awaits that may fail. Centralize error handling where possible.

- Web APIs overview
    - DOM: querySelector, addEventListener, classList, dataset, etc.
    - Storage: localStorage, sessionStorage; beware quota and availability in SSR.
    - URL/URLSearchParams for manipulating query strings.

- Tooling quick notes
    - Package managers: npm, yarn, pnpm. Lockfiles aren’t compatible across tools.
    - Linting/formatting: ESLint, Prettier.
    - Testing: Jest/Vitest, React Testing Library for component tests.
    - TypeScript: superset of JS with types; .ts/.tsx; improves code reading by surfacing intent and
      contracts.

---

## 2) React Primer (for Next.js context)

- Components: function components return JSX.
- State: useState/useReducer for local; Context/Zustand/Redux for global.
- Effects: useEffect for client-side side-effects; runs after render in browser only.
- Memoization: useMemo/useCallback to avoid unnecessary recalculations/props changes.
- Suspense: async boundaries; useful with Next.js streaming and data fetching.

---

## 3) Next.js Fundamentals (v13+ with App Router)

- Key idea: React framework that supports hybrid rendering (SSR/SSG/ISR) with file-based routing and
  server components by default.

- Project structure (app router)
    - app/ is root for routes.
    - Each folder can contain: page.tsx (route), layout.tsx (wrapping UI), loading.tsx (skeleton),
      error.tsx (error boundary), not-found.tsx.
    - Route groups: (group)/ to organize without affecting URL.
    - Dynamic routes: [id]/, catch-all: [...slug]/, optional catch-all: [[...slug]].

- Server vs Client Components
    - Server Components by default in app/. They run on the server, can fetch data directly, and
      don’t include client JS by default.
    - Client Components require 'use client' at the top; needed for stateful hooks (useState,
      useEffect), event handlers, browser-only APIs.
    - You can nest Client inside Server; props must be serializable.

- Data fetching (app router)
    - Use the Web fetch API directly in Server Components. Next.js augments fetch with caching and
      revalidation.
    - Caching and revalidation:
        - Default: request memoization per request and full-route caching for static fetches.
        - Control with fetch(url, { cache: 'no-store' }) for dynamic; or { next: { revalidate:
          60 } } for ISR.
        - Segment-level revalidation via export const revalidate = 60; or
          revalidatePath/revalidateTag in actions.
    - Parallel and sequential fetching: Run Promise.all where possible to reduce TTFB.

- Rendering modes
    - SSR: dynamic rendering per request (no-store, cookies/headers usage, unstable data).
    - SSG: static generation at build time.
    - ISR: static generation with revalidation windows.
    - Streaming: Server Components + Suspense allow sending parts of the page early.

- Mutations and Server Actions
    - Server Actions: async functions exported from Server Components or marked with 'use server'.
      Can mutate DB then revalidatePath/revalidateTag.
    - Alternative: API Routes/Route Handlers and client fetch/POST.

- Routing extras
    - Route Handlers (app/api/*/route.ts): define HTTP verbs (GET, POST, etc.). Return NextResponse.
    - Middleware (middleware.ts): Edge runtime function runs before requests; useful for auth,
      rewrites, locale.
    - Redirects/Rewrites: next.config.js or NextResponse.rewrite/redirect in middleware/route
      handlers.

- Assets and UI utilities
    - next/image for optimized images; next/font for font optimization; next/link for SPA-style
      navigation.
    - Metadata: export const metadata in layout/page for SEO.

- Configuration and environment
    - next.config.js: experimental flags, redirects/rewrites, images, i18n, basePath.
    - Env vars: prefix with NEXT_PUBLIC_ to expose to client; otherwise server-only.

- Deployment
    - Works best on Vercel; also supports Node servers or edge runtimes. Ensure correct adapters if
      self-hosted.

---

## 4) Reading a Next.js Codebase: Checklist

1) Identify router

- app/ present? You’re using App Router (v13+). Otherwise pages/ (legacy Pages Router).

2) Map routes

- For app/: list directories with page.tsx; inspect layouts; note dynamic segments.
- Check (group) folders and parallel routes (@slot).

3) Locate data fetching

- Search for fetch(, getServerSideProps/getStaticProps (pages router), or server actions ('use
  server').
- For fetch: inspect cache/no-store/revalidate options; note tags used for revalidateTag.

4) Determine component type

- 'use client' at top? Then it’s a Client Component. Expect useEffect, DOM use.
- No directive? Likely a Server Component; data fetching is allowed directly.

5) State management and mutations

- Look for context providers, Zustand/Redux stores, or form actions (server actions) and API route
  POST handlers.

6) Auth and middleware

- Open middleware.ts for auth/locale. Check app/api/auth routes or next-auth config if present.

7) Performance and caching

- Ensure expensive fetches are cached or deduped; use parallel fetching. Watch for accidental
  dynamic mode caused by reading cookies/headers in server components.

8) Config and env

- next.config.js for image domains, redirects, experimental flags.
- .env* files; verify NEXT_PUBLIC_ usage for client code.

---

## 5) Common Pitfalls and How to Recognize Them

- JavaScript
    - Confusing == and === leading to unexpected coercions.
    - this inside callbacks; use arrow functions or bind.
    - Mutating arrays/objects when React expects immutability.
    - Async in loops: forEach with async functions doesn’t await; use for...of or Promise.all.

- React/Next.js
    - Missing dependency arrays in useEffect -> rerun loops.
    - Using browser-only APIs in Server Components (will fail at build).
    - Marking everything 'use client' increases bundle size unnecessarily.
    - In app router, accessing cookies/headers makes the route dynamic unintentionally.
    - Misconfigured fetch caching causing stale or overly dynamic pages.

---

## 6) Practical Snippets

- Equality and coercion (avoid == unless intentional)

```js
0 == false; // true (coercion)
0 === false; // false
null == undefined; // true
null === undefined; // false
```

- Event loop ordering

```js
console.log('A');
queueMicrotask(() => console.log('micro'));
Promise.resolve().then(() => console.log('then'));
setTimeout(() => console.log('timeout'), 0);
console.log('B');
// A, B, micro, then, timeout (microtasks before macrotask)
```

- Safe fetch wrapper

```js
export async function safeJson(url, init) {
  const res = await fetch(url, init);
  if (!res.ok) throw new Error(`HTTP ${res.status} ${res.statusText}`);
  return res.json();
}
```

- Next.js route handler (app router)

```ts
// app/api/users/route.ts
import {NextResponse} from 'next/server';

export async function GET() {
  const users = await fetch(process.env.API + '/users', {cache: 'no-store'}).then(r => r.json());
  return NextResponse.json(users);
}

export async function POST(req: Request) {
  const body = await req.json();
  // ... create user
  return NextResponse.json({ok: true}, {status: 201});
}
```

- Server Action example

```ts
// app/users/actions.ts
'use server';
import {revalidatePath} from 'next/cache';

export async function createUser(data: FormData) {
  const name = data.get('name');
  await fetch(process.env.API + '/users', {method: 'POST', body: JSON.stringify({name})});
  revalidatePath('/users');
}
```

- Client and Server component split

```tsx
// app/users/page.tsx (Server Component)
import UsersList from './UsersList';

export default async function Page() {
  const users = await fetch(process.env.API + '/users', {next: {revalidate: 60}}).then(r => r.json());
  return <UsersList users={users}/>;
}

// app/users/UsersList.tsx (Client Component)
'use client';
import {useState} from 'react';

export default function UsersList({users}) {
  const [q, setQ] = useState('');
  const filtered = users.filter(u => u.name.toLowerCase().includes(q.toLowerCase()));
  return (
      <div>
        <input value={q} onChange={e => setQ(e.target.value)} placeholder="Search"/>
        <ul>{filtered.map(u => <li key={u.id}>{u.name}</li>)}</ul>
      </div>
  );
}
```

- Middleware example

```ts
// middleware.ts
import {NextResponse} from 'next/server';
import type {NextRequest} from 'next/server';

export function middleware(req: NextRequest) {
  const {pathname} = req.nextUrl;
  if (pathname.startsWith('/admin')) {
    const auth = req.cookies.get('auth');
    if (!auth) return NextResponse.redirect(new URL('/login', req.url));
  }
  return NextResponse.next();
}

export const config = {matcher: ['/admin/:path*']};
```

- Pages router reference (legacy)

```ts
// pages/users.tsx
export async function getServerSideProps() {
  const users = await fetch(process.env.API + '/users', {cache: 'no-store'}).then(r => r.json());
  return {props: {users}};
}

export default function Users({users}) { /* ... */
}
```

---

## 7) Debugging Tips

- Add console.log judiciously on server (Node) and client (browser devtools). On server components,
  logs appear in server logs.
- Network tab: verify fetch requests, caching headers, and responses.
- React DevTools: component tree, props, state; highlight re-renders.
- Next dev overlays show route segment status (static vs dynamic) and cache revalidation.

---

## 8) Glossary

- SSR: Server-Side Rendering per request.
- SSG: Static Site Generation at build time.
- ISR: Incremental Static Regeneration—rebuilds after a time window.
- RSC: React Server Components—rendered on the server, streamed to client.
- Server Action: Server-side function callable from client via form/actions.

---

## 9) Further Resources

- JavaScript
    - MDN Web Docs: https://developer.mozilla.org/
    - You Don’t Know JS (book series)
- React and Next.js
    - Next.js Docs: https://nextjs.org/docs
    - React Docs: https://react.dev
    - App Router Guide: https://nextjs.org/docs/app

---

If you need a shorter cheatsheet, search within this file for: "Checklist", "Snippets", or "
Pitfalls".
