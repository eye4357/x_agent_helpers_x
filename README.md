# x_agent_helpers_x

Python project.

## Operational Lessons Learned

- When `gh run watch` renders no output, close CI deterministically by querying exact run metadata with `gh run view --json databaseId,headSha,status,conclusion,name` or `gh api repos/<owner>/<repo>/actions/runs/<run_id>` and validating `headSha` matches the pushed commit.
- For deterministic CI closure logs, capture both run id and head SHA in the same step so run-to-commit binding remains explicit in evidence.
- For ordinal-token test discovery, remember code identifiers use underscore form (for example, `ninety_fifth`) while docs use hyphenated text (for example, `ninety-fifth`).
- For deterministic adjacent nibble slices, keep each new token exactly one nibble longer than the prior token and mirror the same literal across tests, changelog, change-control, and AGENTS entries.
- Keep ordinal continuity explicit with compact range notation instead of telescoping per-slice bullets; for example, record `third -> ... -> seventy-eighth` and call out only exceptional lexical boundaries.
- For fast deterministic slice prep, use one anchor read near the latest nibble test and one anchor read in each governance file, then apply all additive updates in one patch to reduce drift risk.
- When commit scope is intentionally repo-local, stage only governed files for that repo so unrelated helper-note edits remain uncommitted unless explicitly intended.
- When a deterministic hardening stream becomes open-ended, define a release terminal slice in change control before closing so follow-up increments move to a newly scoped release instead of extending the current one indefinitely.