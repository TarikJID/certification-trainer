# Module 4: Agentic Architecture & Orchestration

Domain weight on the exam: 27% — the largest single domain. It is taught
fourth, not first, because most of it presupposes tool use and the context
window (Module 1), the general subagent notion and permission modes (Module
2), and MCP (Module 3).

---

## Lesson 4.1: The Agent Loop and the query() Function

### Concept: The Agent Loop
The core execution cycle powering autonomous agents built with the Claude
Agent SDK — the fully formalized version of Module 1's general "tools in a
loop" pattern. Claude receives a prompt along with the system prompt, tool
definitions, and conversation history; it evaluates the current state and
either responds with text, requests one or more tool calls, or both; the
SDK executes each requested tool and feeds the results back to Claude; this
"evaluate → call tools → receive results" cycle repeats until Claude
produces a response with no tool calls, at which point the loop ends and a
final result is returned. One full cycle of evaluate+execute is a "turn."
The loop can be bounded with `max_turns`/`maxTurns` (tool-use round trips)
and `max_budget_usd`/`maxBudgetUsd` (spend), and produces a final
`ResultMessage` whose `subtype` (`success`, `error_max_turns`,
`error_max_budget_usd`, `error_during_execution`,
`error_max_structured_output_retries`) tells the caller how the loop ended.

**Example:** For "Fix the failing tests in auth.ts": Turn 1, Claude calls
`Bash` to run `npm test` and gets 3 failures back; Turn 2, Claude calls
`Read` on `auth.ts` and `auth.test.ts`; Turn 3, Claude calls `Edit` then
re-runs `npm test`, which now passes; Final turn, Claude replies with text
only, ending the loop after 4 turns.

**Source:** Agentic Architecture & Orchestration research — The Agent Loop
(key), https://code.claude.com/docs/en/agent-sdk/agent-loop

### Concept: The query() Function and ClaudeAgentOptions/Options
The basic programmatic entry point of the Claude Agent SDK. `query()` is the
function that starts (or continues) an agent loop: it accepts a `prompt`
and an `options` object (`ClaudeAgentOptions` in Python, `Options` in
TypeScript) that configures everything about how the agent behaves —
allowed/disallowed tools, permission mode, model, system prompt, MCP
servers, subagent definitions, turn/budget limits, and session continuation
flags — and returns an async iterator of messages consumed with `async
for`. Nearly every other capability in this module (subagent invocation,
hooks, permission modes, session resume/fork) is configured as a field on
this same options object.

**Example:** `query(prompt="Review utils.py for bugs...",
options=ClaudeAgentOptions(allowed_tools=["Read","Edit","Glob"],
permission_mode="acceptEdits"))` starts an agent loop that streams messages
while auto-approving the three listed tools.

**Source:** Agentic Architecture & Orchestration research — The query()
Function and ClaudeAgentOptions/Options (prerequisite), https://code.claude.com/docs/en/agent-sdk/quickstart

---

## Lesson 4.2: Coordinator–Subagent Orchestration

### Concept: Coordinator–Subagent Orchestration Pattern
A multi-agent architecture in which a main ("coordinator") agent delegates
focused subtasks to separate agent instances called subagents, each with
its own isolated conversation, specialized system prompt, and (optionally)
restricted tool set — the full depth of the general subagent notion
introduced in Module 2. Subagents are invoked through the built-in `Agent`
tool. This pattern is the primary way the Agent SDK scales beyond a single
flat conversation: the coordinator decides, turn by turn, what to delegate
and to whom, and receives only each subagent's final summarized result
rather than its full working transcript. Subagents can themselves spawn
further subagents (nested delegation), bounded by a configurable depth
limit.

**Example:** A `code-reviewer` subagent (tools restricted to `Read`, `Grep`,
`Glob`, model `sonnet`) and a `test-runner` subagent (tools `Bash`, `Read`,
`Grep`) are both defined via the `agents` parameter of
`ClaudeAgentOptions`/`Options`. A coordinator prompt "Review the
authentication module for security issues" causes Claude to invoke the
`code-reviewer` subagent automatically because its `description` field
matches the task.

**Source:** Agentic Architecture & Orchestration research —
Coordinator–Subagent Orchestration Pattern (key), https://code.claude.com/docs/en/agent-sdk/subagents

