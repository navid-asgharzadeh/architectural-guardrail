# Chapter 2: Setting Up Claude Code on Your Local Machine

## Core Idea
Getting Claude Code running in your terminal is deliberately minimal: an account, one install command, one launch flag, and a single CLAUDE.md file. The author's thesis is that a plain CLAUDE.md is the *absolute minimum* needed to produce production-grade code — no skills, no subagents required.


**Portable invariant**: One minimal rules file plus branch-per-feature works on any agent, with zero extra tooling — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Minimum Viable Guardrail**: one CLAUDE.md file with a single rule ("don't use git") is enough to start building production-grade code.
  - When to use: at setup time, before any feature work.
  - How: create `CLAUDE.md` inside a `claude/` folder in the project root; add the git rule at the bottom of the file; resist adding skills, subagents, and "crazy workflows" until something works. "First figure out something that's working and then improve it as you go."

- **Branch-per-feature isolation**: every unit of work gets its own git branch, never main.
  - When to use: before every new feature, experiment, or one-shot prompt attempt.
  - How: open a second terminal and run `git checkout -b <branch-name>`; iterate by creating sibling branches off a known-good checkpoint (e.g. V5.1, V5.2) until the result is satisfactory; only then is work merged forward.

## Key Concepts
- **$100/month plan**: recommended Claude tier; the author and most course members are on it and never need the $200 plan's 20x usage.
- **Terminal-first workflow**: Claude Code is run inside the VS Code integrated terminal for coding; the web UI/desktop app is reserved for research and brainstorming.
- **`--dangerously-skip-permissions`**: launch flag that gives Claude full bypass access — it stops asking for approve clicks on every action and simply does the work.
- **Destructive-by-default operation**: running Claude with skipped permissions is a "do it at your own risk" decision; the author accepts it because he trusts the system and knows it works.
- **"Don't use git" rule**: a note appended to the bottom of CLAUDE.md instructing Claude not to touch git without an explicit request.
- **Session-history corruption failure**: Claude sometimes reads its own commit history, assumes the entire commit range is "what it's working on," and undoes finished work to rebuild it — costing the author his progress multiple times.
- **Versioned branches**: keeping multiple feature branches (a website builder required five full attempts, V1–V5) so main is never corrupted and competing versions can be compared.
- **Stack-agnostic setup**: the course uses a Next.js app as the guardrail template, but any stack works.

## Mental Models
- **Use the terminal for code, the UI for chat**: think of Claude Code as the compiler and the desktop app as the whiteboard.
- **Think of `--dangerously-skip-permissions` as autopilot**: you trade approval clicks for trust — set it only after you believe the system behaves.
- **Think of branches as checkpoints, not just features**: each branch is a save slot; from any good checkpoint you can fork N versions and keep the best.
- **Use the "minimum viable guardrail" when overwhelmed**: one rule that works beats ten workflows that don't.

## Anti-patterns
- **Letting Claude read git history freely**: it may treat the entire commit history as its current session and undo completed work to "rebuild" it — the author lost real work this way, multiple times. Fix: the bottom-of-CLAUDE.md "don't use git" note.
- **Building on main**: any corruption from an experiment or one-shot prompt hits the one branch you can't afford to lose. Fix: `git checkout -b` before every feature.
- **Over-engineering setup day one**: loading up on skills, subagents, and complex workflows before anything works creates overwhelm without producing code. Fix: ship the minimum, improve as you go.
- **Giving up on install when the command isn't found**: on some shells the freshly installed binary isn't on PATH until restart. Fix: close and reopen the terminal, then re-verify.

## Code Examples

```bash
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
# macOS install (from the Claude Code quick start guide)
curl -fsSL https://claude.ai/install.sh | bash

# Verify the install
claude

# Launch Claude with full bypass — no per-action approve clicks
claude --dangerously-skip-permissions

# Branch-per-feature: always start work on a new branch
git checkout -b v5.1
```

- **What it demonstrates**: the complete lifecycle — install from the official quick start, verify, launch inside the project folder (opened in VS Code) with permissions skipped, and fork a fresh branch before any work.

## Worked Example
Setting up the guardrail environment end to end:

1. **Account**: open Claude's pricing page, click "Try Claude," create an account, and subscribe to the $100/month plan (the $200 tier's 20x usage is unnecessary).
2. **Install**: search "install Claude Code," open the Quick Start guide, copy the install command matching your OS, run it in the terminal, and verify with `claude`. If the command isn't found, restart the terminal and try again.
3. **Project**: create (or drop) a project folder into VS Code so Claude can access the files; open the integrated terminal in that project.
4. **Launch**: run `claude --dangerously-skip-permissions` — accepting that this is a destructive, at-your-own-risk operation that bypasses approval prompts.
5. **Guardrail file**: create `CLAUDE.md` in a `claude/` folder; append the rule at the bottom of the file: don't use git without explicit request.
6. **Branch**: in a second terminal, run `git checkout -b <feature-branch>`. Iterate by forking new branches off known-good checkpoints (V5.1, V5.2, ...) until the result looks right.

## Key Takeaways
- Buy the $100/month plan, not $200 — the guardrail system never needs 20x usage.
- Run Claude in the terminal for coding; use the desktop app only for research/brainstorming.
- Verify installs with `claude` and restart the terminal if the command isn't found.
- Launch with `--dangerously-skip-permissions` deliberately — it's destructive but eliminates approval friction.
- Put "don't use git" at the bottom of CLAUDE.md to stop Claude from undoing its own committed work.
- Create a new branch per feature so main never gets corrupted and competing versions can coexist.

## Connects To
- **Ch 1**: why — the guardrail system's promise of hands-off production code only holds if Claude can act freely, which requires this setup.
- **Ch 3**: why — the CLAUDE.md file created here is the architectural guardrail's home; the next chapter shows what else goes into it.
