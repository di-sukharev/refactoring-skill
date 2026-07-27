---
name: refactoring
description: Behavior-preserving refactoring workflow covering developer-experience (DX) refactors and strict component-first UI refactors. Use when asked to refactor code, improve DX, make code easier to understand, navigate, test, debug, extend, or safely change without altering behavior, or to clean up pages, screens, and routes so visual styles live inside reusable components while external code controls only layout composition. Includes repository discovery, friction mapping, scoped refactor selection, challenge review, minimal implementation, validation, and a "no changes needed" outcome.
---

# Refactoring

Perform a pragmatic, behavior-preserving refactor. Optimize the codebase for the next developer or agent who must understand, change, test, debug, or extend the touched surface. Do not refactor for aesthetics alone. A valid result is: `No code changes needed`.

Two modes share this workflow:

- **DX mode** (default): reduce developer-experience friction on any surface.
- **UI mode**: everything in DX mode plus the "UI Mode" rules below. Activate it when the target is UI code — pages, screens, routes, components, styling — or when the user asks for UI refactoring.

## Goal

Improve development experience without changing product behavior unless the user explicitly asks for behavior changes.

Prioritize improvements that make future work:

- Easier to understand: clearer ownership, fewer hidden assumptions, less mental stack.
- Easier to navigate: predictable file boundaries, names, entrypoints, and call flow.
- Easier to change: smaller blast radius, less duplicate decision logic, better locality.
- Easier to test: important behavior reachable through proportional, stable tests.
- Easier to debug: clearer error paths, logs/status visibility when already part of the surface, fewer opaque side effects.
- Easier to review: focused diffs, reduced incidental complexity, contracts that are obvious from code.

Do not count code movement as a DX win by itself. The improvement must reduce real friction for a plausible next change.

## Operating Rules

- Follow local agent instructions and repository conventions first.
- Preserve unrelated or user-owned changes. Check `git status --short --branch` before editing.
- Preserve public API, database/schema contracts, response formats, error shapes, permissions, security behavior, and business behavior unless the user explicitly asks otherwise.
- Prefer one small high-value improvement over a broad rewrite.
- Do not impose a new architecture unless the current structure is clearly blocking comprehension, testing, or safe change.
- Do not make non-trivial code changes until the challenge checkpoint ends with `PROCEED_WITH_SCOPE` or `NARROW_SCOPE`.
- Use Quick Mode for small, obvious, local changes where public behavior and contracts are unchanged, the owning layer is clear, and validation is cheap.
- Before changing production code with meaningful behavior, contract, state, persistence, auth, routing, serialization, or cross-layer risk, lock important existing behavior with a focused characterization test when the behavior is not already covered and the repository has a proportional test layer.

## Discovery

Understand how a future developer would work on this surface before proposing changes:

1. Read useful project context: `README.md`, architecture docs, local agent instructions, package scripts, CI config, and relevant tests.
2. Infer the stack, validation workflow, module boundaries, naming conventions, and existing architecture.
3. Inspect current diffs and avoid touching unrelated modified files.
4. Identify the owning layer for the behavior under review.
5. Trace the normal development path: where a change would start, what files must be understood, what tests/checks prove it, and what mistakes are easy to make.

When broad code discovery is needed and subagents are available, delegate repository search to a subagent. Ask for a compact evidence map only: `path:line`, symbol or route name, relevant snippet or signature, and why it matters. Verify critical findings before editing.

## Friction Candidates

Look for high-impact DX friction, especially:

- The next change requires reading too many unrelated files to find the owner.
- Business decisions are embedded in transport/UI handlers, controllers, jobs, middleware, ORM/query code, API clients, serializers, validators, or framework-specific code.
- God files, services, or components mix orchestration, persistence, formatting, permissions, external calls, and state changes.
- Duplicate decisions or validation rules force future edits in multiple places.
- Names, module boundaries, or file placement obscure what owns a behavior.
- Hidden dependencies on time, randomness, env, globals, singletons, network, filesystem, or framework context make tests and local reasoning hard.
- `shared`, `common`, or `utils` areas are dumping grounds with unclear ownership.
- Important behavior is only testable through slow end-to-end paths when a narrower stable boundary should exist.
- Setup, scripts, fixtures, or test data make the intended validation path unclear or unnecessarily slow.
- Error, loading, empty, retry, or recovery paths are hard to inspect when they are part of the touched workflow.
- Code encourages the next feature to copy a confusing or brittle pattern.

