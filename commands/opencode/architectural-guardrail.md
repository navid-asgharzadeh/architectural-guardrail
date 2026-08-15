---
description: Apply the architectural guardrail skill (Web Prodigies' AI-coding system) — topics, chapters, or porting it to a codebase
---

The user invoked /architectural-guardrail.

First, load the architectural-guardrail skill from `~/.agents/skills/architectural-guardrail/SKILL.md` (read that file and follow its instructions).

Then handle the user's input:

$ARGUMENTS

Rules:
- For a topic or framework name (e.g. "rate limiting", "white line method"): find it in the skill's Topic Index, read the corresponding chapter file(s), and answer from those.
- For a chapter number (e.g. "ch10"): read that chapter file and summarize/apply it.
- For applying the skill to a repo: follow the skill's "Applying to an Existing Codebase" section and `adaptation.md` — port the invariant, never the snippet.
- If no argument is given: load and present the skill's Core Frameworks as a reference.
- Never copy raw passages verbatim — synthesize; chapters are already summaries.
