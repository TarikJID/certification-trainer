# Research: Agentic Architecture & Orchestration (Domain 1, 27%)

Bullets received: 48 (Task 1.1: 6, 1.2: 8, 1.3: 8, 1.4: 6, 1.5: 6, 1.6: 6, 1.7: 8)
Bullets covered: 48 / 48 (every bullet is named in at least one concept's `Teaches:` field)

---

## Prerequisite concepts

- Concept: Claude Messages API tool-use contract (`tool_use` / `tool_result` blocks)
  Type: prerequisite
  Teaches: supports 1.1-K1, 1.1-K2, 1.1-S1, 1.1-S2, 1.1-S3
  Definition: Tool use is a contract between an application and Claude: the application defines tool schemas, Claude decides when and how to call them, and the application executes the operation. When Claude wants to use a tool, the API response contains one or more `tool_use` content blocks (tool name + JSON input). The calling application executes the tool and sends the output back as a `tool_result` block in the next request. Claude never executes code itself — it only emits structured requests that the client fulfills.
  Example: A weather tool is defined with a JSON schema for `location`. Claude's response contains a `tool_use` block `{"name": "get_weather", "input": {"location": "Boston"}}`; the application calls the weather API and returns `{"type": "tool_result", "tool_use_id": "...", "content": "68°F, sunny"}` in the next request.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works (tier 2, official Claude Platform Docs)

- Concept: Claude Agent SDK `query()` / `ClaudeAgentOptions` (agent options object)
  Type: prerequisite
  Teaches: supports 1.3-K1, 1.3-K3, 1.5-K1, 1.5-K2, 1.5-S1, 1.5-S2, 1.7-K1, 1.7-K2, 1.7-S1, 1.7-S2
  Definition: The Claude Agent SDK's `query()` function (TypeScript) or `ClaudeSDKClient`/`query()` (Python) is the entry point for running an agent session. It accepts an options object (`ClaudeAgentOptions` in Python, `Options` in TypeScript) that configures `allowedTools`, `agents` (subagent definitions), `hooks`, and session controls such as `resume` and `fork_session`. This options object is the mechanism through which subagents, hooks, and session behavior are all wired together.
  Example: `query(prompt="Review the auth module", options=ClaudeAgentOptions(allowed_tools=["Read","Grep","Agent"], agents={...}, hooks={...}))` configures a single call with subagents and hooks enabled at once.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents (tier 2, official Claude Code / Agent SDK Docs)

- Concept: Sessions as persisted conversation history
  Type: prerequisite
  Teaches: supports 1.3-K4, 1.7-K1, 1.7-K2, 1.7-K4, 1.7-S1, 1.7-S2, 1.7-S3
  Definition: "A session is the conversation history the SDK accumulates while your agent works. It contains your prompt, every tool call the agent made, every tool result, and every response. The SDK writes it to disk automatically so you can return to it later." Sessions are identified by a session ID captured from the result message, and this ID is the handle used by `resume`, `continue`, and `fork_session`.
  Example: After a `query()` call finishes, the application reads `session_id` from the `ResultMessage` and stores it so a later process can pass it back in as `resume=session_id` to continue the same conversation.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions (tier 2, official Claude Code / Agent SDK Docs)

- Concept: Heterogeneous data formats from MCP tools
  Type: prerequisite
  Teaches: supports 1.5-K1, 1.5-S1
  Definition: Different backend systems exposed through Model Context Protocol (MCP) tools often return the same kind of information (e.g., timestamps or status) in inconsistent formats — one tool might return a Unix timestamp, another ISO 8601, another a numeric status code — because each MCP server wraps a different underlying system with its own conventions. An agent reasoning directly over these mixed formats is prone to misinterpretation (e.g., comparing a Unix timestamp to an ISO date string).
  Example: `get_customer` returns `created_at: 1699999999` (Unix time) while `lookup_order` returns `order_date: "2023-11-14T10:00:00Z"` (ISO 8601); without normalization, the agent must reconcile two different date encodings itself.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.5 Skills-S1 (tier 1, official exam guide)

- Concept: The augmented LLM (retrieval, tools, memory)
  Type: prerequisite
  Teaches: supports 1.6-K1, 1.6-K3, 1.6-S1, 1.6-S3
  Definition: "The basic building block of agentic systems is an LLM enhanced with augmentations such as retrieval, tools, and memory." Current models can actively use these augmentations themselves — generating their own search queries, selecting appropriate tools, and deciding what to retain — and this augmented LLM is the substrate on which both fixed workflow patterns (like prompt chaining) and fully autonomous agent loops are built.
  Example: A code-review LLM augmented with a `read_file` tool and a memory of prior findings can decide, call by call, which file to open next, rather than following a hardcoded file list.
  Source: https://www.anthropic.com/engineering/building-effective-agents (tier 2, official Anthropic engineering blog)

---

## Task Statement 1.1 — Design and implement agentic loops for autonomous task execution

- Concept: The agentic loop (client-tool loop)
  Type: key
  Teaches: 1.1-K1, 1.1-S1
  Definition: The agentic loop is the canonical control-flow pattern for client-executed tools: (1) send a request with the `tools` array and the user message; (2) Claude responds with `stop_reason: "tool_use"` and one or more `tool_use` blocks; (3) execute each tool and format outputs as `tool_result` blocks; (4) send a new request containing the original messages, the assistant's response, and a user message with the `tool_result` blocks; (5) repeat from step 2 while `stop_reason` is `"tool_use"`. The loop exits on any other stop reason (most commonly `"end_turn"`), meaning Claude has produced a final answer.
  Example: A support agent loop calls `get_customer`, receives a `tool_use` stop reason, executes the lookup, appends the `tool_result`, and re-sends the conversation; Claude then calls `lookup_order`, and the cycle repeats until Claude returns `end_turn` with a final message to the customer.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works (tier 2, official Claude Platform Docs)

- Concept: `stop_reason`-driven loop termination
  Type: key
  Teaches: 1.1-K1, 1.1-S1
  Definition: `stop_reason` is the single source of truth for agentic loop control. The relevant values are `"tool_use"` (Claude is requesting a tool call — execute and continue the loop) and `"end_turn"` (Claude finished naturally — return the response as final). Other values (`max_tokens`, `stop_sequence`, `pause_turn`, `refusal`) require their own handling but are not the primary loop-continuation signal. Control flow must branch on this field rather than on any other signal.
  Example: `while response.stop_reason == "tool_use": execute tools; append results; response = client.messages.create(...)` — the loop naturally falls through to return the response once `stop_reason` becomes `"end_turn"`.
  Source: https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons (tier 2, official Claude Platform Docs)

- Concept: Tool-result context accumulation between iterations
  Type: key
  Teaches: 1.1-K2, 1.1-S2
  Definition: Each iteration of the agentic loop must append both the assistant's tool-use response and the corresponding `tool_result` block(s) to the running message list before the next request. This is what allows Claude to "reason about the next action": the model only knows what happened in a prior tool call because that call's result is explicitly present in the conversation history sent on the next turn. Omitting or truncating this step breaks the model's ability to build on previous findings.
  Example: `messages.append({"role": "assistant", "content": response.content}); messages.append({"role": "user", "content": tool_results})` — sent as-is (with no extra assistant text after the tool_result) on the following request so Claude can incorporate the new information.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works (tier 2, official Claude Platform Docs)

- Concept: Model-driven decision-making vs. pre-configured workflows ("workflows" vs. "agents")
  Type: key
  Teaches: 1.1-K3
  Definition: Anthropic distinguishes two architectures for LLM systems. "Workflows" are "systems where LLMs and tools are orchestrated through predefined code paths" — the sequence of steps and tool calls is fixed by the developer in advance. "Agents," by contrast, are "systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks" — the model itself reasons, at each step, about which tool to call next based on the current context, rather than following a scripted decision tree.
  Example: A workflow might always call `validate_input` → `transform` → `save` in that fixed order for every request. An agent given the same three tools instead decides per-request whether `validate_input` is even needed, and may loop back to `transform` again if `save` fails — because the model, not the code, is choosing the next action.
  Source: https://www.anthropic.com/engineering/building-effective-agents (tier 2, official Anthropic engineering blog)

- Concept: Anti-patterns in agentic loop control flow
  Type: key
  Teaches: 1.1-S3
  Definition: The Claude Platform docs identify specific anti-patterns to avoid when building an agentic loop: (1) parsing the model's natural-language text to infer whether it wants to call a tool, instead of checking `stop_reason == "tool_use"`; (2) using an arbitrary hardcoded iteration cap (e.g., `for i in range(10)`) as the primary stopping mechanism, which can truncate the conversation mid-task; and (3) treating the presence of assistant text content as a completion signal, since Claude can emit text alongside a `tool_use` block on a turn that is not yet finished. The loop should terminate only when `stop_reason` indicates completion (e.g., `end_turn`), with iteration caps used only as a safety net, not the primary control mechanism.
  Example: Code that does `if "I will now call" in response.text: call_tool()` is fragile and wrong; code that does `if response.stop_reason == "tool_use": execute_tools()` is correct and matches the documented pattern.
  Source: https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons (tier 2, official Claude Platform Docs)

---

## Task Statement 1.2 — Orchestrate multi-agent systems with coordinator-subagent patterns

- Concept: Hub-and-spoke coordinator architecture
  Type: key
  Teaches: 1.2-K1, 1.2-S4
  Definition: In a hub-and-spoke multi-agent architecture, a single coordinator ("hub") agent manages all inter-subagent communication, error handling, and information routing; subagents ("spokes") never communicate directly with one another. All output from a subagent returns to the coordinator, and any information one subagent needs from another's findings must be relayed by the coordinator. This centralization gives the coordinator complete observability into the system, consistent error handling in one place, and controlled information flow, at the cost of the coordinator becoming a routing bottleneck if designed poorly.
  Example: A web-search subagent's findings do not go directly to the synthesis subagent; they return to the coordinator, which then explicitly includes them in the prompt it constructs when it spawns the synthesis subagent.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.2 Knowledge-K1 (tier 1, official exam guide); corroborated by the orchestrator/lead-agent role in https://www.anthropic.com/engineering/multi-agent-research-system (tier 2, official Anthropic engineering blog)

- Concept: Subagent context isolation and explicit context passing
  Type: key
  Teaches: 1.2-K2, 1.3-K2, 1.3-S1
  Definition: Subagents run in their own conversation and do not automatically inherit the coordinator's (or any other subagent's) conversation history, tool results, or system prompt. "Unless the subagent is a fork, its context window starts fresh, with no parent conversation... The only content you pass from parent to subagent is the [Agent tool's] prompt string, so include any file paths, error messages, or decisions the subagent needs directly in that prompt." Because subagents also do not share memory between separate invocations, any findings from a prior agent (e.g., web-search results, document-analysis output) must be copied explicitly into the next subagent's prompt text — nothing is inherited implicitly.
  Example: To have a synthesis subagent build on a search subagent's results, the coordinator's prompt to the synthesis subagent must literally contain the search results as text (e.g., "Here are the search findings: ..."), not just a reference like "use the results from before."
  Source: https://code.claude.com/docs/en/agent-sdk/subagents, section "What subagents inherit" (tier 2, official Claude Code / Agent SDK Docs)

