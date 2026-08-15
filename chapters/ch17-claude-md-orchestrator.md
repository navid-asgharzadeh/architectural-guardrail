# Chapter 17: The CLAUDE.md Orchestrator — One File That Connects the Whole Guardrail

## Core Idea
CLAUDE.md is the orchestrator: a single, deliberately short instruction file (86 lines in the course project, only ~25 in a real codebase like Mochi) that wires together every mechanism built so far — pattern recognition, the guardrail script, layered architecture, TypeScript lock-in, validation, UI conventions, and delivery rules — proving you don't need 500-agent workflows, just a good architecture plus prompting strategy.


**Portable invariant**: One short instruction file orchestrates mechanisms; it never explains them — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **The Orchestrator File (CLAUDE.md as connector)**
  - Formulation: "the orchestrator, the main thing that connects all of the concepts we've just learned."
  - When to use: the single instruction file for any guardrail-coded project.
  - How: create a dedicated folder containing CLAUDE.md, plus `settings/`, `skills/`, and `agents/` subfolders for future growth (skills and agents aren't required — they're "awesome actually" and optional). Keep it ~25–86 lines. This file is *proof of concept* that you don't need a unique, crazy workflow — only a good architecture and the prompting strategies.

- **The Pattern Recognition System section (the "never invent" rule)**
  - Formulation: "Everything you're building in this app is from a pattern recognition model. So you should never invent your own architecture."
  - When to use: as the FIRST section of CLAUDE.md — it establishes the operating constraint before anything else.
  - How: always verify the pattern already exists in the codebase or don't use it; no pulling in your own libraries; no starting a new pattern without confirming it exists; the app runs on a source-of-truth global architecture that is "already established in the app."

- **The Repetition Thesis**
  - Formulation: repeat the same core concepts across sections because it "sticks with AI more."
  - When to use: whenever a rule is load-bearing — say it multiple times in different sections rather than once.
  - How: the instructor admits this is a theory, "not rock solid," just a thesis — but he practices it by restating pattern/globalization/GRP rules in every relevant section.

- **The GRP-First Rule**
  - Formulation: "Before creating any type, function, constant, or component, GRP for it first" — using the guardrail script's keyword lookup, never raw file search.
  - When to use: at the start of every creation task; written into CLAUDE.md so it's enforced by instruction, not habit.
  - How: GRP (guardrail script) with keyword+line headers; raw search (instructor cites "Rn") dumps entire file contents into the context window — "horrible."

## Key Concepts
- **Pattern recognition model**: everything built in the app comes from existing patterns; AI must never invent architecture, pull in unestablished libraries, or start a new pattern before confirming one exists.
- **Build into it**: "your job is to build into it" — never create your own stuff; the architecture is a source of truth you plug into.
- **Globalize what you write**: nothing tightly coupled; the app stays modular so future features are plug-in rather than rewrite — "swapping text stacks etc. should all be easy."
- **Don't create blindly**: only build something new when existing architecture can support the new feature.
- **Keyword + line header (source-of-truth line)**: every file carries a keyword and line reference at the top; keywords are the fastest path to reach that piece of code — this is what keeps context pollution at zero and stops session-limit burnout.
- **Token discipline instruction**: you can literally tell AI "your goal is to prioritize session limits as much as possible and lower token consumption as much as possible" — it will comply to some degree.
- **Slot architecture**: Figma-style slots in a component — a section of the component is a slot; you pass a component up as a prop from the child into the parent so one global header serves many pages with custom functionality.
- **Global folder convention**: global components live in a `global` folder, route-only components live in the route's `_components` folder (standard Next.js practice) — "global component inside global folder means global component," and AI understands that better too.
- **Token-based styling**: never hardcode values like `text-white`; use the tokens Tailwind already ships with, so changing `global.css` propagates everywhere and light/dark mode keeps working.
- **Barrel exports for AI**: index files are intentional — "when AI can find an index file, it will find everything else from there" — a readability tradeoff made for AI navigation, not human reading.
- **Finish-the-feature rule**: never skip a feature or leave it incomplete; AI tends to leave a small complex piece unfinished and ask for next steps — the instruction blocks that.

