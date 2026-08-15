# Chapter 5: Globalization — One Source of Truth for Server and Client

## Core Idea
Globalization means finding logic your app needs in more than one place (server, client, multiple components), extracting it into one shared, plan-structured source of truth, and letting both sides consume it — so AI (and you) never have to guess which version is canonical.


**Portable invariant**: One canonical helper per concern, shared by every layer that needs it — port this rule to any stack (see adaptation.md)
## Frameworks Introduced

- **The Key Drawer (Globalization Analogy)**: every reusable piece of logic is a key; you keep exactly one drawer for all keys instead of hiding each key in a different room.
  - When to use: any time the same concept (a feature gate, a limit, a permission) must exist on both server and client, or in multiple components.
  - How: identify logic that is repeated across layers, extract it once as a standalone helper, and make both sides consume that helper — never re-implement per layer.

- **Resource Map (Source of All Truth)**: a single TypeScript data structure that maps over all plan keys and generates, per resource, the permissions, limits, and messaging for that plan.
  - When to use: when decisions like "can this user's plan do X?" must be answered identically by server logic, client logic, and UI components.
  - How:
    1. Define plan keys once (`free`, `starter`, `pro`, `enterprise`, plus a special `portal` plan that bypasses internal payments).
    2. Enumerate resources (members, invitations, organization, roles, custom branding…).
    3. For each resource define: `name`, `description`, `upgrade message`, and a `limit` whose naming is locked to the same plan keys.
    4. Wire the resource entries to navigation/settings so one edit changes the whole app.

- **Client-Side Helper → Universal Function**: since "the client side version is nothing but just JavaScript," the logic in a client helper can be lifted into a plain function callable from both server and client.
  - When to use: whenever you catch yourself writing a "client-side version" and a "server-side version" of the same rule.
  - How: extract all the complexity (plan lookup, usage comparison, upgrade decision) into a function whose inputs are just parameters — the resource being accessed and the cached plan/usage data — and that throws an error when criteria aren't met. Wrappers on either side are fine; the rule lives in one place.

- **TypeScript Lockin** (previewed here, full treatment later): use TypeScript as much as possible so the type system itself throws "echo signals" — compile errors that tell AI exactly what body/shape to follow, and inject microcontext (e.g., "this plan name doesn't exist") when it makes a mistake.
  - When to use: from the moment you define the globalized structure — the data structure is created with TypeScript precisely so misuse fails loudly everywhere.
  - How: hardcode the constructor for the dataset so plan names, resource fields, and limits are typed; then any wrong plan name (e.g., a stray `"free"` string) shows errors across every page that touches it.

## Key Concepts

- **Globalization**: removing parts of your code and setting them as a standard that powers all other features — one spot, like a key drawer, instead of five scattered copies.
- **Feature gate**: a permission/limit check that must exist at both server level and front-end level (page access, form actions, components).
- **Code drift**: the divergence that happens when you rebuild the same logic per layer — client version, server version, then a component-level version AI invents because it can't tell which one is canonical.
- **Resource**: a named capability area (e.g., custom branding, members) with a description, upgrade message, limit, and per-plan permission gates.
- **Plan keys**: the canonical list of plan identifiers (`free`, `starter`, `pro`, `enterprise`, `portal`) that everything else locks onto.
- **Cached plan/usage**: a cached variable, available on both server and client, holding the user's plan and what they've consumed; its structure mirrors the resource map so they can be compared directly.
- **Resource helper function**: a reusable function taking (resource, cached data) and throwing an error when the criteria fail — pluggable into components like a plugin.
- **Microcontext injection**: the type/definition itself teaches AI the shape, so an error gives the AI the exact context it needs to self-correct.
- **Protected procedure**: the frequently-used server entry point (e.g., tRPC protected procedure) into which the permission logic is plugged by default, so AI inherits the global pattern without having to search for it.
- **Permission gate**: per-resource create/read/update/delete permissions attachable to roles or accounts; removing a permission removes it from that resource.

## Mental Models

- Think of globalization as **a plugin**: you plug the helper into client components, into page components, into server procedures — it doesn't live anywhere; it's consumed everywhere.
- Think of the resource map as **app-wide settings**: change one entry (a limit, an upgrade message) and every surface in the app changes with it.
- Use **"source of truth + frequent surface"** when wiring AI: don't just create the global pattern — plug it into the most-used abstraction (the protected procedure) so AI finds the pattern by finding the thing it already uses constantly.
- Use **"server checks first, then compare"** when enforcing limits: the server does a quick check, then the shared helper does the plan-vs-usage comparison — the check order may vary per side, the logic never does.

## Anti-patterns

