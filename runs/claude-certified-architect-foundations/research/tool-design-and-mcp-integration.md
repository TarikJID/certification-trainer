Domain: Tool Design & MCP Integration (18% of the exam)

Description: Addresses designing effective tool interfaces with clear descriptions,
implementing structured error responses for MCP tools, distributing tools across
agents appropriately, integrating MCP servers into workflows, and selecting
built-in tools effectively.

Revision note (rework round 1): this revision adds concepts requested by the
evaluator after it extracted the exam guide's actual task-statement text for this
domain (Domain 2, tasks 2.1–2.5). New or rescoped entries are marked accordingly.
Two sub-claims named by the exam guide could not be verified in any official
source after dedicated searching; they are listed explicitly as unsourced near the
end of the Key Concepts section rather than defined from memory.

---

## Key Concepts

- Concept: Tool definition structure (name, description, input_schema)
  Type: key
  Definition: A user-defined tool passed to the Claude API is specified as an object with `name` (matching `^[a-zA-Z0-9_-]{1,128}$`), `description` (a detailed plaintext explanation of what the tool does, when it should be used, and how it behaves), `input_schema` (a JSON Schema object defining expected parameters), and an optional `input_examples` array of schema-valid example inputs. This schema is what Claude reads to decide whether and how to call the tool.
  Example: A `get_weather` tool is defined with `name: "get_weather"`, a description explaining it returns current weather for a location, and an `input_schema` requiring a `location` string and allowing an optional `unit` enum of `celsius`/`fahrenheit`.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: Best practices for writing tool descriptions
  Type: key
  Definition: Anthropic's guidance states that "extremely detailed descriptions" are "by far the most important factor in tool performance." A good description should cover what the tool does, when it should (and shouldn't) be used, what each parameter means, and any caveats — written as if explaining the tool "to a new hire on your team," making implicit context (query formats, niche terminology, resource relationships) explicit. Anthropic recommends at least 3-4 sentences per description, more for complex tools.
  Example: A poor description "Gets the stock price for a ticker." is contrasted with a good one: "Retrieves the current stock price for a given ticker symbol. The ticker symbol must be a valid symbol for a publicly traded company on a major US stock exchange like NYSE or NASDAQ. The tool will return the latest trade price in USD. It should be used when the user asks about the current or most recent price of a specific stock. It will not provide any other information about the stock or company."
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools ; https://www.anthropic.com/engineering/writing-tools-for-agents

- Concept: Ambiguous or overlapping tool descriptions cause misrouting [NEW — task 2.1]
  Type: key
  Definition: When multiple tools have vague, near-identical, or overlapping purposes — especially similar names — Claude can select the wrong one or supply incorrect parameters. Anthropic's engineering guidance names this directly as one of "the most common failures," giving as an example two similarly-named tools that are easy to confuse. The namespacing guidance in the same source family states plainly: "When tools overlap in function or have a vague purpose, agents can get confused about which ones to use." The fix is to give each tool a distinct, unambiguous name and description, or to namespace/consolidate overlapping tools so their boundaries are clear.
  Example: A tool set containing both `notification-send-user` and `notification-send-channel` is prone to misrouting because their names and likely descriptions are easy to conflate; Claude may send a user notification when a channel notification was intended, or vice versa.
  Source: https://www.anthropic.com/engineering/advanced-tool-use ; https://www.anthropic.com/engineering/writing-tools-for-agents

- Concept: Splitting generic tools into purpose-specific tools with defined input/output contracts [NEW — task 2.1]
  Type: key
  Definition: The inverse failure mode from having too many overlapping tools is having one tool that is too generic to give Claude a clear contract to reason about. Anthropic's guidance is: "Make sure each tool you build has a clear, distinct purpose," and recommends building "a few thoughtful tools targeting specific high-impact workflows" rather than one broad, do-everything tool wrapping raw API/software functionality. A single generic `list_contacts` tool that returns everything and forces Claude to page through irrelevant data token-by-token should instead be split (or replaced) with purpose-specific tools such as `search_contacts` or `message_contact`, each with a narrower, well-defined input and output.
  Example: Instead of one generic `list_contacts` tool that dumps an entire address book for Claude to scan, expose a purpose-specific `search_contacts` tool (input: a query string; output: only matching contacts) so the tool's contract matches the actual task.
  Source: https://www.anthropic.com/engineering/writing-tools-for-agents

