# Chapter 14: The Security Gap — Paths That Bypass the Block Layer

## Core Idea
The block layer (`protectedProcedure`) is the source of truth for all security, so any part of the application that does not consume it — such as third-party components rendered directly on a page — silently violates every security practice you put in place; the fix is a page-level check in the layout that reuses the resources source of truth, not another router.


**Portable invariant**: Any surface bypassing the gate gets a check at its closest common ancestor — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Gap analysis on the block layer**:
  - When to use: whenever you add any rendering or data path — especially third-party components — ask whether it flows through the `protectedProcedure`; if not, you have a security gap.
  - How: treat the block layer as the *only* source of security truth. Walk each page/component and classify every path as "consumes the block" or "does not consume the block." A single non-consuming path = every security practice (permissions, audit, rate limiting) is void for that path, because all of it lives inside the block.

- **Layout-level guard check (reuse the resources source of truth)**:
  - When to use: when a page renders third-party components (e.g., Clerk's built-in billing components) that never hit your tRPC procedures, so the block layer never fires.
  - How: put a checker in `layout.tsx` (or the adapter layer). Reuse the same resources object built earlier — the `requirePermissions` definitions and the nav items — combine the request's pathname with the user's permissions, and determine whether the user may access this page. Run as many checks as needed; if access fails, throw/redirect. The layout placement is deliberate: it also covers any future third-party tool that doesn't go through an adapter.

- **Simplest-fix selection**:
  - When to use: when a security gap is found and the tempting solution is to build new plumbing (a new router/procedure to push the components through).
  - How: reject the new-procedure approach — third-party components can't meaningfully be passed through your procedure. Pick the smallest layer that already wraps everything: the layout. Acknowledge it's "decent enough, not the best way" — the goal is to close the hole now and leave room for improvement.

## Key Concepts
- **Security gap**: any code path in the app that renders or interacts without consuming the block layer — one such path voids every security practice, since all enforcement lives inside the block.
- **Third-party component surface**: components supplied by a provider (Clerk in the course) that render via the provider's own component, not through your tRPC procedure.
- **Copy-the-link access**: because a page with no tRPC calls fires no permission checks, anyone can copy the URL of a restricted page (e.g., billings) and open it directly, even without permission.
- **Resources source of truth**: the registry from earlier modules — `requirePermissions` definitions plus nav items — reused in the layout check so access logic is not reinvented.
- **Layout check**: a server-rendered gate in `layout.tsx` that reads the pathname, matches it against nav items in resources, and combines that with the user's permissions to allow or deny the page.
- **Sidebar visibility**: hiding a nav link via the resources source of truth is a UX decision, not access control — the page itself must still be gated.

## Mental Models
- Think of **the guardrail as a fence whose gate is the block layer**: every security check (permissions, audit, rate limiting) stands at that one gate; a third-party component is a hole in the fence, and walking through the hole skips the gate entirely.
- Use **"visible ≠ authorized"** when reviewing UI: the sidebar already hides the billings link for users without permission, and the service/router layers already throw on API calls — but a page that makes no API calls is still reachable by pasting the URL.
- Think of **the layout check as a catch-all net**: it doesn't replace the block layer; it covers exactly the traffic that can't reach the block, including future third-party tools that bypass adapters.
- Use **"close the hole, then improve"** when choosing a fix: a decent layout check shipped today beats a perfect abstraction that takes a week — the instructor explicitly says this isn't the best way, just a solid one.

## Anti-patterns
- **Trusting UI visibility as security**: hiding a page in the sidebar does nothing when the page renders no protected calls — a copied link reaches it with zero checks firing.
- **Wrapping third-party components in a new procedure**: the instinct is to create another router or procedure so the components "go through" the block — but provider components render through their own component; the simplest fix is a layout check, not new plumbing.
- **Assuming provider components inherit your guardrail**: Clerk components are third-party surfaces that don't go through your `protectedProcedure`; treating them as protected because "everything is protected" is the exact gap this module exists to close.
- **Over-engineering the fix for one hole**: the instructor notes the layout check can live at the adapter layer too and that better ways exist — but the priority is closing the gap, not building a general framework.

## Code Examples
No code is shown in this module — the instructor describes the implementation in prose: in `layout.tsx`, use the same resources source of truth created earlier (the `requirePermissions` resources), read the pathname, combine it with the nav items on the resources object and the user's permissions, and decide access; run as many checks as needed, and throw/redirect on failure. The shape is a layout-level guard, with the adapter layer as an alternative placement.

- **What it demonstrates**: the fix reuses the existing registry instead of inventing a parallel permission system.

## Worked Example
The app renders billing-plan cards using Clerk's built-in components, chosen because Clerk's setup is easier and ships components out of the box. Those components render through Clerk itself — they never call the app's tRPC procedure, so the block layer (session, permissions, audit, rate limiting) never runs for this page. Now give a team member a permission that excludes the billings page. Two defenses already work: the sidebar hides the link (the app uses the resources source of truth for visibility), and the service/router layers throw on any API request from that page, kicking the user back to the dashboard. But the billings page itself makes no API calls, so nothing goes through the tRPC procedure. Result: the restricted user simply copies the billings page URL, pastes it, and opens the page directly — the gap. The fix: a checker in `layout.tsx` that reuses the same resources object — matching the request pathname against nav items and the user's permissions, running as many checks as needed, and throwing/redirecting when access is denied. Placed at the layout, it also covers any other third-party tool that doesn't go through an adapter. The instructor flags this as one decent implementation, not the best — improvement is welcome.

## Key Takeaways
- Treat the block layer as the single source of security truth: any path that doesn't consume it voids every security practice for that path.
- Audit for the gap systematically — third-party components (Clerk and similar providers) are the classic non-consuming surfaces.
- Never treat UI visibility as authorization: a hidden sidebar link is cosmetic when the page fires no protected calls.
- Close a gap with a layout-level check that reuses the resources source of truth (`requirePermissions`, nav items, pathname, user permissions) — not with a new router or procedure.
- Put the check at the layout so future third-party tools that bypass adapters are covered too.
- Ship a decent fix and note where it could improve (adapter layer, more checks) — closing the hole beats perfect abstraction.

## Connects To
- **Ch 10**: block layer — this module repairs the case where the block is *not* consumed; the block remains the source of truth.
- **Ch 6**: registry — the layout check reuses the resources source of truth and its nav items rather than reinventing access logic.
- **Ch 9**: server-only — the layout check is a server-rendered enforcement point, consistent with server-side gating.
- **Ch 13**: rate limiting — a path that bypasses the block also bypasses the rate limiting wired into it; the gap is a rate-limit gap too.
- **Ch 15**: inline injection — further hardening that assumes the page-level and block-level gates from this chapter are in place.
