# Agentic Architecture & Orchestration

Domain weight: 27% of the CCAR-F exam (largest domain). Scope per the domain description: building autonomous agent loops, orchestrating multi-agent systems with coordinator-subagent patterns, configuring subagent invocation and context passing, implementing multi-step workflows with enforcement and handoff patterns, using Agent SDK hooks for tool interception, designing task decomposition strategies, and managing session state.

All concepts below are sourced from Anthropic's own Claude Agent SDK / Claude Code documentation (code.claude.com/docs, platform.claude.com/docs, docs.claude.com). The official CCAR-F exam guide PDF could not be parsed as text by the fetch tool (binary/FlateDecode PDF stream); scoping was therefore done from the domain description provided by the orchestrator, cross-checked against the SDK's own "Capabilities" table (Built-in tools, Hooks, Subagents, MCP, Permissions, Sessions), which maps directly onto the domain's listed sub-topics.

---

## KEY CONCEPTS

### Concept: The Agent Loop
Type: key
Definition: The core execution cycle that powers autonomous agents built with the Claude Agent SDK. Claude receives a prompt along with the system prompt, tool definitions, and conversation history; it evaluates the current state and either responds with text, requests one or more tool calls, or both; the SDK executes each requested tool and feeds the results back to Claude; this "evaluate → call tools → receive results" cycle repeats until Claude produces a response with no tool calls, at which point the loop ends and a final result is returned. One full cycle of evaluate+execute is called a "turn." The loop can be bounded with `max_turns`/`maxTurns` (tool-use round trips) and `max_budget_usd`/`maxBudgetUsd` (spend), and produces a final `ResultMessage` whose `subtype` (`success`, `error_max_turns`, `error_max_budget_usd`, `error_during_execution`, `error_max_structured_output_retries`) tells the caller how the loop ended.
Example: For the prompt "Fix the failing tests in auth.ts": Turn 1, Claude calls `Bash` to run `npm test` and gets 3 failures back; Turn 2, Claude calls `Read` on `auth.ts` and `auth.test.ts`; Turn 3, Claude calls `Edit` then re-runs `npm test`, which now passes; Final turn, Claude replies with text only ("Fixed the auth bug, all three tests pass now"), ending the loop after 4 turns.
Source: https://code.claude.com/docs/en/agent-sdk/agent-loop

---

### Concept: Coordinator–Subagent Orchestration Pattern
Type: key
Definition: A multi-agent architecture in which a main ("coordinator") agent delegates focused subtasks to separate agent instances called subagents, each with its own isolated conversation, specialized system prompt, and (optionally) restricted tool set. Subagents are invoked through the built-in `Agent` tool. This pattern is the primary way the Agent SDK scales beyond a single flat conversation: the coordinator decides, turn by turn, what to delegate and to whom, and receives only each subagent's final summarized result rather than its full working transcript. Subagents can themselves spawn further subagents (nested delegation), bounded by a configurable depth limit.
Example: A `code-reviewer` subagent (tools restricted to `Read`, `Grep`, `Glob`, model `sonnet`) and a `test-runner` subagent (tools `Bash`, `Read`, `Grep`) are both defined via the `agents` parameter of `ClaudeAgentOptions`/`Options`. A coordinator prompt "Review the authentication module for security issues" causes Claude to invoke the `code-reviewer` subagent automatically because its `description` field matches the task; the subagent explores the codebase in its own isolated context and returns a concise finding, which is the only content added to the coordinator's context.
Source: https://code.claude.com/docs/en/agent-sdk/subagents

---

### Concept: Subagent Invocation (Automatic and Explicit)
Type: key
Definition: The mechanism by which a subagent is triggered to run. There are two modes: (1) Automatic invocation — Claude reads each defined subagent's `description` field and decides on its own, based on the current task, whether to route work to that subagent via the `Agent` tool; (2) Explicit invocation — the calling prompt names the subagent directly (e.g., "Use the code-reviewer agent to check the authentication module"), which bypasses matching and guarantees that specific subagent runs. Subagents run in the background by default unless the caller requests a synchronous ("foreground") result, and their spawn depth, concurrency, and cumulative spend can be capped via environment variables (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`) and the `maxBudgetUsd`/`max_budget_usd` query option.
Example: Writing "Use the code-reviewer agent to check the authentication module" in the prompt forces Claude to invoke the named `code-reviewer` subagent rather than deciding for itself, whereas a vaguer prompt like "review my auth code" leaves Claude to match the task against subagent descriptions automatically.
Source: https://code.claude.com/docs/en/agent-sdk/subagents

---

