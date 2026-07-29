---
name: refactoring
description: Behavior-preserving refactoring workflow for maintainability and developer experience across frontend and backend code, run as an iterative loop with fresh independent reviewer agents until validation passes and the result scores at least 9.5/10 or has no actionable findings. Use when the user invokes `/refactoring`, asks to refactor or clean up code, improve DX or maintainability, polish a task's implementation before finishing, or enforce component-owned styling in UI code. Includes task-scoped discovery, friction mapping, challenge review, characterization tests, minimal implementation, an independent review loop, and a "no changes needed" outcome.
---

# Refactoring

Perform a pragmatic, behavior-preserving refactor for maintainability and developer experience, frontend or backend. Default use: polish the current task's changes before finishing; an explicit target (file, module, flow) works too. Prove the result with a loop of fresh independent reviewers until it scores at least 9.5/10 or has no actionable findings. `No code changes needed` is a valid result.

Two modes: **DX** (default) and **UI** — DX plus the "UI Mode" ruleset below, when its gate matches.

## Worth Doing?

A refactor is worth doing only when a plausible next developer task becomes easier: to understand (clear ownership, fewer hidden assumptions), navigate (predictable boundaries, names, entrypoints), change (small blast radius, each decision in its owning layer), test (behavior reachable through proportional stable tests), debug (clear error paths, fewer opaque side effects), or review (focused diffs, obvious contracts). Moving code around or making it look cleaner is not value; when that is all there is, report `No code changes needed`.

## Rules

- Follow local agent instructions and repository conventions first.
- Preserve behavior and contracts: public API, schemas, response formats, error shapes, permissions, security, business behavior. Behavior changes happen only when the user explicitly asks — otherwise split them off as a separate product task.
- Keep the worktree safe: check `git status --short --branch` first; leave unrelated changes alone; stage, commit, stash, reset, or push only when the user asks.
- Prefer one small high-value improvement over a broad rewrite; keep the existing architecture unless it clearly blocks comprehension, testing, or safe change.
- Prefer decoupling over DRY: unify duplicated code only when its copies must always agree; when they change for different reasons, leave them duplicated rather than binding them to a shared abstraction.
- Non-trivial edits start only after the challenge checkpoint approves, and finish only with the review-loop acceptance signal or an honest incomplete report.
- The review score moves only through code changes or evidence: fix findings, or record an evidence-backed rejection.

## Scope

Default target: the code produced or modified during the current task. Identify task-owned files and hunks from the task history, not from `git status` alone; when ownership is ambiguous, ask the user. Edits may extend into related code when the accepted scope requires it — for example, moving a decision to its owning layer — with minimal blast radius and a recorded reason.

## Discovery

Read enough to work like the next developer: project docs and agent instructions, package scripts, CI config, relevant tests; infer the stack, module boundaries, naming, and validation workflow. In end-of-task mode, stay on the changed surface and its immediate neighbors. Delegate broad search to a subagent when available and ask for a compact evidence map (`path:line`, symbol, why it matters); verify critical findings before editing.

## Friction To Hunt

- The next change needs too many unrelated files, or has several competing owners.
- Business decisions sit in transport or infrastructure code: handlers, controllers, jobs, middleware, ORM/queries, API clients, serializers.
- One file mixes orchestration, persistence, formatting, permissions, external calls, and state changes.
- The same decision or validation rule is duplicated across places that must always agree, so one future edit means several places.
- Hidden dependencies — time, randomness, env, globals, singletons, network, filesystem — block local reasoning and tests.
- Important behavior is only provable through slow end-to-end paths; setup, scripts, or fixtures obscure the intended validation path.
- Error, loading, empty, and retry paths are hard to inspect where they are part of the touched flow.
- The code teaches the next feature to copy a confusing pattern.
- Frontend: components or hooks mix data fetching, business decisions, and presentation; view state lives at the wrong level; many components bind to raw server shapes instead of a stable view boundary; a component's look is assembled by its callers (see the UI Mode gate).

## Ownership Boundaries

Keep each answer with its owner, following the existing architecture: "what should happen" → application/domain; "how do we talk to an external system" → infrastructure/adapters; "how do we receive input and return output" → entry/transport/UI; "how is this assembled for this runtime" → wiring/composition; "how does this component look" → inside the component; "where does it sit among siblings" → the layout wrapper outside.

## Propose 1–3 Scopes

For each scope state: the concrete friction, the next task that becomes easier, the preserved contracts, the smallest useful change, the main risk, the focused validation signal, and whether a characterization test is needed, already exists, or would be disproportionate.

Then decide: **proceed** (strong impact, narrow scope, clear ownership, practical validation) · **narrow** (real issue, too-broad scope or unclear validation) · **nothing** (value is aesthetic or the next task is implausible) · **separate product task** (improvement requires user-visible change).

Calibration:

