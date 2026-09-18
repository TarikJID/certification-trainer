# Domain 1 Research — Agentic Architecture & Orchestration

Bullets received: 48 (24 Knowledge, 24 Skills)
Bullets covered: 48 (48/48)
Concepts: 34 (30 key, 4 prerequisite)

---

## Prerequisite concepts

- Concept: The Claude Messages API request/response cycle and `tool_use` content blocks
  Type: prerequisite
  Teaches: 1.1-K1, 1.1-K2, 1.1-S1, 1.1-S2
  Definition: The Claude API is a stateless request/response API: each call to the Messages endpoint sends the full conversation so far (system prompt, prior turns, tool definitions) and receives back a response containing one or more content blocks. When Claude wants to use a tool, the response includes a `tool_use` content block naming the tool and its input arguments, alongside a top-level `stop_reason` field. This request/response mechanic is the substrate the agentic loop is built on: the loop is simply a program that keeps calling this endpoint, feeding tool outputs back in, until the model stops requesting tools.
  Example: A single Messages API call with `messages=[{"role":"user","content":"What's the weather in Boston?"}]` and a `get_weather` tool defined returns a response with `stop_reason: "tool_use"` and a content block `{"type":"tool_use","name":"get_weather","input":{"location":"Boston"}}`.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/build-a-tool-using-agent

- Concept: MCP tool results and the `isError` pattern
  Type: prerequisite
  Teaches: 1.5-K1, 1.5-S1
  Definition: The Model Context Protocol specification defines two distinct error-reporting mechanisms for tools: protocol-level JSON-RPC errors (unknown tool, invalid arguments, server error) and tool execution errors, which are reported inside the tool result itself via a boolean `isError` field (`true` on failure; omitted or `false` means success) rather than as a protocol-level error — this lets the calling model see the failure as ordinary result content and potentially self-correct, instead of the call simply failing at the transport level. Separately, the spec lets each tool declare its own optional `outputSchema` for structured results, and places no requirement that different tools (or different servers) represent the same kind of real-world data the same way. That per-tool freedom is exactly why the same concept — a timestamp, a status — can come back shaped differently from one MCP tool than from another, which is what makes a normalization step necessary before an agent reasons over results pulled from multiple MCP tools together.
  Example: A `tools/call` response reporting a failure returns `{"result": {"content": [{"type": "text", "text": "Failed to fetch weather data: API rate limit exceeded"}], "isError": true}}` — the model sees this as normal tool-result content it can reason about, rather than the call erroring out at the protocol level. Separately, an `orders` tool's own declared `outputSchema` might represent `status` as an integer code while a `billing` tool's schema represents `status` as a string enum — both are valid, spec-compliant tool results, but an agent consuming both together must reconcile the difference itself.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools (formal definition of `isError` and per-tool `outputSchema`); the exam guide's Appendix technologies list also names "isError flag" as in-scope — runs/claude-certified-architect-foundations/sources/exam-guide.txt, line 976

- Concept: Permission evaluation order for tool calls
  Type: prerequisite
  Teaches: 1.4-K1, 1.4-S1, 1.5-K2, 1.5-S2
  Definition: Before any tool call executes, the Agent SDK runs it through a fixed evaluation order: hooks first, then deny rules, then ask rules, then the active permission mode, then allow rules, then a `canUseTool` callback. Hooks are evaluated before everything else, and a hook that returns a `deny` decision blocks the call regardless of what allow rules, permission modes, or later steps would otherwise decide — including in the most permissive `bypassPermissions` mode. This ordering is what makes hooks the correct mechanism for guarantees that must never be skipped: nothing downstream of a hook can override its denial.
  Example: An agent running in `bypassPermissions` mode (which auto-approves nearly everything) still has a `process_refund` call blocked if a registered `PreToolUse` hook denies it — the hook runs before the permission mode is even consulted.
  Source: https://code.claude.com/docs/en/agent-sdk/permissions

- Concept: Session identifiers and transcript storage
  Type: prerequisite
  Teaches: 1.7-K1, 1.7-S1
  Definition: Every Claude Code / Agent SDK session is identified by a session ID and its full transcript (prompts, tool calls, tool results, responses) is persisted to disk automatically — locally under `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`. A session can additionally be given a human-readable name/tag via session-management functions (`renameSession`/`rename_session`, `tagSession`/`tag_session`), which is what makes it possible to resume "a specific named investigation" rather than only the most recent session or a raw UUID.
  Example: After a research session finishes, calling `rename_session(session_id, "q3-competitor-analysis")` lets a developer later resume it by name instead of having to remember or look up the UUID.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions

