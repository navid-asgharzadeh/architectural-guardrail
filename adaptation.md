# Adaptation — Porting the Guardrail to Your Stack

The course's code is reconstructed from a transcript and assumes a specific stack (Next.js, tRPC, Prisma, Upstash, Clerk, ESLint, TypeScript). **Port the invariant, never the snippet.** This file tells you how.

## Porting workflow

1. **Read the invariant** — find the layer in the Invariant Table below (or the chapter's `Portable invariant` line). The invariant is the rule that must hold in any stack.
2. **Locate the equivalent surface** — find where your framework already runs "all requests through one point" (middleware, guards, dependencies, wrappers). Map the construct, not the syntax.
3. **Hand-build ONE exemplar** — write the first instance yourself (one wrapped route, one service file). AI imitates existing patterns; give it a local pattern to imitate.
4. **Encode enforcement** — make the invariant a compiler/lint/runtime error (typed check, linter rule, `throw`), never a doc. Then let AI replicate the exemplar.

## Stack mapping

| Course construct | Next.js | TanStack Start | Framework-agnostic invariant |
|---|---|---|---|
| tRPC `baseProcedure` → `publicRoute`/`protectedRoute` chain | `middleware.ts` for global concerns + a shared route-handler wrapper (`withGuard(handler, {resource})`) | server middleware (`server.ts`) + shared server-function wrapper | One chain point every request passes through; keep 3 variants max (base, public, protected) |
| Upstash rate limiter | `@upstash/ratelimit` or Next middleware + Redis | same Redis-based limiter | Any centralized limiter (Redis). If none: one limiter module as the plug, with the multi-instance gap marked loudly |
| Clerk | Clerk SDK or NextAuth/Auth.js | better-auth or any session adapter | Any auth provider + app-owned adapter layer (provider does its job, adapter does yours) |
| Prisma ("from day one") | Prisma or Drizzle | Drizzle or Kysely | One provider-plug module owns the connection; ONLY service files touch it. Existing driver (e.g. `pg`) stays — wrapping it in one module satisfies the invariant; adopting Prisma in an existing app IS the refactor ch11 warns against |
| `import "server-only"` | native `server-only` package | server functions are server-only by design | Module boundary + runtime guard (`throw` if imported client-side). Without a client bundle this is architectural discipline, not an exploit defense |
| ESLint structural rules (service-file shape) | ESLint flat config | ESLint or Biome | Any linter. None configured? Add one, or fall back to runtime/type guards — the weakest acceptable option; instructions alone drift |
| TypeScript lockin (`keyof typeof`, no `any`) | TS (native) | TS (native) | Typed: derive all types from one source. Untyped (JS): single-source constants + zod schemas + JSDoc typedefs; echo signal becomes a runtime `throw` + lint |
| GrabScript (file-first search) | `rg` + SOT keywords in inline comments | same | Any search tool: list files → filter by SOT keyword → read survivors only |
| CLAUDE.md orchestrator | CLAUDE.md | CLAUDE.md | Any agent instruction file (CLAUDE.md / AGENTS.md), short, pattern rules first |

## Invariant table (all 18 layers)

| Layer | Invariant |
|---|---|
| ch01 Mindset | The guardrail is a system of layers, independent of tooling; direction over code-writing |
| ch02 Setup | One minimal rules file + branch-per-feature; same shape on any agent |
| ch03 Overview | Enforce rules via errors, not context — any compiler/linter/type system can carry the echo signal |
| ch04 Pattern recognition | AI imitates the most-repeated local pattern; repetition teaches, not docs |
| ch05 Globalization | One canonical helper per concern, shared by every layer that needs it |
| ch06 Registry | Source of truth embedded in the most-used code path |
| ch07 TypeScript lockin | Types derived from one source; violation fails the build |
| ch08 Normalizing identity | Adapter layer between providers and app-owned shapes |
| ch09 Server-only | Security-critical code cannot run client-side |
| ch10 Block | One gate all traffic passes through; permissions/audit/limits live there |
| ch11 DB isolation | Only service files touch the database |
| ch12 Org scoping | Tenant identity resolved server-side, never from input or prompt |
| ch13 Rate limiting + audit | Shared limiter middleware with per-service limits; audit built into the shared gate |
| ch14 Security gap | Any surface bypassing the gate gets a check at its closest common ancestor (layout/middleware) |
| ch15 Inline injection | Context delivered at the moment of reading (inline), not as bulk docs |
| ch16 GrabScript | File-list-first search with planted keywords before reading contents |
| ch17 CLAUDE.md | One short instruction file orchestrating mechanisms; never explains them |
| ch18 White Line | False-claim prompt forces search-first; loud stubs; consumable ships before consumer |

## TanStack Start exemplar

```ts
// [structural sketch — port the invariant, not this code]
// withGuard: the block, TanStack Start flavor — max 3 variants (base/public/protected)
export function withGuard<T>(fn: ServerFn<T>, opts: { resource: string; limit: LimitKey }) {
  return createServerFn('POST', async (input: T, ctx) => {
    const ev = getRequestEvent();            // org-scoping: session lives here, never in input
    const session = ev.context.session;
    const limit = getLimit(opts.limit);      // SOT registry (ch06); unknown key throws at startup
    if (import.meta.env.PROD) await rateLimiter.hit(ev.request.headers.get('x-forwarded-for'), limit); // dev exception (ch13)
    audit({ resource: opts.resource, orgId: session.orgId, userId: session.userId }); // audit block built once (ch13)
    return fn(input, { orgId: session.orgId }); // ctx injection (ch12): orgId from server, never from input
  });
}
```
- `server.ts` middleware: use it for global concerns only — the guard lives in the shared wrapper, not scattered hooks.
- Dev exception rationale (tRPC hot reload) is tRPC-specific: port it as "skip limits when not PROD".

## Reconstructed-code convention

Code blocks in the chapters are reconstructed from the transcript — structural sketches, not literal. Chapters mark them with:
`// [reconstructed from transcript — structural sketch; port the invariant, not this code]`
Do not import them verbatim into a different stack.

## Porting checklist

- [ ] Listed the layers this feature touches → read their invariants
- [ ] Mapped each construct to a stack equivalent (table above)
- [ ] Built one exemplar by hand before letting AI replicate
- [ ] Encoded the invariant as an error (type/lint/throw), not a comment
- [ ] Loudly marked gaps (multi-instance limiter, untyped stack fallbacks) — never silently wrong