- A handler validates, checks permission, formats, and makes a domain decision → move only the decision to its owner; the handler stays the request/response boundary.
- A 900-line service → pick one scenario or repeated decision inside it; "split it into controllers, repositories, and factories because it is big" is not a scope.
- A long component whose sections map cleanly to product states with an obvious owner → leave it; size alone is not friction.
- Two flows that look alike but change for different reasons → keep them apart; resemblance alone is not friction.
- A page assembles a card's look from utility classes while a wrapper overrides its padding → the card owns its look behind a `variant`, the page keeps layout only (when the UI gate matches).
- The refactor would expose a user-observable inconsistency → preserve current behavior and split a product task.

## Quick Mode

Small, local, obviously behavior-preserving changes with a clear owner and cheap validation — a tooling-verified rename, a fixture moved next to its tests, a simpler conditional covered by existing checks — skip the proposal, checkpoint, and review loop: inspect context, change, validate, report the friction reduced. Auth, permissions, persistence, routing, contracts, serialization, cross-layer state, async side effects, generated code, and merging duplicated code into a shared abstraction always take the full pipeline.

## Challenge Checkpoint

Before implementation, have a fresh challenge agent (most capable available model, high reasoning, read-only) attack the proposal: real problem or stylistic preference? which future task gets easier? is this the smallest scope? what could break? is there a simpler improvement? what test locks the behavior first? At most two rounds of disagreement; without subagents, run the same attack yourself as a separate critical pass. End with exactly one of `PROCEED_WITH_SCOPE` / `NARROW_SCOPE` / `NO_CHANGES_NEEDED` / `SEPARATE_PRODUCT_TASK`; implement only on the first two.

## Characterization Tests

Before editing production code with meaningful behavior, lock current behavior with a focused test, green against the code as it is, at the most proportional existing layer: observable behavior and contracts, not implementation details; narrow enough to fail on an accidental behavior change in the accepted scope. Expected when the refactor touches contracts, state, persistence, auth, routing, serialization, async effects, or cross-layer ownership. For mechanical changes, existing checks (typecheck, lint, build, focused tests) are proportional — add tests for value, not for process. If no proportional layer exists, say so and use the fastest reliable signal instead. A characterization test locks intended behavior; encode a known bug as expected only when the user explicitly wants the bug preserved.

## Implement

For each accepted scope: make the minimal connected diff; place each decision in its owning layer — fix it where it is made, not in the children; improve names, boundaries, fixtures, or scripts only where they serve the accepted friction; add dependencies, interfaces, patterns, or shared abstractions only when they pay for themselves in this change; finish what you start, with churn and leftover TODOs kept out of the diff. Validate each slice with the most relevant project checks (from scripts, docs, Makefile, CI); use TDD where behavior-adjacent changes have a proportional test layer.

## Independent Review Loop

Prove every non-trivial refactor with fresh independent reviewers. Quick Mode and `No code changes needed` are the only exits that skip it.

Independence: each scoring pass is a new agent in an isolated context — same filesystem and project instructions, none of your conversation, reasoning, or prior review output. The reviewer inspects `git status`, diffs, and validation output itself. Clarification from the same reviewer is not a new scoring pass.

Protocol:

1. Validate the touched surface (smallest meaningful tests, typecheck, lint, build); get green before every scoring pass; record commands and results.
2. Spawn one fresh reviewer (most capable available model, high reasoning) with the template below.
3. Treat output as findings, not orders: fix actionable in-scope findings; record evidence-backed rejections of wrong or stale ones; log out-of-scope findings as future cleanup for the report — expanding scope takes user approval. A score below 9.5 with explicitly no actionable findings gets one question about what blocks 9.5: an actionable answer is a finding, anything else means accept the signal. A malformed review (no score, vague concerns) gets one repair request, then a fresh reviewer.
4. After meaningful fixes or rejections, loop with a new fresh reviewer; once a reviewer reports an actionable finding, its score no longer counts for acceptance.
5. Accept when validation is green, the latest reviewer has no unresolved actionable findings, and it scores at least 9.5/10 or explicitly reports none.
6. Default limit: five scoring passes; two consecutive passes producing no code changes and only repeated, rejected, or preference-level comments is stagnation. Hitting either without acceptance is an incomplete outcome — report the exact blocker without lowering the bar. Persist further only when the user asked for persistence.
7. Without subagents, run the scoring pass yourself on a clean re-read with the same anchors and limits, and disclose in the report that review was not independent.

Anchors: **10** — behavior demonstrably preserved, targeted friction removed, easy to inherit, complete green evidence. **9.5** — no actionable findings remain, only optional nits. **Below 9.5** — an actionable finding remains: behavior or contract risk, friction not actually reduced, unjustified new complexity, scope creep, or missing/failing validation evidence. The score summarizes findings; it never overrides them or red validation.

Reviewer prompt template (fill in the scope details):

