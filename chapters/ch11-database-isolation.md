# Chapter 11: Database Isolation

## Core Idea
Wrap all database access behind a single service layer (backed by Prisma) so business logic never touches the DB provider directly — swapping Supabase for Neon or any Postgres provider then costs roughly zero code instead of a full-app refactor.


**Portable invariant**: Only service files touch the database — port this rule to any stack (see adaptation.md)
## Frameworks Introduced

- **Database Isolation Module**: the only file(s) that may connect to the database is a *service* layer; all business logic lives above it and calls the service.
  - When to use: from day one, before you write your first query — retrofitting it later is a refactor (the instructor lived this).
  - How: pick an ORM/provider that sits between your code and any concrete database (Prisma), then route every read/write through a service file marked server-only.

- **The Linear Path (router → procedure → service → Prisma → database)**: every request travels one fixed route: the router layer gets hit first, the tRPC procedure is the block in between, and the service is the one that contacts the database with Prisma's help.
  - When to use: when structuring any new feature or module.
  - How: map out the linear path before building. If you can see the pattern, AI will most likely follow it too.

- **tRPC as the "Secret Sauce"**: tRPC ties the layers together and auto-generates end-to-end TypeScript, so AI is *told* what to do by the compiler instead of being *told* by prompts.
  - When to use: any time the guardrail needs type-safe enforcement of the path.
  - How: define routers, chain procedures, and let inferred types force AI to pass the required inputs (permission gates, resources, zod schemas).

## Key Concepts

- **Service layer**: the only place in the codebase that connects to the database; it imports `server-only` and contains nothing but DB access.
- **Router layer**: holds the business logic — where AI performs domain operations (e.g., "how many pages can render at once").
- **tRPC procedure**: a middleware-like block between router and service; the "heart of everything" — chaining a `protectedProcedure` bakes auth and gates into every endpoint.
- **protectedProcedure**: a pre-built procedure variant that chains org scoping, onboarding gate, membership check, role gate, permission gate, and more; seeing it on an endpoint is a quick confidence check that it's protected.
- **ctx object**: server-side cache of request-scoped data (organization, active organization, user permissions) available to every procedure.
- **Prisma**: ORM layer that decouples code from provider, enabling Supabase, Neon, or "pretty much most Postgres databases" — the instructor prefers Neon (what Mochi runs on).
- **Zod schema input**: procedure inputs are validated against a zod schema so any consumer (human or AI) always passes correct data.

## Mental Models

- Think of the DB provider as a **cable you can unplug**: provider-specific SDKs wired directly into your app are soldered joints; Prisma + a service layer is a standard plug.
- Think of the path as a **plug-in architecture**: routers import routers like plugging in cables — the same strategy the instructor follows in the lexical editor.
- Use "**protected procedure in front = protected**" as your scan heuristic: if AI built a feature and the endpoint starts with `protectedProcedure`, you're confident to some degree it's gated.
- Think of TypeScript errors as **guardrails, not annoyances**: errors force AI to follow the architecture (pass permissions, resources) without you writing it in the prompt every time.

## Anti-patterns

- **Using the provider SDK in the app root**: people create a Supabase instance and call `supabase.getData()` throughout the app — the moment the database is slow or unpleasant, "you've kind of screwed yourself over." Normalizing identity (isolating the provider) would have solved it.
- **Prompting every cross-cutting concern**: without a procedure that chains rate limiting, plan gates, audit logs, usage counting, etc., you'd have to tell AI to implement each one individually in every prompt — and AI has to remember them all.
- **Skipping isolation until you "need" it**: the instructor skipped it in Mochi and changing the database "became hell" — he had to refactor all the code; he was lucky it happened early when stripping it out was easier.

## Code Examples

Service layer (server-only, DB access only):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
import "server-only";

export async function listAutomations(organizationId: string) {
  return prisma.automation.findMany({ where: { organizationId } });
}
```

Router layer (business logic lives here):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
export const automationRouter = router({
  list: organizationProcedure
    .input(listAutomationsSchema)
    .query(async ({ ctx, input }) => {
      // business logic on top of the query
      return listAutomations(input.organizationId);
    }),
});
```

Procedure chaining (what `protectedProcedure` bakes in, as commented steps):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// request setup        — captures start, sets up ctx object
// org scoping          — onboarding gate, membership check,
//                        role gate, permission gate, path resolution
// per-resource rate limiting
// plan limit gates
// feature flag gate
// pagination
// context enrichment
// usage counter        — increments/decrements customer usage
// exception handlers   — errors, success, log request (audit log)
```

- **What it demonstrates**: business logic is separated from database logic; every endpoint inherits the full guardrail stack by chaining one procedure.

## Worked Example

The instructor's own failure: while building Mochi, he used a database without putting the full database isolation module in place. When he wanted to change it, it became hell — a full refactor of every place the code touched the DB. He got lucky: it happened early in the build, so stripping it out was manageable. Had isolation existed from day one, the same change would have cost "one line of code, or maybe no code at all."

The fix he now ships: Prisma bridges all database access (Supabase, Neon, most Postgres — Mochi itself is hosted on Neon). Then in the guardrail's codebase, the path is explicit: `routers/index` imports and connects all routers (plug-in architecture), each router (e.g., `memberRouter`) lists its methods — `list`, `update`, `delete` — all using the `protectedProcedure`. Business logic (e.g., a website-builder rule: only three members may collaborate on a page in real time, otherwise reject) sits in the router. The service behind it imports `server-only` and only accesses the database, taking `organizationId` from the validated input. Because input schemas are zod-validated, the contract is identical no matter who consumes it — even AI on a different page gets the exact same types.

## Key Takeaways

1. Build database isolation on day one — retrofitting after provider SDKs are woven through the app is a refactor, not a config change.
2. Use Prisma to normalize the provider: you can switch among Supabase, Neon, or most Postgres databases at near-zero cost.
3. Enforce the linear path: router (business logic) → procedure (middleware/guardrails) → service (only DB access) → Prisma → database.
4. Make the service the single choke point that connects to the database, and mark it `server-only`.
5. Chain cross-cutting concerns (auth gates, rate limits, plan gates, feature flags, usage counting, audit logs) into the `protectedProcedure` so neither you nor AI has to re-implement them per feature.
6. Validate all procedure inputs with zod schemas so every consumer gets identical, correct contracts.
7. Reuse logic needed on both server and client via a shared helper (the globalization strategy), optionally with a cache layer in between.

## Connects To

- **Ch 10**: block layer — the linear path starts with the blocks AI sees; this chapter shows what happens after the block (procedure/service/DB).
- **Ch 12**: the next module — data layer hardening (and the quiz question teased at the end: how Mochi AI knows the org ID to pass to these endpoints).
