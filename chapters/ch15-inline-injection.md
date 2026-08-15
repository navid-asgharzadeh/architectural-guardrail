# Chapter 15: Inline Injection

## Core Idea
Inline Injection is a microcontext strategy: deliver tiny pieces of context to the AI exactly when it needs them — as inline comments on top of code blocks — instead of dumping huge documentation files and folders into the context window before implementation starts.


**Portable invariant**: Context is delivered at the moment of reading, not as bulk docs — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Inline Injection**:
  - When to use: whenever two parts of the codebase (part A and part B) must connect at the context level, and the AI needs to know that connection the moment it reads the code.
  - How: place one inline comment on top of every block of code (preferably at the top of a function/block, not every line). The AI is first given a path so it knows where to look; as it walks that path, inline comments guide it.
- **What/Why/How/Where comment pattern**:
  - When to use: writing any inline injection comment.
  - How: each inline comment answers four questions — *what* the block does, *why* it exists, *how* it works, and *where* it connects to other parts of the app. Answering these lets AI understand how two different concepts connect.
- **Source-of-truth architecture (enforcement)**:
  - When to use: as the standing goal of the entire guardrail.
  - How: when you know where something can or will live in your app, it's easier to route the entire app. The guardrail concepts don't rely on context alone — they rely mainly on linting, TypeScript, and error handling; that is how AI navigates and figures out what's correct.

## Key Concepts
- **Microcontext injection**: delivering minimal context at the moment of need, instead of bulk context upfront.
- **Inline comment**: a comment placed on top of a code block; the whole mechanism of inline injection ("That's it. That's all you need to do.").
- **Context-window consumption**: the cost driver inline injection attacks — it can cut most of your context window usage with a small tweak.
- **Path-then-guide routing**: you give the AI a path to follow; when it starts looking into that path, inline documentation takes over as the guide.
- **Docs folder (for AI)**: external documentation whose full contents must be injected into the context window — the thing inline injection replaces.
- **Client-facing docs**: a docs folder kept for members/community learning, explicitly *not* for documenting architecture to AI; AI does not use it.
- **Block-level placement**: inline comments belong at the top of a function or block — not on every single line (code "looks horrible").
- **Making it easy for AI**: the guardrail's design target; it is not built to make the code nicer for human dev teams.

## Mental Models
- Think of inline comments as **breadcrumbs at each fork in the road** — the AI picks them up at the moment it stands at that code block, not as a map it must load wholesale in advance.
- Use a docs folder for **humans (client-facing)**, inline comments for **AI** — two audiences, two delivery mechanisms; never mix them.
- Think of inline injection as **just-in-time context delivery** vs. docs folders as **just-in-case bulk dumps**.
- The guardrail steers AI primarily through **linters, TypeScript, and error handling**; prose comments are the secondary channel — enforcement beats explanation.

## Anti-patterns
- **Docs folder as AI context**: when the Mochi architecture was stripped out and rebuilt, the outdated docs folder survived all the way to the last release month — and the AI ignored it the whole time, still following the correct architecture. Outdated docs can't mislead AI because AI doesn't care about them; inline comments outperform documentation files. "Just delete it. You don't need it."
- **Inline comment on every single line**: allowed when building with AI but makes the code horrible to look at; prefer the top of the function/block.
- **Treating inline comments as team documentation**: the point is not readability for humans on teams; it's lowering friction for AI. Writing them with human-documentation habits defeats the purpose.

## Code Examples
No code was shown on screen in this module. The pattern described (comment at the top of a block, answering what/why/how/where) reconstructs as:

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// WHAT: cache lookup for user session, shared by auth + billing
// WHY: avoids double-decoding the token on every request
// HOW: LRU cache keyed by session id, TTL 5min
// WHERE: invalidated by session-events.ts -> onSessionRevoke()
function getSession(id: string): Session { ... }
```
- **What it demonstrates**: one compact comment at the top of a block that tells the AI how this piece connects to part B (session invalidation) without loading any docs file.

## Worked Example
The instructor poses the core problem: if part A and part B are supposed to connect at the context level, how can AI know this at the time it's reading that piece of code? The old way: create a docs folder with files that feed context — but that forces the *entire* file's context into the context window. The inline-injection way: when the AI reaches the block where the connection matters, it finds the answer already in the comment above the code. The AI is first given a path so it knows exactly where to look; once it's looking, the inline comments answer what/why/how/where and it knows how the two concepts connect. Result: the small change of moving context into inline comments can "probably cut most of your context window" consumption — with a plausible drop from a $200/month plan to $100/month (not a guarantee, and not a claim that a $5 plan becomes a $200 plan).

## Key Takeaways
1. Put an inline comment on top of every block of code — that's the entire inline-injection technique.
2. Every inline comment answers four questions: what, why, how, where.
3. Prefer the top of a function/block over every-line comments; every-line is permissible for AI but ugly.
4. Inline injection is a just-in-time replacement for docs folders; the AI ignores docs folders anyway (proven by the Mochi stale-docs story).
5. Keep any docs folder you have strictly client-facing (humans learning the system), never AI-facing.
6. The guardrail's real steering force is linting + TypeScript + error handling; context is only the assist.
7. Expect context consumption to drop drastically — likely enough to halve an API subscription tier.

## Connects To
- **Ch 14**: security gap — the previous module, which sets up why this is "one of the most important modules in the entire course."
- **Ch 16**: guardrail script — answers the open question left here: how does AI actually find the files/location to ingest all that inline context ("the GP first module"), via the path you give it.
- **Ch 17**: CLAUDE.md orchestrator — the top-level wiring that hands the AI its starting path so inline injection can take over from there.
