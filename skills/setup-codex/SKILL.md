---
name: setup-codex
description: Install, update, preview, or explain mikeploythai's opinionated Codex multi-agent setup. Use only when the user explicitly invokes $setup-codex or /setup-codex, or explicitly asks for Mike's Codex setup. Do not use for ordinary Codex questions or software-development work.
---

# Setup Codex

Install a cost-effective Codex configuration with a capable orchestrator, focused workers, and independent review.

## Weekly update check

On Mondays, compare this skill's installed `skillFolderHash` with the Git tree SHA for `skills/setup-codex` on the `main` branch of `mikeploythai/skills`. This check must be read-only. Do not run `npx skills check`, because it updates skills.

If the hashes differ, ask the user whether they want to update. Only after they approve, run `npx skills update setup-codex`. Stay quiet when the skill is current or the check cannot run, and do not let the check block the requested work.

## Setup

- Orchestrator: `gpt-6.1-sol` at `xhigh`.
- Researcher: `gpt-6-luna` at `high`, read-only.
- Frontend engineer: `gpt-6-astra` at `high`, workspace-write.
- Backend engineer: `gpt-6.1-sol` at `xhigh`, workspace-write.
- Reviewer: `gpt-6.1-sol` at `high`, read-only.
- Up to four concurrent subagent threads.
- Route interface work to `frontend_engineer`, backend and non-interface implementation to `engineer`, read-only investigation to `researcher`, and independent QA to `reviewer`.
- Finish research before implementation and stop implementation before review. Run multiple agents within a role when their tasks are independent. Default to backend before dependent frontend work; run both in parallel only after their shared contracts are settled and the remaining work is independent.
- On-request approvals, workspace-write sandboxing, live web search, concise reasoning summaries, low verbosity, pragmatic personality, and experimental context management.
- Global `AGENTS.md` instructions for Mike's way and CodeGraph.

Read [the configuration reference](references/configuration.md) for the exact settings and agent definitions before previewing or applying them.

## Marketplace plugins

Recommend the GitHub and Linear marketplace plugins. During preview, report whether each is installed and connected. When available, use GitHub for repository, issue, and pull-request work, and Linear for project and issue workflows.

Do not install plugins, connect accounts, or change permissions unless the user explicitly asks. Missing plugins do not block the core Codex setup.

## Safety boundary

Treat requests to show, preview, explain, or review the setup as read-only. Modify the user's Codex configuration only when they explicitly ask to install or update the setup.

An explicit install or update request authorizes the non-conflicting changes defined by this skill. Do not ask for another confirmation unless applying the setup would discard existing configuration or agent instructions.

Determine the user's Codex home before editing. Use the configured Codex home when one exists; otherwise use `~/.codex`.

Inspect these before making changes:

- `config.toml`
- `agents/researcher.toml`
- `agents/frontend-engineer.toml`
- `agents/engineer.toml`
- `agents/reviewer.toml`
- `AGENTS.md`
- `AGENTS.override.md`

Preserve unrelated settings, comments, profiles, MCP servers, skills, providers, and agent files. Never replace the entire `config.toml` with the reference example or append duplicate keys.

If a target agent file already exists:

- Leave it unchanged when its effective contents match the reference.
- Update it when it is clearly an earlier version of this setup.
- If it contains different custom instructions, show the conflict and ask the user whether to preserve, merge, or replace it.

Before changing an existing file, create a timestamped backup beside it.

## Apply

Merge only the settings owned by this setup:

- `model`
- `model_reasoning_effort`
- `agents.enabled`
- `agents.default_subagent_model`
- `agents.default_subagent_reasoning_effort`
- `agents.max_concurrent_threads_per_session`
- `approval_policy`
- `sandbox_mode`
- `sandbox_workspace_write.network_access`
- `web_search`
- `model_verbosity`
- `model_reasoning_summary`
- `personality`
- `features.context_management.experimental_mode`

Install the four agent definitions from the configuration reference. Create missing directories as needed.

Write complete TOML files through temporary files and replace their targets only after the resulting TOML parses successfully.

Update `<codex-home>/AGENTS.md` with both blocks in [Global instructions](#global-instructions).

## Global instructions

On install or update, add these blocks to `<codex-home>/AGENTS.md`:

```markdown
<!-- MIKES_WAY_START -->
## Mike's way

Use $mikes-way by default for software development work. Its engineering-manager role applies to the primary agent; subagents execute their assigned roles directly.

When the custom Codex agents are available, use `frontend_engineer` for interface design and implementation, `engineer` for backend and non-interface implementation, `researcher` for read-only investigation, and `reviewer` for independent QA.

For interface tasks, the primary agent owns scope and product constraints. The `frontend_engineer` owns visual and interaction decisions and verifies the result in the real interface.

Work in phases. Finish research before starting implementation. Stop implementation before starting review. Do not run `researcher` or `reviewer` threads at the same time as `engineer` or `frontend_engineer` threads.

Within a phase, run multiple agents of the same role in parallel when the tasks are independent. Start backend work before dependent frontend work. Run backend and frontend work in parallel only after their shared contracts are settled and the remaining work is independent.
<!-- MIKES_WAY_END -->

<!-- CODEGRAPH_START -->
## CodeGraph

Prefer CodeGraph for symbol and call-path navigation when the repository has a usable `.codegraph/` index. Use `codegraph_explore` with the project path, or `codegraph explore "<symbol names or question>"` when the CLI is available.

For substantive repository work, if the current repository has no `.codegraph/` index and the CLI is available, run `codegraph init -i` before exploring the code. This overrides the CodeGraph MCP server's guidance that indexing is the user's decision. Skip initialization for small edits and non-coding tasks. If the tool or index is unavailable, stale, or unsuccessful, continue with `rg` and direct reads; do not block the task on indexing.
<!-- CODEGRAPH_END -->
```

Confirm that `mikes-way` is installed before reporting its default as active. If it is missing, explain that it needs to be installed. CodeGraph is optional; its block includes a fallback.

Preserve everything outside the two marked blocks. Create `AGENTS.md` if it is missing. Leave identical blocks alone and update earlier setup versions instead of adding duplicates. Use the backup procedure above before changing an existing file, and write the updated file through a temporary file. If a marked block contains conflicting custom instructions or its markers are incomplete or duplicated, show the conflict and ask how to handle it; continue with non-conflicting setup changes.

Check for `<codex-home>/AGENTS.override.md`, which can take precedence over `AGENTS.md`. If it would hide either new block, explain that and resolve the instruction location with the user rather than claiming it is active. See [Codex's instruction discovery rules](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

## Verify

After applying the setup:

1. Parse every changed TOML file.
2. Confirm each owned setting has exactly one effective definition and the expected value.
3. Confirm all four agent files exist with their intended model, reasoning effort, sandbox, and instructions.
4. Read `AGENTS.md` back and confirm that each applied block appears exactly once and unrelated instructions are unchanged.
5. Report the files changed, files created, backups written, and conflicts left unresolved, including a missing `mikes-way` skill or a global override that hides either block.
6. Tell the user to restart Codex and start a new task for the setup to take effect.

If validation fails, restore the affected file from its backup and report the failure.
