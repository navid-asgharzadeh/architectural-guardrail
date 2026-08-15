# Chapter 8: Normalizing Identity — An Adapter Layer Over Third-Party Providers

## Core Idea
Start with third-party libraries (never build from scratch — popular libraries give AI the documentation data it needs to stay consistent), then add a thin adapter layer that "normalizes" what those libraries give you — identity, roles, editors — so your app's shared resource stays stable even when you swap or extend the provider underneath.


**Portable invariant**: An adapter layer sits between providers and app-owned shapes — port this rule to any stack (see adaptation.md)
## Frameworks Introduced

- **Third-Party First (The "More Data" Rule)**: use widely-adopted third-party libraries instead of custom-built systems, because the more public data AI has access to, the more consistent its output becomes.
  - When to use: for anything foundational — authentication, editors, form builders — especially if you're a beginner.
  - How: pick the widely-used library (Next.js, Clerk, Lexical); custom code has no external documentation or links for AI to reference, so best practices get invented per-feature instead of inherited. Building from scratch forces you to document "really, really well," and even then AI has far less signal.

- **Provider Tradeoff Evaluation**: two library offerings in the same space each force a compromise — you choose one or the other, not both.
  - When to use: when picking between competing libraries (e.g., Better Auth vs Clerk) before writing the adapter.
  - How: compare along the two axes that matter for your app. Better Auth: open source, maximum flexibility, but none of the dashboard features. Clerk: speed, built-in components, session management, audits, dashboard control — but paid tiers and hard limits (10 roles per application). You must compromise on flexibility or convenience.

- **Normalizing Identity (Adapter Layer)**: a middle layer with a plugin-type system that wraps third-party providers, extracts the common things you need, and exposes them through one shared resource — so you can extend the library's features and swap providers without being locked in.
  - When to use: when a third-party tool restricts a feature you need (e.g., custom roles beyond Clerk's 10-role limit), or when the provider might need to be swapped in the future. Not always applicable — it's a targeted pattern, not a universal one.
  - How:
    1. Identify what your app needs that the provider can't give (role-based permissions with unlimited customized roles).
    2. Create an adapter type (e.g., `adapter membership`) with the extra property (`role`) plus helper functions that give permissions for that role.
    3. Keep the provider doing only what it's good at — Clerk handles authentication and account members; the adapter owns the role system (custom roles are NOT stored on Clerk's end).
    4. Build one shared adapter resource; per provider, write a small connection layer that mirrors that provider's identity into the shared shape.
    5. Later, swapping providers means writing a new connection layer, not rewriting the app.

- **Plugin-Type Extension (Build on the Baseline)**: pick third-party tools with plugin architecture, then write your own plugins/extensions on top instead of replacing the tool.
  - When to use: when you need a feature the baseline library doesn't have, or a complex feature you'd otherwise rebuild per surface.
  - How: consume the plugin interface and write your own adapters (signature element, input field with properties, dynamic variables) — one extension then works everywhere the base tool renders.

## Key Concepts

- **Normalizing identity**: extracting the common things from a provider into a standardized, app-owned shape so the rest of the app doesn't depend on the provider's specifics.
- **Adapter layer**: the middle layer connecting your app to third-party libraries; it owns the features the provider can't (or shouldn't) hold.
- **Adapter membership**: the normalized member object — carries a `role` property and permission helpers on top of what the auth provider manages.
- **Shared adapter / shared resource**: the single canonical adapter in the app; each auth provider gets a connection layer that mirrors into it.
- **Connection layer**: the per-provider translation code (Clerk branch, Better Auth branch) that feeds the shared resource.
- **Plugin-type system**: a library's extension mechanism (Lexical plugins) you build custom adapters into rather than forking the library.
- **Server-side rendering (as a library choice)**: a selection criterion — Lexical renders server-side, so one editor serves many surfaces of the app.
- **Lock-in avoidance**: the adapter exists so that changing providers later is a localized change, not a rewrite.

## Mental Models

- Think of the adapter as **a universal remote**: each provider is a different brand of device; the adapter speaks to all of them, and the app only ever holds the remote.
- Use **"provider does its job, adapter does yours"** when splitting responsibility: Clerk authenticates and manages members; the role system — the part your app relies on — lives in the adapter, not in Clerk.
- Think of plugin-type libraries as **a baseline you extend, not a box you fit inside**: you build signature, input, and dynamic-variable plugins on Lexical's baseline instead of rebuilding a Notion-style block editor.
- Use **"build once on the baseline, render everywhere"** when a component appears in many places: one editor, built on Lexical, is reused in tickets, the website builder's rich text, invoices, and contracts — expansions propagate everywhere for free.

## Anti-patterns

- **Building from scratch when a third-party exists**: custom code has no external docs or examples for AI to reference, so AI can't anchor on best practices — you lose the consistency that the "more data" rule buys, and features (a block editor, auth) take forever.
- **Storing your custom domain on the provider**: putting custom roles inside Clerk hits the 10-role cap and couples your business logic to a paid-tier limit; the role system belongs in the adapter where it's a few lines of code you control.
- **Expecting the adapter to always work**: normalization is not universal — some tools can't be adapted cleanly; the instructor flags it explicitly as a pattern that "will not always work."
- **Swapping providers without connection layers**: if provider-specific data leaks into app code, changing auth later means rewriting every consumer; without per-provider connection layers mirrored into the shared resource, you're locked in despite having an "adapter" in name.
- **Assuming you must install both libraries**: normalization is not "install Clerk and Better Auth together" — it's extracting the common things into your own layer, not doubling dependencies.