UI-specific friction (full rules in "UI Mode"):

- Callers pass `style`, `className`, inline CSS, utility classes, or framework equivalents to change how a component looks.
- A component's visual definition is split between the component and its call sites.
- Wrappers override padding, color, border, background, dimensions, radius, shadow, or typography of their children.
- Local page components duplicate existing design primitives, or a component is an unfinished base whose final look is assembled at call sites.
- Props mirror CSS directly (`padding`, `background`, `borderRadius`, `shadow`) instead of expressing semantic modes.

## Responsibility Boundaries

Use the existing architecture where possible.

- Entry/transport/UI: accept input, read context, call application logic, map result or error to output.
- Application/use-case: own the user or business scenario, permissions, transactions, orchestration, and coordination.
- Domain/rules: hold pure rules, invariants, decisions, state transitions, calculations, and domain errors.
- Infrastructure/adapters: talk to databases, ORM, external APIs, queues, storage, filesystem, SDKs, and platform services.
- Wiring/composition: assemble concrete dependencies.
- Component visuals vs layout: a component owns how it looks; callers and layout wrappers own only how it is placed among other elements.

Heuristics:

- If code answers "what should happen?", keep it in application/domain.
- If code answers "how do we talk to an external system?", keep it in infrastructure.
- If code answers "how do we receive input and return output?", keep it in entry/transport/UI.
- If code answers "how should this be assembled for this runtime?", keep it in wiring/composition.
- If a style answers "how does this component look?", keep it inside the component. If it answers "where does it sit among siblings?", keep it outside in a layout wrapper.

## UI Mode

### Core Principle

Keep every component's visual styling inside the component itself. External callers must not pass cosmetic overrides such as `style`, `className`, CSS objects, utility classes, or framework-specific equivalents to change how a component looks.

Default stance: exposing `className`, `style`, or equivalent styling escape hatches from product components is a bad API in most cases. It usually creates a parallel, undocumented design system at call sites and makes components harder to reason about, test, reuse, and safely change.

External code may control only:

- semantic component modes, such as `variant`, `size`, `tone`, `state`, `disabled`, `colorScheme`, `fullWidth`, and similar domain-level props;
- layout composition through wrapper/layout components that own direction, spacing, alignment, positioning, and grid/flex behavior.

### Bad Practices

Treat these as refactoring targets:

- Passing `style`, `className`, inline CSS, utility classes, or equivalent props to change component appearance.
- Splitting a component's visual definition between the component and its callers.
- Letting wrappers override padding, color, border, background, dimensions, radius, shadow, typography, or other cosmetic properties.
- Using a component as an unfinished base whose final visual design is assembled at the call site.
- Hardcoding the same visual rule in one place while exposing it as a prop in another.
- Adding props that mirror CSS directly, such as `padding`, `background`, `borderRadius`, `shadow`, or `borderColor`, when those props affect visual identity rather than layout composition.

### Allowed Exceptions

Use exceptions sparingly. An exception is acceptable only when the API is intentionally low-level, the styling surface is constrained, and the component does not claim ownership of the final visual identity being overridden.

Acceptable exceptions may include:

- layout primitives such as `Stack`, `Grid`, `Box`, `Spacer`, or page shell components whose job is placement, spacing, alignment, or responsive structure;
- framework or third-party adapters that must pass through host element props for accessibility, portals, animations, measurement, focus management, virtualization, or integration constraints;
- design-system primitives that are explicitly documented as unfinished building blocks rather than product components;
- CSS custom properties or slot props when they expose a narrow semantic surface, not arbitrary visual overrides;
- test-only or instrumentation-only props that do not alter the component's appearance.

When keeping an exception:

- keep the prop name and documentation honest about its scope, such as `containerClassName`, `layoutClassName`, or `slotProps`, instead of a vague styling escape hatch;
- prevent cosmetic overrides on product components such as cards, buttons, banners, fields, nav items, dialogs, and status views;
- prefer semantic props before escape hatches;
- explain the exception in the component or refactor report when it is not obvious from the component category.

### Target Architecture

