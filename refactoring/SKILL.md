---
name: refactoring
description: >-
  Refactors code without a behavior change, so that the next task is easier,
  and checks the result with loop-code-review. Use when the user asks to
  refactor or clean up code.
---

## Goal

A refactor is worth doing only if it makes code easier to understand, find, change, test, debug, or review in a plausible next task.
If no rule fits a case, choose the smallest change that makes the next task easier. `No code changes needed` is a valid result.

## Rules

- Keep behavior and contracts: public APIs, schemas, response formats, error shapes, permissions, security, and business rules. Report a necessary behavior change as a separate product task, unless the user asks for it.
- Keep the existing architecture, unless it blocks understanding, tests, or safe changes.
- Prefer decoupling to DRY. Merge copies of code only if they must always agree.
- If `loop-code-review` is not available, stop. Ask the user to install it from https://github.com/di-sukharev/loop-code-review-skill.
- Leave unrelated changes as they are. Do not stash, reset, or check out files. Do not commit or push, unless the user asks.
- Do not use a browser for visual checks.
- Follow project rules. Write to agents in English and to the user in the user's language.

## Agents

- Claude Code: `subagent_type: general-purpose`, `model: opus`.
- Codex: Sol, `reasoning_effort: high`, `fork_turns: "none"`. Use the longest `wait_agent` timeout.
- The user's model and effort choices override these settings.
- Send the challenger the Challenger brief section verbatim, then the task context. In UI mode, the brief also includes the UI mode section. Do not poll the challenger.
- As a Claude Code subagent, start agents in the foreground. Do not continue an agent, because its replies do not reach you. Start a new one with the same settings. Give it the brief, the task context, the last report, and one request only.
- If an agent cannot start, stop and report.

## Steps

A change is sensitive if it touches auth, permissions, stored data, routing, contracts, serialization, shared state, asynchronous effects, or generated code. A change that merges copied code or moves ownership across layers is also sensitive.
A change is quick if it is local and not sensitive, has a clear owner and cheap checks, and clearly keeps behavior. An example is a rename that a tool makes. For a quick change, skip steps 3, 4, and 7.

1. Target. By default, the target is the code that the current task changed. Find it from the task history, not only from `git status`. If you are not sure what belongs to the task, ask the user. The user can also name a file, module, or flow.
2. Friction. Read the project instructions, scripts, related tests, the target, and its direct neighbors. In unfamiliar code, use `code-scout` if it is available. Look for these problems:
   - the next change needs many unrelated files, or the code has competing owners;
   - business decisions sit in handlers, controllers, jobs, middleware, queries, API clients, or serializers;
   - one file mixes orchestration, storage, formatting, permissions, external calls, and state changes;
   - a rule has copies in places that must always agree;
   - hidden dependencies, such as time, random values, environment, globals, network, or files, block local reasoning and tests;
   - only slow end-to-end paths can prove important behavior, or setup hides how to run the checks;
   - error, loading, empty, and retry paths are hard to find in the touched flow;
   - the code teaches the next feature a confusing pattern;
   - UI components mix data loading, business decisions, and presentation, keep state at the wrong level, or use raw server data;
   - callers build the look of a component.
3. Proposal. Propose one to three scopes. Use the rules in the Challenger brief. For each scope, name the friction and the next task that becomes easier. Also name the contracts to keep, the smallest useful change, the main risk, and the checks.
4. Challenge. Start a fresh challenger. The task context is the repository path, the target, the proposal, and the user's constraints. Continue only with the scopes that get `PROCEED` or `NARROW`. For `NARROW`, use the narrower scope. If a scope that the user asked for gets `SKIP`, ask the user. You can dispute a decision once, with evidence. The next decision is final.
5. Tests. Before you change code with meaningful behavior, make sure that focused tests cover its current behavior. Add only the missing tests. Test observable behavior and contracts at the cheapest existing layer. Each test must fail if the refactor changes this behavior. Run the tests before your edits. They must pass. Record the command and the result.
   A sensitive change always needs these tests. For a mechanical change, such as a move or a rename, the existing checks are enough. If no suitable test layer exists, say so and use the fastest reliable check. Do not write a test that expects a known bug. Report the bug as a separate product task.
6. Implement. For each accepted scope, make the smallest connected change, and finish it.
   - Put each decision in the layer that owns it. Change a decision where it is made, not where it is used.
   - Change names, boundaries, fixtures, or scripts only where they serve the accepted friction.
   - Change docs only where setup, contracts, or decisions change.
   - Add a dependency, interface, pattern, or shared abstraction only if it pays for itself in this change.
   - You can change code outside the target if a scope needs it. Report why.
   - Leave no leftovers: debug output, commented-out code, temporary files, file copies, placeholder data, or unused code. Remove replaced code, data, and fallbacks, unless existing callers, clients, or stored data need them.
   - Run the relevant project checks after each scope. Fix the failures that the refactor causes, and report other failures.
   - If a scope must grow, or a quick change is sensitive, go back to step 3.

   Then read the diff as a new developer. Undo each scope that does not make the next task easier. If no scope is left, report `No code changes needed`.
