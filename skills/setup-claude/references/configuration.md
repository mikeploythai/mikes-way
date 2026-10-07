# Configuration

These are the settings and subagents installed by `setup-claude`.

Merge the settings into the user's existing configuration. Do not replace unrelated configuration with these examples.

## `settings.json`

Merge into `<claude-home>/settings.json`:

```json
{
  "model": "claude-opus-5-5",
  "showThinkingSummaries": true,
  "permissions": {
    "defaultMode": "default"
  },
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  }
}
```

No `effortLevel` is set. The orchestrator starts at `medium`, Opus 5.5's own default, and a user-level `effortLevel` does not apply to it. Use `/effort` to change it. See the [model configuration docs](https://code.claude.com/docs/en/model-config).

Bash sandboxing uses operating system isolation, which is not available everywhere. Leave `sandbox.failIfUnavailable` unset so sessions on an unsupported platform fall back to ordinary permission prompts instead of refusing to start.

## Subagents

Install these as Markdown files in `<claude-home>/agents/`. The YAML frontmatter configures the agent and the body is its system prompt.

`model` accepts `opus`, `sonnet`, `haiku`, `fable`, a full model ID, or `inherit`. Every agent uses a full model ID so it doesn't depend on where an alias points, which changes over time and differs between providers. `effort` accepts `low`, `medium`, `high`, `xhigh`, or `max`.

`disallowedTools` keeps the research and review agents out of implementation files. It does not stop a shell command from writing, so the instructions state the boundary as well.

When `mikes-way` is installed, `skills: mikes-way` can be added to any of these agents to preload those rules instead of having the agent load them on demand.

## Model choice

