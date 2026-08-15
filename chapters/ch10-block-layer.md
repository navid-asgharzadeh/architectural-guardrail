# Chapter 10: The Block Layer

## Core Idea
The Block is the heart of the architectural guardrail: a single normalized entry point that *all* traffic — from user interaction, direct endpoint calls, or AI vibe coding — must pass through before reaching the backend, with session management, plan/access checks, permissions, audit logs, and rate limiting wired in so enforcement happens by default instead of by instruction.


**Portable invariant**: One gate all traffic passes through; session, plan, audit, permissions, and limits live there — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Block type design (all traffic through one gate)**:
  - When to use: when you need one place where every request — legitimate UI traffic, someone hacking an endpoint directly, or AI-generated code calling the API — is checked before it hits the server.
  - How: place the block between the client and backend; route all traffic through it; wire in session management, plan/access caching, the resource registry (source of truth), audit logs, permission checks, and rate limiting so no path around it exists.

- **Protected-route naming (compress the guardrail into one word AI remembers)**:
  - When to use: when you want AI to apply the block pattern without reasoning about it — an easily referenceable, frequently used word it already knows from the codebase.
  - How: name the block pattern `protectedRoute` (or `protectedProcedure`); use it so frequently across the app that it becomes a pattern; AI then only has to remember one word and everything it builds ends up flowing through the block.

- **TypeScript-as-signal (compiler feedback guides AI mid-build)**:
  - When to use: when AI is filling in a protected route and you want the type system — not the prompt — to tell it exactly what is required to fully build the protected path.
  - How: encode the block's contracts in types so TypeScript signals (errors/completions) tell AI exactly what's needed while it uses the protected route.

## Key Concepts
- **Block**: the gate between client and backend that all traffic passes through before reaching the server.
- **Traffic**: every kind of request — user interaction, direct endpoint calls (hacking), or AI vibe coding against the API.
- **Protected route**: the memorable, frequently used pattern name for the block's entry point, so AI reuses it automatically.
- **Org scoping**: every data fetch must be scoped to the org (needs an org ID), so one org's data can never leak into another.
- **Cross-contamination**: data breach caused by a missing scoping layer — one tenant reads another tenant's customer data.
- **Optimistic approach**: caching (server-side and client-side) plus invalidation, so the UI feels blazing fast and users perceive instant responsiveness.
- **Audit-by-git-diff**: since every endpoint must use the protected route, reviewing AI's git changes reduces to checking that each new API endpoint used it.
- **Source of truth (resources)**: the registry definitions the block consults when deciding access — reused inside the block, not reinvented per feature.

## Mental Models
- Think of **the block as a border checkpoint**: no traffic — user, hacker, or AI — reaches the backend without passing the same inspections.
- Use **name-the-gate** when teaching AI a guardrail: one word (`protectedRoute`) replaces a paragraph of instructions; AI copies patterns, not prose.
- Think of **bugs as solvable but breaches as permanent**: customers forgive a fixable bug; a data leak or org cross-contamination cannot be unsolved after the fact.
- Use **git diff as your audit tool**: if every endpoint must route through the block, a code review is one click — open the diff, see the endpoint, confirm it used the protected route.

## Anti-patterns
- **Any route that bypasses the block**: the moment one endpoint (or AI-generated code) reaches the server without going through the block, permission checks, audit logs, and rate limiting silently vanish for that path — the guardrail is only as strong as its least-observed entrance.
- **Missing org scoping on data fetches**: fetching data without scoping it to the org ID produces cross-contamination — a breach, not a bug; no amount of bug-fixing repairs leaked customer data.
- **Expecting AI to compose the security wiring from memory**: if the pattern isn't a single memorable word used everywhere, AI will invent its own endpoints and skip the checks; compress the guardrail into a name.
- **Treating the block as client-only or server-only**: the block must be reusable on the client too, or you lose the optimistic, super-fast experience that was a design requirement.

## Code Examples
No code is shown in this section of the course — the block's contents are specified as a requirements list (session management, plan caching, resources, audit logs, permission checks, rate limiting) plus the `protectedRoute` pattern name; the implementation arrives in later modules.

## Worked Example
Three orgs store customer data on the same backend. Every feature AI builds goes through the `protectedRoute`. One day AI writes a new endpoint that fetches customer records without org scoping — it omits the org ID filter. Because the endpoint went through the block, rate limiting and audit logs still fired, but the fetch itself was unscoped: one org's user can now read another org's customer data. This is the scenario the block exists to prevent: the guardrail's value isn't bug-free code (bugs are solvable and customers forgive them) — it's making the *unfixable* failures (cross-contamination, breaches) structurally impossible by scoping all data to the org at the block layer.

## Key Takeaways
- Build all the annoying, critical concerns (session, plan/access check, resources, audit log, permissions, rate limiting) into the block once — then every AI-built feature inherits them for free.
- Give the block a single memorable name (`protectedRoute`) and use it everywhere; AI remembers one word, not a security checklist.
- Design the block for reuse on the client as well as the server, enabling optimistic updates with proper invalidation.
- Let TypeScript do the guiding: type contracts make the compiler tell AI what a fully built protected path needs.
- Review AI output like QA, not like a security auditor: check the diff, confirm the endpoint used the protected route, then test the feature — the code is already built to production.
- Distinguish bugs (solvable, forgiven) from breaches (unfixable: cross-contamination, leaked customer data) and design the block to eliminate the second category.

## Connects To
- **Ch 6**: registry — the block consults the resource registry (source of truth) for access decisions.
- **Ch 5**: globalization — the block is the global pattern applied to the entire backend, not one feature.
- **Ch 9**: server-only — the block's protected route is the server-side enforcement point.
- **Ch 11**: database isolation — org scoping at the block layer pairs with database-level isolation to prevent cross-contamination.
