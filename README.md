# Skills

My skills for building software with coding agents.

Each skill lives in its own folder under `skills/`, with its own instructions and references.

## Installation

With Node.js and npm available, run:

```sh
npx skills add mikeploythai/skills
```

Follow the prompts to choose the skills, agent, and installation scope. See the [Skills CLI docs](https://skills.sh/docs/cli) for other options.

For a manual install, copy the folders you want from `skills/` into your agent's skill directory. Keep each folder's instructions and references together. Use the skill directory documented by your agent.

## Available skills

### [Mike's way](skills/mikes-way/README.md)

My rules and preferences for how agents work: delegate, reuse what's already there, verify changes, and keep contributor docs useful. Includes model and API design, interface guidance, [stack preferences](skills/mikes-way/references/stack-preferences.md), [frontend toolchain guidance](skills/mikes-way/references/frontend-toolchain.md), and code defaults that respect the repo's existing conventions.

### [Setup Claude](skills/setup-claude/README.md)

Install, update, preview, or explain mikeploythai's opinionated Claude Code multi-agent setup. It uses Opus 5.5 for orchestration, frontend implementation, and review, Sonnet 5.5 for backend implementation and research, and preserves unrelated Claude Code configuration.

### [Setup Codex](skills/setup-codex/README.md)

Install, update, preview, or explain mikeploythai's opinionated Codex multi-agent setup. It uses GPT-6 Sol for orchestration, backend implementation, and review, GPT-6 Astra for frontend work, GPT-6 Luna for research, and preserves unrelated Codex configuration.

## Optional companions

These aren't required, but heavily encouraged to install:

- [Ponytail](https://github.com/DietrichGebert/ponytail) for simpler code and less unnecessary building.
- [Cloudflare's Kumo design skill](https://github.com/cloudflare/kumo/tree/main/skills/kumo-design) for interface guidance. Using it doesn't mean adopting the Kumo library.
- [Jakub Krehel's skills](https://github.com/jakubkrehel/skills) for focused interface design and review.
- [Emil Kowalski's skills](https://github.com/emilkowalski/skills) for interface polish, component design, and animation decisions.
- [shadcn's skill](https://ui.shadcn.com/docs/skills) for projects that use shadcn/ui.
- [CodeGraph](https://github.com/colbymchenry/codegraph) for finding symbols and tracing call paths when a usable index is available. Otherwise the agent uses `rg` and reads the files.

Missing companions shouldn't hold up tasks. The agent can recommend one when it would help once, but won't install it without approval.

See [design task routing](skills/mikes-way/references/interface-design.md#choosing-skills-by-task) for which skills handle prototyping, implementation, reviews, motion planning, testing, and terminology.

## Credits

The bundled references include Lauren Tan's Unslop and a standalone adaptation of Blast radius. See [upstream notices](skills/mikes-way/references/upstream-notices.md) for sources, changes, and license text.