```text
Review this behavior-preserving refactor independently. You have no parent conversation history; derive findings only from repository state and command output you inspect yourself. Stay read-only: no edits, staging, commits, or pushes.

Repository: <repository root path>
Refactor intent: <friction targeted; behavior claimed preserved; whether UI mode applies>
Scope: <files/hunks owned by the refactor; unrelated active changes to ignore>
Validation evidence: <commands run and their results; rerun what you need>

First reconstruct the change: what the refactored code owns, its important flow, what changed structurally, what observable behavior must stay unchanged. If a part resists explanation after reasonable context, name the exact symbol and the future change this ambiguity makes risky.

Hunt in severity order: (1) behavior or contract changes introduced — API, response shapes, error paths, permissions, persistence, ordering, UI states; (2) whether the claimed friction actually got easier, with each decision in its owning layer, or code merely moved; (3) new indirection, patterns, or coupling without current value — including shared abstractions binding code that changes for different reasons; (4) edits beyond the stated scope; (5) test evidence — proportional characterization coverage that would fail on a plausible regression, with no mocks faking the outcome under test; (6) if UI mode applies — remaining call-site cosmetic overrides, visual ownership, semantic props, layout-only wrappers, key visual and accessibility states.

Base findings on repository evidence; keep the existing architecture as the baseline rather than imposing a new one, demanding reuse for uniformity, or pushing to merge look-alike code that changes for different reasons; report preferences as optional nits, not violations. Actionable findings first, with file/line and concrete impact; say clearly if there are none; then a brief understanding summary. Score 10 = friction removed, behavior preserved, complete evidence; 9.5 = no actionable findings, only optional nits; below 9.5 = an actionable finding remains or evidence is missing/failing. End with the numeric score and the concrete issue blocking 9.5, if any.
```

## UI Mode

Gate: the user asks for UI or styling refactoring, or the accepted scope is UI code in a repository whose conventions already favor component-owned styling. Where the house convention is call-site styling — for example, Tailwind utility classes composed in pages — follow the convention, and at most raise the migration as a proposal with its cost stated honestly.

Principle: a component owns its look — surface, padding, radius, typography, color, borders, shadows, sizing, and state styling — expressed once, inside it. Callers control exactly two things: semantic props (`variant`, `size`, `tone`, `state`, `colorScheme`, `fullWidth`, `disabled`) and placement through layout wrappers (direction, gap/margin as external rhythm, flex/grid behavior, alignment, positioning). A style that describes the look belongs inside the component; a style that describes placement belongs in the wrapper.

Styling escape hatches (`className`, `style`, cosmetic pass-throughs, CSS-mirroring props like `padding` or `borderRadius`) stay off product components — cards, buttons, banners, fields, nav items, dialogs, status views. They are legitimate on: layout primitives (`Stack`, `Grid`, `Box`, page shells); adapters that must pass host-element props for accessibility, portals, focus, measurement, or virtualization; documented low-level design-system building blocks; narrow semantic slots or CSS custom properties; test-only instrumentation. Name such props honestly (`containerClassName`, `layoutClassName`, `slotProps`) and note the justification where it is not obvious from the category.

Workflow, per accepted scope, file by file — finish and validate each slice before the next:

1. Learn the existing UI system first; extend an existing component with a semantic prop when the new use is a variation of that component and changes for the same reasons; otherwise give the new use its own component.
2. Classify every style in the file: internal look vs external placement.
3. Move looks into components; extract local page components into the repository's established component location (a flat `components/` only when no clearer convention exists); keep pages, screens, and routes as composition layers.
4. Keep placement in layout wrappers with layout-oriented APIs.
5. Simplify component APIs: semantic props in; unused, vague, and cosmetic pass-through props out.
6. Preserve behavior: data flow, permissions, routing, persistence, and business logic change only when required to preserve the UI; interactions, accessibility, loading/empty/error states, and responsiveness stay intact.

In disputes, choose the stricter component boundary; align with the existing design system's primitives and naming; re-scope when the work would need broad design decisions or outgrow the requested surface.

Validation per slice: the smallest meaningful check plus, when practical, one user-visible signal (browser check, Playwright screenshot, Storybook story, viewport check); cover the states the refactor could affect — default, hover/focus/active, disabled, loading, empty, error, success, long text, narrow viewport — and accessibility basics (focus order and visibility, labels/roles/aria, keyboard activation, contrast). Note per slice what moved inside components and what stayed as layout.

## Finish

Re-read the final diff as a stranger and confirm: the accepted friction is actually reduced; behavior and contracts are intact; scope stayed small; characterization coverage exists or its absence is justified; checks pass; docs changed only where durable setup, contracts, or decisions changed; the loop ended with acceptance or an honestly recorded blocker. If the refactor stopped being worth it, revert your own changes and report `No useful refactor found`.

Report: result (no changes / changes made / incomplete); mode (DX, or DX plus UI); scopes inspected and changed; checkpoint decision; friction reduced and why it matters for the next change; public contracts preserved; characterization coverage (added, reused, or intentionally skipped, with the reason); loop outcome (scoring passes, final score, acceptance signal or exact blocker); findings rejected, with evidence; out-of-scope findings as suggested future cleanup; validation commands and results; remaining risks; suggested commit message.
