# Chapter 18: Using the Guardrail in Practice + The White Line Method

## Core Idea
Once the architectural guardrail is wired into the codebase, prompting collapses to plain-English feature work — simple features are one-shotted, complex features are broken into prerequisite-ordered chunks — and the White Line Method (a deliberate false claim) forces AI to search for globalized solutions before writing anything new.


**Portable invariant**: A false-claim prompt forces search-first; loud stubs; the consumable ships before the consumer — port this rule to any stack (see adaptation.md)
## Frameworks Introduced

- **White Line Method** (spoken in the transcript as both "white lie method" and "white line method"): we make a false claim that forces AI to search for something that may or may not exist.
  - When to use: when you believe the feature AI is about to build needs globalized infrastructure (a payment/checkout source of truth, a shared editor, a central email service) that should already exist somewhere — or when you suspect old code is still wired in.
  - How:
    1. Assert that the thing exists: "We already have a source of truth for payments and checkout somewhere in this application. Use that and wire invoices into it."
    2. Say "use that" — never "find it". Your default assumption decides AI's first action: assume nothing exists and it writes new code first; assume something exists and it searches first.
    3. Let the claim fail safely: if you're wrong, AI just tells you it doesn't exist — and may then suggest creating the globalized solution, which is a free architecture review ("maybe that's a way to catch another bug or something you thought you should have done in a different way").
    4. Stack with the Gap Method for multi-phase features: use the claimed source of truth first, then connect the checkout flow into the invoice as a second step.

- **Gap Method**: deliberately create something to fail (or be missing) right now so that the right thing surfaces later.
  - When to use: when one prompt actually intertwines two separable, reusable tasks (creating a team invite + sending the invite email).
  - How: build only the first part and stub the second — "Build an invite creation feature. Do not wire up emails yet. Just create a single source of truth function where the email functionality will go later on. For now, just log the message in the console." Mark the gap so it can't be forgotten: `console.warn`, a TODO note, linting, or a dev-only toast that renders "something went wrong" in production.

- **Feature Prerequisite Ordering** (linear workflow): decompose a complex feature into sub-features that are features of themselves, then number them so every prerequisite is built before the feature that consumes it.
  - When to use: any complex feature — website builder, automation builder, anything that "takes technical expertise".
  - How: list the sub-features (canvas, frames, drag-and-drop, styles), then order them; the non-obvious prerequisite wins — build the booking calendar *before* the canvas, so it's ready when the builder must render it.

- **Global-First Architecture Thinking**: before prompting for any complex feature, think at a global scale — how can this be built so it can be reused across the entire app.
  - When to use: the first step of every complex feature; applied live to the website builder's canvas.
  - How: answer three questions in order — the structure of the state, the state management (store), then mutation (how data moves into the state and how it's updated). Use JSON structures so a rendering engine can convert store data into UI.

## Key Concepts

- **Simple feature**: a feature that is a feature of itself (e.g., "lead lists" — a reusable filter that creates a list droppable into an automation run). With the guardrail in place, one-shot it in plain English; the prompt is the big feature plus its small features (create filter button, move leads, delete list).
- **Complex feature**: a feature containing many features-of-themselves (a website builder contains canvas, drag-and-drop, frames, styles) — break it into a numbered prerequisite list and build iteratively, one chunk per prompt.
- **Default assumption**: the premise a prompt opens with determines whether AI's first action is writing or searching — "nothing exists, build this" → create; "this already exists" → search.
- **Source of truth function**: one router/service where a single concern lives (e.g., all email sending for the organization) so automation builders, subscription flows, and failed-account emails all reuse it.
- **Rendering engine**: the engine that takes Redux/JSON data and converts it into elements on the canvas; it's why the canvas itself only needs free-flow, zoom in/out, and "can it render one data structure" — design and styles come later.
- **Two-website storage system**: a deliberate cap on the Redux store (users build roughly two websites at a time) so in-app memory doesn't blow up from storing many full site structures.
- **Pseudo-code prompting**: for non-technical builders — speak your ideas in pseudo-code (not code, just a way to represent code) and ask AI for a roadmap or architectural blueprint, then piece features together.
- **Prompt shortening (40–60%)**: with guardrails, the permission system, and feature gates already wired in, you no longer re-describe them — you focus only on the feature, so prompts shrink by at least 40–60%.
- **Branch-per-feature**: create a new branch for every feature, even iterations of the current one; seeing which files Claude touched gives you confidence and clarity that it made the right move.
- **Session limits as a positive signal**: hitting your session window on top of the guardrail is not a failure — it means you're using what you paid for and getting 5–6x the output of someone without the layers.

## Mental Models

- Think of hitting session limits as **using what you paid for**: members who wire up the guardrail often drop from the $200/month plan to the $100/month plan — the guardrail frees the window.
- Use the White Line Method when **you believe something should have been globalized** — "there's no downside to this. If you're wrong it'll just tell you; if not it'll ask you to create a globalized solution."
- Think of a complex feature as **a prerequisite graph, not one prompt**: number the nodes so all prerequisites are fulfilled before the actual feature is built; then tackle them one by one.
- Use **"your thoughts up top, AI's thoughts below"** when designing architecture: sketch the structure you'd choose (Redux, the fields you'd store), let AI correct it — it's "a much better developer than you are" for architecture — then combine piece by piece and build iteratively.
- Think of the guardrail as **removing 40–60% of every prompt**: what you used to re-state in every prompt is now the system's job, leaving you to describe only the feature.

