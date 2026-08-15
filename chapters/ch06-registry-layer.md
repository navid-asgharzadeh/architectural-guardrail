# Chapter 6: Registry Layer

## Core Idea
Make the registry a source of truth that AI naturally discovers by searching the codebase — then embed it inside a high-traffic construct (the protected procedure) so AI never has to figure out which source of truth to use; it just copies the pattern it finds everywhere.


**Portable invariant**: The source of truth is embedded in the most-used code path — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Embed-the-guardrail (plug the global pattern into the most-used code path)**:
  - When to use: when you've built a global pattern (like the build-permission system) but AI would otherwise have to search and decide which source-of-truth function to use.
  - How: wrap the global logic in a construct used more frequently across the app (e.g., a `protectedProcedure` containing all the permission logic), so every feature AI builds flows through it by default.

- **Permission-per-resource map**:
  - When to use: when team roles need CRUD on some resources but not others (e.g., create/read/delete invitations, but no update).
  - How: attach the same permission gate to each plan/resource, then apply it to a role or account — grant create, read, update, delete per resource, and simply remove the permissions you don't want on a specific resource.

## Key Concepts
- **Registry layer**: the module that stores resource definitions so AI, while searching the code, finds the same patterns and reuses them verbatim.
- **Source of truth**: the single construct AI should copy; the goal is to restrict AI's need to search and reason about which source of truth to use.
- **Protected procedure**: a high-frequency wrapper that already contains all the global permission logic, so AI uses the guardrail without being told to.
- **Permission gate**: the same reusable permission applied to roles/accounts per resource (create/read/update/delete), removable per resource.
- **Globalized upgrade message**: one upgrade prompt per resource (with string interpolation to plug in, e.g., the next plan to promote) shown when a user hits a plan limit.
- **Navigation gate**: the same resource definitions gate sidebar/navigation elements, enforced both client-side and server-side.

## Mental Models
- Use **AI-as-copyist** when designing guardrails: AI searches the code, finds a pattern, and uses "the exact same thing that it sees in the other files" — so make the thing it sees be the guardrail.
- Think of **the protected procedure as a chokepoint**: put the global pattern in the code path used most, and enforcement happens by default rather than by instruction.
- Think of **permissions as a per-resource switchboard**: flip CRUD permissions on or off per resource — flexibility where needed, while the source-of-truth instruction is still followed app-wide.

## Anti-patterns
- **Leaving the source of truth "at a global level" for AI to discover**: AI has to figure out which architecture or function to use each time; instead, enforce the global pattern inside something used frequently so discovery is automatic.
- **One-size-fits-all permission grant**: granting every permission to every resource when some resources shouldn't support them (e.g., updating an invitation) — remove unwanted permissions per resource instead.

## Worked Example
Resources are defined once in the registry. Each resource carries its own permission map and an upgrade message. When a user hits their plan limit and tries to use a resource, the system shows that resource's globalized upgrade message — string interpolation plugs in the next plan (e.g., "the next plan is promoted from here"). The same resource definitions build the sidebar: on page refresh, sidebar elements are locked to the same feature gate, client-side and server-side, so even if AI makes a mistake, the code is still protected.

## Key Takeaways
- Make the registry the source of truth AI discovers by search — AI will reuse exactly what it sees in other files.
- Restrict AI's need to *reason*: embed the global guardrail inside the most-used construct (protected procedure) instead of leaving it to be found at a global level.
- Attach permissions per resource, per role/account, and remove what shouldn't exist per resource (e.g., no "update" on invitations).
- Globalize upgrade messages per resource with interpolation so hitting a plan limit is handled uniformly.
- Reuse the same resource definitions as the navigation gate — enforce both client- and server-side so a mistake by AI can't break protection.

## Connects To
- **Ch 5**: globalization
- **Ch 7**: TypeScript lockin
