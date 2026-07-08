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

Prompt a1: 5.3 Codex default execution (cautious)
gorgeous...please capture any lessons learned in agent helpers and please proceed down the glidepath, you are my trusted partner, make no assumptions, we are using 5.3 codex, continue with deterministic no-design slices, always close local gates and CI, and stop only if there is a true design decision or model/agent-selection risk per the guide

Prompt a2: 5.3 Codex autonomous execution (no reprompt)
gorgeous...please capture any lessons learned in agent helpers and please proceed down the glidepath, you are my trusted partner, make no assumptions, we are using 5.3 codex, continue with deterministic no-design slices, always close local gates and CI, and stop only if there is a true design decision or model/agent-selection risk per the guide.
Operate in autonomous execution mode: continue deterministic no-design slices back-to-back without asking for continue, reprompt, or confirmation after each slice.
For each slice you must: implement the smallest deterministic invariant, update tests/docs/helper lessons, run local gates, commit, push, close CI, and record memory.
Immediately proceed to the next deterministic no-design slice after successful CI closure.
Stop only when one of these occurs: (1) a true design decision is required, (2) a model or agent-selection risk is encountered, (3) a hard external blocker occurs (missing credentials/permissions, unavailable external service, or irrecoverable tool failure).
If stopped, report the exact blocker and the smallest next unblocked action.
Never skip local gates or CI closure, and never batch unrelated design changes.

Prompt: 5.5 if essential
gorgeous...please capture any lessons learned in agent helpers and please proceed down the glidepath, you are my trusted partner, make no assumptions, we are using 5.5, use this if and only if the task requires 5.5-level reasoning, and stop if there is any safe downgrade path to 5.3 codex

Quick workflow
- Start with 5.3 Codex.
- Escalate to 5.5 only for genuine high-complexity reasoning or ambiguous design decisions.
- If 5.5 finds a safe downgrade path, return to 5.3 Codex from the last green checkpoint.