### Concept: Context Passing and Isolation Between Coordinator and Subagent
Type: key
Definition: The rule set governing what information flows from a coordinator into a subagent and back. Unless a subagent is a "fork," its context window starts fresh: it does not receive the parent's conversation history, prior tool results, or the parent's system prompt. The only content passed from parent to subagent is the literal prompt string given to the `Agent` tool call, so any file paths, error messages, or decisions the subagent needs must be included explicitly in that prompt. A non-fork subagent does receive its own `AgentDefinition.prompt` (system prompt), project `CLAUDE.md` (unless `omitClaudeMd` is set), and its assigned tool definitions. Only the subagent's final message is returned to the parent as the Agent tool's result — intermediate tool calls and reads stay inside the subagent and never accumulate in the coordinator's context. This isolation is also what keeps the coordinator's context window "lean" on long or exploratory subtasks.
Example: A `research-assistant` subagent can read dozens of files while investigating a bug; none of those file contents enter the main conversation's context — the coordinator's context only grows by the subagent's final one-paragraph summary, not by every file it read.
Source: https://code.claude.com/docs/en/agent-sdk/subagents

---

### Concept: Dynamic Workflows (Script-Based Multi-Subagent Orchestration)
Type: key
Definition: A mechanism for orchestrating many subagents (dozens to hundreds) via a JavaScript script that a runtime executes outside the conversation's context window, rather than turn-by-turn delegation held in Claude's own context. Claude writes the orchestration script for a described task; the script body can call `agent()` (spawn one subagent), `pipeline()` (run one subagent per item in a list, sequentially per item but agents can run concurrently), and `parallel()` (run a set of agent tasks concurrently and wait for all). Intermediate results live in script variables, not in Claude's context, so a workflow can coordinate far more agents than a turn-by-turn conversation can hold. Workflows run in the background, are resumable within the same session, and are bounded by runtime limits (up to 16 concurrent agents by default, 1,000 agents total per run, up to 4,096 items per `parallel()`/`pipeline()` call).
Example: `use a workflow to audit every route handler under src/routes/ for missing authentication checks, and adversarially verify each finding before reporting it` causes Claude to write a script that lists all matching files with one `agent()` call, then runs a `pipeline()` of one audit-subagent per file, and finally has an independent verifier subagent adversarially review each finding before the workflow returns one merged report.
Source: https://code.claude.com/docs/en/workflows

---

### Concept: Task Decomposition Strategies
Type: key
Definition: The design practice of splitting a large or multi-part task into smaller units of work assigned to separate agents, chosen deliberately for isolation, parallel speed-up, or specialization. The Agent SDK documentation identifies four concrete benefits that motivate a decomposition: context isolation (each subagent's exploration doesn't pollute the parent's context), parallelization (independent subtasks finish in the time of the slowest one instead of the sum of all of them), specialized instructions/knowledge (a tailored system prompt gives each worker only the expertise it needs), and tool restriction (limiting each worker's tool surface to only what its slice of the task requires). Decomposition can be expressed either as several named subagents invoked turn-by-turn by a coordinator, or, at larger scale, as a dynamic workflow script using `pipeline()`/`parallel()` to fan work out across many agents and later reduce/merge their results.
Example: During a code review, running `style-checker`, `security-scanner`, and `test-coverage` subagents simultaneously instead of sequentially is a task-decomposition strategy that trades a single generalist pass for three specialized, parallel passes whose results are then synthesized by the coordinator.
Source: https://code.claude.com/docs/en/agent-sdk/subagents ; https://code.claude.com/docs/en/workflows

---

### Concept: Enforcement and Handoff Patterns (Permission Evaluation and Tool Restriction)
Type: key
Definition: The layered mechanism the SDK uses to control and enforce what a tool call — whether from the main agent or a subagent — is actually allowed to do, and how that control is handed off between coordinator and subagent. Every tool request is evaluated in a fixed order: (1) hooks (can deny outright), (2) deny rules (`disallowed_tools`/settings.json — block even in `bypassPermissions` mode), (3) ask rules (fall through to the approval callback even in `bypassPermissions`), (4) the active permission mode (`default`, `acceptEdits`, `plan`, `dontAsk`, `auto`, `bypassPermissions`), (5) allow rules (`allowed_tools`/settings.json), and (6) the `canUseTool` callback as a last resort. On handoff to a subagent, tool restriction (`AgentDefinition.tools`/`disallowedTools`) and permission mode are inherited from the parent unless explicitly overridden, with the rule that a subagent only runs in `bypassPermissions` mode if the parent session itself does — a subagent cannot escalate its own privilege beyond what the coordinator granted.
Example: A `db-reader` subagent is defined with `tools: ["Bash"]` and `permissionMode: "dontAsk"`; combined with a `PreToolUse` hook that runs a validation script against every `Bash` call (exit 0 allows, exit 2 blocks), the subagent can only ever execute pre-approved read-only queries, regardless of what the coordinator's own permission mode allows.
Source: https://code.claude.com/docs/en/agent-sdk/permissions ; https://code.claude.com/docs/en/sub-agents

