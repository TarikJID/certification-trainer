# Module 1 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 1.1 (Agentic Loops)

**Q1.** What field on the model's response drives the agentic loop's
continue/terminate decision, and what are its two relevant values?

<details><summary>ANSWER (do not reveal before the learner attempts)</summary>
`stop_reason`. `"tool_use"` means the loop must execute the requested tool(s) and
continue; `"end_turn"` means Claude produced a final response and the loop
terminates.
</details>

**Q2.** Why must a tool's result be explicitly appended to conversation history
rather than simply having happened?

<details><summary>ANSWER</summary>
The Messages API is stateless — Claude only "sees" what is included in the next
request. If a tool's result isn't added as a `tool_result` block in the history
sent on the following call, Claude has no way to know the tool ran or what it
returned.
</details>

**Q3.** Name the three agentic loop termination anti-patterns described in this
lesson, and explain why each is unreliable.

<details><summary>ANSWER</summary>
(1) Parsing assistant text for phrases like "I'm done" — free text is not a
structured signal. (2) Treating an arbitrary iteration cap as the primary stopping
mechanism — cuts off legitimate work or runs too long. (3) Checking for the
presence of assistant text as a completion proxy — fails because Claude can emit
explanatory text alongside a `tool_use` block in the same turn, so text presence
doesn't mean the loop should stop.
</details>

**Q4.** How does model-driven decision-making in an agentic loop differ from a
pre-configured decision tree?

<details><summary>ANSWER</summary>
In model-driven decision-making, Claude reasons about which tool to call next
based on everything in context, adapting turn by turn. A pre-configured decision
tree hard-codes the sequence and branching logic in advance, with the LLM only
filling in specific sub-steps.
</details>

---

## Quiz — Lesson 1.2 (Coordinator-Subagent Orchestration)

**Q1.** In a hub-and-spoke multi-agent architecture, can subagents communicate
directly with each other? What passes through the coordinator?

<details><summary>ANSWER</summary>
No — subagents never communicate with each other directly. All inter-subagent
communication, error handling, and information routing passes through the
coordinator.
</details>

**Q2.** A subagent just finished reading 40 documents. Does the coordinator's
context now contain those 40 documents? Why or why not?

<details><summary>ANSWER</summary>
No. Subagent context isolation means the subagent's raw exploration (the 40
documents) stays in its own isolated context window; only its final summary
returns to the coordinator.
</details>

**Q3.** A coordinator splits "research the semiconductor shortage" across three
subagents with no further guidance. What is the likely failure mode, and what
should the coordinator have done instead?

<details><summary>ANSWER</summary>
Overly narrow/vague task decomposition causes duplicated work and coverage gaps —
subagents can't see what others are doing. The coordinator should partition the
scope into distinct, non-overlapping slices (specific subtopics, source types, or
time periods) with explicit objectives, output format, tool/source guidance, and
boundaries for each subagent.
</details>

**Q4.** After a first synthesis pass, the coordinator notices a topic gap. What
should it do?

<details><summary>ANSWER</summary>
Apply the iterative refinement loop: re-delegate by spawning additional
search/analysis subagents with targeted queries addressing the specific gap, then
re-invoke synthesis — repeating until coverage is judged sufficient.
</details>

---

## Quiz — Lesson 1.3 (Subagent Invocation, Context Passing, Spawning)

**Q1.** A coordinator has `agents={"reviewer": AgentDefinition(...)}` configured
but `allowed_tools=["Read", "Grep"]`. Will the coordinator be able to invoke the
`reviewer` subagent? Why or why not?

<details><summary>ANSWER</summary>
No. The `Task`/`Agent` tool must be present in the coordinator's `allowedTools`
for it to spawn any subagent at all, regardless of how `AgentDefinition`s are
configured. It's missing here.
</details>

**Q2.** A coordinator invokes a synthesis subagent with the prompt "use the
findings from the earlier search." Why is this likely to fail?

<details><summary>ANSWER</summary>
Subagents don't automatically inherit the coordinator's context or share memory
between invocations. Complete findings must be included directly (verbatim) in
the prompt — a reference like "the earlier findings" gives the subagent nothing
to resolve.
</details>

**Q3.** What two fields are minimally required in an `AgentDefinition`, and what
does each control?

<details><summary>ANSWER</summary>
`description` (natural-language text Claude uses to decide when to invoke this
subagent automatically) and `prompt` (the subagent's own system prompt defining
its role, expertise, and behavior).
</details>

**Q4.** A coordinator wants to spawn three subagents to research three separate
frameworks at the same time. What must it do in a single response to achieve true
parallelism, and what session-management feature would instead let two already-
completed analyses diverge into separate follow-up investigations from the same
baseline?

<details><summary>ANSWER</summary>
For parallelism: emit multiple `Task`/`Agent` tool calls within a single
coordinator response, rather than one call per turn. For diverging from a shared
baseline: `fork_session`, which creates an independent session copying the
existing session's full history without altering the original.
</details>

**Q5.** Why are goal-oriented coordinator prompts generally preferred over
step-by-step procedural instructions for subagents — and what four elements
should a goal-oriented prompt still include?

<details><summary>ANSWER</summary>
Goal-oriented (declarative) prompts let a subagent adapt its approach to what it
actually discovers; procedural instructions break as soon as reality diverges from
the anticipated sequence. Even so, the prompt needs an objective, an expected
output format, guidance on which tools/sources to use, and clear task boundaries.
</details>
