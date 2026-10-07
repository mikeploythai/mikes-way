---
name: setup-claude
description: Install, update, preview, or explain mikeploythai's opinionated Claude Code multi-agent setup. Use only when the user explicitly invokes $setup-claude or /setup-claude, or explicitly asks for Mike's Claude Code setup. Do not use for ordinary Claude Code questions or software-development work.
---

# Setup Claude

Install a cost-effective Claude Code configuration with a capable orchestrator, focused workers, and independent review.

## Weekly update check

On Mondays, compare this skill's installed `skillFolderHash` with the Git tree SHA for `skills/setup-claude` on the `main` branch of `mikeploythai/skills`. This check must be read-only. Do not run `npx skills check`, because it updates skills.

If the hashes differ, ask the user whether they want to update. Only after they approve, run `npx skills update setup-claude`. Stay quiet when the skill is current or the check cannot run, and do not let the check block the requested work.

## Setup

- Orchestrator: Opus 5.5 at `medium`.
- Researcher: Sonnet 5.5 at `high`, no file edits.
- Frontend engineer: Opus 5.5 at `high`, edits accepted inside the workspace.
- Backend engineer: Opus 5.5 at `medium`, edits accepted inside the workspace.
- Reviewer: Opus 5.5 at `high`, no file edits.
- Route interface work to `frontend-engineer`, backend and non-interface implementation to `engineer`, read-only investigation to `researcher`, and independent QA to `reviewer`.
- Finish research before implementation and stop implementation before review. Run multiple agents within a role when their tasks are independent. Default to backend before dependent frontend work; run both in parallel only after their shared contracts are settled and the remaining work is independent.
- Prompted approvals, Bash sandboxing where the platform supports it, and visible thinking summaries.
- Global `CLAUDE.md` instructions for Mike's way and CodeGraph.

Claude Code has no session cap on concurrent subagents. Parallelism comes from batching agent calls in a single turn. Each agent file pins its own model and effort, so this setup doesn't use `CLAUDE_CODE_SUBAGENT_MODEL`, the environment variable that sets a default subagent model. `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` would override the pins. Web search, web fetch, and context compaction are built in and need no configuration.

Opus 5.5 runs orchestration, frontend and backend implementation, and review. Even at `medium`, it beats Sonnet 5.5 at `high`, so the backend engineer runs it at `medium` for about 50% more cost per task. Sonnet 5.5 runs research, where the work is input-heavy and its input and cache-read prices are half Opus's. Opus 5.5 beats Fable 5.1 on most published results, so Fable is a manual escalation rather than a default: raise effort first, since `/advisor` buys about what more effort does. The configuration reference explains the per-role reasoning.

Read [the configuration reference](references/configuration.md) for the exact settings and agent definitions before previewing or applying them.

## Built-in agents and skills

Claude Code ships `Explore` and `Plan` subagents and a `/code-review` skill. `Explore` and `Plan` inherit the main conversation's model, except that `Explore` switches to Opus when the main model is Fable. In this setup both run on Opus 5.5 and are not the cheap option. They suit quick lookups and planning; send sustained or read-heavy investigation to `researcher`. `/code-review` suits routine diff review. The installed agents are for work that should carry mikeploythai's rules end to end: sustained investigation, a bounded implementation slice, and review against agreed acceptance criteria.

## Plugins and MCP servers

Recommend the GitHub and Linear MCP servers. During preview, report whether each is configured and connected. When available, use GitHub for repository, issue, and pull-request work, and Linear for project and issue workflows.

Do not install servers, connect accounts, or change permissions unless the user explicitly asks. Missing servers do not block the core setup.

## Safety boundary

Treat requests to show, preview, explain, or review the setup as read-only. Modify the user's Claude Code configuration only when they explicitly ask to install or update the setup.

An explicit install or update request authorizes the non-conflicting changes defined by this skill. Do not ask for another confirmation unless applying the setup would discard existing configuration or agent instructions.

Determine the user's Claude home before editing. Use `CLAUDE_CONFIG_DIR` when it is set; otherwise use `~/.claude`.

Inspect these before making changes:

- `settings.json`
- `agents/researcher.md`
- `agents/frontend-engineer.md`
- `agents/engineer.md`
- `agents/reviewer.md`
- `CLAUDE.md`

