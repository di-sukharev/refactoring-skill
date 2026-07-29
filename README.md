# refactoring-skill

Agent skills for pragmatic, behavior-preserving refactoring. The skill format (`SKILL.md` with `name`/`description` frontmatter) works with Claude Code, OpenAI Codex CLI, and other runtimes that support agent skills.

## Skills

### [`refactoring`](refactoring/SKILL.md)

A behavior-preserving refactoring workflow for maintainability and developer experience, universal for frontend and backend code. Designed to run at the end of a task to polish the implementation just produced, and it also works pointed at any file, module, or flow.

How it works:

- **Scope** — defaults to the current task's changes; picks 1–3 small high-value scopes; an explicit `No code changes needed` outcome is valid.
- **Safety** — a challenge checkpoint gates non-trivial edits, characterization tests lock existing behavior, public contracts stay intact.
- **DX mode** (default) — clear ownership boundaries, one owner per decision, decoupling over DRY, proportional tests, smaller blast radius, for any stack.
- **UI mode** (gated) — strict component-first styling on top of DX mode: visual styling lives inside components, callers control only semantic props (`variant`, `size`, `tone`, …) and layout composition. Applies when asked for UI refactoring or when the repo's conventions already favor component-owned styling; it does not fight house conventions like utility-class styling.
- **Independent review loop** — after implementation, fresh independent reviewer agents (no orchestrator context) score the result; the loop fixes actionable findings and repeats until validation passes and a reviewer scores at least **9.5/10** or reports no actionable findings. At most five scoring passes; stagnation is reported as an incomplete outcome, never as success.

The loop engineering mirrors the `loop-code-review` skill; the refactoring content merges two earlier skills, `care-refactoring` and `ui-refactoring`.

## Install

Copy the skill directory into your agent's skills folder.

Claude Code:

```sh
git clone https://github.com/di-sukharev/refactoring-skill.git
mkdir -p ~/.claude/skills
cp -R refactoring-skill/refactoring ~/.claude/skills/
```

Codex CLI:

```sh
git clone https://github.com/di-sukharev/refactoring-skill.git
mkdir -p ~/.codex/skills
cp -R refactoring-skill/refactoring ~/.codex/skills/
```

Codex also reads `~/.agents/skills` and, per project, `.agents/skills`; Claude Code also reads a project's `.claude/skills`. Copy into whichever scope you want the skill in, and copy again to update.

Or hand the repository to your agent and ask it to install the skill into your skills directory — the layout is the standard one, so it can place the folder itself.

## Use

At the end of a task, ask the agent to polish the implementation:

- `/refactoring` — refactor the current task's changes until the independent review scores ≥ 9.5/10 or reports no actionable findings.
- `/refactoring src/billing` — refactor an explicit target.
- `/refactoring clean up app/routes so visual styles live inside components` — UI mode.

In Codex, the skill is invoked as `$refactoring`.

## License

[MIT](LICENSE)