---

## Task Statement 1.1: Design and implement agentic loops for autonomous task execution

- Concept: The Agentic Loop Lifecycle (`stop_reason`-driven control flow)
  Type: key
  Teaches: 1.1-K1, 1.1-S1
  Definition: The agentic loop is the cycle that powers Claude Code and the Agent SDK: send a request to Claude (prompt, system prompt, tool definitions, conversation history), let Claude evaluate the state and either respond with text, request one or more tool calls, or both; execute any requested tools; feed the results back in; and repeat. The loop's continuation decision is driven entirely by the `stop_reason` field on the model's response: `"tool_use"` means Claude wants to call a tool and the loop must execute it and continue; `"end_turn"` means Claude produced a final response with no further tool calls and the loop terminates. Each full cycle of "Claude responds → tools execute → results return" is one turn; a session runs as many turns as needed until an `end_turn` is reached (or a safety limit like `max_turns`/budget is hit).
  Example: For the prompt "Fix the failing tests in auth.ts", turn 1 has Claude call `Bash(npm test)` (`stop_reason: "tool_use"`); turn 2 it calls `Read` on the relevant files; turn 3 it calls `Edit` then re-runs tests; the final turn produces a text-only response with `stop_reason: "end_turn"`, and the loop stops.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 148-149 (Task Statement 1.1 Knowledge bullet, verbatim: "inspecting stop_reason (\"tool_use\" vs \"end_turn\")"); loop mechanics (turn structure, tool execution, repeat-until-no-tool-calls) corroborated by https://code.claude.com/docs/en/agent-sdk/agent-loop, whose own `stop_reason` discussion is scoped to the final turn's value (`end_turn`, `max_tokens`, `refusal`) rather than per-turn `tool_use`/`end_turn` branching — the raw Messages API's per-turn `tool_use`/`end_turn` values are documented via the prerequisite concept's own citation

