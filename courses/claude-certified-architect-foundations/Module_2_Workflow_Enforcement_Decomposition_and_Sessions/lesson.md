# Module 2 — Workflow Enforcement, Task Decomposition, and Session Management

Domain: Domain 1 — Agentic Architecture & Orchestration (27% of exam)
Covers Task Statements 1.4, 1.5, 1.6, 1.7.

**Prerequisites from Module 1:** the agentic loop lifecycle (Lesson 1.1),
hub-and-spoke coordinator/subagent architecture and context isolation (Lesson
1.2), and subagent invocation mechanics (Lesson 1.3).

---

## Lesson 1.4 — Multi-Step Workflows with Enforcement and Handoff Patterns

**Maps to:** Task Statement 1.4: Implement multi-step workflows with enforcement
and handoff patterns.

### Prerequisite concept: Permission evaluation order for tool calls

Before any tool call executes, the Agent SDK runs it through a fixed evaluation
order: hooks first, then deny rules, then ask rules, then the active permission
mode, then allow rules, then a `canUseTool` callback. Hooks run before everything
else, and a hook that returns `deny` blocks the call regardless of what allow
rules, permission modes, or later steps would otherwise decide — including in the
most permissive `bypassPermissions` mode. This ordering is what makes hooks the
correct mechanism for guarantees that must never be skipped.

*Example:* An agent running in `bypassPermissions` mode still has a
`process_refund` call blocked if a registered `PreToolUse` hook denies it — the
hook runs before the permission mode is even consulted.

*Source:* https://code.claude.com/docs/en/agent-sdk/permissions
(Teaches: 1.4-K1, 1.4-S1, 1.5-K2, 1.5-S2 — prerequisite)

### Concept: Programmatic Enforcement vs Prompt-Based Guidance for Workflow Ordering

