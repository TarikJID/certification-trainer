## Domain 1: Agentic Architecture & Orchestration (27%)

Description: Covers designing and implementing the core agentic loop, orchestrating coordinator/subagent
multi-agent systems, configuring subagent spawning and context passing, enforcing multi-step
workflows with hooks and handoffs, decomposing complex workflows into tasks, and managing session
state/resumption/forking with the Claude Agent SDK.

Task statements: 7 · Bullets captured: 48 (24 Knowledge, 24 Skills)

### Task statements

- Task Statement 1.1: Design and implement agentic loops for autonomous task execution
  Knowledge of:
    - 1.1-K1: The agentic loop lifecycle: sending requests to Claude, inspecting stop_reason ("tool_use" vs "end_turn"), executing requested tools, and returning results for the next iteration
    - 1.1-K2: How tool results are appended to conversation history so the model can reason about the next action
    - 1.1-K3: The distinction between model-driven decision-making (Claude reasons about which tool to call next based on context) and pre-configured decision trees or tool sequences
  Skills in:
    - 1.1-S1: Implementing agentic loop control flow that continues when stop_reason is "tool_use" and terminates when stop_reason is "end_turn"
    - 1.1-S2: Adding tool results to conversation context between iterations so the model can incorporate new information into its reasoning
    - 1.1-S3: Avoiding anti-patterns such as parsing natural language signals to determine loop termination, setting arbitrary iteration caps as the primary stopping mechanism, or checking for assistant text content as a completion indicator

- Task Statement 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns
  Knowledge of:
    - 1.2-K1: Hub-and-spoke architecture where a coordinator agent manages all inter-subagent communication, error handling, and information routing
    - 1.2-K2: How subagents operate with isolated context—they do not inherit the coordinator's conversation history automatically
    - 1.2-K3: The role of the coordinator in task decomposition, delegation, result aggregation, and deciding which subagents to invoke based on query complexity
    - 1.2-K4: Risks of overly narrow task decomposition by the coordinator, leading to incomplete coverage of broad research topics
  Skills in:
    - 1.2-S1: Designing coordinator agents that analyze query requirements and dynamically select which subagents to invoke rather than always routing through the full pipeline
    - 1.2-S2: Partitioning research scope across subagents to minimize duplication (e.g., assigning distinct subtopics or source types to each agent)
    - 1.2-S3: Implementing iterative refinement loops where the coordinator evaluates synthesis output for gaps, re-delegates to search and analysis subagents with targeted queries, and re-invokes synthesis until coverage is sufficient
    - 1.2-S4: Routing all subagent communication through the coordinator for observability, consistent error handling, and controlled information flow

- Task Statement 1.3: Configure subagent invocation, context passing, and spawning
  Knowledge of:
    - 1.3-K1: The Task tool as the mechanism for spawning subagents, and the requirement that allowedTools must include "Task" for a coordinator to invoke subagents
    - 1.3-K2: That subagent context must be explicitly provided in the prompt—subagents do not automatically inherit parent context or share memory between invocations
    - 1.3-K3: The AgentDefinition configuration including descriptions, system prompts, and tool restrictions for each subagent type
    - 1.3-K4: Fork-based session management for exploring divergent approaches from a shared analysis baseline
  Skills in:
    - 1.3-S1: Including complete findings from prior agents directly in the subagent's prompt (e.g., passing web search results and document analysis outputs to the synthesis subagent)
    - 1.3-S2: Using structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution
    - 1.3-S3: Spawning parallel subagents by emitting multiple Task tool calls in a single coordinator response rather than across separate turns
    - 1.3-S4: Designing coordinator prompts that specify research goals and quality criteria rather than step-by-step procedural instructions, to enable subagent adaptability

- Task Statement 1.4: Implement multi-step workflows with enforcement and handoff patterns
  Knowledge of:
    - 1.4-K1: The difference between programmatic enforcement (hooks, prerequisite gates) and prompt-based guidance for workflow ordering
    - 1.4-K2: When deterministic compliance is required (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate
    - 1.4-K3: Structured handoff protocols for mid-process escalation that include customer details, root cause analysis, and recommended actions
  Skills in:
    - 1.4-S1: Implementing programmatic prerequisites that block downstream tool calls until prerequisite steps have completed (e.g., blocking process_refund until get_customer has returned a verified customer ID)
    - 1.4-S2: Decomposing multi-concern customer requests into distinct items, then investigating each in parallel using shared context before synthesizing a unified resolution
    - 1.4-S3: Compiling structured handoff summaries (customer ID, root cause, refund amount, recommended action) when escalating to human agents who lack access to the conversation transcript

- Task Statement 1.5: Apply Agent SDK hooks for tool call interception and data normalization
  Knowledge of:
    - 1.5-K1: Hook patterns (e.g., PostToolUse) that intercept tool results for transformation before the model processes them
    - 1.5-K2: Hook patterns that intercept outgoing tool calls to enforce compliance rules (e.g., blocking refunds above a threshold)
    - 1.5-K3: The distinction between using hooks for deterministic guarantees versus relying on prompt instructions for probabilistic compliance
  Skills in:
    - 1.5-S1: Implementing PostToolUse hooks to normalize heterogeneous data formats (Unix timestamps, ISO 8601, numeric status codes) from different MCP tools before the agent processes them
    - 1.5-S2: Implementing tool call interception hooks that block policy-violating actions (e.g., refunds exceeding $500) and redirect to alternative workflows (e.g., human escalation)
    - 1.5-S3: Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance

- Task Statement 1.6: Design task decomposition strategies for complex workflows
  Knowledge of:
    - 1.6-K1: When to use fixed sequential pipelines (prompt chaining) versus dynamic adaptive decomposition based on intermediate findings
    - 1.6-K2: Prompt chaining patterns that break reviews into sequential steps (e.g., analyze each file individually, then run a cross-file integration pass)
    - 1.6-K3: The value of adaptive investigation plans that generate subtasks based on what is discovered at each step
  Skills in:
    - 1.6-S1: Selecting task decomposition patterns appropriate to the workflow: prompt chaining for predictable multi-aspect reviews, dynamic decomposition for open-ended investigation tasks
    - 1.6-S2: Splitting large code reviews into per-file local analysis passes plus a separate cross-file integration pass to avoid attention dilution
    - 1.6-S3: Decomposing open-ended tasks (e.g., "add comprehensive tests to a legacy codebase") by first mapping structure, identifying high-impact areas, then creating a prioritized plan that adapts as dependencies are discovered

- Task Statement 1.7: Manage session state, resumption, and forking
  Knowledge of:
    - 1.7-K1: Named session resumption using --resume <session-name> to continue a specific prior conversation
    - 1.7-K2: fork_session for creating independent branches from a shared analysis baseline to explore divergent approaches
    - 1.7-K3: The importance of informing the agent about changes to previously analyzed files when resuming sessions after code modifications
    - 1.7-K4: Why starting a new session with a structured summary is more reliable than resuming with stale tool results
  Skills in:
    - 1.7-S1: Using --resume with session names to continue named investigation sessions across work sessions
    - 1.7-S2: Using fork_session to create parallel exploration branches (e.g., comparing two testing strategies or refactoring approaches from a shared codebase analysis)
    - 1.7-S3: Choosing between session resumption (when prior context is mostly valid) and starting fresh with injected summaries (when prior tool results are stale)
    - 1.7-S4: Informing a resumed session about specific file changes for targeted re-analysis rather than requiring full re-exploration

---

