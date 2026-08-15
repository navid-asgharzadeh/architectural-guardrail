# Chapter 12: Server-Side Org Scoping — Inject Context at the API, Never in the Prompt

## Core Idea
The org ID is a security boundary the AI must never see, guess, or predict; instead, the authenticated user's session (a `ctx` object) is injected into the API endpoint itself, so every database call is org-scoped at the server — even if the prompt asks for the wrong org.


**Portable invariant**: Tenant identity is resolved server-side, never from input or prompt — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Server-Side Context Injection (the `ctx` object)**: "This org ID should never be plugged in… AI should never think about this org ID. It should be dynamically injected into the call itself — not injected into the prompt, injected into the API endpoint."
  - When to use: any API endpoint that a human or an AI agent can invoke, especially anything an agentic layer (like Mochi AI) will chain together.
  - How: give every endpoint a `ctx` object that carries the session of the user currently logged in and actively working; resolve tenant scope from `ctx` and pass it straight into the database calls.

## Key Concepts
- **`ctx` object**: a context object on each API endpoint that carries the session of the currently logged-in, actively working user.
- **Dynamic org injection**: tenant identity is attached to the API request at runtime on the server, not supplied by the caller.
- **Cross-org contamination**: the security breach where an AI is tricked (e.g. by visiting a page on someone else's website) into believing it belongs to a different org, then reading/writing that org's data.
- **Prompt vs. endpoint injection**: the prompt may literally say "fetch from org ID XYZ," but the endpoint ignores that and fetches from the correct org ID because scope is injected into the API request itself.
- **Org scoping**: the practice of constraining all data access to the session's org, applied uniformly to all API endpoints.
- **Agentic chaining safety**: because scope is server-injected, the AI can chain many operations (form → questions → custom data → site → automations) without ever handling tenant identity.

## Mental Models
- Use server-side injection when tenant identity is a security boundary — think of the org ID as a watermark stamped onto the request by the server, never a field the caller fills in.
- Think of the prompt as untrusted user input: the AI may hallucinate any org ID it wants, and it won't matter, because the truth is attached on the back end.
- Use the "never let AI think" rule when any value could break isolation — the AI should only use patterns and pre-built things, not derive security-relevant values.

## Anti-patterns
- **Letting the AI predict or fetch the org ID**: to predict it, the AI would have to fetch the current website ID, then fetch its org ID via your endpoints — a user can trick it into resolving a different website, creating cross-org contamination.
- **Passing org ID in the prompt for agentic flows**: any value the AI supplies is attacker-controllable; even "harmless" hallucination becomes a security breach.
- **Scoping only some endpoints**: the same `ctx` strategy must be followed in all API endpoints, or future agentic operations will find the gap.

## Code Examples
The module describes rather than displays the code; this snippet reconstructs the mechanism exactly as described (ctx carries the session; scope goes directly into the Prisma call):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// API endpoint — org ID injected server-side, never read from the prompt
export async function listComponents(ctx: Ctx) {
  return prisma.component.findMany({
    where: { orgId: ctx.session.orgId }
  });
}
```

- **What it demonstrates**: the org ID comes from the injected `ctx` session and "gets plugged directly into the Prisma schema or the Prisma calls," so a prompt saying "fetch org XYZ" still returns only the caller's own org data.

## Worked Example
Mochi AI's marketing-campaign chain: the agent builds a form, creates qualification questions, attaches them to custom data, creates a site, attaches the form to it, and sets up automations to send emails — all in one chained run. This is safe only because every endpoint in the chain org-scopes by dynamically injecting the session into the API endpoint. If the org ID lived in the prompt, an attacker could point the chain at someone else's org at any step.

## Key Takeaways
1. Never let the AI see, guess, or fetch the org ID — inject it into the API endpoint, not the prompt.
2. Attach a `ctx` object carrying the authenticated session to every API endpoint.
3. Plug `ctx` scope directly into the Prisma schema or Prisma calls so isolation holds at the database boundary.
4. Apply the same strategy to all endpoints uniformly — future agentic operations will rely on it.
5. Even when the prompt says "fetch from org ID XYZ," the endpoint must fetch from the correct org ID regardless.
6. Treat AI as an untrusted caller at the API layer: the server owns tenant identity, always.

## Connects To
- **Ch 11**: database isolation — separating business logic from database logic, services with `import "server-only"`, zod-validated procedure inputs
- **Ch 13**: rate limiting / audit logs