## Mental Models
- **Think of CLAUDE.md as a conductor, not documentation** — each section doesn't explain how to build; it points at a mechanism that already exists in the codebase (GRP script, protected procedure, types folder) and states the rule for when to use it.
- **Use "can I globalize this?" as the default question for every new artifact** — components, functions, and types all get the same test; if the answer is even partially yes, globalize.
- **Think of an index/barrel file as a map you leave for AI** — humans find full export dumps overwhelming, but AI connects the dots from one index file; optimize the codebase for the reader that does most of the work.
- **Think of a copied pattern as an alarm, not a convenience** — "if you notice yourself copying a pattern of another component, that's a signal to globalize it instead."

## Anti-patterns
- **Inventing architecture**: the cardinal sin — pattern must exist first or the code must not be written; new libraries and unconfirmed new patterns are banned outright.
- **Raw file search instead of GRP**: searching the codebase with a plain search tool pulls full file context for every match into the context window, burning session limits — the exact pollution the keyword+line system exists to prevent.
- **Inline types**: creating a type in the same file as the code that uses it (common in production teams, but the instructor hates it); all types belong in one `libs`/types file because "it makes AI's life easier" — and the CLAUDE.md explicitly points AI at the types folder.
- **`any`, `unknown`, and TS bypass commands**: never use them; never hardcode types — "rock solid" TypeScript only, with Prisma types as the source of all truth.
- **Hardcoded style values**: `text-white` and friends break light/dark mode and decouple the UI from `global.css`; tokens are the only allowed vocabulary.
- **Leaving a feature incomplete**: AI will sometimes declare a piece "future work," leave a complex sliver unfinished, and ask for next steps; must be explicitly forbidden.
- **Undocumented framework renames**: Next.js renaming `middleware` to `proxy` was "the worst change for AI" because training data can't distinguish the two files — if your framework has such a rename, document it in CLAUDE.md to avoid ambiguity.

## Code Examples
Reconstructed structure of the 86-line CLAUDE.md, section by section (from the walkthrough; wording condensed but faithful to what was described):

```markdown
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
# CLAUDE.md

## Pattern Recognition System
- Everything you're building in this app is from a pattern recognition model.
  You should never invent your own architecture.
- Verify the pattern already exists in the codebase, or don't use it.
  No pulling in your own libraries. No new patterns without confirmation.
- The app runs on a source-of-truth global architecture
  (already established in the app).
- Your job is to build INTO it. Never create your own stuff.
- Globalize what you write — the app stays modular; future features are
  plug-in rather than rewrite. Nothing should be tightly coupled.
  Swapping text stacks should be easy.
- Don't create blindly. Only when existing architecture can support the
  new feature should you build something new.

## GRP Script
- The whole codebase has a source-of-truth keyword + line at the top of
  each file. These keywords are the fastest way to reach that piece of code.
  This keeps context pollution at zero and stops session-limit burnout.
- Goal: prioritize session limits as much as possible and lower token
  consumption as much as possible.
- Before creating any type, function, constant, or component:
  GRP for it first. Do NOT use raw search — it dumps whole-file context
  into the window. That is horrible.
- When creating something new, add the keywords.
- Don't create duplicate functions. "I have already created all types.
  If you search for them, you will most likely find them."

## Layered Architecture
- Every important router endpoint needs to use this.
- Routers consume the protected procedure and hold business logic,
  validation, orchestration decisions.
- The service layer is the only thing that touches the database directly.
- Source-of-truth line goes at the top of each file.
- Remember to wire permissions — the permission system is part of the
  architectural guardrail.
- Note: Next.js renamed middleware to proxy. [Document which file is which,
  since AI lacks training data to tell them apart.]

## TypeScript
- Never use `any` or `unknown`. Never hardcode types.
  No TypeScript bypass commands.
- Dynamic TypeScript is the only types you can use. Prisma types are the
  source of all truth — put types in the types folder, not inline in files.
- Run a TypeScript check every single time before handing over.

## Validation
- Zod validates all inputs. (Zod is preferred because it throws errors.)

## Inline Context Injection
- [The inline-injection rules repeated here, plus the comment structure]

## UI
- Globalize components when possible. Check whether a reusable one exists
  first and use it if so.
- Never duplicate a component with similar functionality. Extend or
  compose what's there.
- If you notice yourself copying a pattern of another component, that's a
  signal to globalize it instead.
- Global components go in the global folder. Route-only components go in
  the route's _components folder (Next.js practice).
- Design every component for reuse — no hardcoded logic.
- Follow the design themes already in the app.
- Never hardcode values (e.g. `text-white`). Use the Tailwind tokens.
  Changing global.css then propagates; hardcoded values break dark mode.

## Delivery
- Barrel exports / index files are intentional: when AI finds an index
  file, it finds everything else from there.
- Never skip a feature or leave it incomplete. Finish start to finish,
  and send a screenshot that it works (e.g. via Playwright MCP).
- [Most important command — the guardrail invocation]
```