### Concept: Subagent Invocation (Automatic and Explicit)
The mechanism by which a subagent is triggered to run. Automatic invocation:
Claude reads each defined subagent's `description` field and decides on its
own, based on the current task, whether to route work to that subagent via
the `Agent` tool. Explicit invocation: the calling prompt names the
subagent directly ("Use the code-reviewer agent to check the authentication
module"), bypassing matching and guaranteeing that specific subagent runs.
Subagents run in the background by default unless the caller requests a
synchronous ("foreground") result, and their spawn depth, concurrency, and
cumulative spend can be capped via environment variables
(`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`,
`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`) and the `maxBudgetUsd`/
`max_budget_usd` query option.

**Example:** Writing "Use the code-reviewer agent to check the
authentication module" forces Claude to invoke that named subagent, whereas
a vaguer prompt like "review my auth code" leaves Claude to match the task
against subagent descriptions automatically.

**Source:** Agentic Architecture & Orchestration research — Subagent
Invocation (key), https://code.claude.com/docs/en/agent-sdk/subagents

### Concept: Context Passing and Isolation Between Coordinator and Subagent
The rule set governing what information flows from a coordinator into a
subagent and back. Unless a subagent is a "fork," its context window starts
fresh: it does not receive the parent's conversation history, prior tool
results, or the parent's system prompt. The only content passed from parent
to subagent is the literal prompt string given to the `Agent` tool call, so
any file paths, error messages, or decisions the subagent needs must be
included explicitly in that prompt. A non-fork subagent does receive its
own `AgentDefinition.prompt` (system prompt — Module 1), project CLAUDE.md
(unless `omitClaudeMd` is set — Module 2), and its assigned tool
definitions. Only the subagent's final message is returned to the parent as
the Agent tool's result — intermediate tool calls and reads stay inside the
subagent and never accumulate in the coordinator's context. This isolation
is also what keeps the coordinator's context window (Module 1) "lean" on
long or exploratory subtasks.

**Example:** A `research-assistant` subagent can read dozens of files while
investigating a bug; none of those file contents enter the main
conversation's context — the coordinator's context only grows by the
subagent's final one-paragraph summary.

**Source:** Agentic Architecture & Orchestration research — Context Passing
and Isolation Between Coordinator and Subagent (key), https://code.claude.com/docs/en/agent-sdk/subagents

---

## Lesson 4.3: Task Decomposition and Dynamic Workflows

### Concept: Task Decomposition Strategies
The design practice of splitting a large or multi-part task into smaller
units of work assigned to separate agents, chosen deliberately for
isolation, parallel speed-up, or specialization. The Agent SDK documentation
identifies four concrete benefits: context isolation (each subagent's
exploration doesn't pollute the parent's context), parallelization
(independent subtasks finish in the time of the slowest one instead of the
sum of all of them), specialized instructions/knowledge (a tailored system
prompt gives each worker only the expertise it needs), and tool restriction
(limiting each worker's tool surface to only what its slice of the task
requires — Module 3's "distributing tools across subagents"). Decomposition
can be expressed either as several named subagents invoked turn-by-turn by
a coordinator, or, at larger scale, as a dynamic workflow script (next
concept).

**Example:** During a code review, running `style-checker`,
`security-scanner`, and `test-coverage` subagents simultaneously instead of
sequentially trades a single generalist pass for three specialized,
parallel passes whose results are then synthesized by the coordinator.

**Source:** Agentic Architecture & Orchestration research — Task
Decomposition Strategies (key), https://code.claude.com/docs/en/agent-sdk/subagents ; https://code.claude.com/docs/en/workflows

### Concept: Dynamic Workflows (Script-Based Multi-Subagent Orchestration)
A mechanism for orchestrating many subagents (dozens to hundreds) via a
JavaScript script that a runtime executes outside the conversation's
context window, rather than turn-by-turn delegation held in Claude's own
context. Claude writes the orchestration script for a described task; the
script body can call `agent()` (spawn one subagent), `pipeline()` (run one
subagent per item in a list, sequentially per item but agents can run
concurrently), and `parallel()` (run a set of agent tasks concurrently and
wait for all). Intermediate results live in script variables, not in
Claude's context, so a workflow can coordinate far more agents than a
turn-by-turn conversation can hold. Workflows run in the background, are
resumable within the same session, and are bounded by runtime limits (up to
16 concurrent agents by default, 1,000 agents total per run, up to 4,096
items per `parallel()`/`pipeline()` call).

**Example:** "use a workflow to audit every route handler under
src/routes/ for missing authentication checks, and adversarially verify
each finding before reporting it" causes Claude to write a script that
lists all matching files with one `agent()` call, then runs a `pipeline()`
of one audit-subagent per file, and finally has an independent verifier
subagent adversarially review each finding before the workflow returns one
merged report.

**Source:** Agentic Architecture & Orchestration research — Dynamic
Workflows (key), https://code.claude.com/docs/en/workflows

---

## Lesson 4.4: Enforcement, Handoff, and Hooks

### Concept: Enforcement and Handoff Patterns (Permission Evaluation and Tool Restriction)
The layered mechanism the SDK uses to control and enforce what a tool call —
from the main agent or a subagent — is actually allowed to do, and how that
control is handed off between coordinator and subagent. Every tool request
is evaluated in a fixed order: (1) hooks (can deny outright — next concept),
(2) deny rules (`disallowed_tools`/settings.json — block even in
`bypassPermissions` mode), (3) ask rules (fall through to the approval
callback even in `bypassPermissions`), (4) the active permission mode
(`default`, `acceptEdits`, `plan`, `dontAsk`, `auto`, `bypassPermissions` —
Module 2 introduced the CLI-level versions of these modes), (5) allow rules
(`allowed_tools`/settings.json), and (6) the `canUseTool` callback as a last
resort. On handoff to a subagent, tool restriction
(`AgentDefinition.tools`/`disallowedTools` — Module 3) and permission mode
are inherited from the parent unless explicitly overridden, with the rule
that a subagent only runs in `bypassPermissions` mode if the parent session
itself does — a subagent cannot escalate its own privilege beyond what the
coordinator granted.

**Example:** A `db-reader` subagent is defined with `tools: ["Bash"]` and
`permissionMode: "dontAsk"`; combined with a `PreToolUse` hook that runs a
validation script against every `Bash` call (exit 0 allows, exit 2 blocks),
the subagent can only ever execute pre-approved read-only queries,
regardless of what the coordinator's own permission mode allows.

**Source:** Agentic Architecture & Orchestration research — Enforcement and
Handoff Patterns (key), https://code.claude.com/docs/en/agent-sdk/permissions ; https://code.claude.com/docs/en/sub-agents

### Concept: Agent SDK Hooks for Tool Interception
Callback functions, run in the calling application's own process (not
inside the agent's context window), that fire at defined points in the
agent loop so custom code can observe or alter agent behavior. Documented
hook events: `PreToolUse` (fires before a tool executes; can validate
inputs or block dangerous commands), `PostToolUse` (fires after a tool
returns; audit outputs or trigger side effects), `UserPromptSubmit` (fires
when a prompt is sent; inject additional context), `Stop` (fires when the
agent finishes; validate the result or save state), `SubagentStart`/
`SubagentStop` (fire when a subagent spawns or completes; track and
aggregate parallel task results), and `PreCompact` (fires before context
compaction — Module 6; archive the full transcript). A hook can be scoped
to specific tools with a `matcher` pattern (e.g. `"Bash"` runs the hook only
for Bash calls; omitting the matcher runs it for every event of that type).
Because a `PreToolUse` hook that rejects a call prevents the tool from
executing at all, hooks are the mechanism used to intercept and enforce
policy on tool calls, independent of and prior to permission-mode/allow-rule
evaluation (see the fixed evaluation order above).

**Example:** Registering a `PreToolUse` hook matched to `"Write|Edit"` that
inspects the file path being written and returns a deny decision whenever
the path falls outside an allow-listed directory blocks the write before it
ever reaches disk, and Claude sees the rejection as the tool's result.

**Source:** Agentic Architecture & Orchestration research — Agent SDK Hooks
for Tool Interception (key), https://code.claude.com/docs/en/agent-sdk/hooks ; https://code.claude.com/docs/en/agent-sdk/agent-loop

---

## Lesson 4.5: Session State Management

### Concept: Session State Management (Continue, Resume, Fork)
The set of mechanisms for persisting and returning to an agent's
conversation state across multiple calls. A "session" is the conversation
history the SDK accumulates while an agent works — the prompt, every tool
call, every tool result, and every response — written to disk
automatically. Three distinct operations manage this state: "Continue"
finds and resumes the most recent session in the current working directory
with no ID tracking required; "Resume" takes a specific captured
`session_id` and picks that session back up with full prior context, used
when there are multiple concurrent sessions or the target isn't the most
recent one; "Fork" creates a brand-new session that starts as a copy of an
existing session's history but diverges from that point onward, leaving the
original session unchanged, so an application can explore an alternative
approach without losing the original thread. The session ID is read from
the `session_id` field on the `ResultMessage`/`SDKResultMessage`.

**Example:** After a first query analyzes an `auth` module and captures its
`session_id`, a second call passes `resume=session_id` with the prompt "Now
implement the refactoring you suggested" — the agent proceeds with full
memory of the prior analysis instead of re-reading the files. A third call
could instead pass `resume=session_id, fork_session=True` with a different
prompt ("outline how OAuth2 would work instead") to explore that
alternative without disturbing the original JWT-focused session.

**Source:** Agentic Architecture & Orchestration research — Session State
Management (key), https://code.claude.com/docs/en/agent-sdk/sessions

> **Cross-reference:** Module 3 (Lesson 3.4) already covered "Model Context
> Protocol (MCP)" in full depth as a prerequisite for this module's task
> decomposition and enforcement patterns above — an MCP server-scoped
> subagent (e.g., one restricted to `mcp__database__*` tools) is a direct
> application of the tool-distribution pattern from Module 3 combined with
> this module's Coordinator–Subagent pattern.
