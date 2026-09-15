# Module 4 Hands-On Exercise

## Design a multi-agent code-audit architecture

You must design an Agent SDK architecture for auditing a large repository
for missing input validation, spanning hundreds of files.

1. **Loop bounds.** Specify `max_turns` and `max_budget_usd` values (with
   brief justification) for the coordinator session, and state which
   `ResultMessage` subtype you'd expect if the audit runs out of turns
   before finishing.

2. **Decompose the task.** Decide whether this is better served by
   turn-by-turn coordinator–subagent delegation or a dynamic workflow
   script. Justify your choice using the four documented decomposition
   benefits from Lesson 4.3, and the scale limits from the same lesson.

3. **Define two subagents/worker roles**: an `auditor` (finds issues) and a
   `verifier` (adversarially re-checks each finding before it's reported).
   For each, specify: its restricted tool set, what information must be
   explicitly included in the prompt handed to it (given context isolation
   rules from Lesson 4.2), and what its final returned message should
   contain.

4. **Enforcement.** Add a `PreToolUse` hook requirement that blocks any
   `Write`/`Edit` call during the audit (it should be read-only end to end).
   Walk through the six-step evaluation order from Lesson 4.4 and identify
   at which step your hook takes effect, and why a hook is a stronger
   guarantee here than just relying on permission mode alone.

5. **Session handoff.** The audit is expected to take multiple sessions
   across a day. Describe how you would use Continue, Resume, and/or Fork
   to (a) pick up the next day where you left off, and (b) safely try an
   alternative auditing strategy on a subset of files without disturbing the
   main run.

### What a strong answer includes
- Explicit numeric bounds with a stated rationale (not just "high enough").
- A decomposition choice that correctly applies the isolation/
  parallelization/specialization/tool-restriction rationale, and that
  respects (or explicitly exceeds and thus rules out turn-by-turn
  delegation for) the workflow scale limits.
- Subagent prompts that include everything the subagent needs (since it
  starts with no parent history), and restricted tool sets appropriate to a
  read-only audit.
- Correct identification that hooks run first in the evaluation order and
  can block a call outright, which is why a hook is used here instead of
  relying only on `disallowedTools` or permission mode.
- Correct differentiation of Continue vs. Resume vs. Fork mapped to the two
  described scenarios.