---

### Concept: Agent SDK Hooks for Tool Interception
Type: key
Definition: Callback functions, run in the calling application's own process (not inside the agent's context window), that fire at defined points in the agent loop so custom code can observe or alter agent behavior. Documented hook events include `PreToolUse` (fires before a tool executes; can validate inputs or block dangerous commands), `PostToolUse` (fires after a tool returns; used to audit outputs or trigger side effects), `UserPromptSubmit` (fires when a prompt is sent; used to inject additional context), `Stop` (fires when the agent finishes; used to validate the result or save state), `SubagentStart`/`SubagentStop` (fire when a subagent spawns or completes; used to track and aggregate parallel task results), and `PreCompact` (fires before context compaction; used to archive the full transcript). A hook can be scoped to specific tools with a `matcher` pattern (e.g., a matcher of `"Bash"` runs the hook only for Bash calls; omitting the matcher runs the hook for every event of that type). Because a `PreToolUse` hook that rejects a call prevents the tool from executing at all (Claude receives a rejection message instead), hooks are the mechanism used to intercept and enforce policy on tool calls, independent of and prior to permission-mode/allow-rule evaluation.
Example: Registering a `PreToolUse` hook matched to `"Write|Edit"` that inspects the file path being written and returns a deny decision whenever the path falls outside an allow-listed directory — this blocks the write before it ever reaches disk, and Claude sees the rejection as the tool's result.
Source: https://code.claude.com/docs/en/agent-sdk/hooks ; https://code.claude.com/docs/en/agent-sdk/agent-loop

---

### Concept: Session State Management (Continue, Resume, Fork)
Type: key
Definition: The set of mechanisms for persisting and returning to an agent's conversation state across multiple calls. A "session" is the conversation history the SDK accumulates while an agent works — the prompt, every tool call, every tool result, and every response — written to disk automatically. Three distinct operations manage this state: "Continue" finds and resumes the most recent session in the current working directory with no ID tracking required; "Resume" takes a specific captured `session_id` and picks that session back up with full prior context, used when there are multiple concurrent sessions or the target isn't the most recent one; "Fork" creates a brand-new session that starts as a copy of an existing session's history but diverges from that point onward, leaving the original session unchanged, so an application can explore an alternative approach without losing the original thread. The session ID is read from the `session_id` field on the `ResultMessage`/`SDKResultMessage`.
Example: After a first query analyzes an `auth` module and captures its `session_id`, a second call passes `resume=session_id` with the prompt "Now implement the refactoring you suggested" — the agent proceeds with full memory of the prior analysis instead of re-reading the files. A third call could instead pass `resume=session_id, fork_session=True` with a different prompt ("outline how OAuth2 would work instead") to explore that alternative without disturbing the original JWT-focused session.
Source: https://code.claude.com/docs/en/agent-sdk/sessions

---

## PREREQUISITE CONCEPTS

### Concept: Tool Use (Function Calling)
Type: prerequisite
Definition: The foundational Claude API capability that lets Claude call functions defined by the developer (client tools) or provided by Anthropic (server tools). The developer specifies what operations are available and the shape of their inputs/outputs (an `input_schema`); Claude decides when and how to call them based on the user's request and each tool's description, returning a structured `tool_use` block that the calling application (for client tools) or Anthropic's infrastructure (for server tools) executes, with the result sent back as a `tool_result`. This request/execute/respond round trip is the single primitive that the agent loop repeats turn after turn, and is a prerequisite for understanding both the Agent Loop and Hooks (which intercept precisely this round trip).
Example: A `get_weather` tool is defined with an `input_schema` requiring a `location` string; Claude responds with `stop_reason: "tool_use"` and a `tool_use` block naming `get_weather` with `{"location": "San Francisco, CA"}`; the application looks up the weather and sends it back as a `tool_result`, after which Claude produces the final text answer.
Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

---

