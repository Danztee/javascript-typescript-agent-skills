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

Install both skills to the agents detected on your machine:

```bash
npx skills add Danztee/javascript-typescript-agent-skills \
  --skill javascript-engineering \
  --skill typescript-engineering
```

Install both skills for specific agents, for example Codex, Claude Code, and Cursor:

```bash
npx skills add Danztee/javascript-typescript-agent-skills \
  --skill '*' \
  --agent codex \
  --agent claude-code \
  --agent cursor \
  --global
```

Install every skill to every supported agent:

```bash
npx skills add Danztee/javascript-typescript-agent-skills --all
```

Use `--global` for user-wide installation. Omit it to install into the current project. The CLI supports many agents, including Codex, Claude Code, Cursor, OpenCode, Windsurf, GitHub Copilot, Gemini CLI, and others; run `npx skills add --help` for the current list.

The `agents/openai.yaml` files contain optional Codex UI metadata. They do not make these skills Codex-only; other agents use the portable `SKILL.md` files.

## Development

Each skill is a directory containing a required `SKILL.md`. Supporting examples live in that skill's `references/` directory. Maintainer pressure scenarios are in `evals/pressure-scenarios.md`.

Test local discovery before pushing:

```bash
npx skills add . --list
```

## License

MIT — see [LICENSE](LICENSE).