- Concept: Coordinator role — decomposition, delegation, aggregation, and dynamic subagent selection
  Type: key
  Teaches: 1.2-K3, 1.2-S1
  Definition: The coordinator (lead agent) is responsible for the full lifecycle of a multi-agent task: it analyzes the incoming query, decides an overall strategy, decomposes the task into subtasks, delegates each subtask to an appropriate subagent, and aggregates/synthesizes the subagents' results into a final answer. Rather than always invoking every available subagent through a fixed pipeline, a well-designed coordinator analyzes query complexity and dynamically decides which subagents are actually needed for a given request.
  Example: For a simple fact-finding query the coordinator might spawn a single search subagent, while for a broad comparative research query it spawns several subagents each covering a distinct facet — the coordinator picks the number and type of subagents based on what the specific query requires rather than always running the full four-agent pipeline.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system (tier 2, official Anthropic engineering blog); corroborated by runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.2 Knowledge-K3 (tier 1)

- Concept: Risks of overly narrow task decomposition
  Type: key
  Teaches: 1.2-K4
  Definition: When a coordinator decomposes a broad topic into subtasks that are individually well-executed but collectively too narrow, the resulting output can have systematic coverage gaps even though every subagent "succeeded" at its assigned slice. Anthropic's engineering write-up on its research system found that vague, high-level task instructions to subagents "often were vague enough that subagents misinterpreted the task or performed the exact same searches as other agents" — and, symmetrically, an overly narrow set of subtask assignments can omit entire relevant sub-domains of the topic even though each individual subtask is completed correctly.
  Example: A coordinator asked to research "the impact of AI on creative industries" that only spawns subagents for "AI in digital art," "AI in graphic design," and "AI in photography" produces a report that is silent on music, writing, and film — not because any subagent failed, but because the decomposition itself never assigned those domains to anyone.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.2 Knowledge-K4 and Sample Question 7 (tier 1, official exam guide); corroborated by https://www.anthropic.com/engineering/multi-agent-research-system (tier 2)