- **What it demonstrates**: the whole guardrail system compresses into ~25–86 lines of *rules and pointers* — each section tells AI which mechanism to use and what to never do, with zero explanation of how the mechanism works internally.

## Worked Example
The **Mochi header slot component** — how the UI globalization rule paid off:

1. The instructor noticed the same header component used across Mochi, looking "very similar" but with differences: sometimes no buttons, sometimes different sizes.
2. Rather than duplicating the header per page, the AI (with a nudge — "I had to tell it, give it some guidance as to how to create the slot") built a **slot-type architecture**: a section of the header block is a slot, filled by passing a component up from the child as a prop into the parent header.
3. Result: custom functionality in the navbar — the dashboard got an "AI generated dashboard" with a button to tweak/switch dates, all editable directly from the header, which "already comes built in with responsive design, light mode, dark mode, whatever you need."
4. Why it happened: AI built the component because it "saw it using it more often" — the CLAUDE.md rule ("copying a pattern is a signal to globalize") plus the recurring header pattern combined into one reusable, slot-based component.

## Key Takeaways
1. Keep CLAUDE.md short — 86 lines is already generous; a real codebase ran on ~25. It's proof you don't need 500-agent workflows.
2. Open with the pattern recognition rule: never invent architecture, never import unestablished libraries, verify patterns exist first.
3. Write the GRP-first rule explicitly — "GRP for it first, never raw search" — and state the token/session-limit goal in plain words.
4. Ban `any`/`unknown`/inline types; point AI at the single types folder (Prisma as source of truth) and require a TypeScript check at every handover.
5. Never hardcode style values; tokens only, so `global.css` stays the single design lever and dark mode survives.
6. Use barrel exports deliberately — they exist to let AI navigate from one index file, not for human readability.
7. Forbid incomplete features: finish start-to-finish and prove it with a screenshot (e.g., Playwright MCP).
8. Document framework renames (middleware→proxy) that AI's training data can't resolve on its own.
9. Set up the folder once (CLAUDE.md + settings/ + skills/ + agents/) so future growth has a home.

## Connects To
- **Ch 16**: the guardrail script — CLAUDE.md's GRP section encodes the keyword+line lookup and the "GRP first, never raw search" rule.
- **Ch 15**: inline context injection — gets its own repetitive section inside the orchestrator, along with the comment structure.
- **Ch 4**: pattern recognition — the first CLAUDE.md section is the module's thesis restated as an instruction ("never invent your own architecture").
- **Ch 7**: TypeScript lock-in — the orchestrator's TypeScript section encodes the `any`/`unknown` ban and Prisma-types-as-truth rule.
- **Ch 18**: usage recap — the "most important command" this chapter ends on is exercised in the final recap.
