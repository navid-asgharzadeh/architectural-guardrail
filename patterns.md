# Patterns — Architectural Guardrail

## Minimum Viable Guardrail
**When to use**: day one of a new AI-built project.
**How**: one CLAUDE.md with one rule (e.g. "never let AI use git"). Beats skills, subagents, and elaborate setups until trust is earned.
**Trade-offs**: less protection early; the guardrail grows layer by layer as the app does.

## Branch-per-feature
**When to use**: before every feature or iteration.
**How**: `git checkout -b` from a known-good checkpoint; main stays pristine; failed attempts are discarded branches (V5.1, V5.2).
**Trade-offs**: merge chore; cheap insurance against AI undoing finished work.

## Errors-over-context ("no context, error only")
**When to use**: every time you want AI to follow a rule.
**How**: make the rule a compile/lint/type error (ESLint, derived types, `server-only`), not a doc. AI fixes errors; it ignores docs.
**Trade-offs**: requires build-system work up front; pays off as app grows.

## Pattern-first (linear path)
**When to use**: before AI builds a feature with no existing precedent.
**How**: establish the pattern manually first (one protectedProcedure, one service shape), then let AI imitate. Test: can you map the linear path from prompt to DB?
**Trade-offs**: one-time handcrafting cost; AI then clones it reliably.

## Key drawer
**When to use**: deciding where new reusable code lives.
**How**: one known location per concern (one spot for keys, one for resource defs, one for helpers). AI reuses what it finds by searching existing patterns.
**Trade-offs**: requires discipline not to scatter; nothing else.

## Globalization + client mirror
**When to use**: logic needed on server AND client (feature gates, plan checks, resource helpers).
**How**: extract into one pure function of parameters that throws on failure; wrapper varies per side, the rule never does. Edit one resource entry, whole app changes.
**Trade-offs**: premature globalization of single-use logic adds indirection.

## Registry + permission-per-resource
**When to use**: resources with differing access needs.
**How**: single resource map (name, description, upgrade message, limit, permissions); flip permissions per resource per role/account; nav gating and upgrade prompts derive from the same map.
**Trade-offs**: resources with identical policies may not need per-resource config.

## TypeScript lockin + bait-and-switch prompting
**When to use**: keeping types in sync across a growing app.
**How**: derive all types dynamically (`keyof typeof` constants, Prisma first); prompt "these types already exist — use those first"; ban `any`/`unknown`. Wrong usage = build error = echo signal.
**Trade-offs**: dynamic types are slightly harder to read than literals.

## ESLint-as-guardrail
**When to use**: rules that must survive code drift (service-file structure, use-server enforcement).
**How**: encode the structural rule in ESLint (e.g. first-child check on service files) — app patterns override CLAUDE.md prose, but not lint errors.
**Trade-offs**: lint config drift is possible; still stronger than instructions.

## Adapter layer + shared adapter
**When to use**: third-party auth/membership providers with limits or swap risk.
**How**: provider does its job, adapter does yours (roles, limits); every connection layer mirrors into one shared adapter; swap = new connection layer, not a rewrite.
**Trade-offs**: indirection; skip when provider is never swapped and limits don't bite.

## server-only + service file
**When to use**: any code that must never run client-side.
**How**: `import "server-only"` at top of the service file; permission logic lives in server-only services, not inside API endpoints (bundled server actions are client-callable by ID).
**Trade-offs**: none for security-critical code.

## Block type design + protectedRoute
**When to use**: designing the security gate.
**How**: one memorable name (protectedRoute) wrapping session, plan check, resources, audit, permissions, rate limiting; every endpoint routes through it. Audit-by-git-diff: check the endpoint used the protected route.
**Trade-offs**: uniformity over per-endpoint specificity.

## Database isolation via Prisma
**When to use**: from day one, before the first DB call.
**How**: service layer = only DB access; router → protectedProcedure → service → Prisma → DB. Provider SDK directly in app code is a solder joint; Prisma + service is a plug.
**Trade-offs**: ORM overhead; provider swap then costs ~zero code.

## ctx-injection (org scoping)
**When to use**: every endpoint an agent might call.
**How**: attach ctx carrying the session; resolve org ID from ctx.session server-side, never from the prompt (prompt-supplied values are attacker-controllable).
**Trade-offs**: none — missing scope is a breach, not a bug.

## Base procedure middleware (rate limiting)
**When to use**: app-wide rate limits, including public routes (DDoS targets public resources).
**How**: one base procedure owns the limiter; each service passes its own limit into the shared checker; protected procedures chain on top.
**Trade-offs**: dev-mode hot reload can spin the limiter locally (~100x bundle) — exempt localhost.

## Layout-level guard check
**When to use**: pages rendering third-party components (Clerk etc.) that never hit the block layer.
**How**: layout.tsx check: pathname × nav items × permissions against the resources source of truth; sidebar hiding is cosmetic, not authorization.
**Trade-offs**: only closes page-level bypass; per-data checks still need the block.

## Inline injection (What/Why/How/Where)
**When to use**: two parts of the codebase that connect at the context level.
**How**: comment at the top of the block answering what/why/how/where, naming the connected file; delete docs folders (AI ignores stale docs; keeps 20–30% of context free).
**Trade-offs**: comments on every line make code unreadable — block-level only.

## GrabScript file-first search
**When to use**: AI must locate a concept but location is unknown.
**How**: script lists candidate files (file list only, ~zero context) → filter by SOT keyword → open only survivors → search deeper at the best fit (reverse search order).
**Trade-offs**: needs SOT keywords planted in inline comments first.

## Orchestrator CLAUDE.md
**When to use**: the file connecting the whole guardrail.
**How**: pattern-recognition rules first ("never invent architecture — verify the pattern exists, then build into it"); globalize-what-you-write; keyword+line headers; token discipline; barrel exports as navigation map; finish-the-feature with screenshot proof. Keep it short (~25-86 lines).
**Trade-offs**: too long = ignored; it orchestrates mechanisms, doesn't explain them.

## White Line Method
**When to use**: AI is about to build duplicated infrastructure.
**How**: open with a false claim ("use the existing source of truth…") that forces AI to search; the search finds the real existing code. Stack with Gap Method: "once done, connect checkout into invoice."
**Trade-offs**: works because default assumption decides write-first vs search-first; use sparingly to keep trust.

## Feature prerequisite ordering
**When to use**: complex features (canvas builder, multi-step flows).
**How**: number sub-features 1-5; the consumable ships before the consumer (booking calendar before canvas); pseudo-code prompt: speak ideas, ask AI for a blueprint, then correct it.
**Trade-offs**: upfront thinking; prevents AI building the roof first.