- **Duplicating logic per layer**: writing a client-side version and a server-side version guarantees divergence; later, AI sees two versions, can't differentiate, and generates a third (component-level) copy — code drift compounds.
- **Globalizing forms and components**: globalization is for tiny processes (feature gates, limits, permissions), not for UI — those stay component-local; only the logic gets extracted.
- **Tightly coupling the rule to server or client context**: if the helper requires server context or client state, it can't be shared — keep the rule independent, and let each side wrap it with its own quick checks.
- **Making AI search for the source of truth**: if the canonical helper sits somewhere obscure, AI has to figure out which source of truth to use — and will guess. Plug the rule into the commonly used abstraction instead.
- **Leaving plan names as free strings**: untyped plan identifiers silently accept typos; TypeScript lockin turns a wrong plan name into errors everywhere, which is the echo signal you want.

## Code Examples

The transcript describes the code on screen rather than pasting it; reconstructed from the instructor's walkthrough (structure and field names as described):

```ts
// [reconstructed from transcript — structural sketch; port the invariant, not this code]
// plan keys — the single canonical list
type PlanKey = "free" | "starter" | "pro" | "enterprise" | "portal";

type Resource = {
  name: string;           // e.g. "custom-branding"
  description: string;
  upgradeMessage: string; // shown in the upgrade modal
  limit: Record<PlanKey, number>; // naming locked to plan keys
  permissions: ("create" | "read" | "update" | "delete")[];
};

// source of all truth — maps over plans, generates structure per resource
const resources: Record<string, Resource> = {
  members: { /* name, description, upgradeMessage, limit, permissions */ },
  invitations: { /* no "update" permission, e.g. */ },
  organization: {},
  roles: {},
  customBranding: {},
};

// shared helper — runs on BOTH server and client
function checkAccess(resource: Resource, cached: { plan: PlanKey; used: number }) {
  if (cached.used >= resource.limit[cached.plan]) {
    throw new Error(resource.upgradeMessage); // triggers upgrade modal
  }
}

// protected procedure — the frequent surface the rule is plugged into by default
protectedProcedure.use(requiredPermission).query(...);
```

- **What it demonstrates**: one typed structure answers "does this plan have access?" identically on server and client; the helper throws (not returns) on failure; `requiredPermission` drags in permissions from the resource entries so the protected procedure enforces the same structure everywhere.

## Worked Example

Feature gates for a project-creation limit:

1. **Define the source of truth**: a `plan keys` constant holds `free`, `starter`, `pro`, `enterprise`, plus `portal` (bypasses internal payments). A hardcoded TypeScript constructor maps over these plans and builds the resource structure — resources include `members`, `invitations`, `organization`, `roles`, `custom branding`. Each resource gets a `name`, `description`, `upgrade message`, and a `limit` whose keys are locked to the same plan names. The entries are also connected to navigation — "settings for the entire app": edit one limit and every page changes.
2. **Cache the user's data**: another source of truth fetches the user's plan and stores it as a cached variable — available on both server and client, and shaped the same as the resource map (plan + consumed usage).
3. **Write the shared helper**: a function takes two parameters — the resource being accessed and the cached data — and throws an error when criteria aren't met ("free tier allows one project; user already has one"). This one function is called from client components (to show the upgrade modal) and from server procedures (to enforce the same rule).
4. **Plug into the frequent surface**: on the server, the check lives inside the `protectedProcedure` as `requiredPermission`, which pulls permissions from `buildPermissionConstants` → the resource entries. The user tries to create a role; the procedure resolves the permission against the exact resource map built above. AI doesn't think about "which permission system?" — it uses the protected procedure and inherits the pattern.
5. **Gate per resource**: each resource carries create/read/update/delete permissions that can be applied to a team member's role or account. To remove a capability (e.g., no "update" on invitations), remove that permission from the resource.

## Key Takeaways

1. Find logic used by multiple layers and globalize it — the client-side version is just JavaScript, so make it a plain helper both sides call.
2. Make the shared helper a pure function of parameters (resource + cached plan/usage); let it throw on failure rather than render UI.
3. Create the data structure with TypeScript so wrong plan names and shapes error everywhere — that's TypeScript lockin, the first layer of the echo signal.
4. Shape the cached plan/usage data exactly like the resource map, so comparison is trivial and structure is consistent on both sides.
5. Don't make AI search for the source of truth — plug the global pattern into the most-used abstraction (the protected procedure) by default.
6. Keep server/client wrappers thin and independent; the rule itself must be tightly coupled to nothing.
7. Only globalize tiny processes (feature gates, limits, permissions) — never forms and components.

## Connects To

- **Ch 4 (pattern recognition)**: globalization produces the exact repeatable patterns that Ch 4's pattern-recognition module trains AI to spot and copy — the resource map is a pattern AI finds while searching the codebase.
- **Ch 6 (registry layer)**: globalization's source of truth becomes a registry — the next module shows how the registry connects to AI so it stops searching and consumes the canonical entries directly.
- **Ch 7 (TypeScript lockin)**: the resource map is built with TypeScript deliberately — Ch 7 expands why maximal TypeScript is the first layer of the echo signal and how microcontext injection teaches AI the right shape.
