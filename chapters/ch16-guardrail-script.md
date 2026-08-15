# Chapter 16: The GrabScript Module — Pointing AI at the Right Files

## Core Idea
The GrabScript (called "GP script" / "Grabcript" in the transcript) is a script you run to give the AI agent a shortlist of candidate files worth reading — a path to follow — so it narrows down by files *first* and only ingests file contents once it has a target, instead of dumping scattered files into the context window.


**Portable invariant**: File-list-first search with planted keywords, before reading any contents — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **GrabScript workflow (file-first search)**: give the script a path → it returns a list of files worth looking into → you filter that list further (e.g. by SOT keyword) → open only the surviving file(s).
  - When to use: at the start of any task where the agent must locate where a concept lives (source of truth, a pattern, a domain).
  - How: run the script with the path you want it to follow; from the file list it emits, apply a keyword filter; read only the few files that remain.
- **SOT (source-of-truth) keywords**: a "Source of truth keywords" section written at the top of the inline documentation of relevant files, listing searchable keywords — some pointing directly at the location where the source of truth exists.
  - When to use: for every architectural concept (e.g. permissions) so the agent can find it by search, not by reading.
  - How: write as many keywords as needed — "not too many but just enough" — including ones that name the exact source-of-truth location; the agent searches these words and finds the concept instantly.
- **Reverse search order**: invert the agent's default behavior. Default = search for a concept, then dump every scattered file's contents into context. Guardrail = first narrow down by files, then search deeper based on where the agent thinks the best fit is.
  - When to use: whenever a concept search would otherwise flood the context window.
  - How: enforce via CLAUDE.md — instruct the agent to run the GrabScript as the first step, before reading any file contents.

## Key Concepts
- **GrabScript**: a script that returns a list of files worth looking into; enforced as the first step via CLAUDE.md; ingests only the file list (tiny context cost) rather than file contents.
- **Path to follow**: the input you give the script when it starts, so it knows exactly where to look.
- **SOT keyword**: a searchable term embedded in inline docs that lets AI find a source of truth instantly without reading context.
- **Linear pattern**: how AI reads an unknown codebase — starts from folder structure, walks app folders, notices tRPC, gradually navigates its way. Slow, context-hungry, and uninformed.
- **AI blindness**: AI has no idea what you're showing or building until it reads files; it must be pointed in the right direction with zero context consumption until context is actually needed.
- **Context window flooding**: the failure the script prevents — scattered documentation consuming ~20–30% of the window before any real work starts.
- **What/Why/How/Where inline comments**: the documentation pattern (from the previous module) that answers these four questions at the top of each function or block — not every line.

## Mental Models
- Think of the GrabScript as a **librarian handing over a catalog card, not the book** — the agent sees titles first, then requests only the volume it needs.
- Think of AI as **blind** — you must point, not describe; pointing costs zero context, describing floods it.
- Use the **file list when the concept is unknown**; use SOT keyword search when the concept is known but its location isn't.
- Treat the context window as a **budget** — the goal is zero consumption until the agent actually needs to read the context.

## Anti-patterns
- **Docs folder as primary navigation**: the instructor's Mochi codebase kept an outdated docs folder until the final release month — AI ignored it and still followed the correct implementation, proving inline comments outperform doc files. Delete the docs folder; it's member-facing, not AI-facing.
- **Scatter-search**: when AI lacks knowledge of a concept it searches all scattered files and dumps their contents into the context window to read. This is the exact behavior the GrabScript inverts.
- **Commenting every line**: inline What/Why/How/Where on every line makes code "look horrible" — keep comments at the top of a function or block.
- **Hand-writing SOT keywords**: they're generated as part of the CLAUDE.md module, not maintained by hand (they'd rot like the docs folder).

## Code Examples
Reconstructed from the terminal demo (the transcript shows the flow; exact script syntax is not printed verbatim):

```bash
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
# Step 1 — GrabScript: list candidate files, contents stay unread
./grabscript.sh --path src
# -> a list of files worth looking into is injected into context (file list only)

# Step 2 — filter the list by SOT keyword
grep -rl "SOT:permissions" src
# -> prisma/... , permissions/index.ts  (four files total)

# Step 3 — open the surviving file: you're at the source of truth
```

- **What it demonstrates**: list-first → filter → single-file deep read; only the final file's contents enter the context window.

## Worked Example
Finding the source-of-truth architecture for **permissions**:

1. **Default (bad)**: AI searches for "permissions" across the repo, finds candidate files scattered everywhere, and dumps all their contents into the context window to figure out which one matters.
2. **Guardrail (good)**: CLAUDE.md tells the agent to run the GrabScript first. It gets a path (the app directory) and emits a list of files worth looking into — that list alone is all that's injected.
3. The agent filters the list by the SOT keyword for permissions, narrowing to four files — e.g. in `prisma` and `permissions/index.ts`.
4. The agent clicks into `permissions/index.ts` and is standing directly on the source-of-truth architecture. Context consumed: one file list plus one file's contents — versus dozens of files in the naive flow.

Result claimed by the instructor: this single change can nearly halve context consumption ("drop your subscription from $200 to $100 per month" being the motivating framing).

## Key Takeaways
1. **Point before you read**: enforce the GrabScript as the first step so the agent gets a file list at near-zero context cost.
2. **Reverse the search order**: narrow by files first, search deeper only where the best fit is — never dump scattered file contents.
3. **Embed SOT keywords in inline docs** at the top of files; some keywords should name the exact source-of-truth location.
4. **Give the script a path to follow** so it knows exactly where to look from the start.
5. **Delete the docs folder**: it drifts, gets ignored, and is not how the guardrail navigates — linting, TypeScript, and error handling do the real navigation work.
6. **Comment at the top of a function or block**, not on every line — density, not volume.
7. Enforce the whole flow via CLAUDE.md, not by discipline — that's the next module.

## Connects To
- **Ch 15**: inline injection — the What/Why/How/Where comments the script's output eventually leads the agent to read.
- **Ch 17**: CLAUDE.md orchestrator — where the GrabScript instruction is enforced and SOT keyword sections are generated, not hand-written.
