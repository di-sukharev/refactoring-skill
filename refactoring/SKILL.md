---
name: refactoring
description: >-
  Refactors code without changing behavior to make the codebase easier to
  maintain. Use when the user asks to refactor or clean up code, or after a
  large implementation task when the changed code needs clearer structure
  or alignment with project practices.
---

Make code easier to understand, change, test, and debug without changing behavior.
Default to code changed by the current task, or the file, module, or flow the user names.
Follow project rules. Use English with agents and the user's language with the user.

## Scope

Name the concrete maintenance problem and a plausible next task that becomes easier.
Choose the smallest connected change that solves it. No code change is a valid result.
Size and similarity alone do not justify a refactor.
Preserve public APIs, schemas, response and error formats, permissions, security, and business rules.
Report behavior changes as separate work unless the user requests them.
Keep the existing architecture unless it prevents a clear, testable solution.

Ask a fresh subagent without chat history to challenge changes beyond a local mechanical edit and all behavior risks listed below.
Choose a capable model and effort for the risk. Follow the user's model and effort choices.
Give it the repository path, target code, proposed change, constraints, and criteria below.
Ask it to check the code, identify what becomes easier, find a smaller solution, and name risks and behavior tests.
Have the challenger read project rules and report findings with file:line evidence.
It must not edit files, run checks, start agents, or use a browser.
Use its findings to keep, narrow, or reject the scope.

## Refactor

Read the target and affected code. Use code-scout for broad research in unfamiliar code when available.
If loop-code-review is required but unavailable, stop before editing and report the missing dependency.
Before edits that can affect behavior, run focused tests of current contracts. Add missing tests for real risks and require passing results.
Use existing checks for mechanical moves and renames.
Require passing behavior tests for changes to security, stored data, contracts, routing, serialization, shared state, asynchronous effects, or generated code.
This also applies to mechanical edits. If coverage is limited, use the fastest reliable check and state what remains unverified.
Report known bugs separately rather than adding tests that require them.

Keep business decisions with their owner. Keep input, output, and external access at their boundaries.
Separate mixed responsibilities where they make changes difficult.
Make time, randomness, configuration, network, and file access explicit where hidden dependencies block tests.
Merge duplicate rules that must always agree. Keep similar code separate when it changes for different reasons.
Write clear code. Prefer declarative code, clear names, and existing project patterns.
Add dependencies and abstractions when they simplify the current change.
Remove dead code, debug output, temporary files, and obsolete fallbacks. Preserve compatibility that callers or stored data still need.
Update setup, fixtures, scripts, and documentation where the refactor affects them.
Finish each connected change and run affected checks. Fix failures caused by the refactor and report unrelated failures.
If the scope must grow, reassess its value and risks before extending it.

## UI

Follow the project's component and styling conventions.
Separate data loading, business decisions, and presentation. Keep state with its owner and convert server data at the boundary.
Move appearance into components when requested or when the project uses this convention. Keep placement in surrounding layouts.
Use semantic props for variants and states. Keep style overrides in documented layout, adapter, and low-level extension points.
Preserve data flow, interactions, accessibility, states, and responsive behavior.
Report human checks for affected states and screen sizes. Do not use a browser for visual checks.

## Finish

Read the final diff against the stated maintenance benefit. Rework or remove refactor changes that do not make the next task easier.
Use existing checks for local mechanical edits without the risks above. Run loop-code-review for other changes, with the scope, contracts, and risks.
Definition of Done: unchanged behavior, removal of the identified maintenance problem, and changes within scope.
Report incomplete if required review is unavailable or remains open, or work is unfinished.
Summarize what became easier, behavior checks, remaining risks, human checks, and review cost.
Keep unrelated changes intact. Do not commit, push, deploy, or change shared data without authorization.