- Concept: Partitioning research scope across subagents to minimize duplication
  Type: key
  Teaches: 1.2-S2
  Definition: To avoid subagents redundantly covering the same ground, the coordinator should assign each subagent a distinct, clearly bounded slice of the task — a specific subtopic, source type, or comparison target — rather than issuing the same broad instruction to every subagent. Anthropic's system uses scaling rules tied to task complexity: "Simple fact-finding requires just 1 agent with 3-10 tool calls, direct comparisons might need 2-4 subagents with 10-15 calls each, and complex research might use more than 10 subagents with clearly divided responsibilities."
  Example: Instead of telling three subagents "research renewable energy trends," the coordinator assigns one subagent "solar policy in the EU," a second "battery storage costs," and a third "wind power adoption in Asia" — three non-overlapping slices of one broad topic.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system (tier 2, official Anthropic engineering blog); corroborated by runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.2 Skills-S2 (tier 1)

- Concept: Iterative refinement loop (coordinator evaluates and re-delegates)
  Type: key
  Teaches: 1.2-S3
  Definition: Rather than treating a single pass of delegate-then-synthesize as final, the coordinator can evaluate synthesis output for coverage gaps, re-delegate targeted follow-up queries to search/analysis subagents to fill those gaps, and re-invoke synthesis — repeating until coverage is judged sufficient. This mirrors the general "evaluator-optimizer" workflow pattern, in which "one LLM call generates a response while another provides evaluation and feedback in a loop," and works best "when it uses clear evaluation criteria, and when iterative refinement... provides measurable value."
  Example: After a first synthesis pass covers only three of five requested comparison dimensions, the coordinator detects the two missing dimensions, sends a search subagent two new targeted queries for exactly those dimensions, and re-runs synthesis with the combined findings.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.2 Skills-S3 (tier 1, official exam guide); corroborated by the evaluator-optimizer workflow pattern in https://www.anthropic.com/engineering/building-effective-agents (tier 2)

