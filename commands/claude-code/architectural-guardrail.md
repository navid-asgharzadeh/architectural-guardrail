---
description: Apply the architectural guardrail skill (Web Prodigies' AI-coding system) — topics, chapters, or porting it to a codebase
argument-hint: [topic | framework | chapter number]
---

# Architectural Guardrail

Load the skill at `~/.agents/skills/architectural-guardrail/SKILL.md` and follow it, then handle the user's input:

$ARGUMENTS

Rules:
- Topic or framework name (e.g. "rate limiting", "white line method") → find it in the skill's Topic Index, read the corresponding chapter file(s), and answer from those.
- Chapter number (e.g. "ch10") → read that chapter file and summarize or apply it.
- Applying the skill to this repo → follow the skill's "Applying to an Existing Codebase" section and `adaptation.md` — port the invariant, never the snippet.
- No argument → present the skill's Core Frameworks as a reference.
- Never copy raw passages verbatim — synthesize; chapters are already summaries.
