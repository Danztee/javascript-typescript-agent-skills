# JavaScript & TypeScript Agent Skills

Reusable engineering guidance for coding agents working in JavaScript and TypeScript repositories.

The skills favor project consistency, correct runtime behavior, and proportionate validation. They deliberately avoid adding defensive checks, dependencies, abstractions, or configuration changes without a real reason.

## Skills

- `javascript-engineering` — JavaScript implementation, debugging, refactoring, and review across browser and Node.js projects.
- `typescript-engineering` — TypeScript implementation, type design, runtime boundaries, compiler behavior, and review.

## Install

List the available skills:

```bash
npx skills add Danztee/javascript-typescript-agent-skills --list
```

Install both into Codex:

```bash
npx skills add Danztee/javascript-typescript-agent-skills \
  --skill javascript-engineering \
  --skill typescript-engineering \
  --agent codex \
  --global
```

Install one skill:

```bash
npx skills add Danztee/javascript-typescript-agent-skills \
  --skill typescript-engineering \
  --agent codex \
  --global
```

## Development

Each skill is a directory containing a required `SKILL.md`. Supporting examples live in that skill's `references/` directory. Maintainer pressure scenarios are in `evals/pressure-scenarios.md`.

Test local discovery before pushing:

```bash
npx skills add . --list
```

Before making the repository public, add a license that matches how you want others to reuse the skills.
