# Module 4 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 4.1 — The Agent Loop and the query() Function

1. What are the four possible `ResultMessage` error subtypes besides
   `success`?
2. What is the difference between what `max_turns` bounds and what
   `max_budget_usd` bounds?
3. What object configures nearly every other Agent SDK capability
   (subagents, hooks, permission modes, sessions)?

### Answers
1. `error_max_turns`, `error_max_budget_usd`, `error_during_execution`,
   `error_max_structured_output_retries`.
2. `max_turns`/`maxTurns` bounds the number of tool-use round trips
   (turns); `max_budget_usd`/`maxBudgetUsd` bounds cumulative spend.
3. `ClaudeAgentOptions` (Python) / `Options` (TypeScript), passed to the
   `query()` function.

---

## Quiz 4.2 — Coordinator–Subagent Orchestration

1. What is the difference between automatic and explicit subagent
   invocation?
2. By default, does a subagent see the coordinator's prior conversation
   history? What does it receive instead?
3. What does the coordinator's context grow by when a subagent finishes its
   work, and what does it NOT grow by?
4. What tool is used to invoke a subagent?

### Answers
1. Automatic: Claude matches the task against each subagent's `description`
   field and decides on its own. Explicit: the prompt names the subagent
   directly, bypassing matching and guaranteeing that specific subagent
   runs.
2. No — a non-fork subagent's context starts fresh. It receives only the
   literal prompt string given to the `Agent` tool call, plus its own
   `AgentDefinition.prompt`, project CLAUDE.md (unless omitted), and its
   assigned tools.
3. It grows only by the subagent's final summarized message; it does not
   grow by the subagent's intermediate tool calls or file reads, which stay
   inside the subagent's own context.
4. The built-in `Agent` tool.

---

## Quiz 4.3 — Task Decomposition and Dynamic Workflows

1. Name the four documented benefits of task decomposition.
2. What is the key architectural difference between turn-by-turn
   coordinator delegation and a dynamic workflow script?
3. Name the three functions a dynamic workflow script can call, and what
   each does.
4. What are the default/documented limits on a dynamic workflow (concurrent
   agents, total agents, items per parallel/pipeline call)?

### Answers
1. Context isolation, parallelization, specialized instructions/knowledge,
   and tool restriction.
2. A workflow script's intermediate results live in script variables
   outside Claude's own context window, executed by a runtime — not held
   turn-by-turn in the conversation — so it can coordinate far more agents
   than a conversation-held delegation could.
3. `agent()` spawns one subagent; `pipeline()` runs one subagent per item
   in a list (sequential per item, but agents can run concurrently);
   `parallel()` runs a set of agent tasks concurrently and waits for all.
4. Up to 16 concurrent agents by default, 1,000 agents total per run, and
   up to 4,096 items per `parallel()`/`pipeline()` call.

---

## Quiz 4.4 — Enforcement, Handoff, and Hooks

1. List the six-step fixed evaluation order for a tool request, in order.
2. Can a subagent run in `bypassPermissions` mode if its parent coordinator
   session does not? Why or why not?
3. Which hook event fires before a tool executes, and what can it do?
4. Which hook event is relevant to context compaction, and what is it
   typically used for?

### Answers
1. (1) hooks, (2) deny rules, (3) ask rules, (4) active permission mode,
   (5) allow rules, (6) the `canUseTool` callback.
2. No — a subagent only runs in `bypassPermissions` mode if the parent
   session itself does; a subagent cannot escalate its own privilege beyond
   what the coordinator granted.
3. `PreToolUse`; it can validate inputs or block dangerous commands outright
   before the tool executes.
4. `PreCompact`, fired before context compaction — typically used to
   archive the full transcript before it gets summarized away.

---

## Quiz 4.5 — Session State Management

1. What are the three session operations, and what does each do?
2. Where does the caller obtain the `session_id` needed to resume a
   session?
3. If you want to try an alternative approach without losing the original
   conversation thread, which operation should you use?

### Answers
1. Continue (resumes the most recent session in the current working
   directory, no ID needed), Resume (picks a specific captured `session_id`
   back up with full prior context), Fork (creates a new session copying
   an existing one's history up to that point, diverging from there,
   leaving the original untouched).
2. From the `session_id` field on the `ResultMessage`/`SDKResultMessage`.
3. Fork — it copies the existing session's history but diverges going
   forward, leaving the original session unchanged.
