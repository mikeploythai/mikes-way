# Setup Claude

Install, update, preview, or explain mikeploythai's opinionated Claude Code multi-agent setup.

See [SKILL.md](SKILL.md) for the installation workflow and [configuration](references/configuration.md) for the settings and subagent definitions.

The setup adds Mike's way and CodeGraph guidance to the global `CLAUDE.md`. Install [Mike's way](../mikes-way/README.md) for its default instruction to work. CodeGraph is optional. This is the Claude Code counterpart to [Setup Codex](../setup-codex/README.md).

## Installation

With Node.js and npm available, run:

```sh
npx skills add mikeploythai/skills --skill setup-claude
```

Follow the prompts to choose the installation scope. See the [Skills CLI docs](https://skills.sh/docs/cli) for other options.

## Usage

Explicitly ask your agent to preview, install, or update the setup:

```text
/setup-claude Preview Mike's Claude Code setup.
```

The skill preserves unrelated configuration and only changes global Claude Code settings when explicitly asked to install or update the setup.

## What it configures

| Setting | Value |
| --- | --- |
| `model` | `claude-opus-5-5` |
| `showThinkingSummaries` | `true` |
| `permissions.defaultMode` | `default` |
| `sandbox.enabled` | `true` |
| `sandbox.autoAllowBashIfSandboxed` | `true` |

Plus four subagents in `<claude-home>/agents/`: a read-only `researcher` on Sonnet 5.5 at `high`, a workspace-writing `frontend-engineer` on Opus 5.5 at `high`, a workspace-writing backend `engineer` on Opus 5.5 at `medium`, and a read-only `reviewer` on Opus 5.5 at `high`. The global `CLAUDE.md` block routes work to those roles and runs research, implementation, and review as separate phases. The orchestrator gets `medium` from Opus 5.5's own default, so no `effortLevel` is set. Updating an earlier install removes its `effortLevel` of `medium`.

Opus 5.5 takes the orchestrator, frontend engineer, backend engineer, and reviewer seats. Even at `medium`, it beats Sonnet 5.5 at `high` on independent benchmarks, and the backend engineer costs about 50% more per task than Sonnet 5.5, not double. Sonnet 5.5 takes the researcher seat, where the work is input-heavy and its input and cache-read prices are half Opus's. Fable 5.1 is a manual escalation: raise effort first, since `/advisor` buys about what more effort does, or use `/model fable` for a full switch.

Some Codex settings have no counterpart here. Claude Code has no session cap on concurrent subagents. Each agent file pins its own model, so the setup doesn't use `CLAUDE_CODE_SUBAGENT_MODEL`, and `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` would override the pins. Web search, web fetch, and context compaction are built in.
