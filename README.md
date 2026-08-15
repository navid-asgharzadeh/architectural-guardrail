# Architectural Guardrail

Agent skill generated from the video course [*Advanced Claude Code for Web Developers (Full Course)*](https://www.youtube.com/watch?v=GCz83HTg2vI) by [Web Prodigies](https://www.youtube.com/@webprodigies).

The skill encodes the course's **architectural guardrail** system — a layered method for building production-grade apps with AI coding agents (pattern recognition, globalization, TypeScript lockin, the block layer, database isolation, inline injection, GrabScript, CLAUDE.md orchestration, the White Line Method).

> **Attribution**: this content is a synthesized summary of Web Prodigies' course — it is NOT the original video content. Please watch the original at the link above and support the channel.

## Install

```bash
npx skills add https://github.com/navid-asgharzadeh/architectural-guardrail --skill architectural-guardrail
```



## Usage

In any project, ask your agent:

- **Apply a pattern** — "apply architectural-guardrail rate limiting" or "add the block layer to this app"
- **Port to your stack** — "port the block layer to this repo (TanStack Start / Express / plain JS)" — the agent reads `adaptation.md` (stack mapping + 18 layer invariants)
- **Study a module** — "what does ch10 teach?" — chapters load on demand
- **Browse** — "what chapters does the architectural-guardrail skill have?"

## Slash commands (optional)

The skill triggers automatically by description, but if you want a `/architectural-guardrail` shortcut:

```bash
# opencode
cp commands/opencode/architectural-guardrail.md ~/.config/opencode/command/
# Claude Code
cp commands/claude-code/architectural-guardrail.md ~/.claude/commands/
```

Restart the agent session after copying. Usage: `/architectural-guardrail rate limiting`, `/architectural-guardrail ch10`, or no args for the Core Frameworks reference.

## File inventory

- `SKILL.md` — core frameworks, chapter index, topic index, porting workflow
- `adaptation.md` — stack mapping table (Next.js / TanStack Start / framework-agnostic) + layer invariants
- `chapters/ch01–ch18.md` — one module per chapter
- `glossary.md` — key terms
- `patterns.md` — techniques and design patterns
- `cheatsheet.md` — security rules, build-order tree, thresholds, tells
- `commands/opencode/` and `commands/claude-code/` — host slash-command wrappers

## Note

Chapter files contain synthesized summaries and reconstructed code sketches, not the book/video text.
