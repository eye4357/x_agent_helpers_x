PROMPT PLAYBOOK

Goal
- Use this file to choose between the default 5.3 Codex execution path and the rare 5.5 escalation path.

Model selection rules
- Use 5.3 Codex by default for deterministic, no-design, test-hardening glidepath work.
- Use 5.5 only for genuinely higher-complexity reasoning, architecture tradeoffs, or ambiguous multi-path design decisions.
- If work can be safely done in 5.3, do not run it on 5.5.

When to use each prompt
- Use the 5.3 Codex prompt for normal autonomous execution, deterministic slices, local gates, CI closure, and helper-lesson capture.
- Use the 5.5 prompt only when there is a real 5.5-level reasoning need; if a safe 5.3 downgrade path exists, stop and hand back to 5.3 Codex.

Prompt: 5.3 Codex default execution
gorgeous...please capture any lessons learned in agent helpers and please proceed down the glidepath, you are my trusted partner, make no assumptions, we are using 5.3 codex, continue with deterministic no-design slices, always close local gates and CI, and stop only if there is a true design decision or model/agent-selection risk per the guide

Prompt: 5.5 if essential
gorgeous...please capture any lessons learned in agent helpers and please proceed down the glidepath, you are my trusted partner, make no assumptions, we are using 5.5, use this if and only if the task requires 5.5-level reasoning, and stop if there is any safe downgrade path to 5.3 codex

Quick workflow
- Start with 5.3 Codex.
- Escalate to 5.5 only for genuine high-complexity reasoning or ambiguous design decisions.
- If 5.5 finds a safe downgrade path, return to 5.3 Codex from the last green checkpoint.