## Code Examples

The transcript describes the adapter code on screen rather than pasting it; reconstructed from the instructor's walkthrough (structure and property names as described):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// normalized identity — the adapter the app actually uses
type AdapterMembership = {
  role: string;                 // custom role — NOT stored on Clerk's end
  // ... identity fields mirrored from the provider
};

// permission helpers for the role live in the adapter layer
function canCreateResource(membership: AdapterMembership) {
  // "very little bit of code" — resolves permissions for the custom role
}

// connection layer — per provider, mirror into the shared adapter
// branch A uses Clerk only (auth + members, no roles)
// branch B uses Better Auth / other providers
function fromClerkUser(clerkUser: ClerkUser): AdapterMembership {
  return { role: /* from your own role store */ };
}
```

- **What it demonstrates**: Clerk handles authentication and account members only; the `role` property and its permission helpers live in the adapter, so the role-based system survives any provider swap and is never constrained by the provider's limits.

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// plugin-type extension: build on the baseline library, not a fork
// Lexical is the open-source, server-rendered editor baseline
// custom plugins expand it:
//   - signature element
//   - input field (with properties)
//   - dynamic variables (e.g., insert {{firstName}} from a connected lead/recipient)
```

- **What it demonstrates**: one Notion-style editor, built by extending Lexical's plugin architecture, is rendered server-side and reused in tickets, rich text, invoices, and contracts; custom features are written once and available everywhere the baseline renders.

## Worked Example

Normalizing identity over Clerk for funnelmod.ai:

1. **Pick the third-party first**: the app starts with Clerk because it's fast, has built-in components, and allows session control from the dashboard. Auth itself is never custom-built.
2. **Hit the provider's wall**: the app needs team members with customized roles — but Clerk caps the entire application at 10 roles, and exceeding that requires upgrading tiers. The app *relies* on roles, so this constraint is unacceptable.
3. **Extract the common thing**: instead of storing custom roles in Clerk, the app defines an adapter membership with a `role` property plus helpers that give permissions for that role. The role-based system — easy to handle, very little code — lives entirely in the adapter layer.
4. **Keep the provider doing its one job**: Clerk continues to handle authentication and the members on the account; custom roles are simply not stored on Clerk's end.
5. **Mirror into the shared resource**: in the app there is one shared adapter (the "shared permission" thing). Different branches use different providers — one branch is Clerk-only, another uses Better Auth and other third-party libraries — and each branch only needs its own connection layer that mirrors into the shared resource.
6. **Stay un-locked**: because everything downstream consumes the adapter, switching providers later means writing a new connection layer — not touching the role logic, the permission helpers, or the UI.

The same move applies to the editor: the Notion-style block editor would take forever to build from scratch (all blocks, components, plus AI functionality). Instead the app leveraged Lexical — open source, plugin-type design, server-side rendering — and built its own plugins (signature element, input field with properties, dynamic variables). The dynamic-variable plugin, the one truly built-from-scratch complex feature, was easy to implement across the whole app because it consumed Lexical's plugin architecture rather than being re-implemented per surface.

## Key Takeaways

1. Prefer widely-used third-party libraries over custom builds — the more data AI has, the more consistent the outcome; custom code gives AI nothing to reference for best practices.
2. Choose providers consciously: Better Auth = maximum flexibility, open source; Clerk = speed, components, session management, audits — and hard limits. You compromise either way.
3. Put features the provider can't do (custom roles, unlimited role counts) in an adapter layer with a plugin-type system — a `role` property plus permission helpers is "very little bit of code" and survives provider swaps.
4. Let the provider do only what it's good at; never store your app-critical domain (custom roles) on the provider's end.
5. Build one shared adapter as the canonical resource; each provider gets a connection layer that mirrors its identity into it — then changing providers is a new connection layer, not a rewrite.
6. When extending a library, prefer ones with plugin architecture (Lexical) and build your own plugins on the baseline; pick server-rendering libraries so one component serves every surface (tickets, rich text, invoices, contracts).
7. Know the pattern's limits: normalizing identity will not always work for every tool — apply it where the provider's restriction is real and the common surface is extractable.

## Connects To

- **Ch 7 (TypeScript lockin)**: the adapter membership and its `role`/permission shape should be defined as types so misuse errors at compile time — normalization gives TypeScript a stable, app-owned shape to lock onto.
- **Ch 9 (server-only)**: normalization chooses libraries by their server capabilities (Lexical renders server-side); the next module explains the server-only concept this builds on.
- **Ch 6 (registry)**: the shared adapter is itself a shared resource — the same "one canonical thing" idea that the registry layer turns into the AI's single source of truth.
- **Ch 5 (globalization)**: this module explicitly combines and extends the globalization strategy — extracting common things from third-party libraries into one normalized layer is globalization applied to external dependencies.