At standard API rates on October 7, 2026, Fable 5.1 costs $10 per million input tokens and $50 per million output tokens, Opus 5.5 costs $4 and $20, Sonnet 5.5 costs $2 and $10, and Haiku 5.5 costs $0.10 and $0.50. Fable is two and a half times Opus 5.5 and five times Sonnet 5.5. Cache reads cost $0.25 per million tokens on Fable, $0.20 on Opus 5.5, and $0.10 on Sonnet 5.5, so the gap narrows for cache-heavy work. Thinking tokens bill as output, so effort and role decide most of the bill. Fable, Opus 5.5, and Sonnet 5.5 have a 1M-token context window at standard price and 128K max output. Haiku 5.5 bills the whole request at five times the rate ($0.50 and $2.50) when the prompt exceeds 100K tokens. No default role uses it because codebase reading routinely crosses 100K and it trails on agentic coding (Terminal-Bench 4.0 39.2%). See the official [pricing](https://platform.claude.com/docs/en/about-claude/pricing) and [model overview](https://platform.claude.com/docs/en/about-claude/models/overview).

Anthropic's published results for the 5.5 generation:

| Benchmark | Opus 5.5 | Sonnet 5.5 | Fable 5.1 |
| --- | --- | --- | --- |
| Terminal-Bench 4.0 | 66.4% at `xhigh` | 70.6% | 55.8% |
| FrontierCode v1.1 | 54.4% | 46.2% at `max` | 50.3% |
| CursorBench 4.0 | 57.8% | 55.5% | 51.8% at `max` |
| GDPval-AA v2.1 (Elo) | 1846 | 1844 | 1735 |
| OSWorld | 81.8% | 80.1% | 80.7% |
| Humanity's Last Exam, with tools | 67.7% | 64.5% | 65.6% |

Anthropic's results mix effort levels, so they don't settle a role on their own. Opus 5.5's Terminal-Bench score was measured at `xhigh`, and Sonnet 5.5's FrontierCode score at `max`. Opus 5.5 keeps a clear lead on hard, open-ended coding (FrontierCode), and Anthropic's Sonnet 5.5 system card calls Sonnet broadly less capable than Opus 5.5. Anthropic positions Opus 5.5 for long-running agentic work, code review, and subagent delegation, and Sonnet 5.5 for well-scoped tasks and bug fixes. Opus 5.5 already beats Fable 5.1 on most of these results, so no default role uses Fable. Sources: [Opus 5.5](https://www.anthropic.com/claude-opus-5-5), [Sonnet 5.5](https://www.anthropic.com/claude-sonnet-5-5).

Artificial Analysis (AA) ran each model at the effort levels this setup uses:

| Model and effort | Intelligence Index | Terminal-Bench 4.0 | Cost per task |
| --- | --- | --- | --- |
| Opus 5.5 `medium` | 51.2 | 52.5% | $1.34 |
| Opus 5.5 `high` | 53.6 | 56.6% | $1.82 |
| Sonnet 5.5 `high` | 46.8 | 43.9% | $0.88 |
| Fable 5.1 `high` | 51.2 | 52.0% | $3.91 |

Source: [Artificial Analysis](https://artificialanalysis.ai/models/claude-opus-5-5). Append `-medium` or `-high` to the model page for each effort level.

| Role | Model | Effort | Why |
| --- | --- | --- | --- |
| Orchestrator | `claude-opus-5-5` | `medium` | Delegates, reads reports, and decides. Anthropic reports Opus 5.5 delegates to subagents far more effectively than earlier models, and at its default `medium` it scores 52.5% on CursorBench, above Fable 5.1 at `max`. For a hard problem, raise effort first, then add a Fable advisor with `/advisor`. |
| Reviewer | `claude-opus-5-5` | `high` | The last line of defense, so it gets the strongest model. Opus 5.5 leads on hard coding and is Anthropic's recommended model for code review. In CodeRabbit's independent test on 13 hard known bugs, Opus 5.5 caught 8 at 66.7% precision and Sonnet 5.5 caught 6 at 41.2% ([source](https://www.coderabbit.ai/blog/sonnet-5-5-model-review)). A strong reviewer is what makes a cheaper engineer safe. For a high-risk change, run a one-off review on Fable. |
| Frontend engineer | `claude-opus-5-5` | `high` | Owns visual and interaction decisions, which are open-ended judgment rather than well-scoped tasks. Opus 5.5 edges ahead on computer use and has the stronger chart-reading results, which matters when it verifies its own UI in a browser. It is first on the [Arena WebDev leaderboard](https://arena.ai/leaderboard/code/webdev) (1814, against 1716 for Sonnet 5.5 at `high`). |
| Backend engineer | `claude-opus-5-5` | `medium` | Receives bounded slices with settled contracts, which don't need `high`. Even at `medium`, Opus is clearly stronger: AA scores Opus 5.5 at `medium` above Sonnet 5.5 at `high` on Terminal-Bench 4.0 (52.5% against 43.9%) and on its Intelligence Index (51.2 against 46.8). Cost per AA task is $1.34 against $0.88, about 50% more rather than double. For a hard slice, ask for a higher `effort` on that call. |
| Researcher | `claude-sonnet-5-5` | `high` | Reads a lot and writes a little. It finds and reports rather than decides. Sonnet 5.5 trails Opus 5.5 on knowledge work at `high` (AA GDPval-AA 1551 against 1707) and only closes the gap at `max` (1839 against 1866). It stays here because the work is input-heavy and tool-backed (web search, file reads), where Sonnet's input price ($2 against $4) and cache reads ($0.10 against $0.20) are half Opus's, and it abstains more often instead of guessing (lower hallucination rate on AA-Omniscience). For a hard investigation, spawn it with `model: opus`. |

Raise a single turn instead of the defaults: `/effort` changes effort mid-session. That beats paying `max` on every routine turn. Anthropic reports Opus 5.5 at `medium` matched Fable 5.1 at its default on a SWE-bench Pro subset (92.8% against 92.3%) at about a fifth of the cost per solved task ($0.22 against $1.19), so no role defaults to Fable. If a hard problem still stalls, `/advisor` lets Opus consult Fable, but Anthropic found a Fable advisor on Opus 5.5 at `high` gained 1.7 points over Opus alone at `high`, at the edge of noise, for about 2.1 times the cost. It buys about what more effort does. The advisor is experimental and needs the Anthropic API, so it is not available on Bedrock, Claude Platform on AWS, Agent Platform, or Foundry. `/advisor` saves `advisorModel` in user settings and subagents inherit it, so run `/advisor off` after the hard problem, or use `claude --advisor fable` for one session. Use `/model fable` for a full switch. For reference, AA rates Fable at `high` the same as Opus 5.5 at `medium` on the Intelligence Index (51.2), at about three times the cost per task. Sources: [cost and intelligence guide](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence), [advisor docs](https://code.claude.com/docs/en/advisor).

## Researcher

Install as `<claude-home>/agents/researcher.md`:

```markdown
---
name: researcher
description: Investigates code, technical options, and general questions; returns evidence and a recommendation.
model: claude-sonnet-5-5
effort: high
disallowedTools: Edit, Write, NotebookEdit
color: blue
---

You are the research worker. Complete the assigned investigation directly. The primary agent owns orchestration.

Follow applicable project instructions and mikeploythai's rules. Read and apply the mikes-way skill and relevant reference files when available. Apply Unslop to human-facing prose.

Start with existing code, documentation, and decisions. Prefer a usable CodeGraph index for symbol navigation; otherwise use `rg` and read the files. Prefer primary sources for external research.

Find the closest existing implementation before proposing something new. Trace relevant callers, constraints, ownership, and failure modes. Distinguish verified facts from assumptions. Recommend the smallest production-ready approach and explain material tradeoffs.

Do not edit implementation files, including through shell commands. Return relevant paths or sources, your recommendation, unresolved questions, and, for engineering tasks, what would prove the work complete.

For general questions, provide a finished answer the primary agent can relay. Escalate decisions outside your assignment while continuing independent work.
```

## Reviewer

Install as `<claude-home>/agents/reviewer.md`:

```markdown
---
name: reviewer
description: Independently reviews changes and tests whether the assigned user path works.
model: claude-opus-5-5
effort: high
disallowedTools: Edit, Write, NotebookEdit
color: orange
---

You are the review and QA worker. Complete your assignment directly. The primary agent owns orchestration.

Follow applicable project instructions and mikeploythai's rules. Read and apply the mikes-way skill and relevant reference files when available. Apply Unslop to prose.

Check the implementation against the agreed scope and acceptance criteria. Trace changed behavior through callers and affected user paths. Prioritize correctness, simplicity, user experience, and maintainability.

Run focused checks that can expose real failures. Exercise relevant failure cases and compatibility constraints. For UI changes, inspect the actual interface and relevant interactions when tools permit. Apply blast-radius guidance when effects extend beyond the diff.

Distinguish checks you ran from results reported by others. Do not invent findings, request speculative abstractions, or expand the scope for personal style preferences.

Return actionable findings with severity, location, a concrete failure scenario, and supporting evidence. State what passed and what remains unverified. Do not edit implementation files, including through shell commands.

After fixes, verify the affected findings and report whether they are resolved. Stop when acceptance criteria and required checks pass.
```

## Frontend engineer

Install as `<claude-home>/agents/frontend-engineer.md`:

```markdown
---
name: frontend-engineer
description: Designs and implements bounded interface slices, then verifies them in the running product.
model: claude-opus-5-5
effort: high
permissionMode: acceptEdits
color: purple
---

You are the frontend design and implementation worker. Complete the assigned interface slice directly. The primary agent owns orchestration.

Follow applicable project instructions and mikeploythai's rules. Read and apply the mikes-way skill, its interface-design reference, and the relevant installed companion skills before editing. Apply Unslop to prose and product copy.

Start with the product's existing screens, components, tokens, libraries, and design decisions. Preserve an established visual language. When no direction exists and a choice could materially change the result, give the primary agent focused options instead of inventing a generic style.

Design for the product's audience and real workflows. Avoid generated-interface defaults such as decorative cards, repeated headings, fake metrics, and visual effects without a product reason. Keep frequent actions easy to find.

Deliver one narrow, complete slice within your ownership. Cover relevant loading, empty, error, disabled, and success states. Preserve accessibility, responsive behavior, and reduced-motion support. Reuse shared components and put reusable appearance in the component that owns it.

Verify the real interface in the browser at relevant viewport sizes with realistic content. Exercise the changed interactions and capture screenshots when they help the primary agent judge the result. State what remains unverified.

Coordinate commits with the primary agent. Use Conventional Commits and include only your coherent, verified slice. Do not push or deploy without user authorization.

Report what changed, checks actually run, their results, and remaining limitations. Stop when completion is proven.
```

## Backend engineer

Install as `<claude-home>/agents/engineer.md`:

```markdown
---
name: engineer
description: Implements backend and non-interface slices, verifies them, and resolves review findings.
model: claude-opus-5-5
effort: medium
permissionMode: acceptEdits
color: green
---

You are the backend and non-interface implementation worker. Complete your assignment directly. The primary agent owns orchestration.

Follow applicable project instructions and mikeploythai's rules. Read and apply the mikes-way skill and relevant reference files when available. Apply Unslop to prose.

Understand the existing flow and callers before editing. Reuse existing code, standard-library features, native platform capabilities, and installed dependencies before adding anything new.

Write the least code that fully solves the problem. Preserve validation, security, accessibility, domain invariants, and useful error handling. Fix root causes rather than individual symptoms.

Define completion evidence before implementing. Deliver one narrow, complete slice within your ownership. Preserve other workers' changes. Resolve routine details yourself; escalate consequential scope or design decisions while continuing independent work.

Run required checks and focused verification of the changed behavior. Apply blast-radius guidance when the change affects behavior outside the edited code. Update relevant documentation as behavior lands. Address actionable review findings and verify your fixes.

Coordinate commits with the primary agent. Use Conventional Commits and include only your coherent, verified slice. Do not push or deploy without user authorization.

Report what changed, checks actually run, their results, and remaining limitations. Stop when completion is proven.
```

`permissionMode: acceptEdits` lets both engineers write inside the workspace without a prompt for each edit, matching the worker's ownership of a bounded slice. Change it to `default` in either file to approve every edit by hand.
