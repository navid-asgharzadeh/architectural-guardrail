# Chapter 7: TypeScript Lockin

## Core Idea
TypeScript exists in an AI-assisted app not to give the developer intellisense but to make the build fail — types are generated from one source of truth so that any AI-written code violating the data structure errors everywhere, and the error is plugged back into the AI as the echo signal.


**Portable invariant**: Types derive from one source; a violation fails the build — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **TypeScript Lockin**: "All TypeScript types should be dynamically generated. You can't hardcode them over and over again. You can't rebuild the types."
  - When to use: every AI-assisted codebase, from day one — the lock only works if the app is built this way from the start.
  - How: (1) derive every type from a single source of truth (Prisma schema, resource-key maps); (2) forbid AI from re-declaring types that already exist; (3) make the generated types strict — no `any`, no `unknown`; (4) run build + lint as part of the loop so violations surface as machine-readable errors.
- **Strict Typing Rule**: a dynamically generated type must be "rock solid" — it cannot be violated in any way, cannot be bypassed with `any`/`unknown`, and must apply consistently across the entire app.
  - When to use: whenever you derive a type (permissions, resources, DB models) — half-strict derivation leaves a hole the AI will find.
  - How: use `keyof typeof` extraction from the canonical constants object; never re-declare keys by hand.
- **Source-of-Truth Types**: "Always use the source-of-truth types that already exist in the app." Prisma is the primary source of truth; you should never create other types unless they are required for something globalized and not already part of the Prisma schema.
  - When to use: any time AI is about to define a new type for something the DB or an existing registry already describes.
  - How: instruct AI to use the Prisma-generated type and expand on top of it if needed, instead of building a parallel user/object type with ids and fields redeclared.
- **Bait-and-Switch Prompting**: tell the AI "types already exist in the app — use those first." The AI assumes the type exists, hunts for it, finds it, and connects to it instead of inventing a duplicate.
  - When to use: whenever you suspect a type probably already exists but you can't remember where; when instructing AI on new features.
  - How: phrase the instruction as an assertion of existence ("use the existing X type"), not a request to build one.
- **Lint-Based Structural Guardrail**: enforce file-structure rules (e.g. first line of a file) via an ESLint config, not via CLAUDE.md prose. When ESLint throws, the error gets plugged into the AI, which fixes it.
  - When to use: for any rule about how files must be structured (e.g. `server-only` at the top of service files).
  - How: add an extended object in the ESLint config whose files field points at the target files (e.g. service files), with a rule checking that the first child (first statement) is the required keyword — roughly analogous to CSS `first-child` selectors.

## Key Concepts
- **Echo signal**: build/lint errors that get fed back to the AI, telling it exactly what it did wrong.
- **Source of truth**: the one canonical definition (Prisma schema, constants object) from which all types are mechanically derived.
- **Dynamic type generation**: extracting types via `keyof typeof` from existing runtime structures instead of hand-writing them.
- **Derived permission structure**: a permission is built as resource → operation → derived permission (e.g. `member_create`), all extracted from the resource key map.
- **Microcontext injection**: mechanism by which the AI gets access to config files (like the ESLint guardrail) while building, so it sees the rule and complies before errors even fire.
- **Red-green prompting**: prompting discipline (detailed in a later module) tied to making the build pass — types that exist are the "green" baseline.
- **Pattern drift**: the app's existing code patterns silently override instructions in CLAUDE.md, because the AI follows what it sees in the codebase.

## Mental Models
- Think of TypeScript as a tripwire, not documentation: the point is that the app "will fail even if AI tries to put that piece of code in here."
- Think of types as the fence: if the fence is derived from one source, adding something outside it (e.g. a `people` permission key) throws an error everywhere.
- Think of the codebase as the real prompt: "no amount of context you give AI will actually work to enforce a rule" — when app patterns contradict instructions, patterns win; fix the patterns (or enforce via lint), not the prose.
- Use the bait-and-switch when you want AI to reuse: assert the type exists and let the AI hunt for it rather than asking it to define one.

## Anti-patterns
- **Rebuilding types per file** (AI's default mistake): creating a fresh user type with ids and names instead of importing the Prisma-generated type — duplicates drift apart and the lockin stops echoing; usually caused by bad pre-existing patterns in the app or by never instructing the AI otherwise.
- **`any` / `unknown` in derived types**: an escape hatch that lets AI bypass the permission structure entirely; the generated types must be rock solid.
- **Enforcing rules in CLAUDE.md only**: putting "always add `use server` on top of the file" in CLAUDE.md worked until the app drifted — a few outdated service files lacked the line, the AI followed the code pattern (not the rule), and ~20–30% of the app was built violating the rule while the author believed it was enforced. Context alone is not enforcement.

## Code Examples
The transcript describes, rather than fully displays, the key definitions. Reconstructed schematically from the description (Next.js + ESLint):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// permission types derived from the resource constants map
type ResourceKey = keyof typeof resourceSources;
// permission = resource → operation → derived permission
// e.g. "member_create": resource, operation, derived permission

// if a new key is added to the source map, everything updates;
// if AI writes a bogus key like "people", the app fails to build
```

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// eslint config: enforce "server-only" as first statement of service files
// an extended object whose files field points at the service files;
// rule checks the first child (first statement) must be the required keyword
```

- **What it demonstrates**: types flow from one source and break loudly; structural rules are enforced by lint config, whose errors get plugged back into the AI.

## Worked Example
**Enforcing `use server` at the top of server-action pages.** The instructor's naive approach: put "always have `use server` on top of the file" in CLAUDE.md. It worked initially, then silently failed — a couple of outdated service pages lacked the line, the AI copied the outdated pattern, and new services were built wrong for 20–30% of the app before it was noticed. Root cause: patterns in the code beat context in CLAUDE.md.

The fix: an ESLint config file with an extended object pointing at the service files, enforcing that the first child of each file is `server-only` (a different, stricter keyword — dedicated module later). Now when AI forgets the line, ESLint throws; the error is plugged into the AI, which adds the line. Combined with microcontext injection, the AI sees the rule while building, so the instructor can "confidently say" every service file will start with `server-only` — not 100% guaranteed, but enforced far beyond what context alone achieved.

## Key Takeaways
1. Generate all types dynamically from a single source of truth — never hardcode, never rebuild.
2. Use Prisma-generated types as the primary source of truth; only create new types when globalized and absent from the Prisma schema.
3. Derived types must be rock solid: no `any`, no `unknown`, no bypasses.
4. Make the build + lint the enforcer: if AI writes a bad key, the app fails — that failure is the echo signal.
5. Prompt reuse as existence: "types already exist in the app, use those first" makes AI hunt for and connect existing types.
6. Never rely on CLAUDE.md context to enforce a rule — the app's existing patterns will silently override it; enforce via lint/errors instead.
7. When a lint rule throws, feed the error back to the AI — it knows exactly what to fix.

## Connects To
- **Ch 6**: registry — resource keys and permission constants live in the registry this chapter derives from.
- **Ch 8**: normalization identity — the next module; types normalize how entities are identified across the app.
- **Red-green prompting module** (later): the prompting discipline for driving builds to green using these locked types.
- **Server-only module** (later): the dedicated keyword the ESLint guardrail enforces on service files.
- **Microcontext injection**: how the AI sees the lint guardrail while building, before errors fire.