- Concept: Tool consolidation and namespacing
  Type: key
  Definition: Rather than wrapping every API endpoint as a separate tool, effective tool design consolidates related, multi-step operations into fewer, more capable tools (e.g., one `schedule_event` tool that checks availability and schedules, instead of separate `list_users`, `list_events`, `create_event` tools), which reduces selection ambiguity. When many tools span multiple services, tools should be namespaced with a service/resource prefix (e.g., `asana_projects_search`, `github_list_prs`) so tool selection stays unambiguous as the tool library grows — MCP clients sometimes do this namespacing automatically.
  Example: Instead of `create_pr`, `review_pr`, `merge_pr` as three tools, expose one `github_pr` tool with an `action` parameter (`create`, `review`, `merge`).
  Source: https://www.anthropic.com/engineering/writing-tools-for-agents ; https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: Tool-set size and selection reliability [NEW — task 2.3]
  Type: key
  Definition: Loading too many tool definitions at once degrades both context efficiency and tool-selection accuracy. Anthropic documents that "a typical multiserver setup (GitHub, Slack, Sentry, Grafana, and Splunk) can consume ~55k tokens in definitions before Claude does any work," and that "Claude's ability to pick the right tool degrades once you exceed 30-50 available tools." The recommended mitigation is to keep only a small non-deferred set — "your 3-5 most frequently used tools" — always loaded, and use on-demand tool discovery (the tool search tool) for the rest so that selection accuracy stays high even across thousands of available tools.
  Example: A workspace aggregating five MCP servers exposes 60+ tools; instead of loading all of them up front, the 3-5 most-used tools (e.g., `github_search_issues`) stay non-deferred while the rest are marked `defer_loading: true` and discovered on demand via the tool search tool.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool

- Concept: tool_choice — controlling how Claude selects tools (auto, any, tool, none) [NEW — task 2.3]
  Type: key
  Definition: The `tool_choice` request parameter governs whether and how Claude must use the tools provided. `auto` (the default when tools are provided) lets Claude decide whether to call any provided tool. `any` requires Claude to use one of the provided tools, without forcing a particular one. `tool` (specified as `{"type": "tool", "name": "..."}`) forces Claude to always use one particular named tool. `none` (the default when no tools are provided) prevents Claude from using any tool. Forcing `any` or `tool` prefills the assistant turn to guarantee a tool call, which suppresses any natural-language explanation before the call; combining forced tool use with `strict: true` on the tool definition additionally guarantees the call's arguments conform exactly to the schema.
  Example: `"tool_choice": {"type": "tool", "name": "get_weather"}` forces Claude to call `get_weather` on this turn even if it would otherwise have answered directly from its own knowledge.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: High-signal tool response design
  Type: key
  Definition: Tool implementations should return only high-signal information to the agent rather than raw or bloated API payloads: prioritize contextual relevance over flexibility, return semantic/stable identifiers (slugs, human-readable names) instead of opaque internal references, and include only the fields Claude needs for its next step. For responses that could be large, implement pagination, range selection, filtering, and/or truncation with sensible defaults, and consider offering a response-detail parameter (concise vs. detailed).
  Example: A `search_contacts` tool that returns a filtered, paginated list of matching contacts (name, email) outperforms a `list_contacts` tool that dumps the entire contact database as raw JSON with internal UUIDs.
  Source: https://www.anthropic.com/engineering/writing-tools-for-agents

- Concept: Claude API tool_result error handling (is_error)
  Type: key
  Definition: When a client tool's execution fails, the application returns a `tool_result` content block with `tool_use_id`, an error message in `content`, and `"is_error": true`. Claude incorporates this into its response and can recover or retry. Error messages should be instructive — stating what went wrong and what to try next (e.g., "Rate limit exceeded. Retry after 60 seconds.") rather than a generic "failed." If Claude's tool call itself is invalid (e.g., missing a required parameter), returning a `tool_result` with `is_error: true` describing the missing parameter lets Claude retry with corrections (Claude typically retries 2-3 times before giving up); using `strict: true` on the tool definition can eliminate invalid calls entirely by guaranteeing schema-conformant inputs.
  Example: `{"type": "tool_result", "tool_use_id": "toolu_01...", "content": "ConnectionError: the weather service API is not available (HTTP 500)", "is_error": true}` causes Claude to tell the user it could not retrieve the weather and suggest trying again later.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls

