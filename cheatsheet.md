# Cheatsheet — Architectural Guardrail

## Security decision rules

| Situation | Do | Because |
|---|---|---|
| Building any endpoint | Route through `protectedRoute` / the block | Security enforced by default, not by remembering |
| Endpoint an agent may call | Attach ctx; resolve org ID from ctx.session | Prompt-supplied org IDs are attacker-controllable |
| Code must never run on client | `import "server-only"` in the service file | Bundled server actions are callable by ID |
| Page renders third-party components | Add layout-level permission check | They never reach the block layer |
| AI might scatter permission checks | Enforce pattern via ESLint/derived types | App code overrides instructions; lint errors don't drift |

## AI behavior rules

- **Never let AI use git** — it reads commit history and undoes finished work (Ch 1, Ch 17)
- **Errors over docs** — make every rule a compile/lint error, not a CLAUDE.md paragraph (Ch 4, Ch 7)
- **GRP-first** — GrabScript (or keyword search) before reading files; file lists cost ~zero context (Ch 16, Ch 17)
- **Bait-and-switch** — "these types already exist — use those first" (Ch 7)

## Build-order decision tree

```
Feature request
├── Simple (one-shot plain English) → prompt directly
└── Complex → number sub-features 1-5
    ├── Consumable first (calendar before canvas)
    ├── Structure → state → mutation (global-first thinking)
    └── Split intertwined tasks; stub second task loudly (Gap Method)
```

## Thresholds & defaults

| Threshold | Value |
|---|---|
| CLAUDE.md length | keep short (~25–86 lines); pattern rules first |
| Docs folder | delete it — AI ignores stale docs (frees 20–30% of context) |
| Inline comments | top of block only, What/Why/How/Where; never every line |
| Prompt savings with guardrail | 40–60% shorter (permissions/gates never re-described) |
| Plan | $100 tier suffices; guardrail lets you hit session limits and still ship 5–6x more |
| Type strictness | ban `any`/`unknown` in derived types |
| Provider wiring | Prisma from day one, or a later swap costs a full refactor |

## Tells & smells

- AI invents files / deviates from architecture → you're vibe coding; install the guardrail (Ch 1)
- AI goes "creative" on architecture → inconsistent codebase patterns; establish one pattern (Ch 4)
- One file deviates from the pattern → noise; align it, don't fork (Ch 4)
- AI ignores your docs → stale or too long; replace with inline injection (Ch 15)
- Missing org scope → breach, not a bug (Ch 12)
- Sidebar hides a link but page makes no API calls → copy-the-link bypass hole (Ch 14)
