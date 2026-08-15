# Chapter 13: Rate Limiting + Audit Logs

## Core Idea
Rate limiting is done at the block level — one base procedure whose middleware every procedure inherits — with a customization layer on top so different services can have different limits. Audit logs are the same trick: the block already knows resource, customer, and user, so every action is traced by default with zero per-router code.


**Portable invariant**: A shared limiter middleware with per-service limits; audit is built into the shared gate — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Base procedure (block-level rate limiting)**:
  - When to use: any production app where many services need rate limiting but at different levels; any app with public routes that can be hit by DDoS.
  - How: extract the rate-limit checker into its own middleware, put that middleware into a dedicated `baseProcedure` ("the word itself probably tells you what it does"), then chain other procedures (public, protected) onto it. Pass in the required limit as a parameter, get the client IP, and plug it into the limiter. Keep the total number of procedures minimal: "one base procedure that does a bunch of stuff and then one public procedure maybe and then one protected procedure and then chain them together."
- **Customization layer on top of a base**:
  - When to use: when rate limiting must happen in one place ("has to be done at the block level") but "different services might need a higher or lower rate limit."
  - How: the base procedure owns the middleware; each service passes its own limit into the shared checker rather than building its own limiter.
- **Client mirror**:
  - When to use: as the payoff of implementing every guardrail — globalization, pattern recognition, source-of-truth — consistently.
  - How: when your app "mirrors patterns," AI will follow those patterns as much as possible. Combine with globalization (server and client type functionality derived into one globalized function) so patterns get reused over and over, "which enforces pattern even more," and AI uses the same reusable interface wherever it needs it, on server or client.

## Key Concepts
- **Rate limiting at the block level**: rate limits must be enforced in a shared block/procedure, not per endpoint, because that is the only layer AI reliably reuses across the whole app.
- **Base procedure**: a procedure whose middleware does the rate-limit check; every other procedure chains onto it.
- **Customization layer**: the per-service limit value passed into the shared checker, giving different services higher or lower limits on top of one base.
- **Client IP key**: the identity used to rate-limit requests ("you have to get the client IP and then plug that in there").
- **Audit log block**: a block inside the protected procedure that traces every action using resource/customer/member context the block already knows — AI never writes per-router audit code.
- **Client mirror**: your app mirroring patterns so AI follows them; the compounding result of all previous modules.
- **Source-of-truth resources**: the single `resources` object (plans, limits, feature gates, permissions) that the plan-limit gate auto-derives from — one place to check limits, check feature gates, and throw errors.

## Mental Models
- Think of rate limiting as **a toll booth on the base road**: every procedure (public or protected) drives through it, but each lane can have its own toll amount.
- Think of audit logs as **a security camera already wired into the building**: because the block knows resource, customer, and member, recording is automatic — you don't install a camera per room.
- Use the **client mirror** when you want AI to write guardrail-shaped code without being told: the codebase's own patterns become the prompt.
- Use the source-of-truth object when adding a new feature: go to one place (`resources`), type out "how much it has per plan and what are the permissions" — "basic English" — and TypeScript throws if you forget something.

## Anti-patterns
- **Building rate limiting only inside the protected procedure**: everything doesn't have to live there; public resources get hit by DDoS ("people might access my website a million times... not just protected stuff"), so the rate-limit aspect must be extracted and globalized for reuse by both public and protected routes.
- **One limiter per service/route**: fails the customization-layer idea — the base owns the middleware and each service passes its limit in, so AI never has to think when building the next resource.
- **Per-router audit logging**: writing audit-log code for each router makes AI regenerate tracing logic constantly; instead inject the audit block into the protected procedure so every action "is already traced" by default.
- **Proliferating procedures**: chain other base procedures only when needed — "try your best to only have as limited resources as possible" (base + public + protected).
- **Skipping the dev exception for the limiter**: tRPC hot reload on localhost can spin the client bundle "100 times," so you'll constantly hit your own limits in development; add an exception in dev.

## Code Examples
From the protected procedure, the plan-limit gate auto-derived from the source-of-truth resources (point nine in the instructor's walkthrough):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// protected procedure, point nine:
// plan limit gate is autoderived from resources
// destructure limits + feature gates from the source-of-truth object
// throw an error in one place if a limit or gate is violated
```

Base procedure with rate-limit middleware (structure as described):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// baseProcedure — the rate-limit middleware lives here
const baseProcedure = t.procedure.use(rateLimitChecker);

// rate limit checker: takes required limit, passes it in,
// keyed by client IP (Upstash-style rate limit)
// in dev: put an exception here, or hot reload will spin
// the tRPC client bundle ~100 times and trip your own limits
```

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// chain: baseProcedure -> protectedProcedure -> specific router
// audit log block is built inside the protected procedure itself:
// resource + customer + member are already known there,
// so every user action is traced by default
```

- **What it demonstrates**: one middleware owns the check, one procedure owns the audit block, and per-service variation is just a passed-in limit — not new code per route.

## Worked Example
Adding rate limiting to a new public service:

1. In the base procedure's middleware, the rate-limit checker already exists (takes a limit, keyed by client IP).
2. Open the source-of-truth `resources` object, add the new resource, and type out "how much it has per plan and then what are the permissions on it" — TypeScript errors if a field is missing. No reconstruction needed: you can "check the limit of the resource... check the feature gate and... throw errors in from one place."
3. Chain the new public procedure onto the base procedure and pass the limit you want — different services get higher or lower limits by passing different values, not by writing new limiter code.
4. In dev, exempt local requests so hot reload doesn't trip the limit; in production the limit is enforced per client IP.
5. Because the pattern lives in the shared block, a brand-new AI session "will automatically know about the context" and build the next resource the same way.

## Key Takeaways
- Rate limiting must be done at the block level, not per endpoint — that's where AI reliably reuses it.
- Different services need different limits: one base checker plus a passed-in limit is the customization layer.
- Extract and globalize whenever possible: the rate-limit middleware serves both public and protected procedures.
- Public routes need rate limits at least as much as protected ones — that's where DDoS pressure lands.
- Audit logs are free: the block already knows resource, customer, and member, so build the audit block once inside the protected procedure and every action is traced.
- Keep procedure count minimal (base + public + protected), chaining more only when necessary.
- The client mirror is the compounding reward: every guardrail implemented makes AI follow your patterns more, server-side and client-side.

## Connects To
- **Ch 10**: block layer — the protected procedure that owns enforcement
- **Ch 12**: data layer — the source-of-truth `resources` object the plan-limit gate auto-derives from
- **Ch 14**: security gap module — the next module in the check-off
