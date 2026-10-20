---
name: refactoring
description: >-
  Make code easier to read, change, debug, and test without changing external
  behavior. Use for refactoring, cleanup, or improving code after implementation.
---

## Scope

Refactor the requested code. Otherwise, use active Git changes, including staged, unstaged, and untracked files. If there are none, inspect the project.

For project-wide work, inspect source files over 1000 lines first, then remaining files over 500, 200, and 100. Exclude generated and third-party code. Size determines where to inspect first, not a required file length.

Skip changes without a concrete DX benefit: easier reading, modification, debugging, or testing.
Change architecture when it makes the code clearer and easier to test. Preserve user-visible behavior and external API contracts. Always report discovered bugs separately.

## Good code

- Each function should do one thing. Extract long blocks into functions, modules, or services named after what they do. Prefer declarative code that describes an operation through named steps and data transformations.
- Decoupling is more important than DRY. Unrelated features should be changeable independently, even if they contain similar code. Share a function when changing the same rule would otherwise require updating several copies.
- Endpoints validate requests, check access, call business operations, and format responses. Put business logic in named functions or service methods. Add a use-case function when several services need coordination.
- Extract related state, effects, and event logic from large components into separate custom-hook functions named after their purpose. Keep calculations that do not use hooks in ordinary functions.
- Store each piece of UI state once. Calculate dependent values instead of keeping copies synchronized. Use one `status` for mutually exclusive loading, success, and error states instead of separate booleans.
- Receive required data and clients through arguments, constructors, or framework dependency injection. Avoid reading or changing mutable globals from business functions.
- Keep a component's colors, typography, borders, padding, and visual states inside that component. Allow appearance changes only through semantic props such as `variant` and `size`. Parents position and space components with layouts or wrappers, without passing `className` or `style` to restyle them.

Remove unused code, obsolete fallbacks, and wrappers that forward calls without simplifying usage or hiding external-library details.
Update every place that uses a function or component when changing its internal interface.

## Checks and review

Run relevant existing tests before risky edits and affected checks afterwards.
Add tests only for meaningful behavior risks; prefer simple integration tests against real endpoints and a test database where relevant.
Delete tests that add no real value, including tests written just for the sake of having tests, cosmetic assertions, tests that restate the implementation, and duplicates that catch no additional failures. Preserve useful behavioral coverage.

Finish with `loop-code-review`, passing the refactoring scope and these criteria. Ask it to score the DX improvement and look for missed refactors, regressions, and overengineering. Fix accepted findings and require `passed`.
If relevant checks fail or review cannot pass, report the refactor as incomplete.

Report the DX improvement, checks, review score, discovered bugs, and remaining risks. Leave final visual verification to the user.
Do not commit or push unless requested.