- Concept: MCP tool result structure and error handling (isError)
  Type: key
  Definition: The MCP specification defines two error-reporting mechanisms for tools. Protocol errors are standard JSON-RPC errors (e.g., code -32602 for invalid arguments, or an unknown-tool error) returned when the request itself is malformed or the tool doesn't exist. Tool execution errors — API failures, invalid input data, business-logic errors — are reported inside a normal (successful) JSON-RPC response as a `CallToolResult` with `isError: true` and a descriptive `content` array, so the calling model/client can inspect and recover from the failure rather than the transport failing outright.
  Example: A weather MCP tool that hits a rate limit returns `{"jsonrpc": "2.0", "id": 4, "result": {"content": [{"type": "text", "text": "Failed to fetch weather data: API rate limit exceeded"}], "isError": true}}` rather than a JSON-RPC protocol error, because the tool executed but the underlying operation failed.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools

- Concept: Structured error categorization and retryability (transient, validation, permission, business-logic) [NEW — task 2.2]
  Type: key
  Definition: Official sources converge on the same categorical distinctions the exam guide names, even though no single page uses one unified field-name schema. The MCP tools specification itself groups tool execution errors into "API failures," "Invalid input data," and "Business logic errors" as the three named failure kinds reported via `isError: true`. The Claude API's HTTP error taxonomy separately distinguishes a `permission_error` (403 — the caller lacks access), an `invalid_request_error` (400 — the request/input must change before it can succeed, i.e. a validation failure), and errors that are explicitly retryable — the API's `rate_limit_error` (429) and `api_error`/`overloaded_error` (5xx) are, per Anthropic, the ones "the official SDKs automatically retry... with exponential backoff." Claude Code's own error handling documentation likewise separates permission-denial errors (e.g., "File is covered by a Read deny rule in your permission settings") from other failure kinds, and states it "retries transient failures up to 10 times with exponential backoff." Together these sources establish the same four-way distinction the exam guide names — transient/retryable, validation, permission, and business-logic — as the basis for structuring a tool's error metadata, even though "errorCategory"/"isRetryable" as literal field names were not found in any official Anthropic or MCP document.
  Example: A refund tool that fails because the requested amount exceeds a policy limit should be reported as a business-logic error (not retryable, needs a different plan) — analogous to MCP's "business logic errors" category — while a downstream payment API timeout should be reported the way Anthropic's SDKs already treat 5xx/429 responses: retryable with backoff.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools ; https://platform.claude.com/docs/en/api/errors ; https://code.claude.com/docs/en/errors

- Concept: Distinguishing access/permission failures from valid empty results [NEW — task 2.2]
  Type: key
  Definition: A tool or search that legitimately finds nothing must not be reported the same way as a tool call that was blocked or failed. Anthropic's tool search tool documentation states this explicitly for its own server-side search: "A search that matches nothing returns a `tool_search_tool_search_result` with an empty `tool_references` array, not an error." That same page's error-code taxonomy (`invalid_tool_input`, `unavailable`, `too_many_requests`, `execution_time_exceeded`) is reserved for genuine failures, kept separate from the empty-but-successful case. Claude Code's error reference draws the same line for file access: a permission `deny` rule produces an explicit denial message ("File is covered by a Read deny rule in your permission settings"), which is a distinct, reportable failure rather than an absence of results.
  Example: A `search_tickets` MCP tool that finds zero matching tickets for a query should return a successful, empty result list — not `isError: true` — whereas a `search_tickets` call the caller lacks permission to run at all should return a clearly labeled permission/access error so the caller doesn't mistake "denied" for "nothing found."
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool ; https://code.claude.com/docs/en/errors

