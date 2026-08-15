---
name: architectural-guardrail
description: "Knowledge base from the video course \"Advanced Claude Code for Web Developers\" by Web Prodigies (@webprodigies, youtube.com/watch?v=GCz83HTg2vI). Use when building production-grade apps with AI coding agents (Claude Code, Codex, Cursor), applying the architectural guardrail / guardrail coding system: pattern recognition, globalization, TypeScript lockin, the block layer, database isolation, inline injection, GrabScript, CLAUDE.md orchestration, the White Line Method, and the 'never let AI use git' rule. Also use when porting or adapting guardrail patterns to an existing codebase or a different stack (Next.js, TanStack Start, Express, plain JS, untyped codebases)."
---

<!-- argument-hint: [topic, framework name, module name, or chapter number] -->

# Advanced Claude Code for Web Developers — The Architectural Guardrail
**Author**: Web Prodigies ([@webprodigies](https://www.youtube.com/@webprodigies)) — source: ["Advanced Claude Code for Web Developers (Full Course)"](https://www.youtube.com/watch?v=GCz83HTg2vI), video transcript | **Length**: ~175 min (344 transcript pages) | **Modules**: 18 | **Generated**: 2026-08-15

## How to Use This Skill

- **Without arguments** — load Core Frameworks below for reference
- **With a topic** — ask about `rate limiting`, `org scoping`, `inline injection`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch10`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

**The Architectural Guardrail** is a layered system that keeps AI writing good code as an app grows. Vibe coding (prompting without structure) is "the absolute worst way to build an application." The system scales because each layer forces AI to imitate proven patterns instead of inventing architecture. Claimed proof: Mochi shipped 1,200 features in 4 months using only AI, with 40–60% shorter prompts and drastically lower token spend. The system is agent-agnostic — works with Claude Code, Codex, Cursor.

**Layer order (the guardrail):**
1. **Mindset** — never let AI use git; branch-per-feature; trust errors over docs. Future-prompt one step higher: prompt toward future reusability so later features are one prompt.
2. **Pattern recognition** — AI never invents architecture; it verifies a pattern exists and builds into it. If you can't map a feature's linear path through existing code, no pattern exists — establish one first. Code drift (AI inventing its own pattern) happens when existing code is inconsistent; one deviating file is noise, majority repetition defines the pattern.
3. **Globalization** — logic needed on server AND client (feature gates, plan checks, resources) is extracted into one pure helper that throws on failure. Key drawer: one known location per concern; AI reuses what it finds by searching.
4. **Registry** — a single resource map (name, description, upgrade message, limit, permissions) as source of truth; nav gating, plan gates, and upgrade prompts all derive from it. AI copies what it finds in the most-used procedure — put the pattern there (chokepoint).
5. **TypeScript lockin** — all types dynamically derived from one source (`keyof typeof` constants, Prisma first); ban `any`/`unknown`; wrong usage errors everywhere (echo signal). Bait-and-switch prompting: "these types already exist — use those first."
6. **Normalizing identity** — provider does its job, adapter does yours; every connection layer mirrors into one shared adapter, so swapping providers is a localized change.
7. **Server-only** — `import "server-only"` in service files; permission logic lives in server-only services, never inside API endpoints (bundled server actions are client-callable by ID).
8. **The block** — the single gate all traffic passes through: one protected procedure (`protectedRoute`) bundling session, plan check, resources, audit, permissions, rate limiting. Audit-by-git-diff: verify the endpoint used the protected route.
9. **Database isolation** — service layer = only DB access (router → protectedProcedure → service → Prisma → DB); Prisma from day one or a provider swap costs a full refactor. ctx-injection: org ID resolved from ctx.session server-side, never from the prompt.
10. **Rate limiting + audit** — one base procedure owns the limiter, each service passes its own limit; public routes chain through it too (DDoS targets public resources). Audit block built once in the protected procedure automatically knows resource, customer, member.
11. **Security gap closure** — pages rendering third-party components (Clerk etc.) never hit the block; add a layout-level check (pathname × nav items × permissions) reusing the resources source of truth. Sidebar hiding is cosmetic, not authorization.
12. **Inline injection** — comments at the top of blocks answering What/Why/How/Where deliver microcontext at the moment AI reads that code. Delete docs folders — AI ignores stale docs; inline injection frees 20–30% of context.
13. **GrabScript (guardrail script)** — given a path, it lists candidate files (file list only, ~zero context), filters by SOT keywords planted in inline comments, then opens only survivors (reverse search order).
14. **CLAUDE.md orchestrator** — the file connecting everything: pattern-recognition rules first ("never invent your own architecture"), globalize-what-you-write, keyword+line headers, token discipline, barrel exports as navigation map, finish-the-feature with screenshot proof. Keep it short (~25–86 lines) — it orchestrates mechanisms, doesn't explain them.

**White Line Method** — when AI would build duplicated infrastructure, open with a false claim ("use the existing source of truth…") that forces AI to search and find the real existing code. The AI's default assumption (write-first vs search-first) decides duplication vs reuse.

**Gap Method** — split intertwined tasks and stub the second loudly ("log to console for now"); loudly-marked gaps get filled, not forgotten.

**Feature prerequisite ordering** — complex features: number sub-features 1-5; the consumable ships before the consumer (booking calendar before canvas); structure → state → mutation.

**Errors-over-context** — the master principle: every rule becomes a compile/lint/type error, because AI fixes errors but ignores docs.

**Token discipline** — the guardrail is built-in token optimization: permissions, gates, and architecture are enforced by code, so prompts shrink 40–60%; hit session limits ON TOP of the guardrail for maximum output (plan $200 → $100 became viable).

---

## Applying to an Existing Codebase

The course code is reconstructed and assumes Next.js + tRPC + Prisma + Upstash + Clerk + TypeScript. **Port the invariant, never the snippet.** For any real repo:

1. Read the layer's `Portable invariant` (in its chapter) or [adaptation.md](adaptation.md)'s invariant table
2. Map each construct to your stack's equivalent via the mapping table ([adaptation.md](adaptation.md) covers Next.js, TanStack Start, and framework-agnostic fallbacks — including untyped JS, existing DB drivers, no linter)
3. Hand-build ONE exemplar in the repo's local style, then let AI replicate it
4. Encode the invariant as an error (type/lint/`throw`), never as a comment — instructions drift, errors don't

If the stack is NOT covered by the mapping table, derive from the invariant row — that's the transferable unit.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-intro-mindset.md) | Intro + Mindset | guardrail coding, future-prompting, anti-vibe-coding |
| [ch02](chapters/ch02-claude-setup.md) | Claude Code Setup | minimum viable guardrail, branch-per-feature, skip-permissions |
| [ch03](chapters/ch03-guardrail-overview.md) | Guardrail at 10,000 Feet | layer system, echo signal, block, microcontext |
| [ch04](chapters/ch04-pattern-recognition.md) | Pattern Recognition | linear path, code drift, errors-over-context |
| [ch05](chapters/ch05-globalization.md) | Globalization | key drawer, resource map, client mirror |
| [ch06](chapters/ch06-registry-layer.md) | Registry Layer | permission-per-resource, chokepoint, AI-as-copyist |
| [ch07](chapters/ch07-typescript-lockin.md) | TypeScript Lockin | derived types, bait-and-switch, ESLint guardrail |
| [ch08](chapters/ch08-normalization-identity.md) | Normalizing Identity | adapter layer, shared adapter, plugin-type extension |
| [ch09](chapters/ch09-server-only.md) | Server-Only | server-only import, bundle-ID exploit defense |
| [ch10](chapters/ch10-block-layer.md) | The Block Layer | block type design, protectedRoute, org scoping |
| [ch11](chapters/ch11-database-isolation.md) | Database Isolation | service layer, Prisma plug, linear path |
| [ch12](chapters/ch12-data-layer-hardening.md) | Server-Side Org Scoping | ctx injection, tenant isolation |
| [ch13](chapters/ch13-rate-limiting-audit.md) | Rate Limiting + Audit Logs | base procedure, customization layer, audit block |
| [ch14](chapters/ch14-security-gap.md) | Security Gap | layout guard, resources reuse, simplest fix |
| [ch15](chapters/ch15-inline-injection.md) | Inline Injection | What/Why/How/Where comments, path-then-guide |
| [ch16](chapters/ch16-guardrail-script.md) | GrabScript | file-first search, SOT keywords, reverse search |
| [ch17](chapters/ch17-claude-md-orchestrator.md) | CLAUDE.md Orchestrator | pattern-first rules, slot architecture, token discipline |
| [ch18](chapters/ch18-usage-white-line.md) | Using the Guardrail + White Line | White Line Method, Gap Method, prerequisite ordering |

## Topic Index

- **Audit logs** → ch10, ch13
- **Base procedure** → ch13
- **Block layer / protectedRoute** → ch03, ch10, ch11, ch14
- **CLAUDE.md** → ch02, ch17
- **Code drift** → ch04
- **ctx / org scoping** → ch12
- **Database isolation / Prisma** → ch03, ch11
- **Echo signal** → ch03, ch04, ch07
- **Gap Method** → ch18
- **Globalization / client mirror** → ch05, ch13
- **GrabScript** → ch16, ch17
- **Inline injection** → ch15, ch16
- **Key drawer** → ch03, ch05
- **Never let AI use git** → ch01, ch02, ch17, ch18
- **Normalizing identity / adapter** → ch08
- **Pattern recognition** → ch04, ch17
- **Rate limiting** → ch13
- **Registry / resources** → ch06, ch14
- **Security gap** → ch14
- **Server-only** → ch09
- **Session limits / token optimization** → ch01, ch18
- **TypeScript lockin** → ch07
- **Vibe coding (anti-pattern)** → ch01, ch04
- **White Line Method** → ch18

## Supporting Files

- [adaptation.md](adaptation.md) — porting guide: stack mapping table (Next.js / TanStack Start / generic) + all 18 layer invariants + 4-step workflow. Read first when applying to an existing codebase
- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — security rules, build-order tree, thresholds, tells

---

## Scope & Limits

This skill covers the course content only — synthesized summaries of the video "Advanced Claude Code for Web Developers (Full Course)" by Web Prodigies ([@webprodigies](https://www.youtube.com/@webprodigies)); it is not the original content. The transcript has speech-to-text garble; obvious misspellings are reconstructed (Claw.md → CLAUDE.md, Superbase → Supabase, Grabcript/GRP script → GrabScript). Names mentioned in the video (Funnomas.ai / Mochi, testimonials) are uncertain. Chapter code blocks are reconstructed structural sketches (marked as such), not literal code — port invariants per [adaptation.md](adaptation.md). For topics beyond this course, check related skills or ask the agent directly.
