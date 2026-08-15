# Chapter 9: Server-Only Module

## Core Idea
The most common AI-created security hole is an API endpoint whose permissions live only in the caller; the fix is making every service function server-only by default, so it is never bundled to the client and can never be invoked directly by a hacker.


**Portable invariant**: Security-critical code can never run client-side — port this rule to any stack (see adaptation.md)
## Frameworks Introduced
- **Pattern-over-reasoning (give AI a pre-existing pattern so it doesn't have to think)**:
  - When to use: whenever a rule matters for security, because "AI can pretty much do whatever it wants" — reasoning about security each time leads to inconsistent results.
  - How: connect the rule to a pattern that has been used before, so AI just starts implementing the pattern instead of deciding.

- **Server-only conversion (replace server actions with server-only functions)**:
  - When to use: any time you have server actions, because a server action can be called from the client and gets bundled and shipped there.
  - How: add the server-only import statement at the top of each service file, which prevents the code from being bundled with the client — it can only run on the server.

## Key Concepts
- **Server action**: a Next.js function that does server stuff, but can also be fired from the client — the name is misleading.
- **Client bundling**: server actions are bundled and sent to the client; the client then technically holds everything needed to invoke the function.
- **Bundle ID exploit**: a hacker who figures out the bundle ID can call the exact server function directly, bypassing all API-endpoint permissions.
- **Server-only import**: the import statement at the top of every service file that excludes the code from the client bundle, so nobody can call it from the client.
- **Service file**: the module containing app logic; every service starts with the server-only import, making it run only on the server.
- **ESLint enforcement**: the same server-only rule is enforced by the ESLint config, so services can only be called on the server — strictly enforced, not just instructed.

## Mental Models
- Use **pattern-over-reasoning** when wiring security into AI workflows: AI doesn't have to think about security; it just has to start implementing the pattern.
- Think of **server actions as client-addressable endpoints**: any function bundled to the client is a callable endpoint — the word "server" in the name grants no protection.
- Think of **the server-only import as a firewall for code paths**: if it's not in the client bundle, there's nothing for the client to call.

## Anti-patterns
- **Auth-inside-each-endpoint**: AI creates one API endpoint for every change in the app and puts authentication checkers, permission feature gates, etc. inside each of them — security becomes scattered, repetitive, and only as good as each endpoint's internals.
- **Relying on endpoints for permissions**: if nothing is done in the server action and you rely on the API endpoints to do protection, the hacker bypasses all permissions and calls the server function directly.
- **Trusting the name "server action"**: assuming a server action only runs on the server — it can be fired from the client, bundled, and reverse-engineered by ID.

## Code Examples
```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
import "server-only";
```
- **What it demonstrates**: the one line at the top of every service file that prevents the file's code from being bundled with the client.

## Worked Example
A Next.js app has a form with a submit button wired to a server action. Because server actions can be called from the client, the function is bundled and sent to the client. An attacker who finds the bundle ID can call the exact server function directly, bypassing every permission check that lives in the API endpoint layer. The smallest step that eliminates the problem entirely: add `import "server-only"` at the top of every service file, converting all server actions into server-only functions. The code is no longer in the client bundle, so it can only run on the server. The guardrail then instructs AI to use this in all services, and the ESLint config strictly enforces it — all services are protected by default.

## Key Takeaways
- Treat every server action as client-callable: it gets bundled and shipped to the client, and a hacker who finds the bundle ID can call it directly.
- Convert all server actions to server-only functions — the smallest step that completely eliminates the direct-call problem.
- Put the server-only import at the top of every service file; it prevents the code from ever being bundled to the client.
- Instruct AI to apply the pattern in all services, then back it with ESLint so it is strictly enforced, not just requested.
- With server-only in place by default, every service is protected and runs only on the server.

## Connects To
- **Ch 8**: normalization identity
- **Ch 10**: the block layer (auth/permission checks connect into the block, examined later)