7. Review. Run `loop-code-review` on the refactor changes. Give reviewers each accepted scope with its friction, contracts, and risk, but not the rest of the proposal. List other uncommitted changes in the files that you changed. In UI mode, add the UI mode section to the task context. The Definition of Done (DoD) is:
   - behavior and contracts do not change;
   - the friction of each accepted scope is gone;
   - the change stays in the accepted scopes.

## Finish

The result is `No code changes needed`, changes, or incomplete. It is incomplete if the review status is open or a scope is unfinished.
Report the result, the removed friction, the next task that becomes easier, the kept contracts, and the behavior tests.
Also report the challenge decisions, rejected findings with reasons, checks, human checks, unresolved issues, and other problems for later.
Also report the cost: agents, rounds, models, efforts, and agent tokens if known.

## UI mode

Use UI mode if the user asks to move styles into components. Also use it for UI code in a project that keeps styles inside components. Otherwise, follow the project's styling convention. You can propose a migration with its cost.

A component owns its look: surface, padding, radius, typography, color, borders, shadows, size, and state styles. Callers control only semantic props, such as `variant`, `size`, `tone`, or `disabled`, and placement through layout wrappers. Placement is direction, gap, outer margin, alignment, flex or grid behavior, and position.

Product components, such as cards, buttons, fields, and dialogs, take no style overrides, such as `className`, `style`, or `padding`. Overrides are correct only on:

- layout primitives, such as `Stack`, `Grid`, `Box`, and page shells;
- adapters that pass host props for accessibility, portals, focus, measurement, or virtualization;
- documented low-level design system parts, narrow semantic slots, and CSS custom properties;
- test instrumentation.

Give such props honest names, such as `containerClassName` or `slotProps`.

When you implement, change one file at a time, and check it before the next. Follow these steps:

1. Learn the existing UI system. If a new use is a variant of a component and changes for the same reasons, add a semantic prop. Otherwise, make a new component where the project keeps components.
2. Sort each style into look or placement. Move each look into its component, and keep placement in layout wrappers. Pages, screens, and routes only compose.
3. Remove unused, vague, and cosmetic props.
4. Keep the data flow, interactions, accessibility, states, and responsive behavior.

If you are not sure, choose the stricter component boundary. Use the names and primitives of the existing design system. If the work needs broad design decisions, narrow the scope.
Report human checks for the states that the change can affect. Examples are hover, focus, disabled, loading, empty, error, long text, and narrow screens. Also report focus order, labels, keyboard use, and contrast.

## Challenger brief

You are the challenger. Find out if each proposed scope makes a plausible next task easier. Assume that it does not, until the evidence shows that it does.

### Work

- Read the project instructions and the code that the proposal names. Search before you read. Read only the ranges that you need.
- For each scope, answer these questions:
  - Is the friction real, or only a preference?
  - Which plausible next task becomes easier?
  - Is this the smallest useful scope?
  - What can break?
  - Is a simpler change enough?
  - Which test locks the behavior first?

### Rules

- Size alone is not friction. Similarity alone is not friction. Code that only moves or looks cleaner is not a result.
- Keep each decision with its owner, in the existing architecture:
  - what must happen: the domain or application layer;
  - how to talk to an external system: an adapter;
  - how to receive input and return output: the entry layer or UI;
  - how to assemble parts for a runtime: the composition layer;
  - how a component looks: the component;
  - where a component sits: the layout around it.
- Examples:
  - A handler validates input, checks permissions, formats output, and makes a domain decision. Move only the decision to its owner. The handler stays the request and response boundary.
  - A service has 900 lines. Choose one scenario or one repeated decision in it. "Split it into controllers, repositories, and factories because it is big" is not a scope.
  - A long component has sections that match product states, and the owner is clear. Leave it.
  - Two flows look alike but change for different reasons. Keep them apart.
  - In UI mode, a page builds the look of a card from utility classes, and a wrapper changes its padding. The card owns its look behind a `variant`. The page keeps only the layout.
  - A refactor would expose an inconsistency that users can see. Keep the current behavior, and propose a separate product task.

### Limits

Do not edit files, run checks, start agents, or use a browser.

### Report

Report briefly and in English, with `file:line` references. Do not paste code or full logs. For each scope, give the main reasons and one decision:

- `PROCEED`: the impact is strong, the scope is narrow, the owner is clear, and the checks are practical.
- `NARROW`: the problem is real, but the scope is too broad or the checks are unclear. Name the narrower scope.
- `SKIP`: the value is only style, or the next task is not plausible.
- `PRODUCT_TASK`: the improvement needs a change that users can see.
