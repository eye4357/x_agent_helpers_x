# Model And Agent Selection Guide

Use this guide before development work begins.

## Required Pre-Development Question

Before starting development, ask explicitly:

> What model is appropriate for this task, and what agent should I use from the available agent list?

Do not assume the cheapest model is appropriate for implementation work. Do not assume the strongest model is always necessary. Choose based on task risk, context length, privacy sensitivity, and validation cost.

## Practical Model Risk Ladder

Use higher-capability models when mistakes would be expensive to find or repair.

| Model tier | Recommended use | Main risk |
| --- | --- | --- |
| GPT 5.5 | Architecture, release decisions, privacy-sensitive workflows, schema/default/template decisions, packaging strategy, multi-repo automation, and ambiguous design tradeoffs. | Cost. |
| GPT 5.4 | Serious coding, release hardening, CI/debug loops, multi-file edits, and most productionization work. | Some judgment loss versus the top tier. |
| GPT 5.3 Codex | Code-heavy implementation, test writing, refactors, CLI/tooling work, and narrow debugging where the requirements are clear. | May be less broadly strategic if it is more code-specialized. |
| GPT 5.4 mini | Narrow, well-specified edits, repetitive assertion additions, formatting, small docs changes, and tasks with cheap validation. | More missed context and more supervision needed. |
| GPT 5 mini | Drafting, summarization, simple local edits, mechanical changes, and low-risk work that is easy to verify. | Highest risk for autonomous development: missed constraints, weaker long-context tracking, and false confidence. |

If GPT 5.5 is unaffordable, prefer GPT 5.4 or GPT 5.3 Codex for development that touches release quality, privacy, public/private boundaries, CI, packaging, schemas, or multi-repo state.

Do not use GPT 5 mini unattended for autonomous glidepath work that must preserve a full sequence such as:

```text
probe -> edit -> focused validation -> ledger update -> full gate -> commit -> push -> CI confirmation -> memory update -> clean-state check
```

GPT 5 mini can be useful when the slice is mechanical and independently verifiable, but the operator should supervise it closely.

## Agent Recommendation From The Current List

Current available specialized agent:

- `Explore`: fast read-only codebase exploration and Q&A.

Recommended default:

- Use the normal coding agent for implementation, edits, validation, commits, CI checks, and release-hardening loops.
- Use `Explore` when the next step is read-only reconnaissance: finding owners, mapping call paths, locating tests, summarizing relevant files, or answering a codebase question before editing.

Do not use `Explore` for direct file edits or commit work. It should return concise findings that the coding agent can act on.

## How To Ask For A Recommendation

When asking which model and agent to use, include:

- The intended task.
- Whether code edits are expected.
- Whether the task touches private data, public release artifacts, schemas, templates, packaging, CI, or multi-repo state.
- Whether the desired mode is autonomous execution, review only, or read-only exploration.
- The acceptable cost level: cheapest workable, balanced, or safest.

Example request:

```text
Before development, recommend the model and agent for this task.
Task: add the next scoped DOCX audit assertion slice.
Risk: public release hardening, no private data, commit/push/CI expected.
Cost preference: balanced, avoid GPT 5.5 unless clearly needed.
Available agents: Explore.
```

Expected recommendation for that example:

```text
Use GPT 5.4 or GPT 5.3 Codex with the normal coding agent. Use Explore only if the controlling code path or next assertion row is unclear and needs read-only reconnaissance first. Avoid GPT 5 mini for unattended execution because the workflow depends on preserving validation, ledger, CI, memory, and clean-state discipline.
```

## Execution Hygiene For Python Work

- Before running any Python command in a terminal, configure the workspace Python environment and use its explicit interpreter command prefix.
- If collection fails with `ModuleNotFoundError` for local modules or missing tools like `pytest`, stop and repair environment selection/dependencies first, then rerun the same validation command.
- Treat interpreter mismatch as an execution-environment issue, not a signal to weaken test scope.

## Stop Conditions

Ask for a higher-capability model or explicit human decision before proceeding when the work involves:

- Renderer behavior changes.
- Schema or default policy changes.
- Template compatibility decisions.
- Privacy boundary decisions.
- Release strategy or packaging changes.
- Ambiguous user-facing behavior.
- A failed validation whose cause changes the understanding of the controlling code path.

When in doubt, use a stronger model for the decision and a cheaper model for the mechanical follow-through after the decision is settled.