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
- On Windows, do not assume the `py` launcher or a local `3.12` interpreter exists; probe explicitly and, if unavailable, rely on the CI Quality Gates result as the authoritative cross-version check before closing the slice.
- For CI confirmation, derive the run id from the workflow page and derive the Quality Gates job id from that run page before polling job status; do not guess ids or reuse stale ids from prior runs.
- On GitHub commit checks pages, short-hash URLs can return 404; use the full 40-character commit SHA for `/commit/<sha>/checks` to resolve the correct CI run deterministically.
- In anonymous HTML fetches, a workflow run page can present stale in-progress snapshots; if this happens, treat commit checks summary (`1 / 1`) on the full-SHA commit page as the terminal fallback signal after you already resolved run and Quality Gates job IDs from the checks page.
- With GitHub Actions REST polling, run status can flip to `completed` slightly before the jobs endpoint reflects terminal job states; re-query `/actions/runs/<id>/jobs` and require `Quality Gates` `completed/success` before declaring slice closure.
- With GitHub Actions REST polling, terminal signals can arrive in either order (`run` first or `Quality Gates` first); close the slice only after both endpoints are terminal-success (`run` is `completed/success` and `Quality Gates` is `completed/success`).
- GitHub Actions can report non-blocking runner deprecation annotations (for example Node runtime migration notices) while `Quality Gates` still succeeds; treat these as maintenance follow-up, not a failed slice, unless a required gate turns non-success.
- For scoped Office audit Markdown assertions, probe rendered report rows in the exact audit code path before adding assertions; values can differ from lower-level structure fixtures (for example, header/footer counts can be `0` in the audit report path).
- For list-valued scoped Office audit metrics (for example `table_grid_widths`), assert the exact Markdown-serialized row value from the rendered audit report (for example `[]`) instead of inferring formatting from lower-level summaries.
- For deterministic metric glidepaths, derive the next row from the full ordered metric list (for example `docx_metrics`) rather than filtered keyword searches; filtered slices can skip intermediate metrics and break sequence discipline.
- For deterministic JSON contract glidepaths, after locking adjacent count scalars, lock neighboring boolean package-presence scalars one at a time (for example `has_core_properties`, then `has_extended_properties`, then `has_styles`) to keep each slice minimal and low churn.
- For deterministic JSON contract glidepaths, when a scalar is already literal-asserted on `generated`, close the symmetric contract immediately by asserting the matching `source` scalar before moving to larger payloads.
- For deterministic per-comparison scalar parity slices, close sibling comparisons in index order (for example `1`, then `2`, then `3`) for the same metric before switching to a different metric family.
- After closing the final sibling index for a scalar metric family, start the next family at the first sibling index and repeat the same ordered closure pattern.
- When a rendered Markdown metric row is already exact and deterministic (for example `part_count`), use that row value as the no-assumption seed for the next JSON scalar parity slice in the same family.
- In repeated sibling assertion blocks, keep each assertion bound to the matching section variable (`resume_2017_docx_section`, `resume_2023_docx_section`, `resume_2024_docx_section`); cross-binding can mask gaps and undermine deterministic per-sibling closure.
- If unauthenticated GitHub REST polling is rate-limited and `gh` is unavailable, confirm run and job completion from the workflow run page HTML (run status plus Quality Gates job status) instead of guessing CI state.
- On Windows/OneDrive worktrees, `git commit`/`git push` can trigger interactive local object cleanup prompts during auto-maintenance; for unattended slices, run git with `-c gc.auto=0 -c maintenance.auto=false` and prefer path-pinned commands (`git -C <repo> ...`) to avoid shell cwd drift.
- If terminal state intermittently resolves `git` as missing in multi-command runs, switch immediately to the absolute executable path (`C:\Program Files\Git\cmd\git.exe`) for deterministic commit/push closure.
- Once absolute Git path fallback is activated in a slice, keep using the same absolute invocation form for status/log/push/final checks through slice closure to avoid parser-mode regressions.

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