- Concept: MCP structured content and output schema
  Type: key
  Definition: An MCP tool definition may include an optional `outputSchema` (a JSON Schema for the tool's expected output). When provided, servers MUST return structured results conforming to it in the result's `structuredContent` field, and clients SHOULD validate against it; for backwards compatibility a tool returning `structuredContent` SHOULD also serialize the same JSON into a `TextContent` block. This lets clients and LLMs parse and validate tool outputs reliably rather than relying on unstructured text.
  Example: A `get_weather_data` tool declares an `outputSchema` requiring `temperature`, `conditions`, and `humidity`; its response includes both a `text` block with the JSON stringified and a `structuredContent` object `{"temperature": 22.5, "conditions": "Partly cloudy", "humidity": 65}`.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools

- Concept: Tool annotations and trust boundary
  Type: key
  Definition: MCP tool definitions may include optional `annotations` — properties describing tool behavior (for display/decision purposes). Because annotations are supplied by the server itself, the specification warns that clients MUST treat tool annotations as untrusted unless they come from a trusted server, since a malicious or compromised server could mislabel a destructive tool as safe.
  Example: A file-deletion MCP tool might be annotated as read-only by a malicious server; a well-designed client does not rely on that annotation alone to skip a user confirmation prompt unless the server is explicitly trusted.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools

- Concept: Distributing tools across (sub)agents
  Type: key
  Definition: Claude Code subagents restrict tool access through a `tools` field (allowlist) and/or a `disallowedTools` field (denylist) in the subagent's frontmatter; if both are set, `disallowedTools` is applied first and then `tools` is resolved against what remains. Both fields accept MCP server-level patterns (`mcp__<server>` or `mcp__<server>__*`) to grant or remove an entire server's tools at once. Regardless of configuration, a fixed set of sensitive tools (e.g., `AskUserQuestion`, `EndConversation`, `ExitPlanMode` unless in plan mode) is always removed from subagents, and background subagents are further restricted to a smaller built-in tool set than foreground subagents. This lets an architect grant a read-only research subagent only `Read, Grep, Glob` while a coding subagent also gets `Write, Edit, Bash`.
  Example: A `code-reviewer` subagent configured with `tools: Read, Glob, Grep` can analyze code but cannot modify files, while a `coordinator` subagent configured with `tools: Agent(worker, researcher), Read, Bash` can only spawn the `worker` and `researcher` subagent types.
  Source: https://code.claude.com/docs/en/sub-agents

- Concept: Integrating MCP servers into workflows
  Type: key
  Definition: Claude Code connects to MCP servers over multiple transports — stdio (local processes, ideal for tools needing direct system access), HTTP (recommended for remote servers, supports OAuth), SSE (deprecated in favor of HTTP), and WebSocket (for servers that push events unprompted) — configured via `claude mcp add --transport <type> <name> <url-or-command>`. Servers can be registered at three scopes: local (private, current project only, stored in `~/.claude.json`), project (team-shared via a version-controlled `.mcp.json` at the project root), and user (private, available across all of a user's projects). When the same server name is defined at multiple scopes, Claude Code uses the single highest-precedence definition (local > project > user > plugin-provided > claude.ai connectors) rather than merging fields.
  Example: A team adds a shared Jira MCP server with `claude mcp add --transport http jira https://mcp.jira.example.com --scope project`, which writes an entry to `.mcp.json` that is checked into version control so every teammate gets the same tool once they approve the project-scoped server.
  Source: https://code.claude.com/docs/en/mcp

- Concept: Environment variable expansion in .mcp.json for credential management [NEW — task 2.4]
  Type: key
  Definition: Claude Code supports environment variable expansion inside `.mcp.json` so teams can share one configuration file without hard-coding machine-specific paths or secrets. Supported syntax: `${VAR}` expands to the value of environment variable `VAR`, and `${VAR:-default}` expands to `VAR` if set, otherwise falls back to `default`. Expansion applies in a server entry's `command`, `args`, `env`, `url`, and `headers` fields. If a referenced variable is unset and has no default, the config still loads: Claude Code reports a missing-variable warning in `claude mcp list` output and leaves the literal `${VAR}` text unexpanded. Separately, Claude Code protects a fixed list of credential-like variable names (e.g. `ANTHROPIC_API_KEY`, `AWS_BEARER_TOKEN_BEDROCK`, `HTTPS_PROXY`) by always reading them as empty inside a remote server's `url`/`headers`, so a shared or plugin-provided `.mcp.json` can never exfiltrate the local user's own credentials to a named server.
  Example: `{"mcpServers": {"api-server": {"type": "http", "url": "${API_BASE_URL:-https://api.example.com}/mcp", "headers": {"Authorization": "Bearer ${API_KEY}"}}}}` lets every teammate share the same committed `.mcp.json` while each supplies their own `API_KEY` locally.
  Source: https://code.claude.com/docs/en/mcp

- Concept: MCP resources as an addressable content catalog [NEW — task 2.4]
  Type: key
  Definition: Resources are one of MCP's three server-exposed primitives, alongside tools and prompts: each resource is "uniquely identified by a URI" and represents data a server wants to share for context — "files, database schemas, or application-specific information." Clients discover the catalog with `resources/list` and fetch a specific item with `resources/read`, without needing the model to guess at and invoke a discovery tool first. Claude Code exposes this to users directly: a resource can be referenced inline with an `@server:resource` mention (documented example: `@github:repos/owner/repo/issues`), which "fetches data from connected MCP servers" for inclusion in context. Because a resource is fetched by a known, stable URI rather than searched for, this pattern avoids the exploratory back-and-forth of a tool call whose job is merely to locate content that the server could already list or address directly.
  Example: Instead of asking Claude to call a `search_issues` tool and hope it finds the right ticket, a user (or the system prompt) references the ticket directly: `Can you analyze @github:repos/owner/repo/issues and suggest a fix?`, letting Claude read the resource's contents via `resources/read` immediately.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/resources ; https://code.claude.com/docs/en/common-workflows

- Concept: Dynamic tool discovery and updates in MCP workflows
  Type: key
  Definition: MCP clients discover a server's tools via a `tools/list` request (paginated) and invoke them via `tools/call`. Servers that declare the `tools` capability with `listChanged: true` can send a `notifications/tools/list_changed` message when their available tools change; Claude Code automatically refreshes the tool list from that server on receiving this notification, without requiring the user to disconnect and reconnect, and — with tool search enabled — can list a newly connected server's tools to Claude mid-turn so they can be used without waiting for the next user message.
  Example: An MCP server that gains a new "create_invoice" tool after a feature flag flips sends `notifications/tools/list_changed`; Claude Code calls `tools/list` again and the new tool becomes available in the same session.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools ; https://code.claude.com/docs/en/mcp

- Concept: Functional selection among Claude Code's file and search tools (Grep, Glob, Read, Write, Edit) [NEW/RESCOPED — task 2.5]
  Type: key
  Definition: Claude Code's built-in tools reference documents distinct, non-overlapping functional purposes for its core file/search tools, and selecting the right one for a task is itself a tested skill. Per the reference: Glob "finds files by name pattern" (file path pattern matching), while Grep searches file contents for patterns — "Where Glob finds files by name, Grep finds lines inside them" (content search). Read "reads the contents of files" (full-file reading, with line numbers, pagination via offset/limit, and support for images/PDFs/notebooks). Write "creates or overwrites files" (a full-file operation, replacing entire contents). Edit "makes targeted edits to specific files" (a narrower, surgical modification via exact string replacement), distinct from Write's whole-file replacement.
  Example: To find which files mention a deprecated config key, use Grep (content search); to find every `*.test.ts` file in a repo, use Glob (path pattern matching); to view a whole file before deciding what to change, use Read; to replace one function's body in place, use Edit; to regenerate an entire generated file from scratch, use Write.
  Source: https://code.claude.com/docs/en/tools-reference

- Concept: Edit tool's uniqueness constraint and recovery from non-unique matches [NEW — task 2.5]
  Type: key
  Definition: The Edit tool requires its `old_string` target to match exactly one location in the file. Per Claude Code's tools reference: "`old_string` must appear exactly once. When it appears more than once, Claude either supplies a longer string with enough surrounding context to pin down one occurrence, or sets `replace_all: true` to replace them all." This is the officially documented recovery path when a targeted Edit call fails on a non-unique match: narrow the match with more surrounding context, or explicitly opt into replacing every occurrence. (Note: official Claude Code documentation was searched specifically for a named "fall back to Read + Write" recovery pattern for this failure and none was found; see the Unsourced section below.)
  Example: An Edit call with `old_string: "return x"` fails because that string appears five times in the file; Claude either expands `old_string` to include several surrounding lines that occur only once, or reissues the call with `replace_all: true` if every occurrence should legitimately change.
  Source: https://code.claude.com/docs/en/tools-reference

- Concept: Restricting built-in tools via Claude Code permissions [RESCOPED — was previously mislabeled "Selecting built-in tools effectively"]
  Type: key
  Definition: Which of Claude Code's built-in tools (Read, Write, Edit, Bash, PowerShell, Glob, Grep, LSP, WebFetch, WebSearch, Agent, Monitor, Artifact, NotebookEdit, task-tracking tools, AskUserQuestion, EnterWorktree/ExitWorktree, EnterPlanMode/ExitPlanMode, SendMessage, Skill, Workflow, and more) are active in a given session can be restricted via CLI flags (`--allowedTools`, `--disallowedTools`, `--tools`) or via path/command-scoped rules in `settings.json` (`permissions.allow` / `permissions.deny`, e.g. `"Edit(src/**)"`, `"Bash(npm run *)"`, `"WebFetch(domain:github.com)"`). This is the access-control layer that complements — but is distinct from — choosing the functionally right tool for a task: it governs what a session or subagent is *permitted* to invoke at all, regardless of which tool would otherwise be the best functional fit.
  Example: A read-only auditing session is started with `claude --allowedTools Read,Grep,Bash` so the agent can inspect code but never call `Write` or `Edit`; a `settings.json` `deny` rule of `"Bash(rm *)"` blocks destructive shell deletions regardless of what else is allowed.
  Source: https://code.claude.com/docs/en/tools-reference

---

## Prerequisite Concepts

- Concept: Tool use (function calling) and the agentic tool-use loop
  Type: prerequisite
  Definition: Tool use (also called function calling) lets Claude call functions defined by the developer (client tools) or provided by Anthropic (server tools). Claude decides whether to call a tool based on the user's request and the tool's description; for client tools, Claude returns a `stop_reason` of `tool_use` with one or more `tool_use` content blocks (each with `id`, `name`, `input`), the application executes the corresponding function, and sends the result back as a `tool_result` block (matched by `tool_use_id`) in a subsequent `user` message so Claude can continue the response.
  Example: Given "What's the weather in San Francisco?", Claude emits a `tool_use` block calling `get_weather` with `{"location": "San Francisco, CA"}`; the application runs its weather lookup and replies with a `tool_result` block containing "15 degrees Celsius, partly cloudy", after which Claude produces the final natural-language answer.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

- Concept: JSON Schema for tool inputs
  Type: prerequisite
  Definition: Both the Claude API's `input_schema` and MCP's `inputSchema`/`outputSchema` fields are JSON Schema objects: a standard, language-agnostic vocabulary for describing the shape of JSON data (types, required properties, enums, nested objects) that validators and language models can both use to understand and constrain valid tool arguments or outputs.
  Example: `{"type": "object", "properties": {"location": {"type": "string"}}, "required": ["location"]}` declares that a tool's input must be an object with a required string `location` property.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: MCP architecture — Host, Client, Server, and primitives
  Type: prerequisite
  Definition: MCP follows a client-server architecture: an MCP Host (an AI application such as Claude Code or Claude Desktop) creates one MCP Client per MCP Server it connects to, and each Client maintains a dedicated connection to its Server. MCP separates a data layer (a JSON-RPC 2.0-based protocol defining capability discovery and core primitives — Tools for AI-invokable actions, Resources for contextual data, and Prompts for reusable interaction templates) from a transport layer (stdio for local processes, or Streamable HTTP for remote servers, which supports bearer tokens, API keys, and OAuth).
  Example: When Claude Code connects to both a local filesystem MCP server (over stdio) and a remote Sentry MCP server (over Streamable HTTP), Claude Code (the Host) instantiates one MCP Client for the filesystem server and another for Sentry, each maintaining its own connection.
  Source: https://modelcontextprotocol.io/docs/learn/architecture

- Concept: JSON-RPC 2.0
  Type: prerequisite
  Definition: JSON-RPC 2.0 is the underlying remote-procedure-call message format MCP uses for all client-server communication: requests carry a `jsonrpc` version, `id`, `method`, and `params`; responses carry either a `result` or an `error` (with a numeric `code` and `message`); notifications are one-way messages with no `id` and expect no response. Understanding this format is necessary to distinguish an MCP protocol-level error from a tool-execution-level error reported inside a successful result.
  Example: A request for a nonexistent tool returns a JSON-RPC error object `{"jsonrpc": "2.0", "id": 3, "error": {"code": -32602, "message": "Unknown tool: invalid_tool_name"}}` rather than a tool result.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools

- Concept: Claude Code subagents
  Type: prerequisite
  Definition: Subagents are specialized AI assistants, each defined with its own system prompt, model, and permissions, that run in an isolated context window separate from the main conversation. Claude automatically delegates matching tasks to a subagent based on its description, and (for non-fork subagents) the subagent does not see the main conversation's history; only its final summary returns to the caller. Claude Code ships built-in subagents (Explore: read-only codebase search; Plan: read-only research for plan mode; general-purpose: full tool access for complex multi-step work) in addition to user-defined ones.
  Example: Instead of running a large log-searching task in the main conversation (flooding it with irrelevant output), Claude Code delegates it to a subagent that reads the logs in its own context and returns only a short summary of findings.
  Source: https://code.claude.com/docs/en/sub-agents

- Concept: Claude Code permission system (settings.json and CLI tool flags)
  Type: prerequisite
  Definition: Claude Code controls which tools a session (or subagent) may use through a permission system: CLI flags (`--allowedTools`, `--disallowedTools`, `--tools`) and `settings.json` `permissions.allow`/`permissions.deny` rules, which can be scoped to specific paths (`Read(src/**)`), commands (`Bash(npm run *)`), or domains (`WebFetch(domain:github.com)`). This permission layer is the mechanism underlying both "selecting built-in tools effectively" and subagent-level tool restriction.
  Example: `{"permissions": {"deny": ["WebSearch", "Bash(rm *)"]}}` in `settings.json` prevents any session using that configuration from performing web searches or running `rm` commands, regardless of what a subagent's own `tools` field allows.
  Source: https://code.claude.com/docs/en/tools-reference

---

## Unsourced (could not be verified in an official source)

- Sub-claim: "System prompt wording has a keyword-sensitive effect that can create unintended *associations between specific tools* (biasing which of several tools gets picked)."
  What was found instead: Official Claude API documentation confirms the related but narrower claim that system prompt wording steers *whether* Claude calls a tool at all versus responding directly (e.g., "Use the tools to investigate before responding." increases tool use; "Use your judgment..." keeps triggering conservative) — see https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview. Dedicated searches of platform.claude.com, code.claude.com, and anthropic.com/engineering (writing-tools-for-agents, advanced-tool-use) did not surface an official statement about keyword-sensitive wording creating unintended associations *between* multiple candidate tools specifically. Not defined as its own concept above to avoid overstating source support.

- Sub-claim: "When Edit fails due to non-unique text matches, the documented/recommended fallback is to use Read + Write instead."
  What was found instead: Claude Code's tools reference documents Edit's actual recovery mechanism for non-unique matches as supplying more surrounding context or setting `replace_all: true` (see the "Edit tool's uniqueness constraint and recovery from non-unique matches" concept above, sourced to https://code.claude.com/docs/en/tools-reference). No official page was found that names "read the whole file, then overwrite it with Write" as the documented fallback strategy for this specific failure, despite targeted searches of code.claude.com/docs/en/tools-reference, code.claude.com/docs/en/common-workflows, and the Anthropic-schema text editor tool docs.

---

## Sources Cited

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool
- https://platform.claude.com/docs/en/api/errors
- https://www.anthropic.com/engineering/writing-tools-for-agents
- https://www.anthropic.com/engineering/advanced-tool-use
- https://modelcontextprotocol.io/specification/2025-06-18/server/tools
- https://modelcontextprotocol.io/specification/2025-06-18/server/resources
- https://modelcontextprotocol.io/docs/learn/architecture
- https://code.claude.com/docs/en/mcp
- https://code.claude.com/docs/en/sub-agents
- https://code.claude.com/docs/en/tools-reference
- https://code.claude.com/docs/en/errors
- https://code.claude.com/docs/en/common-workflows