### Concept: System Prompt
Type: prerequisite
Definition: The initial instruction set that shapes how Claude behaves throughout a conversation or subagent run — its role, capabilities, constraints, and response style. In the Agent SDK, a system prompt can come from three starting points: a minimal SDK default (tool-calling support only), the `claude_code` preset (the full Claude Code CLI prompt, optionally extended with `append`), or a fully custom string supplied via the `systemPrompt`/`system_prompt` option. Understanding system prompts is a prerequisite to the Coordinator–Subagent pattern, since each subagent's specialized behavior is defined by giving it its own distinct `prompt` field in its `AgentDefinition`, separate from the coordinator's own system prompt.
Example: `system_prompt="You are a senior Python developer. Always follow PEP 8 style guidelines."` passed in `ClaudeAgentOptions` replaces the default prompt so every response in that session is shaped by that persona and constraint.
Source: https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts

---

### Concept: The `query()` Function and `ClaudeAgentOptions`/`Options`
Type: prerequisite
Definition: The basic programmatic entry point of the Claude Agent SDK. `query()` is the function that starts (or continues) an agent loop: it accepts a `prompt` (what you want Claude to do) and an `options` object (`ClaudeAgentOptions` in Python, `Options` in TypeScript) that configures everything about how the agent behaves — allowed/disallowed tools, permission mode, model, system prompt, MCP servers, subagent definitions, turn/budget limits, and session continuation flags — and returns an async iterator of messages that the caller consumes with `async for`. Nearly every other capability in this domain (subagent invocation, hooks, permission modes, session resume/fork) is configured as a field on this same options object, making it the prerequisite mechanism a learner must understand before configuring subagents, sessions, or enforcement rules.
Example: `query(prompt="Review utils.py for bugs...", options=ClaudeAgentOptions(allowed_tools=["Read","Edit","Glob"], permission_mode="acceptEdits"))` starts an agent loop that streams messages (Claude's reasoning, tool calls, tool results, and a final `ResultMessage`) while auto-approving the three listed tools.
Source: https://code.claude.com/docs/en/agent-sdk/quickstart

---

### Concept: The Context Window
Type: prerequisite
Definition: The total amount of information available to Claude during a session — it does not reset between turns. It accumulates the system prompt, tool definitions, conversation history, tool inputs, and tool outputs across every turn. Content that stays the same across turns (system prompt, tool definitions, CLAUDE.md) is automatically prompt-cached. When the context window approaches its limit, the SDK automatically compacts the conversation, summarizing older history while keeping recent exchanges and key decisions intact. Understanding the context window is a prerequisite for Context Passing/Isolation between coordinator and subagent (isolation exists specifically to keep the coordinator's context window from growing with every subagent's intermediate work) and for Session State Management (a session's persisted history is exactly what populates the context window on resume).
Example: A long session that reads many large files and runs verbose Bash commands can consume thousands of tokens per turn; once the accumulated context nears the model's limit, the SDK emits a `system` message with `subtype: "compact_boundary"` and replaces older messages with a summary rather than truncating silently.
Source: https://code.claude.com/docs/en/agent-sdk/agent-loop

---

### Concept: Model Context Protocol (MCP)
Type: prerequisite
Definition: An open standard for connecting AI agents to external tools and data sources — databases, APIs like Slack or GitHub, or custom in-process tools — without the developer writing custom tool-calling implementations for each one. In the Agent SDK, MCP servers can run as local stdio processes, connect over HTTP/SSE, or execute directly in-process ("SDK MCP servers"), and are configured via the `mcpServers`/`mcp_servers` option or a `.mcp.json` file. MCP tools follow the naming convention `mcp__<server-name>__<tool-name>` and require explicit permission (e.g., via `allowedTools` wildcards like `mcp__github__*`) before Claude can call them. Understanding MCP is a prerequisite for Task Decomposition Strategies (specialized subagents are frequently scoped to a specific MCP server's tools, e.g., a database-only worker) and for Enforcement/Handoff Patterns (deny/allow rules and subagent `disallowedTools` explicitly reference MCP server-level patterns such as `mcp__server` or `mcp__*`).
Example: A `filesystem` MCP server started with `command: "npx", args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]` and `allowedTools: ["mcp__filesystem__*"]` gives the agent read/list access to a specific directory tree through standardized MCP tools rather than a hand-written file-access tool.
Source: https://code.claude.com/docs/en/agent-sdk/mcp

---

## SOURCES CITED

- https://code.claude.com/docs/en/agent-sdk/agent-loop
- https://code.claude.com/docs/en/agent-sdk/subagents
- https://code.claude.com/docs/en/workflows
- https://code.claude.com/docs/en/agent-sdk/permissions
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/agent-sdk/hooks
- https://code.claude.com/docs/en/agent-sdk/sessions
- https://code.claude.com/docs/en/agent-sdk/overview
- https://code.claude.com/docs/en/agent-sdk/quickstart
- https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts
- https://code.claude.com/docs/en/agent-sdk/mcp
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
