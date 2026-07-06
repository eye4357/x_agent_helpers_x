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
- After finishing a numeric scalar family (for example `part_count`), start the next boolean scalar family (for example `has_theme`) at sibling index `1` and keep the same ordered sibling closure pattern.
- For boolean scalar sibling closures with stable values, define one expected boolean constant and reuse it across sibling assertions to minimize churn while keeping contracts explicit.
- After reusing one expected boolean constant across sibling closures, finish the family through the last sibling index before switching to any new metric family.
- After closing one boolean family, move to the next adjacent boolean metric family at sibling index `1` and continue deterministic sibling order.
- While advancing sibling indices within the current boolean family, keep one shared expected constant name for that metric (`expected_docx_has_font_table`) until the family is closed.
- When closing the final sibling in a metric family, mirror that final-index addition in changelog, release notes, and change-control within the same slice so release docs stay synchronized.
- After closing a boolean family, advance to the next boolean metric in the existing structure-key order (for example `has_font_table` then `has_web_settings`) before skipping ahead.
- Within the active boolean family, keep one expected constant name unchanged across sibling indices (for example `expected_docx_has_web_settings`) to reduce churn and review noise.
- After the final sibling of `has_web_settings` is closed, hand off directly to the next adjacent boolean family start (`has_footnotes` index `1`) without mixing families.
- At a new boolean family start, introduce the matching expected constant name immediately (`expected_docx_has_footnotes`) and keep it stable across sibling closures.
- While progressing through `has_footnotes` siblings, reuse the same expected constant variable without renaming to preserve deterministic review diffs.
- After `has_footnotes` comparison 3 closes, move directly to the next adjacent boolean family start (`has_endnotes` comparison 1) without opening parallel families.
- At `has_endnotes` family start, declare `expected_docx_has_endnotes` once and reuse it unchanged for sibling closures to keep diffs deterministic.
- While advancing `has_endnotes` sibling indices, keep the same expected constant and assertion shape to preserve one-step deterministic diffs.
- After `has_endnotes` comparison 3 closes, hand off directly to the next adjacent boolean family start (`has_custom_xml` comparison 1) without opening parallel families.
- At `has_custom_xml` family start, declare `expected_docx_has_custom_xml` once and reuse it unchanged across sibling closures for deterministic low-churn diffs.
- While advancing `has_custom_xml` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `has_custom_xml` comparison 3 closes, continue with the next adjacent metric family in existing structure-key order rather than opening parallel families.
- At `header_count` family start, declare `expected_docx_header_count` once and reuse it unchanged across sibling closures for deterministic low-churn diffs.
- While advancing `header_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `header_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `footer_count` family start, declare `expected_docx_footer_count` once and reuse it unchanged across sibling closures for deterministic low-churn diffs.
- While advancing `footer_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `footer_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `body_child_counts` family start, declare one expected dict object from the rendered contract (`{"p": 1, "sectPr": 1}`) and reuse it across sibling closures.
- While advancing `body_child_counts` sibling indices, keep the same expected dict constant and assertion shape to preserve deterministic one-step diffs.
- After `body_child_counts` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `external_relationship_count` family start, declare `expected_docx_external_relationship_count` once and reuse it unchanged across sibling closures.
- While advancing `external_relationship_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `external_relationship_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `external_hyperlink_relationship_count` family start, declare `expected_docx_external_hyperlink_relationship_count` once and reuse it unchanged across sibling closures.
- While advancing `external_hyperlink_relationship_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `external_hyperlink_relationship_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `font_names` family start, declare `expected_docx_font_names` from the rendered contract list (`["Calibri", "Symbol"]`) and reuse it unchanged across sibling closures.
- While advancing `font_names` sibling indices, keep the same expected list constant and assertion shape to preserve deterministic one-step diffs.
- After `font_names` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `fonted_run_count` family start, declare `expected_docx_fonted_run_count` once and reuse it unchanged across sibling closures.
- While advancing `fonted_run_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `fonted_run_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `hyperlink_count` family start, declare `expected_docx_hyperlink_count` once and reuse it unchanged across sibling closures.
- While advancing `hyperlink_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `hyperlink_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `indented_paragraph_count` family start, declare `expected_docx_indented_paragraph_count` once and reuse it unchanged across sibling closures.
- While advancing `indented_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `indented_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `italic_run_count` family start, declare `expected_docx_italic_run_count` once and reuse it unchanged across sibling closures.
- While advancing `italic_run_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `italic_run_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbered_paragraph_count` family start, declare `expected_docx_numbered_paragraph_count` once and reuse it unchanged across sibling closures.
- While advancing `numbered_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbered_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_abstract_count` family start, declare `expected_docx_numbering_abstract_count` once and reuse it unchanged across sibling closures.
- While advancing `numbering_abstract_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_abstract_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_abstract_ids` family start, declare `expected_docx_numbering_abstract_ids` once and reuse it unchanged across sibling closures.
- While advancing `numbering_abstract_ids` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_abstract_ids` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_level_count` family start, declare `expected_docx_numbering_level_count` once and reuse it unchanged across sibling closures.
- While advancing `numbering_level_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_level_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_level_fonts` family start, declare `expected_docx_numbering_level_fonts` once from the rendered contract list and reuse it unchanged across sibling closures.
- While advancing `numbering_level_fonts` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_level_fonts` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_level_texts` family start, declare `expected_docx_numbering_level_texts` once using the escaped rendered glyph contract list (for example `["\u2022"]`) and reuse it unchanged across sibling closures.
- While advancing `numbering_level_texts` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- If `numbering_level_texts` comparison 1 already has generated/source parity while comparisons 2 and 3 only have value contracts, close parity in sibling order (`comparison 2` then `comparison 3`) before moving to `numbering_num_count`.
- After `numbering_level_texts` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `numbering_num_count` family start after `numbering_level_texts` closure, begin again at sibling index `1` with explicit generated/source parity before advancing to siblings `2` and `3`.
- At `numbering_num_count` family start, declare `expected_docx_numbering_num_count` once and reuse it unchanged across sibling closures.
- While advancing `numbering_num_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_num_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- After `numbering_num_count` family closure, hand off to `numbering_num_ids` at comparison `1` by adding explicit generated/source parity first, then close comparisons `2` and `3` in order.
- At `numbering_num_ids` family start, declare `expected_docx_numbering_num_ids` once from the rendered contract list and reuse it unchanged across sibling closures.
- While advancing `numbering_num_ids` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `numbering_num_ids` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- After `numbering_num_ids` family closure, hand off to `page_margins` at comparison `1` with explicit generated/source parity, then close comparisons `2` and `3` using sibling-specific expected maps when rendered margins differ.
- At `page_margins` family start, declare `expected_docx_page_margins` from the rendered contract map for comparison 1 and keep assertion shape explicit.
- While advancing `page_margins` sibling indices, preserve the same assertion shape but switch to sibling-specific expected maps when rendered margins differ (for example comparison 2 left/right `1800` versus `1440` on adjacent siblings).
- After `page_margins` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- After `page_margins` family closure, hand off to `page_size` at comparison `1` by adding explicit generated/source parity first, then close comparisons `2` and `3` in order using the shared expected page-size map.
- At `page_size` family start, declare `expected_docx_page_size` once from the rendered contract map and reuse it unchanged across sibling closures.
- While advancing `page_size` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `page_size` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- When adding parity assertions near dict-based expected contracts (for example `page_margins` maps), keep assertions outside the literal block to avoid syntax-break inserts during deterministic single-line patches.
- After `page_size` family closure, hand off to `paragraph_count` at comparison `1` by adding explicit generated/source parity first, then close comparisons `2` and `3` in order using the shared expected scalar.
- At `paragraph_count` family start, declare `expected_docx_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- After `paragraph_count` family closure, hand off to `run_count` at comparison `1` by adding explicit generated/source parity first, then close comparisons `2` and `3` in order using the shared expected scalar.
- At `run_count` family start, declare `expected_docx_run_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `run_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `run_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `styled_paragraph_count` family start, declare `expected_docx_styled_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `styled_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `styled_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `aligned_paragraph_count` family start, declare `expected_docx_aligned_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `aligned_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `aligned_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `spaced_paragraph_count` family start, declare `expected_docx_spaced_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `spaced_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `spaced_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `tab_stopped_paragraph_count` family start, declare `expected_docx_tab_stopped_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `tab_stopped_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `tab_stopped_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `table_count` family start, declare `expected_docx_table_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `table_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `table_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `table_grid_widths` family start, declare `expected_docx_table_grid_widths` once from the rendered list contract and reuse it unchanged across sibling closures.
- While advancing `table_grid_widths` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `table_grid_widths` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `table_paragraph_count` family start, declare `expected_docx_table_paragraph_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `table_paragraph_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `table_paragraph_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `table_row_count` family start, declare `expected_docx_table_row_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `table_row_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `table_row_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `underline_run_count` family start, declare `expected_docx_underline_run_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `underline_run_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- After `underline_run_count` comparison 3 closes, continue with the next adjacent metric family in structure-key order without opening parallel families.
- At `bold_run_count` family start, declare `expected_docx_bold_run_count` once from the rendered scalar contract and reuse it unchanged across sibling closures.
- While advancing `bold_run_count` sibling indices, keep the same expected constant and assertion shape to preserve deterministic one-step diffs.
- At `bold_run_count` comparison 3 closure, immediately hand off to the next adjacent metric family and repeat c1->c2->c3 closure with unchanged gate/CI evidence standards.
- At `colored_run_count` family start, define one expected scalar constant from the rendered DOCX audit contract and reuse it unchanged across sibling closures.
- While progressing `colored_run_count` siblings, only add the next comparison index assertions and keep constant/value source unchanged to preserve deterministic c1->c2->c3 diffs.
- At `colored_run_count` comparison 3 closure, hand off immediately to the next adjacent metric family under the same deterministic gate-and-CI closure cadence.
- When extending DOCX `normalized_text.line_count` sibling coverage, keep the scalar expectation (`1`) fixed across comparisons 1/2/3 and advance one sibling index per slice.
- At DOCX `normalized_text.line_count` comparison 3 closure, record family completion and hand off to the next adjacent deterministic assertion family with unchanged gate and CI closure standards.
- For DOCX `normalized_text.line_count` source-side family start, keep scalar expectation `1` fixed and advance source assertions in sibling order `comparison 1 -> 2 -> 3`.
- While progressing source-side `normalized_text.line_count`, add only the next comparison index assertion and keep the scalar value contract unchanged to preserve one-step deterministic diffs.
- At source-side `normalized_text.line_count` comparison 3 closure, record family completion and hand off to the next adjacent deterministic assertion family with unchanged gate/CI closure discipline.
- For DOCX `normalized_text.sha256` deterministic shape contracts, use a fixed scalar expectation (`64`) and advance sibling comparisons one index at a time while asserting both generated and source payload lengths.
- While progressing DOCX `normalized_text.sha256` siblings, keep the length contract fixed at `64` and add only the next comparison pair to preserve deterministic one-step diffs.
- At DOCX `normalized_text.sha256` comparison 3 closure, record family completion and hand off to the next adjacent deterministic assertion family under unchanged local/CI closure rules.
- For DOCX `normalized_text.sha256` equality-family start, advance sibling comparisons one index at a time while asserting explicit generated/source hash equality per comparison.
- While progressing DOCX `normalized_text.sha256` equality siblings, add only the next comparison-pair equality assertion to preserve deterministic one-step diffs.
- At DOCX `normalized_text.sha256` equality comparison 3 closure, record family completion and hand off to the next adjacent deterministic assertion family under unchanged local/CI closure discipline.
- After closing DOCX sibling equality families, close adjacent XLSX singleton contracts at comparison `0` explicitly (generated/source parity) before introducing new multi-sibling families.
- For XLSX singleton `normalized_text.sha256` shape contracts at comparison `0`, keep the fixed hash-length expectation (`64`) and close generated/source length parity in one deterministic slice.
- After XLSX singleton sha256-length closure at comparison `0`, close explicit generated/source sha256 equality in the next adjacent one-step slice before moving to a new metric family.
- For XLSX singleton structure keys at comparison `0`, close adjacent keys by explicit generated/source parity assertions (for example `part_names`) before introducing literal value-shape contracts.
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `content_type_overrides` immediately after `part_names`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `sheet_count` immediately after `content_type_overrides`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `sheet_names` immediately after `sheet_count`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `sheets` immediately after `sheet_names`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `styles` immediately after `sheets`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `workbook_relationship_targets` immediately after `styles`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `workbook_relationship_type_counts` immediately after `workbook_relationship_targets`).
- Continue XLSX singleton structure-key closure at comparison `0` in adjacent order by adding one explicit generated/source parity assertion per slice (for example `worksheet_part_count` immediately after `workbook_relationship_type_counts`).
- Continue XLSX singleton structure-key closure at comparison `0` by closing any remaining unpaired structure map/scalar contracts with one explicit generated/source equality assertion per slice (for example `root_relationship_type_counts`) while preserving one-slice deterministic diffs.
- Continue XLSX singleton structure-key closure at comparison `0` by closing any remaining unpaired structure map/scalar contracts with one explicit generated/source equality assertion per slice (for example `has_core_properties`) while preserving one-slice deterministic diffs.
- Continue XLSX singleton structure-key closure at comparison `0` by closing any remaining unpaired structure map/scalar contracts with one explicit generated/source equality assertion per slice (for example `has_extended_properties`) while preserving one-slice deterministic diffs.
- Continue XLSX singleton structure-key closure at comparison `0` by closing any remaining unpaired structure map/scalar contracts with one explicit generated/source equality assertion per slice (for example `has_styles`) while preserving one-slice deterministic diffs.
- Continue XLSX singleton structure-key closure at comparison `0` by closing any remaining unpaired structure map/scalar contracts with one explicit generated/source equality assertion per slice (for example `part_count`) while preserving one-slice deterministic diffs.
- After top-level XLSX structure parity is fully explicit, continue deterministically into nested `styles` subkeys by sorted order with one generated/source parity assertion per slice (for example `bold_font_indexes` first).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `border_colors` immediately after `bold_font_indexes`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `border_count` immediately after `border_colors`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `cell_style_count` immediately after `border_count`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `cell_xfs` immediately after `cell_style_count`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `cell_xfs_count` immediately after `cell_xfs`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `fill_count` immediately after `cell_xfs_count`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `fill_fg_colors` immediately after `fill_count`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `font_colors` immediately after `fill_fg_colors`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `font_count` immediately after `font_colors`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `font_names` immediately after `font_count`).
- Continue nested XLSX `styles` subkey closure in sorted order with one generated/source parity assertion per slice (for example `italic_font_indexes` immediately after `font_names`).
- After completing the nested XLSX `styles` subkey parity family, continue to the next adjacent DOCX family by adding one generated/source comparison-level parity assertion per slice (for example `underline_run_count` comparison `1` before comparisons `2` and `3`).
- Continue DOCX `underline_run_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- In commit-checks HTML polling, treat `data-conclusion="success"` as non-terminal while `data-job-status` is still `in_progress`; close the slice only when status is `completed`, conclusion is `success`, and a checks success marker is present.
- After closing DOCX `underline_run_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family with one generated/source parity assertion per deterministic slice.
- Continue the next adjacent DOCX metric family with one comparison-level generated/source parity assertion per slice (for example `colored_run_count` comparison `1` first, then comparisons `2` and `3`).
- Continue DOCX `colored_run_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- After closing DOCX `colored_run_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family with one generated/source parity assertion per deterministic slice.
- Continue the next adjacent DOCX metric family in adjacent comparison order with one generated/source parity assertion per slice (for example `aligned_paragraph_count` comparison `1` first, then comparisons `2` and `3`).
- Continue DOCX `aligned_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- After closing DOCX `aligned_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family with one generated/source parity assertion per deterministic slice.
- Continue DOCX `spaced_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `spaced_paragraph_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `spaced_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`tab_stopped_paragraph_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `tab_stopped_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `tab_stopped_paragraph_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `tab_stopped_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`table_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `table_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `table_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `table_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`table_grid_widths`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `table_grid_widths` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `table_grid_widths` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `table_grid_widths` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`table_paragraph_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `table_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `table_paragraph_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `table_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`table_row_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `table_row_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `table_row_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `table_row_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`bold_run_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `bold_run_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- After closing DOCX `bold_run_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`underline_run_count`) with one generated/source parity assertion per deterministic slice.
- For DOCX `style_definition_count` comparison-level parity closure, add one generated/source parity assertion per deterministic slice in sibling order (`comparison 1`, then `2`, then `3`) before switching families.
- Continue DOCX `style_definition_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `style_definition_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family with one generated/source parity assertion per deterministic slice.
- Continue DOCX `part_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `part_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `part_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_theme` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_theme` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_theme` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`has_font_table`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_font_table` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_font_table` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_font_table` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`has_web_settings`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_web_settings` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_web_settings` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_web_settings` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`has_footnotes`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_footnotes` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_footnotes` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_footnotes` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`has_endnotes`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_endnotes` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_endnotes` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_endnotes` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`has_custom_xml`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `has_custom_xml` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `has_custom_xml` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `has_custom_xml` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`header_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `header_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `header_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `header_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`footer_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `footer_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `footer_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `footer_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`body_child_counts`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `body_child_counts` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `body_child_counts` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `body_child_counts` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`external_relationship_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `external_relationship_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `external_relationship_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `external_relationship_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`external_hyperlink_relationship_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `external_hyperlink_relationship_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `external_hyperlink_relationship_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `external_hyperlink_relationship_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`font_names`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `font_names` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `font_names` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `font_names` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`fonted_run_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `fonted_run_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `fonted_run_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `fonted_run_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`hyperlink_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `hyperlink_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `hyperlink_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `hyperlink_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`indented_paragraph_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `indented_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `indented_paragraph_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `indented_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`italic_run_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `italic_run_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `italic_run_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `italic_run_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`numbered_paragraph_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `numbered_paragraph_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `numbered_paragraph_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `numbered_paragraph_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`numbering_abstract_count`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `numbering_abstract_count` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `numbering_abstract_count` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `numbering_abstract_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`numbering_abstract_ids`) with one generated/source parity assertion per deterministic slice.
- Continue DOCX `numbering_abstract_ids` comparison-level parity closure in adjacent order by adding one generated/source parity assertion per slice (`comparison 2` after `comparison 1`, then `comparison 3`).
- Continue DOCX `numbering_abstract_ids` comparison-level parity closure by adding `comparison 3` generated/source parity immediately after closing `comparison 2`, before moving to the next family.
- After closing DOCX `numbering_abstract_ids` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`underline_run_count`) with one generated/source parity assertion per deterministic slice.
- If a handoff line conflicts with the live assertion block, use the next in-file adjacent metric that is missing comparison-level generated/source parity; do not reopen already-closed families.
- Continue DOCX `numbering_level_count` comparison-level parity closure in adjacent order by adding `comparison 2` generated/source parity immediately after closing `comparison 1`, then close `comparison 3` before moving families.
- After closing DOCX `numbering_level_count` comparison-level parity for comparisons `1`, `2`, and `3`, continue to the next adjacent DOCX structure metric family (`numbering_level_fonts`) with one generated/source parity assertion per deterministic slice.
- In repeated sibling assertion blocks, keep each assertion bound to the matching section variable (`resume_2017_docx_section`, `resume_2023_docx_section`, `resume_2024_docx_section`); cross-binding can mask gaps and undermine deterministic per-sibling closure.
- If unauthenticated GitHub REST polling is rate-limited and `gh` is unavailable, confirm run and job completion from the workflow run page HTML (run status plus Quality Gates job status) instead of guessing CI state.
- On Windows/OneDrive worktrees, `git commit`/`git push` can trigger interactive local object cleanup prompts during auto-maintenance; for unattended slices, run git with `-c gc.auto=0 -c maintenance.auto=false` and prefer path-pinned commands (`git -C <repo> ...`) to avoid shell cwd drift.
- If terminal state intermittently resolves `git` as missing in multi-command runs, switch immediately to the absolute executable path (`C:\Program Files\Git\cmd\git.exe`) for deterministic commit/push closure.
- Once absolute Git path fallback is activated in a slice, keep using the same absolute invocation form for status/log/push/final checks through slice closure to avoid parser-mode regressions.
- If post-interrupt parser artifacts reject quoted absolute paths, switch to the short-path executable form (`C:\Progra~1\Git\cmd\git.exe`) for deterministic git status/log/push/rev-parse commands until closure is complete.

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