- Concept: Appending Tool Results to Conversation History
  Type: key
  Teaches: 1.1-K2, 1.1-S2
  Definition: After each tool executes, its result is not discarded — it is added to the conversation as a new message (a `tool_result` block associated with the originating `tool_use` block's ID) and included in the next request sent to Claude. This is how the model "sees" what happened: nothing about a tool's outcome is available to Claude for its next decision unless it has been explicitly appended to the history sent on the following API call. The context window accumulates the system prompt, tool definitions, and this growing conversation history across every turn of the session.
  Example: After `Bash(npm test)` returns "3 failing tests", the SDK yields a `UserMessage` carrying that output, which becomes part of the history in the next call to Claude — enabling Claude's next turn to reason about which file to read based on the specific failures reported, rather than guessing blind.
  Source: https://code.claude.com/docs/en/agent-sdk/agent-loop

- Concept: Model-Driven Decision-Making vs Pre-configured Decision Trees
  Type: key
  Teaches: 1.1-K3
  Definition: An agentic loop lets Claude decide, turn by turn, which tool to call next based on everything in context — this is model-driven (or "agent") behavior, distinct from a hard-coded workflow where the sequence of steps and branching logic is authored in advance by a developer and the LLM is invoked only to fill in specific sub-steps. Anthropic's own distinction: workflows are systems where LLMs and tools are orchestrated through predefined code paths; agents are systems where the LLM dynamically directs its own process and tool usage, maintaining control over how it accomplishes the task. Agentic loops (Task Statement 1.1) are explicitly the latter — Claude, not a decision tree, chooses the next action.
  Example: A pre-configured decision tree for customer support might hard-code "if refund requested, always call get_customer then process_refund." An agentic loop instead lets Claude decide — based on the conversation — whether to call `lookup_order` first, ask a clarifying question, or escalate, adapting the sequence to the specific request rather than following a fixed script.
  Source: https://www.anthropic.com/research/building-effective-agents

- Concept: Agentic Loop Termination Anti-Patterns
  Type: key
  Teaches: 1.1-S3
  Definition: Three implementation mistakes undermine a correctly designed loop: (1) parsing the assistant's natural-language text for phrases like "I'm done" to decide whether to stop, which is unreliable because free text is not a structured signal; (2) treating an arbitrary iteration cap (e.g., "stop after 10 loops") as the *primary* stopping mechanism rather than a safety backstop, which cuts off legitimate multi-step work or, if set too high, lets the loop run needlessly long; and (3) checking for the presence of assistant text content as a proxy for "the task is complete," which fails because Claude can emit explanatory text alongside a `tool_use` block in the same turn. The correct mechanism is the structured `stop_reason` field, not any of these heuristics.
  Example: An implementation that stops the loop as soon as Claude's response contains any text (anti-pattern 3) would terminate prematurely on a turn where Claude writes "Let me check the logs first" and then also requests a `Read` tool call in the same response — the loop should have continued because `stop_reason` is still `"tool_use"`.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 160-162 (Task Statement 1.1 Skills bullet, verbatim), corroborated by https://code.claude.com/docs/en/agent-sdk/agent-loop ("Turns continue until Claude produces output with no tool calls")

---

## Task Statement 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

- Concept: Hub-and-Spoke Coordinator Architecture
  Type: key
  Teaches: 1.2-K1, 1.2-S4
  Definition: In a hub-and-spoke (orchestrator-worker) multi-agent design, one coordinator ("lead") agent sits at the hub and every subagent is a spoke that communicates only with the coordinator — subagents never talk to each other directly. All inter-subagent communication, error handling, and information routing passes through the coordinator. This topology gives the coordinator full observability into what every subagent is doing, lets it apply consistent error handling across all of them, and keeps information flow controlled and auditable, at the cost of making the coordinator a bottleneck if it is not designed to delegate effectively.
  Example: Anthropic's Research feature uses a lead agent that spawns web-search and document-analysis subagents; those subagents never communicate with each other — each reports its findings back only to the lead agent, which decides what happens next.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

- Concept: Subagent Context Isolation
  Type: key
  Teaches: 1.2-K2
  Definition: A subagent's context window starts fresh: it does not automatically inherit the coordinator's conversation history, prior tool calls, or tool results. The only channel from coordinator to subagent is the prompt string passed when the subagent is invoked (plus its own system prompt/`AgentDefinition` and, unless disabled, project `CLAUDE.md`). This isolation is a deliberate design choice — it is what lets subagents explore verbose content (dozens of files, long search results) without any of that bulk accumulating in the coordinator's own context.
  Example: A `research-assistant` subagent can read 40 source documents while investigating a topic; none of those 40 documents' raw content enters the coordinator's context — only the subagent's final summary does.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents

- Concept: Coordinator Responsibilities — Decomposition, Delegation, Aggregation, Dynamic Subagent Selection
  Type: key
  Teaches: 1.2-K3, 1.2-S1
  Definition: The coordinator is responsible for the whole lifecycle of a request: breaking it into subtasks (decomposition), assigning subtasks to the right subagent(s) (delegation), and combining subagent outputs into a coherent final result (aggregation). A well-designed coordinator also analyzes each incoming query's complexity and decides which subagents are actually needed for it, rather than always routing every request through the same fixed pipeline of every available subagent — a simple factual query might need one subagent, while a broad comparative research question might need five.
  Example: A coordinator receiving "What is the capital of France?" invokes no subagents and answers directly, while a coordinator receiving "Compare the semiconductor supply chains of three countries" spawns three or more subagents scoped to distinct sub-questions.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

- Concept: Risks of Overly Narrow Task Decomposition
  Type: key
  Teaches: 1.2-K4
  Definition: When a coordinator gives subagents insufficiently scoped or insufficiently detailed task boundaries, the result is not efficient parallel coverage but duplicated work, coverage gaps, or subagents that fail to find necessary information — because each subagent, operating in isolation, cannot see what the others are doing and has no way to detect overlap or gaps on its own. Vague task assignments (e.g., "research the semiconductor shortage" with no further guidance) are the direct cause; the fix is giving every subagent an explicit objective, output format, tool/source guidance, and task boundaries.
  Example: A lead agent split "semiconductor shortage" into three subagents with vague scopes; one investigated the 2021 automotive chip crisis while two others redundantly investigated 2025 supply chains — narrow, ambiguous decomposition produced duplicate work and left other angles (e.g., the 2021 crisis's downstream effects) uncovered.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

- Concept: Partitioning Research Scope Across Subagents to Minimize Duplication
  Type: key
  Teaches: 1.2-S2
  Definition: To avoid the duplication risk above, a coordinator assigns each subagent a distinct, non-overlapping slice of the problem — a specific subtopic, source type, or time period — rather than letting multiple subagents independently pursue the same broad question. Effective partitioning requires the coordinator to give each subagent an explicit objective, expected output format, guidance on which tools/sources to use, and clear boundaries on what is and is not in scope for that subagent.
  Example: Instead of three subagents each told to "research the semiconductor shortage," the coordinator assigns one to "2021 automotive chip crisis root causes," one to "2024-2026 supply chain diversification efforts," and one to "policy responses (CHIPS Act and equivalents)" — three clearly bounded, non-overlapping slices.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

- Concept: Iterative Refinement Loop (Coordinator Re-delegation)
  Type: key
  Teaches: 1.2-S3
  Definition: After subagents report back and a synthesis step produces a draft result, the coordinator evaluates that output for gaps or weaknesses, and — if coverage is insufficient — re-delegates: it spawns additional search/analysis subagents with more targeted queries addressing the specific gap, then re-invokes synthesis. This loop repeats until the coordinator judges coverage sufficient, rather than accepting the first synthesis pass unconditionally.
  Example: A first synthesis pass on "AI regulation trends" lacks any EU-specific detail; the coordinator notices the gap, spawns a new subagent scoped to "EU AI Act provisions," and re-runs synthesis once that subagent reports back.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

---

## Task Statement 1.3: Configure subagent invocation, context passing, and spawning

- Concept: The Task/Agent Tool and the `allowedTools` Requirement
  Type: key
  Teaches: 1.3-K1
  Definition: Subagents are spawned through a dedicated tool — named `Task` in the exam guide's terminology and in the `system:init` tools list, and surfaced as `"Agent"` in `tool_use` blocks on current SDK versions (older SDK versions emit `"Task"` in both places; code that detects subagent invocation should match either name for compatibility). For a coordinator to be able to invoke subagents at all, this tool must be present in its `allowedTools`/`allowed_tools` configuration — a coordinator without `"Task"`/`"Agent"` in its allowed tools cannot spawn subagents, regardless of how its `AgentDefinition`s are configured.
  Example: `ClaudeAgentOptions(allowed_tools=["Read", "Grep", "Glob", "Agent"], agents={...})` — omitting `"Agent"` from that list means Claude can never invoke any of the defined subagents even though they exist in `agents`.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents; naming corroborated by runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 190-191

- Concept: Explicit Context Passing to Subagents
  Type: key
  Teaches: 1.3-K2, 1.3-S1
  Definition: Because a subagent's context window starts fresh (see Subagent Context Isolation), any information the subagent needs — prior findings, file paths, error messages, decisions already made — must be written directly into the prompt string passed to the `Task`/`Agent` tool call. There is no automatic inheritance and no shared memory between separate subagent invocations; two calls to the same subagent type do not share state unless a fork or an explicit resume is used. Complete findings from prior agents (e.g., full web search results and document analysis output) must be included verbatim in the prompt given to a downstream subagent such as a synthesis agent, not merely referenced.
  Example: When invoking a synthesis subagent, the coordinator's prompt includes the full text of the web-search subagent's findings and the full text of the document-analysis subagent's findings pasted directly into the prompt — not a pointer like "see the earlier search results," which the synthesis subagent has no way to resolve.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents

- Concept: `AgentDefinition` Configuration
  Type: key
  Teaches: 1.3-K3
  Definition: Subagents defined programmatically are configured through an `AgentDefinition` (or its filesystem equivalent, a markdown file in `.claude/agents/`) with, at minimum, a `description` (natural-language text Claude uses to decide when to invoke this subagent automatically) and a `prompt` (the subagent's own system prompt defining its role, expertise, and behavior). Optional fields include `tools` (restricting which tools the subagent can use — if omitted, it inherits every tool available to subagents), `model` (overriding which model runs the subagent), and several others (`disallowedTools`, `skills`, `maxTurns`, `background`, etc.).
  Example: `AgentDefinition(description="Expert code review specialist. Use for quality, security, and maintainability reviews.", prompt="You are a code review specialist...", tools=["Read","Grep","Glob"], model="sonnet")` defines a read-only reviewer subagent that Claude will invoke automatically when a task matches its description.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents

- Concept: Structured Data Formats Separating Content from Metadata
  Type: key
  Teaches: 1.3-S2
  Definition: When subagents pass findings up to the coordinator or across to another subagent, mixing narrative content and attribution metadata (source URL, document name, page number, publication date) into a single unstructured blob makes it easy for that attribution to get lost or garbled during later summarization. The fix is to require subagents to output findings in a structured format — e.g., a claim plus an evidence excerpt plus a separate source-URL/document-name field — so that content and metadata travel together but distinctly, and downstream agents (like a dedicated citation-processing agent) can reliably pair the two back up.
  Example: Anthropic's research system routes all findings through a dedicated CitationAgent that processes the documents and the research report together to identify the specific source location for every claim — a step only possible because each finding was captured with its claim and its source metadata as separate, structured fields rather than merged prose.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

- Concept: Parallel Subagent Spawning in a Single Turn
  Type: key
  Teaches: 1.3-S3
  Definition: A coordinator can spawn multiple subagents to run concurrently by emitting several `Task`/`Agent` tool calls within a single response, rather than issuing one call, waiting for its result, and only then issuing the next. Because independent subtasks running in parallel finish in the time of the slowest one rather than the sum of all of them, this yields substantial latency reduction for workloads made of genuinely independent subtasks. Anthropic's research system spawns 3-5 subagents in parallel (with each subagent itself using 3+ tools in parallel), and reports this cut research time by up to 90% for complex queries compared to serial execution.
  Example: A coordinator investigating "compare React, Vue, and Svelte adoption trends" emits three `Task` calls in one turn — one subagent per framework — instead of researching React, waiting for the result, then starting Vue.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system; parallel tool execution mechanics corroborated by https://code.claude.com/docs/en/agent-sdk/agent-loop

- Concept: Goal-Oriented Coordinator Prompts (vs Step-by-Step Procedural Instructions)
  Type: key
  Teaches: 1.3-S4
  Definition: Coordinator prompts for subagents should specify what a good outcome looks like — an objective, an output format, and quality criteria to evaluate against — rather than a rigid, literal sequence of steps to follow. Declarative (goal-based) instructions let a subagent adapt its approach to what it actually discovers along the way, while procedural (step-based) instructions break as soon as reality diverges from the anticipated sequence. The contract still needs to be concrete, though: Anthropic's guidance is that a subagent's task description needs an objective, an output format, guidance on which tools/sources to use, and clear task boundaries — omitting any of these four causes the subagent to drift, even though none of them is a literal step-by-step procedure.
  Example: "Investigate whether Company X has faced regulatory action in the past 3 years; report findings as a list of {date, regulator, outcome, source} entries; only use primary regulatory filings and major news outlets" is goal-oriented and adaptable. "First search for 'Company X lawsuit', then open the third result, then search for 'Company X SEC'..." is procedural and brittle.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system

---

## Task Statement 1.4: Implement multi-step workflows with enforcement and handoff patterns

- Concept: Programmatic Enforcement vs Prompt-Based Guidance for Workflow Ordering
  Type: key
  Teaches: 1.4-K1, 1.4-K2, 1.5-K3, 1.5-S3
  Definition: There are two fundamentally different ways to make an agent follow a required order of operations: prompt-based guidance (instructing Claude in its system prompt, e.g. "always verify identity before processing refunds") relies on the model reliably following that instruction every time, and prompt instructions alone have a non-zero failure rate — the model can occasionally skip, misorder, or forget a stated rule under pressure from other context. Programmatic enforcement (hooks, prerequisite gates evaluated in code before a tool executes) makes compliance deterministic: the enforcement runs outside the model's own reasoning and cannot be argued around or forgotten. When a business rule requires guaranteed compliance — such as verifying identity before any financial operation — the deterministic mechanism (hooks/gates), not prompt wording, is the correct choice.
  Example: A prompt instruction "never process a refund before verifying the customer" can fail if the model, deep in a long multi-turn conversation, loses track of whether verification already happened. A `PreToolUse` hook that programmatically checks whether `get_customer` has returned a verified customer ID before allowing `process_refund` to execute cannot fail this way — it either sees a verified ID in state or it doesn't.
  Source: https://code.claude.com/docs/en/agent-sdk/hooks; deterministic vs probabilistic framing corroborated by runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 210-214, 234-236

- Concept: Programmatic Prerequisite Gates (Blocking Downstream Tool Calls)
  Type: key
  Teaches: 1.4-S1
  Definition: A prerequisite gate is a `PreToolUse` hook (or equivalent permission rule) that inspects the tool being requested and the state accumulated so far, and denies the call outright if a required earlier step has not completed. Because hooks are evaluated before deny rules, ask rules, permission mode, and allow rules in the SDK's permission evaluation order, a hook's denial cannot be overridden by any looser setting elsewhere in the configuration — this is what makes the gate deterministic rather than advisory.
  Example: A `PreToolUse` hook matching `process_refund` checks whether a `verified_customer_id` has been set in session state by a prior `get_customer` call; if not, it returns `permissionDecision: "deny"` with a reason explaining that `get_customer` must run first, and Claude receives that denial as the tool result rather than being allowed to process the refund.
  Source: https://code.claude.com/docs/en/agent-sdk/permissions; hook mechanics from https://code.claude.com/docs/en/agent-sdk/hooks

- Concept: Multi-Concern Request Decomposition with Parallel Investigation
  Type: key
  Teaches: 1.4-S2
  Definition: A customer message that raises several distinct issues at once (e.g., "my order arrived damaged AND I was charged twice") should be decomposed into its constituent items rather than handled as one undifferentiated request. Each item is then investigated using the same shared context (so facts discovered while investigating one concern — like the verified customer ID — don't need to be re-established for the next), and the individual findings are synthesized into a single unified resolution presented to the customer, rather than several disconnected partial answers.
  Example: For "my order arrived damaged and I was charged twice," the agent investigates the damage claim (checking order status, damage policy) and the double-charge claim (checking billing records) using the same already-verified customer context, then presents one combined resolution: a replacement item plus a duplicate-charge refund.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 222-223 (Task Statement 1.4 Skills bullet, verbatim)

- Concept: Structured Handoff Summaries for Human Escalation
  Type: key
  Teaches: 1.4-K3, 1.4-S3
  Definition: When an agent escalates mid-process to a human, that human typically has no access to the full conversation transcript, so an unstructured "here's what happened" note is insufficient. A structured handoff protocol compiles the specific facts a human agent needs to act immediately: customer identifying details, a root-cause analysis of the issue, any relevant amounts (e.g., a proposed refund amount), and a recommended action — organized as discrete fields rather than freeform prose, so the receiving human doesn't have to reconstruct the situation from scratch.
  Example: An escalation summary reading `{customer_id: "C-4471", root_cause: "Shipping carrier lost package per tracking API", refund_amount: "$89.99", recommended_action: "Approve full refund; carrier claim already filed"}` lets a human agent act in seconds, versus a transcript dump the human would have to read end-to-end.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 215-216, 224-225 (Task Statement 1.4 Knowledge and Skills bullets, verbatim)

---

## Task Statement 1.5: Apply Agent SDK hooks for tool call interception and data normalization

- Concept: `PostToolUse` Hooks for Tool Result Transformation and Normalization
  Type: key
  Teaches: 1.5-K1, 1.5-S1
  Definition: A `PostToolUse` hook fires after a tool returns its result and, before that result reaches the model, can inspect and rewrite it. This is the mechanism for normalizing heterogeneous data formats coming from different tools or MCP servers — e.g., converting Unix timestamps and ISO 8601 strings to one consistent date format, or mapping numeric status codes and string statuses to one vocabulary — so that Claude reasons over a single consistent representation instead of having to infer equivalences itself. The hook's callback returns `hookSpecificOutput` with `updatedToolOutput` (or the deprecated MCP-only `updatedMCPToolOutput`) to replace the tool's output before Claude sees it, or `additionalContext` to append information alongside the original result.
  Example: A `PostToolUse` hook matched to `mcp__orders__*` and `mcp__billing__*` converts every timestamp field it finds to ISO 8601 and every status field to a shared enum (`"shipped" | "delivered" | "pending"`) before the combined order-and-billing data reaches Claude, eliminating the need for the model to reconcile `status_code: 3` against `status: "SHIPPED"` itself.
  Source: https://code.claude.com/docs/en/agent-sdk/hooks

- Concept: `PreToolUse` Hooks for Compliance Interception
  Type: key
  Teaches: 1.5-K2, 1.5-S2
  Definition: A `PreToolUse` hook fires before a tool call executes and can inspect the tool name and its proposed input to enforce a business rule, returning `permissionDecision: "deny"` (with a `permissionDecisionReason` explaining why, so the model doesn't retry the same call) to block a policy-violating action outright. Beyond simply denying, a hook can also redirect: its denial reason can instruct Claude toward an alternative path, such as escalating to a human, rather than merely failing silently.
  Example: A `PreToolUse` hook matched to `process_refund` inspects `tool_input.amount`; if it exceeds $500, the hook returns `permissionDecision: "deny"` with reason "Refunds over $500 require human approval — call escalate_to_human instead," which both blocks the over-threshold refund and steers the model toward the correct alternative workflow.
  Source: https://code.claude.com/docs/en/agent-sdk/hooks

---

## Task Statement 1.6: Design task decomposition strategies for complex workflows

- Concept: Prompt Chaining — Fixed Sequential Decomposition
  Type: key
  Teaches: 1.6-K1, 1.6-S1
  Definition: Prompt chaining decomposes a task into a sequence of fixed steps, where each LLM call processes the output of the previous one, optionally with programmatic checks inserted between steps to validate progress before continuing. It is the right choice when a task can be cleanly and predictably broken into known subtasks in advance — the sequence of stages doesn't depend on what earlier stages discover, only their content does. It trades some latency (multiple sequential calls) for higher accuracy, since each individual call is a simpler, more focused task than the whole problem at once.
  Example: A predictable multi-aspect code review pipeline: step 1 analyzes each changed file individually for local issues; step 2 (a separate LLM call) takes the outputs of step 1 and performs a cross-file integration pass looking for issues that only show up when files are considered together. The two stages are fixed and always run in that order.
  Source: https://www.anthropic.com/research/building-effective-agents

- Concept: Dynamic Adaptive Decomposition (Orchestrator-Workers Pattern)
  Type: key
  Teaches: 1.6-K1, 1.6-K3, 1.6-S1
  Definition: Dynamic decomposition uses a central (orchestrator/coordinator) LLM call to determine subtasks at run time based on the specific input, rather than following a pre-fixed sequence — the key difference from prompt chaining is that the subtasks themselves aren't known in advance. This suits open-ended investigation tasks where the right next step depends on what was just discovered: an adaptive investigation plan generates its next subtasks based on the findings of the current ones, rather than committing to a full plan upfront.
  Example: Software refactoring across an unfamiliar codebase can't have its exact file list and change list predicted before investigation begins — the orchestrator discovers which files matter and what changes are needed as it goes, generating new subtasks (or new subagent delegations) in response to what earlier steps found.
  Source: https://www.anthropic.com/research/building-effective-agents; corroborated by https://www.anthropic.com/engineering/multi-agent-research-system (lead agent "decides whether more research is needed" and adapts its strategy)

- Concept: Per-File Local Analysis Plus Cross-File Integration Pass (Avoiding Attention Dilution)
  Type: key
  Teaches: 1.6-K2, 1.6-S2
  Definition: A single LLM call asked to review many files at once at high depth tends to produce inconsistent depth across files and miss issues — attention dilution, where the model's effective attention per file drops as the total input grows. The prompt-chaining fix specific to code review is to split the work into two passes: a local pass that analyzes each file individually (so each file gets full attention), followed by a separate cross-file integration pass that looks specifically for issues that only emerge from how files interact (e.g., a changed function signature in one file that a caller in another file no longer matches).
  Example: A 40-file pull request is reviewed as 40 independent per-file passes (each flags local issues like unhandled exceptions or unclear naming) plus one final pass given a summary of all 40 files' changes together, specifically tasked with finding integration problems like an API contract mismatch between two of the changed files.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 250-251, 258-259 (Task Statement 1.6 Knowledge and Skills bullets, verbatim)

- Concept: Adaptive Investigation Planning for Open-Ended Tasks
  Type: key
  Teaches: 1.6-S3
  Definition: For an open-ended task with no predefined subtask list (e.g., "add comprehensive tests to a legacy codebase"), an effective decomposition strategy is staged and adaptive rather than fully planned upfront: first map the codebase's structure (directories, modules, dependencies), then identify high-impact areas (most-used modules, historically buggy code, untested critical paths) from that map, then create a prioritized plan — and let that plan continue to adapt as dependencies and complications are discovered during execution, rather than treating the initial plan as fixed.
  Example: Told to add tests to an undocumented legacy service, an agent first explores the directory tree and import graph to build a structural map, identifies the payment-processing module as high-impact (heavily depended-upon, currently untested), prioritizes it first in the test plan, and then — upon discovering mid-task that the payment module has an undocumented dependency on a deprecated internal library — adjusts the remaining plan to address that dependency before continuing.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 260-262 (Task Statement 1.6 Skills bullet, verbatim); general pattern corroborated by https://www.anthropic.com/research/building-effective-agents

---

## Task Statement 1.7: Manage session state, resumption, and forking

- Concept: Named Session Resumption (`--resume`)
  Type: key
  Teaches: 1.7-K1, 1.7-S1
  Definition: Resuming a session takes a specific session identifier — including a human-assigned name, via the CLI's `--resume <session-name>` — and continues that exact prior conversation with its full accumulated context (files read, analysis performed, actions taken) restored. This differs from "continue," which always picks up the single most-recently-used session in the current directory with no name or ID required: resume is necessary whenever an application must return to one specific session among several (e.g., one session per user, or one named investigation thread revisited across separate work sessions), rather than only ever the latest one.
  Example: A developer runs `claude --resume auth-refactor-investigation` days after starting that named session to continue exactly where that specific investigation left off, even though several other unrelated sessions have run in the same directory since.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions; CLI flag syntax corroborated by runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 267-268

- Concept: Session Forking for Divergent Exploration (`fork_session`)
  Type: key
  Teaches: 1.3-K4, 1.7-K2, 1.7-S2
  Definition: Forking creates a new, independent session that starts as a copy of an existing session's full history, from that shared baseline; the original session's ID and history are left completely unchanged, so both the fork and the original can be resumed and extended separately afterward. This is the mechanism for exploring divergent approaches from a common analysis point — e.g., having already understood a codebase or a research topic once, you can branch into two (or more) different directions without re-doing the shared upfront work and without either branch contaminating the other.
  Example: After a session has fully analyzed an authentication module (baseline), forking it lets one branch explore "how would this look with JWT?" while the original session is separately resumed to explore "how would this look with OAuth2?" — both branches share the same initial analysis but diverge independently from there, and neither affects the other's history.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions

- Concept: Informing Resumed Sessions About File Changes
  Type: key
  Teaches: 1.7-K3, 1.7-S4
  Definition: A session persists conversation history — not the filesystem. If files the agent previously analyzed have been modified outside that session (by a teammate, a merge, or a separate tool run) since the session last saw them, the agent's in-context understanding of those files is now stale, and it will reason from outdated assumptions unless explicitly told what changed. The fix is to inform the resumed session about the specific files that changed (and ideally how), enabling targeted re-analysis of just those files rather than requiring the agent to re-explore the entire codebase from scratch.
  Example: Resuming a session that previously analyzed `payment.py` and `orders.py` with the prompt "Note: `payment.py` was modified since you last read it to add idempotency keys — re-check your earlier refactoring plan against the new version" lets the agent re-read only that one file and adjust, instead of blindly proceeding on stale assumptions or re-reading the entire codebase.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions ("Sessions persist the conversation, not the filesystem"); scenario corroborated by runs/claude-certified-architect-foundations/sources/exam-guide.txt, lines 271-272, 282-283

- Concept: Fresh Session with Structured Summary vs Resuming with Stale Tool Results
  Type: key
  Teaches: 1.7-K4, 1.7-S3
  Definition: Resuming a session is not always the more reliable choice. When the prior session's accumulated tool results (file contents, search results, API responses) are likely stale or the transcript is too large/noisy to be worth carrying forward, it is often more robust to start a fresh session and inject a concise, structured summary of what was learned (key facts, decisions, file paths) directly into the new prompt, rather than resuming and hoping the model correctly discounts outdated tool output buried in a long history. The choice is a judgment call: resume when prior context is mostly still valid; start fresh with an injected summary when it is not.
  Example: After a long exploratory session whose tool outputs (old file contents, since-changed search results) are now mostly obsolete, the developer instead starts a new session with a prompt like "Here is what we established previously: [structured summary of key facts and decisions]. Proceed from here," avoiding the risk that the agent treats stale tool results in a resumed transcript as still current.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions ("Don't rely on session resume ... often more robust than shipping transcript files around")

---

## Coverage summary

| Task Statement | Bullets | Concepts teaching them |
|---|---|---|
| 1.1 | K1,K2,K3,S1,S2,S3 (6) | Agentic Loop Lifecycle; Appending Tool Results; Model-Driven vs Decision Trees; Termination Anti-Patterns |
| 1.2 | K1,K2,K3,K4,S1,S2,S3,S4 (8) | Hub-and-Spoke; Context Isolation; Coordinator Responsibilities; Risks of Narrow Decomposition; Partitioning Scope; Iterative Refinement Loop |
| 1.3 | K1,K2,K3,K4,S1,S2,S3,S4 (8) | Task/Agent Tool & allowedTools; Explicit Context Passing; AgentDefinition; Session Forking; Structured Data Formats; Parallel Spawning; Goal-Oriented Prompts |
| 1.4 | K1,K2,K3,S1,S2,S3 (6) | Programmatic Enforcement vs Prompt Guidance; Prerequisite Gates; Multi-Concern Decomposition; Structured Handoff Summaries |
| 1.5 | K1,K2,K3,S1,S2,S3 (6) | PostToolUse Normalization; PreToolUse Compliance Interception; Programmatic Enforcement vs Prompt Guidance |
| 1.6 | K1,K2,K3,S1,S2,S3 (6) | Prompt Chaining; Dynamic Adaptive Decomposition; Per-File + Cross-File Passes; Adaptive Investigation Planning |
| 1.7 | K1,K2,K3,K4,S1,S2,S3,S4 (8) | Named Session Resumption; Session Forking; Informing Resumed Sessions of File Changes; Fresh Session vs Stale Resumption |

Total bullets: 6+8+8+6+6+6+8 = 48. All 48 covered.