Build components with clear, semantic APIs. Express visual variants through explicit product-level props such as `variant`, `size`, `tone`, `state`, `colorScheme`, `fullWidth`, `disabled`.

Use layout wrappers for placement concerns only. A wrapper may own direction (row/column), spacing (gap, padding, margin when it describes external layout rhythm), flex/grid behavior, alignment and justification, positioning and responsive placement. Do not use wrapper components to change the visual skin of their children.

In mature codebases, follow the existing component ownership model first. Place extracted components in the repository's established location, such as feature-level component folders, shared UI packages, route-local component directories, or design-system modules. Use a flat `components/` folder only when the repository does not already have a clearer convention.

### UI Workflow

Work sequentially through files. For each file, finish the refactor and validation for that slice before moving to the next one.

1. Inspect the current UI system: nearby components, existing design primitives, layout wrappers, style utilities, and route/page conventions. Check whether a suitable component already exists before creating a new one; prefer extending an existing component with a semantic prop over creating a duplicate.
2. In the target file, identify style props, class overrides, inline styles, utility-class assembly, local page components, and repeated cosmetic fragments. Separate each style into external layout versus internal visual styling.
3. Move visual styling into components. Extract local page/screen/route components into the closest established component location. Keep pages, screens, and routes as composition layers.
4. Keep layout outside via wrapper/layout components with layout-oriented, not cosmetic, APIs.
5. Simplify component APIs: remove harmful `style`/`className`/override/cosmetic pass-through props, remove unused props, merge vague props into clearer semantic props. Avoid CSS-mirroring props for visual design.
6. Preserve behavior: keep interactions, accessibility behavior, loading states, empty states, error states, and responsive behavior intact.

### UI Decision Rules

- In disputed cases, choose the stricter component boundary.
- Do not leave temporary override mechanisms behind.
- Do not keep a local page component just because it is used once.
- Do not create a new component when an existing component can be reused or cleanly extended.
- Do not make the smallest textual diff if it leaves unclear ownership of visual styling.
- Do not weaken the existing design system. Align with established primitives and naming when they exist.
- Treat `className`, `style`, and equivalent props as disallowed on product components unless a narrow exception is clearly justified.
- If a proposed exception would allow arbitrary cosmetic overrides, replace it with semantic variants, slots with constrained styling, or a layout wrapper.
- Stop and re-scope if the refactor requires broad design decisions, changes visible product behavior, or creates a migration larger than the requested surface.

### UI Validation

- Run the smallest meaningful formatter, typecheck, lint, unit test, component test, build, or browser check for the touched surface.
- Prefer at least one user-visible validation signal when practical: local browser inspection, Playwright screenshot, Storybook story, component preview, responsive viewport check, or visual regression test.
- Check important states that the refactor could affect: default, hover/focus/active, disabled, loading, empty, error, success, long text, and narrow viewport.
- Include accessibility-sensitive states when relevant: focus order, focus ring visibility, labels, roles, aria state, keyboard activation, and contrast.
- If validation fails, fix it before moving to the next file.

After each file or coherent file group, report briefly: what was wrong before, what visual styling moved inside components, what stayed outside as layout, which props were added/simplified/removed, which local components moved into established locations, why the result is cleaner, and what validation was run.

Expected result: pages, screens, and routes become clean composition layers; components become predictable, reusable, visually self-contained units; the codebase no longer relies on cosmetic style pass-throughs, scattered hardcoded styling, or call-site overrides to assemble final UI.

## Scope Proposal

Before editing, propose at most 1-3 small high-value scopes. One scope may be a file, endpoint, component, hook, use case, job, module boundary, script, fixture, or business flow.

For each proposed scope, state:

- Concrete friction (DX or UI).
- Plausible next developer task that would benefit.
- Preserved behavior and public contracts.
- Smallest useful change.
- Main risk.
- Focused validation signal.
- Whether a pre-refactor characterization test is needed, already exists, or would be disproportionate.

## Quick Mode

Use Quick Mode only when all of these are true:

- The change is small, local, and obviously behavior-preserving.
- The owning layer and affected files are clear from nearby code.
- Public API, user-visible behavior, contracts, permissions, persistence, and error shapes are unchanged.
- The improvement has concrete DX value beyond aesthetics.
- Validation is cheap through an existing focused test, typecheck, lint, build, or direct runtime check.

Examples:

- Rename a confusing helper or local variable when usages are clear and tooling can verify references.
- Move a test fixture closer to the tests that own it when imports remain straightforward.
- Clarify a script name, README validation command, or test setup note that already reflects existing behavior.
- Simplify a small conditional or extraction when the resulting owner is more obvious and behavior is covered by existing checks.

In Quick Mode, skip the full scope proposal and challenge checkpoint. Still inspect the affected context, preserve behavior, run proportional validation, and report the concrete friction reduced.

Do not use Quick Mode for auth, permissions, persistence, routing, contracts, serialization, cross-layer state, async side effects, generated code, or changes where the primary proof would require broad E2E validation.

## Decision Matrix

Use this matrix to decide whether to proceed, narrow, skip, or split the work:

| Factor | Strong Signal | Weak Signal |
| --- | --- | --- |
| Impact | A plausible next task becomes easier to understand, change, test, debug, or review. | The code merely looks cleaner or more stylistically pleasing. |
| Behavior risk | Contracts, state, auth, persistence, routing, serialization, or cross-layer behavior could be affected. | Rename, local move, fixture cleanup, or isolated implementation simplification. |
| Confidence | The owning layer, callers, and validation path are clear. | Ownership is unclear or important callers are unknown. |
| Validation cost | A focused existing or easy characterization test/check can prove the change. | Only slow, brittle, or broad validation can catch regressions. |
| Scope fit | One flow, file, module boundary, endpoint, component, hook, use case, script, or fixture. | Broad "clean up architecture" or multiple unrelated surfaces. |

Decision rule:

- Proceed when impact is strong, scope is narrow, ownership is clear, and validation is practical.
- Narrow when the issue is real but the proposed scope is broad or validation is unclear.
- Do nothing when the benefit is mostly aesthetic or the next-developer task is not plausible.
- Split into a product task when preserving behavior would require changing user-visible behavior, contracts, data, permissions, or rollout assumptions.

## Calibration Examples

Good scope (DX):

- Friction: a route handler validates input, checks permission, formats the response, and makes a domain decision.
- Better refactor: move the domain decision to an application/domain owner, keep the handler as the request/response boundary, and protect observable response behavior with an existing or new focused test.

Good scope (UI):

- Friction: a page assembles a card's final look from utility classes and inline styles at every call site, and a wrapper overrides the card's padding and background.
- Better refactor: give the card component full ownership of its visual identity, expose a semantic `variant`/`tone` prop for the differing cases, keep the page as layout-only composition, and verify key visual states.

Too broad:

- Friction: a service file is 900 lines.
- Poor refactor: split it into controllers, repositories, interfaces, and factories because the file is large.
- Better refactor: choose one scenario or repeated decision in that file and improve only that ownership boundary.

No changes needed:

- Observation: a component is long, but its sections map cleanly to product states, the next owner is obvious, and validation is clear.
- Result: leave it alone. Size alone is not friction.

Separate product task:

- Discovery: a refactor exposes inconsistent error behavior that users can observe.
- Result: do not "fix it while refactoring" unless the user explicitly accepts the behavior change. Preserve current behavior or split the behavior change into a product task.

## Challenge Checkpoint

If subagents are available, ask a fresh challenge agent to validate the proposed scope before implementation. Use the most capable available model with high or extra-high reasoning. The challenge agent must not write code.

Ask it to answer:

- Is this a real DX or UI-ownership problem or just a stylistic preference?
- What future task becomes easier after this change?
- Is the scope small enough?
- What behavior or public contract could be accidentally broken?
- Is there a simpler improvement?
- Is this overengineering?
- What existing or new characterization test should protect the behavior before refactoring?
- Should we proceed, narrow the scope, do nothing, or turn this into a separate product task?

The main agent and challenge agent may do at most two short rounds of disagreement.

If subagents are not available, perform the same challenge as a separate critical pass yourself and summarize the conclusion.

End with exactly one decision:

- `PROCEED_WITH_SCOPE`
- `NARROW_SCOPE`
- `NO_CHANGES_NEEDED`
- `SEPARATE_PRODUCT_TASK`

Do not change non-trivial code unless the final decision is `PROCEED_WITH_SCOPE` or `NARROW_SCOPE`. Quick Mode changes are the only exception.

## Characterization Tests

For refactors that preserve behavior, prefer a green characterization test before editing production code:

- Use the highest-value existing test layer that is proportional to the scope.
- Test observable behavior, contracts, boundaries, and invariants, not incidental implementation details.
- Keep the test narrow enough to fail for an accidental behavior change in the accepted scope.
- Run the characterization test before production edits and confirm it passes against current behavior.
- If no suitable test layer exists or adding a test would be disproportionate, state that explicitly and use the fastest reliable validation path instead.

Characterization coverage is expected when the refactor touches meaningful behavior, contracts, state, persistence, auth, routing, serialization, async side effects, or cross-layer ownership. For small mechanical changes, do not add tests only to satisfy process; rely on existing focused checks, typecheck, lint, build, or a direct runtime signal when that is proportional.

Do not invent a heavy test harness just to satisfy this step. Do not encode known bugs as desired behavior unless the user explicitly wants the bug preserved.

## Implementation Loop

For each accepted scope:

1. Inspect surrounding code and tests.
2. Identify the concrete friction, preserved behavior, public contracts, risks, and smallest useful change.
3. Add or identify the focused characterization test if important behavior is not already covered, then run it before production edits.
4. Make the minimal connected diff. In UI mode, follow the "UI Workflow" order within the scope.
5. Move decisions to the owner layer; do not patch symptoms in children, leaf helpers, or low-level adapters when the wrong decision is made higher up.
6. Improve names, boundaries, fixtures, scripts, or tests only where they directly reduce the accepted friction.
7. Run the most relevant project checks discovered from scripts, docs, Makefile, or CI. In UI mode, also apply "UI Validation".

Use TDD for non-trivial behavior, contract, auth, persistence, routing, query, validation, or state-transition changes when the repository has a proportional test layer. For pure behavior-preserving movement where tests already cover the path, keep test changes focused.

## What Counts As A Win

Prefer changes with observable developer value:

- A future change has one obvious owner instead of several competing places.
- A repeated rule or state transition is represented once at the right boundary.
- A slow or brittle validation path is replaced by a proportional focused check.
- A confusing setup or test command becomes discoverable through existing scripts/docs.
- A component, hook, service, route, or job no longer mixes unrelated responsibilities.
- An error or failure path becomes easier to reason about without changing the user-facing contract.
- A page, screen, or route becomes a composition layer while a product component owns its full visual identity.

If the only benefit is "this looks cleaner", do not proceed.

## Avoid

- Rewriting from scratch.
- Adding new frameworks or dependencies unless clearly necessary.
- Adding an interface, port, repository, factory, mediator, event bus, or CQRS pattern without current value.
- Splitting simple code into many files just to look clean.
- Creating unrelated cleanup or doc churn.
- Leaving TODOs instead of completing the current refactor.
- Leaving temporary styling override mechanisms behind.
- Adding architecture that makes the code harder to read.
- Moving business scenarios into infrastructure/query/client code.
- Letting infrastructure/framework details leak into pure business logic.
- Creating a new component when an existing one can be reused or cleanly extended.
- Weakening an existing design system with parallel primitives or naming.
- Renaming or moving code when it makes git history harder to follow without enough DX payoff.
- Adding docs that mirror code instead of making setup, validation, ownership, or durable decisions clearer.

## Pre-Final Review

Review the diff as a fresh reviewer:

- Confirm the accepted friction is actually reduced.
- Confirm behavior is preserved.
- Confirm scope stayed small.
- Confirm public contracts are unchanged.
- Confirm characterization coverage was added, already existed, or was intentionally skipped with a reason.
- Confirm tests/checks pass, or clearly explain what could not be run and why.
- Confirm documentation changes are only made when durable architecture, setup, operations, contracts, user flows, or engineering decisions changed.

If the refactor no longer looks worth it, revert only your own changes and report `No useful refactor found`.

## Final Report

Report concisely:

- Overall result: no changes, changes made, or partial validation.
- Mode used: DX, UI, or both.
- Scopes inspected.
- Challenge checkpoint decision.
- Scopes changed.
- Friction reduced and why it matters for the next change.
- Public contracts preserved.
- Characterization coverage added, reused, or intentionally skipped.
- Primary signal status.
- Secondary signal status: tests/checks run.
- Docs status.
- Remaining risks or suggested future cleanup.
- Suggested commit message when changes are ready.