There are two ways to make an agent follow a required order of operations:
prompt-based guidance (instructing Claude, e.g. "always verify identity before
processing refunds") relies on the model reliably following that instruction, and
prompt instructions alone have a non-zero failure rate. Programmatic enforcement
(hooks, prerequisite gates evaluated in code) makes compliance deterministic —
it runs outside the model's own reasoning and cannot be forgotten. When a business
rule requires guaranteed compliance, the deterministic mechanism is correct.

*Example:* A prompt instruction "never process a refund before verifying the
customer" can fail if the model loses track deep in a long conversation. A
`PreToolUse` hook that programmatically checks whether `get_customer` has
returned a verified ID before allowing `process_refund` cannot fail this way.

*Source:* https://code.claude.com/docs/en/agent-sdk/hooks; corroborated by
exam-guide.txt
(Teaches: 1.4-K1, 1.4-K2)

### Concept: Programmatic Prerequisite Gates (Blocking Downstream Tool Calls)

A prerequisite gate is a `PreToolUse` hook (or equivalent permission rule) that
inspects the tool being requested and accumulated state, and denies the call
outright if a required earlier step has not completed. Because hooks are
evaluated before deny/ask/permission-mode/allow rules, a hook's denial cannot be
overridden by any looser setting elsewhere in the configuration.

*Example:* A `PreToolUse` hook matching `process_refund` checks whether a
`verified_customer_id` has been set by a prior `get_customer` call; if not, it
returns `permissionDecision: "deny"` with a reason explaining `get_customer` must
run first.

*Source:* https://code.claude.com/docs/en/agent-sdk/permissions;
https://code.claude.com/docs/en/agent-sdk/hooks
(Teaches: 1.4-S1)

### Concept: Multi-Concern Request Decomposition with Parallel Investigation

A customer message raising several distinct issues at once (e.g. "my order
arrived damaged AND I was charged twice") should be decomposed into constituent
items rather than handled as one undifferentiated request. Each item is
investigated using the same shared context, and findings are synthesized into a
single unified resolution.

*Example:* For "my order arrived damaged and I was charged twice," the agent
investigates the damage claim and the double-charge claim using the same
already-verified customer context, then presents one combined resolution.

*Source:* exam-guide.txt (Task 1.4 Skills bullet)
(Teaches: 1.4-S2)

### Concept: Structured Handoff Summaries for Human Escalation

When an agent escalates mid-process, the receiving human typically has no access
to the full transcript, so an unstructured note is insufficient. A structured
handoff protocol compiles: customer identifying details, root-cause analysis, any
relevant amounts, and a recommended action — as discrete fields, not freeform
prose.

*Example:* `{customer_id: "C-4471", root_cause: "Shipping carrier lost package per
tracking API", refund_amount: "$89.99", recommended_action: "Approve full refund;
carrier claim already filed"}` lets a human act in seconds.

*Source:* exam-guide.txt (Task 1.4 Knowledge and Skills bullets)
(Teaches: 1.4-K3, 1.4-S3)

**Quiz:** see `quiz.md`, Lesson 1.4.

---

## Lesson 1.5 — Agent SDK Hooks for Tool Call Interception and Data Normalization

**Maps to:** Task Statement 1.5: Apply Agent SDK hooks for tool call interception
and data normalization.

### Prerequisite concept: MCP tool results and the `isError` pattern

The Model Context Protocol defines two distinct error mechanisms: protocol-level
JSON-RPC errors, and tool execution errors reported inside the tool result via a
boolean `isError` field (`true` on failure) rather than as a protocol-level error
— letting the model see the failure as ordinary content and potentially
self-correct. Separately, each tool can declare its own `outputSchema`, and the
spec places no requirement that different tools represent the same real-world
data the same way — which is exactly why a timestamp or a status can come back
shaped differently from one MCP tool than another, making normalization
necessary before an agent reasons over results from multiple tools together.

*Example:* `{"result": {"content": [{"type":"text","text":"Failed to fetch weather
data: API rate limit exceeded"}], "isError": true}}` — the model sees this as
normal content it can reason about. An `orders` tool's `status` might be an
integer code while a `billing` tool's `status` is a string enum — both valid,
but an agent consuming both must reconcile the difference.

*Source:* https://modelcontextprotocol.io/specification/2025-06-18/server/tools;
exam-guide.txt Appendix
(Teaches: 1.5-K1, 1.5-S1 — prerequisite; this course returns to MCP error
categorization in depth in Module 3, Lesson 2.2)

### Concept: `PostToolUse` Hooks for Tool Result Transformation and Normalization

A `PostToolUse` hook fires after a tool returns its result and, before that
result reaches the model, can inspect and rewrite it. This is the mechanism for
normalizing heterogeneous data formats from different tools/MCP servers — e.g.
converting Unix timestamps and ISO 8601 strings to one date format, or mapping
numeric and string status codes to one vocabulary. The hook returns
`hookSpecificOutput` with `updatedToolOutput` to replace the tool's output, or
`additionalContext` to append information.

*Example:* A `PostToolUse` hook matched to `mcp__orders__*` and `mcp__billing__*`
converts every timestamp to ISO 8601 and every status to a shared enum before the
combined data reaches Claude.

*Source:* https://code.claude.com/docs/en/agent-sdk/hooks
(Teaches: 1.5-K1, 1.5-S1)

### Concept: `PreToolUse` Hooks for Compliance Interception

A `PreToolUse` hook fires before a tool call executes and can inspect the tool
name and proposed input to enforce a business rule, returning
`permissionDecision: "deny"` (with a reason) to block a policy-violating action
outright. A hook can also redirect: its denial reason can instruct Claude toward
an alternative path such as escalating to a human.

*Example:* A `PreToolUse` hook matched to `process_refund` inspects
`tool_input.amount`; if it exceeds $500, the hook denies with reason "Refunds
over $500 require human approval — call escalate_to_human instead," both
blocking the refund and steering toward the correct alternative.

*Source:* https://code.claude.com/docs/en/agent-sdk/hooks
(Teaches: 1.5-K2, 1.5-S2)

### Concept: Deterministic Guarantees (Hooks) vs Probabilistic Compliance (Prompts) — restated for 1.5

This is the same Programmatic Enforcement vs Prompt-Based Guidance principle from
Lesson 1.4, applied specifically to the hook mechanism: choose hooks over
prompt-based enforcement whenever business rules require guaranteed compliance,
because hooks run outside model reasoning and cannot be argued around.

*Example:* Relying only on a system-prompt instruction "never process refunds
over $500 without approval" leaves a non-zero chance the model processes one
anyway under unusual phrasing; a `PreToolUse` hook enforcing the same rule cannot
be talked past.

*Source:* https://code.claude.com/docs/en/agent-sdk/hooks; exam-guide.txt
(Teaches: 1.5-K3, 1.5-S3)

**Quiz:** see `quiz.md`, Lesson 1.5.

---

## Lesson 1.6 — Task Decomposition Strategies for Complex Workflows

**Maps to:** Task Statement 1.6: Design task decomposition strategies for complex
workflows.

### Concept: Prompt Chaining — Fixed Sequential Decomposition

Prompt chaining decomposes a task into a sequence of fixed steps, where each LLM
call processes the previous one's output, optionally with programmatic checks
between steps. It suits tasks that can be cleanly and predictably broken into
known subtasks in advance, trading some latency for higher per-step accuracy.

*Example:* A predictable code review pipeline: step 1 analyzes each changed file
individually; step 2 (a separate call) takes step 1's outputs and performs a
cross-file integration pass. The two stages are fixed and always run in order.

*Source:* https://www.anthropic.com/research/building-effective-agents
(Teaches: 1.6-K1, 1.6-S1)

### Concept: Dynamic Adaptive Decomposition (Orchestrator-Workers Pattern)

Dynamic decomposition uses a central orchestrator call to determine subtasks at
run time based on the specific input, rather than a pre-fixed sequence — the
subtasks themselves aren't known in advance. This suits open-ended investigation
where the right next step depends on what was just discovered.

*Example:* Software refactoring across an unfamiliar codebase can't have its
exact file/change list predicted upfront — the orchestrator discovers which
files matter as it goes, generating new subtasks in response to findings.

*Source:* https://www.anthropic.com/research/building-effective-agents;
corroborated by https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.6-K1, 1.6-K3, 1.6-S1)

### Concept: Per-File Local Analysis Plus Cross-File Integration Pass (Avoiding Attention Dilution)

A single LLM call asked to review many files at once at high depth tends to
produce inconsistent depth and miss issues — attention dilution. The fix specific
to code review: split into a local pass analyzing each file individually, plus a
separate cross-file integration pass looking for issues that only emerge from how
files interact.

*Example:* A 40-file pull request is reviewed as 40 independent per-file passes
(each flags local issues) plus one final pass given a summary of all 40 files'
changes together, specifically tasked with finding integration problems.

*Source:* exam-guide.txt (Task 1.6 Knowledge and Skills bullets)
(Teaches: 1.6-K2, 1.6-S2)

### Concept: Adaptive Investigation Planning for Open-Ended Tasks

For an open-ended task with no predefined subtask list, an effective strategy is
staged and adaptive: first map the codebase's structure, then identify high-impact
areas from that map, then create a prioritized plan — and let that plan continue
to adapt as dependencies and complications are discovered during execution.

*Example:* Told to add tests to an undocumented legacy service, an agent first
explores the directory/import graph to build a structural map, identifies the
payment-processing module as high-impact, prioritizes it, then adjusts the plan
mid-task upon discovering an undocumented dependency.

*Source:* exam-guide.txt (Task 1.6 Skills bullet); corroborated by
https://www.anthropic.com/research/building-effective-agents
(Teaches: 1.6-S3)

**Quiz:** see `quiz.md`, Lesson 1.6.

---

## Lesson 1.7 — Managing Session State, Resumption, and Forking

**Maps to:** Task Statement 1.7: Manage session state, resumption, and forking.

### Prerequisite concept: Session identifiers and transcript storage

Every Claude Code / Agent SDK session is identified by a session ID, and its full
transcript is persisted to disk automatically — locally under
`~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`. A session can be given a
human-readable name/tag via `renameSession`/`tagSession`, which is what makes it
possible to resume "a specific named investigation" rather than only the most
recent session or a raw UUID.

*Example:* After a research session finishes, calling
`rename_session(session_id, "q3-competitor-analysis")` lets a developer later
resume it by name.

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions
(Teaches: 1.7-K1, 1.7-S1 — prerequisite)

### Concept: Named Session Resumption (`--resume`)

Resuming takes a specific session identifier — including a human-assigned name,
via `--resume <session-name>` — and continues that exact prior conversation with
its full accumulated context restored. This differs from "continue," which always
picks up the most-recently-used session with no name required.

*Example:* `claude --resume auth-refactor-investigation` days later continues
exactly where that named investigation left off, even though other unrelated
sessions have run since.

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions; CLI flag syntax
corroborated by exam-guide.txt
(Teaches: 1.7-K1, 1.7-S1)

### Concept: Session Forking for Divergent Exploration — full skill coverage

Building on the forking definition introduced in Lesson 1.3 (`AgentDefinition`
lesson, 1.3-K4): `fork_session` creates an independent session from a shared
baseline, leaving the original's history unchanged, so both can be resumed and
extended separately.

*Example:* After a session has fully analyzed an authentication module, forking
it lets one branch explore JWT while the original explores OAuth2 — both share
the same initial analysis but diverge independently.

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions
(Teaches: 1.3-K4, 1.7-K2, 1.7-S2)

### Concept: Informing Resumed Sessions About File Changes

A session persists conversation history — not the filesystem. If files the agent
previously analyzed have been modified outside the session since it last saw
them, its in-context understanding is stale unless explicitly told what changed.
The fix: inform the resumed session about the specific files that changed,
enabling targeted re-analysis rather than full re-exploration.

*Example:* Resuming with "Note: `payment.py` was modified since you last read it
to add idempotency keys — re-check your refactoring plan against the new
version" lets the agent re-read only that file.

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions; scenario
corroborated by exam-guide.txt
(Teaches: 1.7-K3, 1.7-S4)

### Concept: Fresh Session with Structured Summary vs Resuming with Stale Tool Results

Resuming is not always the more reliable choice. When prior tool results are
likely stale or the transcript is too large/noisy, it is often more robust to
start a fresh session and inject a concise, structured summary of what was
learned, rather than resuming and hoping the model correctly discounts outdated
tool output. Resume when prior context is mostly still valid; start fresh with an
injected summary when it is not.

*Example:* After a long exploratory session whose tool outputs are now mostly
obsolete, a developer starts a new session with "Here is what we established
previously: [structured summary]. Proceed from here."

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions
(Teaches: 1.7-K4, 1.7-S3)

**Quiz:** see `quiz.md`, Lesson 1.7.

---

## Module 2 summary of bullets taught

Lesson 1.4: 1.4-K1, 1.4-K2, 1.4-K3, 1.4-S1, 1.4-S2, 1.4-S3
Lesson 1.5: 1.5-K1, 1.5-K2, 1.5-K3, 1.5-S1, 1.5-S2, 1.5-S3
Lesson 1.6: 1.6-K1, 1.6-K2, 1.6-K3, 1.6-S1, 1.6-S2, 1.6-S3
Lesson 1.7: 1.3-K4 (full skill coverage), 1.7-K1, 1.7-K2, 1.7-K3, 1.7-K4, 1.7-S1,
1.7-S2, 1.7-S3, 1.7-S4