---

## Task Statement 1.3 — Configure subagent invocation, context passing, and spawning

- Concept: The Task/Agent tool and the `allowedTools` requirement
  Type: key
  Teaches: 1.3-K1
  Definition: Subagents are spawned through a dedicated tool — named `Task` in earlier SDK versions and `Agent` in current versions (the SDK still accepts `"Task"` as an alias) — that the coordinator's model calls just like any other tool. For a coordinator to be able to invoke subagents at all, its `allowedTools` (or `allowed_tools`) configuration must include this tool; without it, the coordinator has no mechanism to spawn subagents, no matter how its subagents are defined. Claude detects and matches `subagent_type` to route to the correct subagent definition when the tool is called.
  Example: `ClaudeAgentOptions(allowed_tools=["Read", "Grep", "Glob", "Agent"], agents={...})` — omitting `"Agent"`/`"Task"` from `allowed_tools` here would mean Claude Code can never spawn the defined subagents, even though `agents` is configured.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.3 Knowledge-K1 (tier 1, official exam guide); corroborated by https://code.claude.com/docs/en/agent-sdk/subagents, sections "Create subagents" and "Detect subagent invocation" (tier 2), which document the `Agent`/`Task` tool name and the `allowedTools` usage pattern

- Concept: `AgentDefinition` configuration
  Type: key
  Teaches: 1.3-K3
  Definition: Each subagent is configured with an `AgentDefinition` object. The required fields are `description` (a natural-language description of when to use this agent, which Claude uses for automatic delegation matching) and `prompt` (the subagent's system prompt defining its role, expertise, and behavior). Optional fields include `tools` (an allow-list restricting which tools the subagent can use — if omitted, it inherits every tool available to subagents) and `model` (overriding the model used for that specific subagent, e.g. a stronger model for high-stakes reviews).
  Example: `AgentDefinition(description="Expert code review specialist. Use for quality, security, and maintainability reviews.", prompt="You are a code review specialist...", tools=["Read","Grep","Glob"], model="sonnet")` defines a read-only, Sonnet-backed review subagent that Claude will select automatically for review-shaped requests.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents, section "AgentDefinition configuration" (tier 2, official Claude Code / Agent SDK Docs)

- Concept: Fork-based session management (`fork_session`)
  Type: key
  Teaches: 1.3-K4, 1.7-K2, 1.7-S2
  Definition: Forking creates a new session that starts as a copy of an existing session's full history but then diverges independently from that point forward. "The fork gets its own session ID; the original's ID and history stay unchanged. You end up with two independent sessions you can resume separately." This lets a team explore multiple divergent approaches from one shared analysis baseline (e.g., two different refactoring strategies) without either branch affecting the other or requiring the original analysis to be redone.
  Example: After analyzing an authentication module in session A, the team forks A into session B to explore "outline how OAuth2 would work here," while resuming the original session A separately to continue down a JWT-based approach — both branches share the same initial codebase analysis but diverge after that point.
  Source: https://code.claude.com/docs/en/agent-sdk/sessions, section "Fork to explore alternatives" (tier 2, official Claude Code / Agent SDK Docs)

- Concept: Structured data formats separating content from metadata
  Type: key
  Teaches: 1.3-S2
  Definition: When passing findings between agents, content (the claim or extracted text) should be structurally separated from attribution metadata (source URL, document name, page number, publication date) rather than folded into an unstructured blob of prose, so that downstream agents (and the final report) can preserve exactly which source backs which claim. Anthropic's own research pipeline implements this with a dedicated citation pass: "a separate CitationAgent reads the raw documents AND the final report... [and] attaches each claim to a specific URL," which only works because the claim and its supporting source were kept as distinguishable, structured data through the pipeline rather than merged into free text.
  Example: A search subagent returns findings as `{"claim": "Revenue grew 12% YoY", "source_url": "https://...", "document": "Q3 earnings call transcript", "page": 4}` rather than a paragraph like "According to the transcript, revenue grew 12%," so the synthesis agent can carry the URL forward even after rewriting the claim's wording.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.3 Skills-S2 (tier 1, official exam guide); corroborated by the CitationAgent pattern in https://www.anthropic.com/engineering/multi-agent-research-system (tier 2)

- Concept: Parallel subagent spawning
  Type: key
  Teaches: 1.3-S3
  Definition: A coordinator can spawn multiple subagents concurrently by emitting several `Task`/`Agent` tool calls within a single response, rather than issuing them one at a time across separate turns. Because subagents are independent, concurrent invocations finish in roughly the time of the slowest one rather than the sum of all of them. Anthropic's research system reports that "the lead agent spins up 3-5 subagents in parallel rather than serially," which was one of the two parallelization changes that "cut research time by up to 90% for complex queries."
  Example: A coordinator response contains three `Agent` tool_use blocks in the same turn — one each for `web-search`, `document-analysis`, and `fact-check` subagents — so all three run at once instead of the coordinator waiting for each to finish before starting the next.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system (tier 2, official Anthropic engineering blog); corroborated by the "Parallelization" benefit in https://code.claude.com/docs/en/agent-sdk/subagents (tier 2) and runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.3 Skills-S3 (tier 1)

- Concept: Goal-oriented (vs. procedural) coordinator prompts for subagent adaptability
  Type: key
  Teaches: 1.3-S4
  Definition: A coordinator's prompt to a subagent should specify the research goal, expected output format, tool guidance, and clear task boundaries — not a rigid, step-by-step procedure. Anthropic found that vague or under-specified task descriptions caused subagents to duplicate work or misinterpret scope, and the fix was giving subagents "detailed task descriptions... including objectives, output formats, tool guidance, and clear boundaries" that define what to accomplish and why, leaving the subagent free to adapt how it gets there rather than being locked into a prescribed sequence of actions.
  Example: Instead of instructing a subagent "First call web_search with query X, then call web_search with query Y, then summarize," the coordinator instructs it "Determine whether company Z's Q3 revenue guidance was met; report the figure and your source," letting the subagent choose its own search strategy to reach that goal.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system (tier 2, official Anthropic engineering blog); corroborated by runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.3 Skills-S4 (tier 1)

---

## Task Statement 1.4 — Implement multi-step workflows with enforcement and handoff patterns

- Concept: Programmatic enforcement vs. prompt-based guidance
  Type: key
  Teaches: 1.4-K1, 1.4-K2, 1.5-K3, 1.5-S3
  Definition: Workflow ordering can be enforced two ways. Prompt-based guidance tells the model, in natural language, what order to follow (e.g., "always verify the customer before processing a refund") — the model usually complies but compliance is probabilistic and has a non-zero failure rate. Programmatic enforcement (hooks, prerequisite gates) uses code outside the model's control to physically block a disallowed action from executing, regardless of what the model attempts — this yields a deterministic guarantee. When business rules require guaranteed compliance (e.g., identity verification before a financial operation), programmatic enforcement is required because prompt instructions alone cannot provide that guarantee.
  Example: Production logs showing an agent skips `get_customer` and calls `lookup_order` on a bare customer name 12% of the time demonstrate that a prompt instruction ("always verify identity first") is insufficient; a `PreToolUse` hook that denies `lookup_order`/`process_refund` calls until a verified customer ID exists in context closes that gap deterministically.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.4 Knowledge-K1/K2, Task Statement 1.5 Knowledge-K3, and Sample Question 1 (tier 1, official exam guide); corroborated by the deterministic blocking capability described in https://code.claude.com/docs/en/agent-sdk/hooks (tier 2)

- Concept: Structured handoff protocol for mid-process escalation
  Type: key
  Teaches: 1.4-K3, 1.4-S3
  Definition: When an in-progress multi-step task must be escalated to a human who has no access to the conversation transcript, the handoff must package everything the human needs as structured data rather than relying on the human to reconstruct context: customer/case details, root-cause analysis of what went wrong or why the case is out of scope for autonomous resolution, and a recommended action. Compiling this summary at the point of escalation (rather than leaving the human to read a raw transcript) lets the human agent act immediately.
  Example: An escalation payload of `{"customer_id": "C-4471", "root_cause": "Damaged item outside standard 30-day window", "refund_amount": "$89.50", "recommended_action": "Approve as goodwill exception"}` gives a human agent everything needed to decide in seconds, without reading the underlying conversation.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.4 Knowledge-K3 and Skills-S3 (tier 1, official exam guide)

- Concept: Programmatic prerequisite gating
  Type: key
  Teaches: 1.4-S1
  Definition: A programmatic prerequisite blocks a downstream tool call from executing until an upstream prerequisite step has been completed and its result verified, implemented as code-level logic (typically a hook) rather than a model instruction. This guarantees a required ordering (e.g., identity verification before any financial action) independent of what the model decides to do.
  Example: A `PreToolUse` hook checks whether a verified `customer_id` is present in accumulated conversation state before allowing `process_refund` to execute; if `get_customer` has not yet returned a verified ID, the hook returns `permissionDecision: "deny"` and the refund call never reaches the backend, no matter what the model attempted.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.4 Skills-S1 and Sample Question 1 (tier 1, official exam guide); implementation mechanism corroborated by the `PreToolUse` `permissionDecision` pattern in https://code.claude.com/docs/en/agent-sdk/hooks (tier 2)

- Concept: Decomposing multi-concern requests with shared-context parallel investigation
  Type: key
  Teaches: 1.4-S2
  Definition: When a single customer (or user) request bundles multiple distinct concerns, the request should be decomposed into its constituent items, each investigated using a shared context (so findings about one item can inform another where relevant), and the results synthesized into one unified resolution rather than answering only the first concern mentioned or producing several disjointed replies.
  Example: A message that says "my order #123 arrived damaged and I was also double-charged on order #456" is split into two distinct investigation threads (damage claim on #123; billing dispute on #456), both investigated with access to the same customer/account context, then combined into a single reply that resolves both.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.4 Skills-S2 (tier 1, official exam guide)

---

## Task Statement 1.5 — Apply Agent SDK hooks for tool call interception and data normalization

- Concept: `PreToolUse` hooks (outgoing tool-call interception)
  Type: key
  Teaches: 1.5-K2, 1.5-S2
  Definition: A `PreToolUse` hook fires when the SDK is about to execute a tool call, before it runs. The callback receives the tool name and input and returns a decision via `hookSpecificOutput`: `permissionDecision` set to `"allow"`, `"deny"`, or `"ask"` (plus `"defer"`), an optional `permissionDecisionReason` explaining the decision to the model, and optionally `updatedInput` to rewrite the call's arguments before it executes. A `"deny"` decision blocks the tool call outright and is deterministic — it cannot be overridden by another hook returning `"allow"` (when multiple hooks or rules apply, `deny` always takes priority).
  Example: A hook matched on `matcher: "process_refund"` inspects `tool_input.amount`; if the amount exceeds $500 it returns `{"hookSpecificOutput": {"permissionDecision": "deny", "permissionDecisionReason": "Refunds over $500 require human escalation"}}`, which blocks the refund call and can be paired with logic that redirects the conversation toward `escalate_to_human`.
  Source: https://code.claude.com/docs/en/agent-sdk/hooks, sections "How hooks work" and "Outputs" (tier 2, official Claude Code / Agent SDK Docs)

- Concept: `PostToolUse` hooks (tool-result transformation and normalization)
  Type: key
  Teaches: 1.5-K1, 1.5-S1
  Definition: A `PostToolUse` hook fires after a tool call returns a result, before the model processes it. The callback can set `additionalContext` to append information to the tool result, or set `updatedToolOutput` to replace the tool's output entirely — giving it the ability to rewrite raw, heterogeneous tool output into a normalized form before the model ever sees it. This is the mechanism for reconciling MCP tools that return the same kind of data (dates, statuses) in different encodings.
  Example: A `PostToolUse` hook matched on multiple MCP order/customer tools converts every incoming timestamp field — whether a Unix epoch integer or an ISO 8601 string — into a single consistent ISO 8601 format via `updatedToolOutput` before the result is appended to the conversation, so the model always reasons over one date format regardless of which backend tool produced the value.
  Source: https://code.claude.com/docs/en/agent-sdk/hooks, sections "Available hooks" and "Outputs" (tier 2, official Claude Code / Agent SDK Docs)

(See "Programmatic enforcement vs. prompt-based guidance" under Task Statement 1.4 for 1.5-K3 and 1.5-S3.)

---

## Task Statement 1.6 — Design task decomposition strategies for complex workflows

- Concept: Prompt chaining
  Type: key
  Teaches: 1.6-K1, 1.6-K2, 1.6-S1, 1.6-S2
  Definition: "Prompt chaining decomposes a task into a sequence of steps, where each LLM call processes the output of the previous one." It trades some latency for higher accuracy by making each individual call an easier, more focused subtask, and is best suited to tasks that can be "easily and cleanly decomposed into fixed subtasks" known in advance. Anthropic notes developers can "add programmatic checks (gates) on any intermediate steps to ensure the process is still on track" between chain steps.
  Example: A code review pipeline runs a fixed sequence: first a pass that analyzes each file individually for local issues, then a second, separate pass that takes all per-file outputs and looks specifically for cross-file integration problems — a fixed two-step chain, not something the model decides dynamically per review.
  Source: https://www.anthropic.com/engineering/building-effective-agents (tier 2, official Anthropic engineering blog)

- Concept: Orchestrator-workers pattern (dynamic adaptive decomposition)
  Type: key
  Teaches: 1.6-K1, 1.6-K3, 1.6-S1, 1.6-S3
  Definition: In the orchestrator-workers pattern, "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results." Unlike prompt chaining, "subtasks aren't pre-defined, but determined by the orchestrator based on the specific input" — making this pattern well suited to complex, open-ended tasks where the number and nature of subtasks cannot be predicted in advance and instead must be generated adaptively based on what is discovered along the way.
  Example: For "add comprehensive tests to a legacy codebase," an orchestrator first explores the codebase structure, then — based on what it finds — dynamically generates a prioritized set of testing subtasks (which files, in what order) that could not have been fully enumerated before the exploration happened.
  Source: https://www.anthropic.com/engineering/building-effective-agents (tier 2, official Anthropic engineering blog)

- Concept: Multi-pass review decomposition (per-file local pass + cross-file integration pass)
  Type: key
  Teaches: 1.6-K2, 1.6-S2
  Definition: For predictable, multi-aspect reviews such as large code reviews, task decomposition should split the work into a per-file local-analysis pass (each file reviewed individually for its own issues) plus a separate cross-file integration pass (specifically looking for problems that only appear when files are considered together, such as inconsistent interfaces or data-flow mismatches). Splitting the work this way avoids "attention dilution" — the degradation that occurs when a single pass tries to hold both fine-grained per-file detail and broad cross-file relationships in view at once.
  Example: Reviewing a 40-file pull request as one giant single-pass review risks missing that `UserService.ts` and `AuthController.ts` disagree about a field's type; running a dedicated cross-file integration pass after the per-file passes complete is designed specifically to catch that kind of issue.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.6 Knowledge-K2 and Skills-S2 (tier 1, official exam guide)

- Concept: Adaptive investigation planning for open-ended tasks
  Type: key
  Teaches: 1.6-K3, 1.6-S3
  Definition: For open-ended tasks whose scope cannot be fully specified up front, an effective decomposition strategy is to first map the relevant structure (e.g., explore the codebase or problem space), then identify high-impact areas based on that exploration, then create a prioritized plan — and to let that plan continue adapting as new dependencies or constraints are discovered during execution, rather than committing to a full fixed task list at the outset.
  Example: Tasked with "add comprehensive tests to a legacy codebase," an agent first surveys which modules have zero test coverage, identifies the modules with the highest change frequency (highest risk) as priority targets, drafts a plan starting there, and revises the plan's ordering when it discovers an untested shared utility that several "already covered" modules actually depend on.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.6 Knowledge-K3 and Skills-S3 (tier 1, official exam guide); corroborated by the orchestrator-workers pattern in https://www.anthropic.com/engineering/building-effective-agents (tier 2)

---

## Task Statement 1.7 — Manage session state, resumption, and forking

- Concept: Named session resumption (`--resume <session-name>`)
  Type: key
  Teaches: 1.7-K1, 1.7-S1
  Definition: A Claude Code session can be given a descriptive name (at startup with `claude -n <name>`, mid-session with `/rename <name>`, or via the session picker), and once named, `claude --resume <name>` "resumes the named session directly," restoring its full conversation history so work can continue exactly where it left off — including across separate CLI invocations run at different times. This is distinct from `--continue`, which always resumes only the most recently used session in the current directory rather than a specific named one.
  Example: A developer names an investigation session `claude -n refund-flow-audit`; days later, from the same or a different terminal, `claude --resume refund-flow-audit` reopens that exact conversation with its accumulated findings intact.
  Source: https://code.claude.com/docs/en/sessions, sections "Resume a session" and "Name your sessions" (tier 2, official Claude Code Docs); terminology corroborated by runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.7 Knowledge-K1 (tier 1)

(See "Fork-based session management (`fork_session`)" under Task Statement 1.3 for 1.7-K2 and 1.7-S2.)

- Concept: Informing a resumed session about file changes made outside it
  Type: key
  Teaches: 1.7-K3, 1.7-S4
  Definition: When a session is resumed after the underlying files it previously analyzed have been modified (by a human, another tool, or another session), the agent's prior tool results in that conversation history no longer reflect the current state of those files — the model will otherwise keep reasoning from stale reads. The resumed session should be explicitly told which specific files changed so it re-reads and re-analyzes only the affected files (a targeted re-analysis), rather than either reasoning from outdated results or being forced into a full, expensive re-exploration of everything it looked at before.
  Example: After a developer manually edits `payment_service.py` between two work sessions, resuming with "Note: `payment_service.py` was modified since we last looked at it — please re-read it before continuing" lets the agent correct only that file's understanding instead of silently giving advice based on the pre-edit version.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.7 Knowledge-K3 and Skills-S4 (tier 1, official exam guide)

- Concept: Choosing session resumption vs. a fresh session with an injected summary
  Type: key
  Teaches: 1.7-K4, 1.7-S3
  Definition: Resuming a session carries forward its complete tool-call history, which is valuable when that prior context is still mostly valid, but becomes a liability when much of it is stale (e.g., many of the tool results reflect a system state that has since changed) — in that case, starting a new session and manually injecting a concise, structured summary of what was learned is more reliable than resuming, because it lets the operator control exactly what carries forward and discard outdated tool results rather than leaving them in context for the model to (possibly) reason from incorrectly. The choice is a judgment call based on how much of the prior context remains valid, not a default preference for one approach.
  Example: After a two-week gap during which the target codebase was substantially refactored, starting fresh with "Here is a summary of the prior architecture analysis: [3 bullet points]" avoids the risk of the agent trusting dozens of now-stale file reads that a straight `--resume` would otherwise carry forward.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt, Task Statement 1.7 Knowledge-K4 and Skills-S3 (tier 1, official exam guide)

---

## Coverage summary

- Bullets received: 48
- Bullets covered: 48
- Key concepts: 30
- Prerequisite concepts: 5
- UNSOURCED concepts: 0