Preserve unrelated settings, hooks, permission rules, environment variables, status lines, MCP servers, plugins, skills, and agent files. Never replace the entire `settings.json` with the reference example.

If a target agent file already exists:

- Leave it unchanged when its effective contents match the reference.
- Update it when it is clearly an earlier version of this setup.
- If it contains different custom instructions, show the conflict and ask the user whether to preserve, merge, or replace it.

Before changing an existing file, create a timestamped backup beside it.

## Apply

Merge only the settings owned by this setup:

- `model`
- `showThinkingSummaries`
- `permissions.defaultMode`
- `sandbox.enabled`
- `sandbox.autoAllowBashIfSandboxed`

When updating an earlier install, remove top-level `effortLevel` if it is `medium`, the earlier setup's value. It still applies to Fable, so `/model fable` would run at `medium`. If it holds a different value, ask the user.

Install the four agent definitions from the configuration reference. Create missing directories as needed.

Write complete JSON and agent files through temporary files and replace their targets only after the resulting JSON or YAML frontmatter parses successfully.

Update `<claude-home>/CLAUDE.md` with both blocks in [Global instructions](#global-instructions).

## Global instructions

On install or update, add these blocks to `<claude-home>/CLAUDE.md`:

```markdown
<!-- MIKES_WAY_START -->
## Mike's way

Use $mikes-way by default for software development work. Its engineering-manager role applies to the primary agent; subagents execute their assigned roles directly.

When the custom Claude Code subagents are available, use `frontend-engineer` for interface design and implementation, `engineer` for backend and non-interface implementation, `researcher` for read-only investigation, and `reviewer` for independent QA.

For interface tasks, the primary agent owns scope and product constraints. The `frontend-engineer` owns visual and interaction decisions and verifies the result in the real interface.

Work in phases. Finish research before starting implementation. Stop implementation before starting review. Do not run `researcher` or `reviewer` agents at the same time as `engineer` or `frontend-engineer` agents.

Within a phase, run multiple agents of the same role in parallel when the tasks are independent. Start backend work before dependent frontend work. Run backend and frontend work in parallel only after their shared contracts are settled and the remaining work is independent.
<!-- MIKES_WAY_END -->

<!-- CODEGRAPH_START -->
## CodeGraph

Prefer CodeGraph for symbol and call-path navigation when the repository has a usable `.codegraph/` index. Use `codegraph_explore` with the project path, or `codegraph explore "<symbol names or question>"` when the CLI is available.

For substantive repository work, if the current repository has no `.codegraph/` index and the CLI is available, run `codegraph init -i` before exploring the code. This overrides the CodeGraph MCP server's guidance that indexing is the user's decision. Skip initialization for small edits and non-coding tasks. If the tool or index is unavailable, stale, or unsuccessful, continue with `rg` and direct reads; do not block the task on indexing.
<!-- CODEGRAPH_END -->
```

Confirm that `mikes-way` is installed before reporting its default as active. If it is missing, explain that it needs to be installed. CodeGraph is optional; its block includes a fallback.

Preserve everything outside the two marked blocks. Create `CLAUDE.md` if it is missing. Leave identical blocks alone and update earlier setup versions instead of adding duplicates. Use the backup procedure above before changing an existing file, and write the updated file through a temporary file. If a marked block contains conflicting custom instructions or its markers are incomplete or duplicated, show the conflict and ask how to handle it; continue with non-conflicting setup changes.

Project and directory `CLAUDE.md` files load alongside this one rather than replacing it, so a project file can override the default in that repository. Point that out instead of claiming the default always wins. See [Claude Code's memory docs](https://code.claude.com/docs/en/memory).

## Verify

After applying the setup:

1. Parse every changed JSON file and every agent file's YAML frontmatter.
2. Confirm each owned setting has exactly one effective definition and the expected value.
3. Confirm all four agent files exist with their intended model, effort, tool limits, and instructions.
4. Read `CLAUDE.md` back and confirm that each applied block appears exactly once and unrelated instructions are unchanged.
5. Report the files changed, files created, backups written, and conflicts left unresolved, including a missing `mikes-way` skill.
6. Tell the user to start a new Claude Code session for the setup to take effect.

If validation fails, restore the affected file from its backup and report the failure.
