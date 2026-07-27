# refactoring-skills

Agent skills for pragmatic, behavior-preserving refactoring. The skill format (`SKILL.md` with `name`/`description` frontmatter) works with Claude Code, OpenAI Codex CLI, and other runtimes that support agent skills.

## Skills

### [`refactoring`](refactoring/SKILL.md)

One workflow, two modes:

- **DX mode** (default) — scoped developer-experience refactors: repository discovery, friction mapping, 1–3 small high-value scopes, a challenge checkpoint before non-trivial edits, characterization tests, proportional validation, and an explicit "no changes needed" outcome.
- **UI mode** — strict component-first UI refactoring on top of DX mode: visual styling lives inside components, callers never pass cosmetic `style`/`className` overrides, external code controls only semantic props (`variant`, `size`, `tone`, …) and layout composition through wrapper components.

This skill is a merge of two earlier skills, `care-refactoring` and `ui-refactoring`.

## Install

Clone the repo and symlink the skill directory into your agent's skills folder.

Claude Code:

```sh
git clone git@github.com:di-sukharev/refactoring-skills.git
ln -s "$PWD/refactoring-skills/refactoring" ~/.claude/skills/refactoring
```

Codex CLI:

```sh
ln -s "$PWD/refactoring-skills/refactoring" ~/.codex/skills/refactoring
```

## Use

Ask for a refactor and let the skill trigger, or invoke it explicitly:

- `/refactoring improve DX of src/billing without changing behavior`
- `/refactoring clean up app/routes so visual styles live inside components`