## Anti-patterns

- **Letting AI use git**: the first rule of the mindset module — the number of times AI has messed with git and changed a bunch of stuff "is just brutal."
- **Intertwining two tasks in one prompt**: "build the invite system and send the email" entangles invite creation with email delivery and loses the globalized email solution that automation builders, subscription providers, and failed-account recovery could all reuse.
- **Assuming nothing exists by default**: framing a prompt as "add a checkout form to the invoices page with Stripe" makes AI write new code first, missing an existing source of truth — even a highly detailed version of that prompt ("the UI should look similar to other checkout forms") is still incorrect because the structure is wrong.
- **Re-describing the guardrail in every prompt**: re-specifying permissions, feature gates, and guardrails wastes the 40–60% you saved; those layers already live in the system and CLAUDE.md.
- **Building the canvas before the calendar**: the intuitive first feature (canvas) isn't the right first build; the consumable (booking calendar) goes first so it exists when the builder needs to render it.
- **Forgetting your own gaps**: a gap left unmarked silently disappears from context; mark it with `console.warn`, a TODO note, linting, or a dev-only toast so you (or AI) can figure out something's wrong.
- **Trusting "I already removed it"**: the lexical-editor member insisted the old code was gone — the false-claim prompt proved it was still wired in. Verify claims with a forced search, don't accept them.
- **Building from your own architecture only**: "my architecture might be horrible" — don't rely on just your brain; use AI's expertise for the technical architecture part.

## Worked Example

**The White Line Method — lexical editor case.** A community member built a Notion-like custom text editor from scratch, and the instructor suggested migrating to Lexical — explicitly telling them to remove the custom editor *first*. Weeks later the app lagged on every keystroke: the old editor's iteration loop was still re-reading every character and triggering a refresh. On the coaching call the member insisted the old code was gone. The instructor prompted with a false claim:

> "I have two incorrect architectures for the editor that have been mixed — I have the custom flow and I also have the lexical editor, which is what I want. First, verify that my claim is true. Next, completely strip out anything related to the editor so that there are zero references to the editor in my app."

Result: the old stuff was still wired into the Lexical editor. The false claim forced a search that revealed what both the human and AI had missed.

**Checkout form case.** Bad: "Add a checkout form to the invoices page with Stripe." Correct, stacking White Line + Gap:

> "We already have a source of truth for payments and checkout somewhere in this application. Use that and wire invoices into it. Once that's done, connect that source of truth checkout flow into the invoice."

**Mochi Redux bug case.** The website builder had a stale-data bug the instructor suspected wasn't cache — he prompted: "There is a bug in the Redux store. The data is being overwritten when I'm on a different page. I know this exists because it has existed in the past. I need you to deep dive and solve this bug immediately." AI went to the absolute root cause (a singleton model falling back to cached data instead of Redux data) and found additional bugs along the way.

**Live build recap — booking calendar first.** The instructor's most complex AI-only build was his website builder. Ordering the sub-features: (1) booking calendar, (2) canvas, (3) frames, (4) drag-and-drop, (5) styles. Everyone assumes canvas is first; the booking calendar goes first so it's ready to render when the builder needs it. Then, global-first thinking for the canvas: define the data structure first (website: id, org, data; element: id, type, name, and an `elements` array because one element can have multiple children — a calendar is just a special element id mapped in the rendering engine), choose state management (Redux, capped to a two-website system so in-app memory doesn't blow up), then define mutation. After the architecture is locked, building the canvas is simple: free-flow, zoom in/out, and "can it render one of these data structures" — forget the design until the prototype proves itself.

## Key Takeaways

1. Never let AI use git — the first rule of the mindset module, restated as the closing rule of the CLAUDE.md build.
2. Hitting session limits on top of the guardrail is a good sign: you're getting 5–6x output, and members typically downgrade from the $200 to the $100 plan as the window frees up.
3. Prompts shorten 40–60% once guardrails, permissions, and feature gates live in the system — describe only the feature.
4. Simple feature = one-shot plain English (big feature + small features); complex feature = prerequisite-numbered chunks built iteratively.
5. Open a prompt with the false claim "we already have a source of truth for X — use that" to force a search; the default assumption decides whether AI writes or searches, and a wrong claim costs nothing.
6. Stack the Gap Method on top: stub the missing global ("for now, just log the message in the console") so gaps are deliberate, marked, and filled later.
7. Create a new branch for every feature, even iterations — seeing the files Claude touched restores confidence that it made the right move.

## Connects To

- **Ch 17**: CLAUDE.md orchestrator — the "never let AI use git" rule lives in CLAUDE.md, and everything wired there is what makes prompts shrink 40–60%.
- **Ch 1**: mindset module — never let AI use git is rule one; session limits are a multiplier, not a failure.
- **Ch 5**: globalization — the White Line Method is the prompting technique for finding the sources of truth the globalization layer establishes.
