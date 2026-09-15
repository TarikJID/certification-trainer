# Module 3: Tool Design & MCP Integration

Domain weight on the exam: 18%. Builds directly on Module 1 (tool use, JSON
Schema, tool definition structure) and Module 2 (settings/permission modes,
the general subagent notion, the Explore subagent).

---

## Lesson 3.1: Writing Effective Tool Descriptions

### Concept: Best practices for writing tool descriptions
Anthropic's guidance states that "extremely detailed descriptions" are "by
far the most important factor in tool performance." A good description
covers what the tool does, when it should (and shouldn't) be used, what each
parameter means, and any caveats — written as if explaining the tool "to a
new hire on your team," making implicit context (query formats, niche
terminology, resource relationships) explicit. Anthropic recommends at least
3-4 sentences per description, more for complex tools. This builds directly
on Module 1's basic `name`/`description`/`input_schema` shape.

**Example:** A poor description, "Gets the stock price for a ticker," is
contrasted with a good one: "Retrieves the current stock price for a given
ticker symbol. The ticker symbol must be a valid symbol for a publicly
traded company on a major US stock exchange like NYSE or NASDAQ. The tool
will return the latest trade price in USD. It should be used when the user
asks about the current or most recent price of a specific stock. It will
not provide any other information about the stock or company."

**Source:** Tool Design & MCP Integration research — Best practices for
writing tool descriptions (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools ; https://www.anthropic.com/engineering/writing-tools-for-agents

### Concept: Ambiguous or overlapping tool descriptions cause misrouting
When multiple tools have vague, near-identical, or overlapping purposes —
especially similar names — Claude can select the wrong one or supply
incorrect parameters. Anthropic's engineering guidance names this directly
as one of "the most common failures." The namespacing guidance states
plainly: "When tools overlap in function or have a vague purpose, agents can
get confused about which ones to use." The fix is to give each tool a
distinct, unambiguous name and description, or to namespace/consolidate
overlapping tools so their boundaries are clear (see Lesson 3.2).

**Example:** A tool set containing both `notification-send-user` and
`notification-send-channel` is prone to misrouting because their names and
likely descriptions are easy to conflate; Claude may send a user
notification when a channel notification was intended, or vice versa.

**Source:** Tool Design & MCP Integration research — Ambiguous or
overlapping tool descriptions cause misrouting [exam task 2.1] (key), https://www.anthropic.com/engineering/advanced-tool-use ; https://www.anthropic.com/engineering/writing-tools-for-agents

### Concept: System prompt wording steers whether Claude calls a tool at all
Beyond a tool's own description, the surrounding system prompt shapes
whether Claude calls a tool in the first place. With the default
`tool_choice` of `auto`, "this boundary is steerable through your system
prompt": a light instruction such as "Use the tools to investigate before
responding" increases tool use; a stronger instruction such as "Always call
a tool first before responding" pushes further toward always calling a
tool; and conversely, "Use your judgment about whether to call a tool or
respond directly" keeps triggering conservative. This is a narrower claim
than the next concept: it's about whether a tool gets called at all, not
about biasing selection among several specific candidate tools.

**Example:** A system prompt saying "Use your judgment about whether to call
a tool or respond directly" leaves Claude free to answer "What's 2+2?"
without invoking any tool, whereas "Always call a tool first before
responding" pushes Claude toward invoking a tool even for a question it
could answer directly.

**Source:** Tool Design & MCP Integration research — System prompt wording
steers whether Claude calls a tool at all (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

### Concept: Keyword-sensitive system prompt wording can create unintended tool associations
> **Sourcing note:** this is a stronger, more specific claim than the one
> above, sourced only to the official CCAR-F Exam Guide (Task Statement
> 2.1), not to Anthropic's vendor documentation. A dedicated search of
> platform.claude.com, code.claude.com, and anthropic.com/engineering did
> not find it stated there. Treat it as an exam-guide fact, distinct from
> the vendor-documented claim above.

The exam guide names, as a Task 2.1 knowledge item, "the impact of system
prompt wording on tool selection: keyword-sensitive instructions can create
unintended tool associations," paired with the skill of "reviewing system
prompts for keyword-sensitive instructions that might override well-written
tool descriptions." This asserts that particular wording in a system prompt
can, via keyword sensitivity, become associated with a specific tool in a
way that was not intended — to the point of overriding what an otherwise
well-written tool description would have produced.

**Example:** Reviewing a system prompt for language that repeats a keyword
also present in one tool's name or description, in a way unrelated to that
tool's intended use, is the guide-named review skill for catching this
failure mode before it produces an unintended tool association.

**Source:** Claude Certified Architect – Foundations Exam Guide, Task
Statement 2.1 (exam-guide-only; not found in vendor documentation).

---

## Lesson 3.2: Structuring the Tool Set

### Concept: Splitting generic tools into purpose-specific tools with defined input/output contracts
The inverse failure mode from having too many overlapping tools is having
one tool that is too generic to give Claude a clear contract to reason
about. Anthropic's guidance: "Make sure each tool you build has a clear,
distinct purpose," recommending "a few thoughtful tools targeting specific
high-impact workflows" rather than one broad, do-everything tool wrapping
raw API/software functionality.

**Example:** A single generic `list_contacts` tool that returns everything
and forces Claude to page through irrelevant data token-by-token should
instead be split (or replaced) with purpose-specific tools such as
`search_contacts` or `message_contact`, each with a narrower, well-defined
input and output.

**Source:** Tool Design & MCP Integration research — Splitting generic
tools into purpose-specific tools [exam task 2.1] (key), https://www.anthropic.com/engineering/writing-tools-for-agents

### Concept: Tool consolidation and namespacing
Rather than wrapping every API endpoint as a separate tool, effective tool
design consolidates related, multi-step operations into fewer, more capable
tools (e.g., one `schedule_event` tool that checks availability and
schedules, instead of separate `list_users`, `list_events`, `create_event`
tools), which reduces selection ambiguity. When many tools span multiple
services, tools should be namespaced with a service/resource prefix (e.g.,
`asana_projects_search`, `github_list_prs`) so tool selection stays
unambiguous as the tool library grows — MCP clients sometimes do this
namespacing automatically.

**Example:** Instead of `create_pr`, `review_pr`, `merge_pr` as three tools,
expose one `github_pr` tool with an `action` parameter (`create`, `review`,
`merge`).

**Source:** Tool Design & MCP Integration research — Tool consolidation and
namespacing (key), https://www.anthropic.com/engineering/writing-tools-for-agents ; https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

### Concept: Tool-set size and selection reliability
Loading too many tool definitions at once degrades both context efficiency
and tool-selection accuracy. Anthropic documents that "a typical multiserver
setup (GitHub, Slack, Sentry, Grafana, and Splunk) can consume ~55k tokens
in definitions before Claude does any work," and that "Claude's ability to
pick the right tool degrades once you exceed 30-50 available tools." The
recommended mitigation: keep only a small non-deferred set — "your 3-5 most
frequently used tools" — always loaded, and use on-demand tool discovery
(the tool search tool) for the rest.

**Example:** A workspace aggregating five MCP servers exposes 60+ tools;
instead of loading all of them up front, the 3-5 most-used tools (e.g.,
`github_search_issues`) stay non-deferred while the rest are marked
`defer_loading: true` and discovered on demand via the tool search tool.

**Source:** Tool Design & MCP Integration research — Tool-set size and
selection reliability [exam task 2.3] (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool

### Concept: tool_choice — controlling how Claude selects tools (auto, any, tool, none)
The `tool_choice` request parameter governs whether and how Claude must use
the tools provided: `auto` (default when tools are provided) lets Claude
decide whether to call any provided tool; `any` requires Claude to use one
of the provided tools, without forcing a particular one; `tool` (specified
as `{"type": "tool", "name": "..."}`) forces Claude to always use one
particular named tool; `none` (default when no tools are provided) prevents
any tool use. Forcing `any` or `tool` prefills the assistant turn to
guarantee a tool call, suppressing any natural-language explanation before
the call; combining forced tool use with `strict: true` on the tool
definition additionally guarantees the call's arguments conform exactly to
the schema (Module 5 covers `strict: true` and structured outputs in
depth).

**Example:** `"tool_choice": {"type": "tool", "name": "get_weather"}` forces
Claude to call `get_weather` on this turn even if it would otherwise have
answered directly from its own knowledge.

**Source:** Tool Design & MCP Integration research — tool_choice [exam task
2.3] (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools . This same concept and source is also cited from the
Prompt Engineering & Structured Output research as "Defining tools and
forcing tool use (tool_choice)" — taught once, here.

### Concept: High-signal tool response design
Tool implementations should return only high-signal information to the
agent rather than raw or bloated API payloads: prioritize contextual
relevance over flexibility, return semantic/stable identifiers (slugs,
human-readable names) instead of opaque internal references, and include
only the fields Claude needs for its next step. For responses that could be
large, implement pagination, range selection, filtering, and/or truncation
with sensible defaults, and consider offering a response-detail parameter
(concise vs. detailed).

**Example:** A `search_contacts` tool that returns a filtered, paginated
list of matching contacts (name, email) outperforms a `list_contacts` tool
that dumps the entire contact database as raw JSON with internal UUIDs.

**Source:** Tool Design & MCP Integration research — High-signal tool
response design (key), https://www.anthropic.com/engineering/writing-tools-for-agents

---

## Lesson 3.3: Error Handling for Tools and MCP

### Concept: Claude API tool_result error handling (is_error)
When a client tool's execution fails, the application returns a
`tool_result` content block with `tool_use_id`, an error message in
`content`, and `"is_error": true`. Claude incorporates this into its
response and can recover or retry. Error messages should be instructive —
stating what went wrong and what to try next (e.g., "Rate limit exceeded.
Retry after 60 seconds.") rather than a generic "failed." If Claude's tool
call itself is invalid (e.g., a missing required parameter), returning a
`tool_result` with `is_error: true` describing the missing parameter lets
Claude retry with corrections (Claude typically retries 2-3 times before
giving up); `strict: true` on the tool definition can eliminate invalid
calls entirely by guaranteeing schema-conformant inputs (Module 5).

**Example:** `{"type": "tool_result", "tool_use_id": "toolu_01...",
"content": "ConnectionError: the weather service API is not available (HTTP
500)", "is_error": true}` causes Claude to tell the user it could not
retrieve the weather and suggest trying again later.

**Source:** Tool Design & MCP Integration research — Claude API tool_result
error handling (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls . This concept and source is also cited from the
Prompt Engineering & Structured Output research as "Validation and retry
loops for tool calls" — taught once, here, and extended in Module 5 with
the batch/structured-output angle.

### Concept: MCP tool result structure and error handling (isError)
The MCP specification defines two error-reporting mechanisms. Protocol
errors are standard JSON-RPC errors (e.g., code -32602 for invalid
arguments, or an unknown-tool error), returned when the request itself is
malformed or the tool doesn't exist. Tool execution errors — API failures,
invalid input data, business-logic errors — are reported inside a normal
(successful) JSON-RPC response as a `CallToolResult` with `isError: true`
and a descriptive `content` array, so the calling model/client can inspect
and recover from the failure rather than the transport failing outright.

**Example:** A weather MCP tool that hits a rate limit returns
`{"jsonrpc": "2.0", "id": 4, "result": {"content": [{"type": "text", "text":
"Failed to fetch weather data: API rate limit exceeded"}], "isError":
true}}` rather than a JSON-RPC protocol error, because the tool executed but
the underlying operation failed.

**Source:** Tool Design & MCP Integration research — MCP tool result
structure and error handling (key), https://modelcontextprotocol.io/specification/2025-06-18/server/tools

### Concept: Structured error categorization and retryability
Official sources converge on the same categorical distinctions the exam
guide names, even though no single page uses one unified field-name schema.
The MCP tools specification groups tool execution errors into "API
failures," "Invalid input data," and "Business logic errors." The Claude
API's HTTP error taxonomy separately distinguishes a `permission_error`
(403), an `invalid_request_error` (400, a validation failure), and errors
that are explicitly retryable — `rate_limit_error` (429) and
`api_error`/`overloaded_error` (5xx) are the ones "the official SDKs
automatically retry... with exponential backoff." Claude Code's own error
handling separates permission-denial errors from other failure kinds and
"retries transient failures up to 10 times with exponential backoff."
Together these establish the same four-way distinction the exam guide
names — transient/retryable, validation, permission, and business-logic —
even though literal field names like "errorCategory"/"isRetryable" were not
found in any official document.

**Example:** A refund tool that fails because the requested amount exceeds
a policy limit should be reported as a business-logic error (not retryable,
needs a different plan) — analogous to MCP's "business logic errors"
category — while a downstream payment API timeout should be treated the way
Anthropic's SDKs already treat 5xx/429 responses: retryable with backoff.

**Source:** Tool Design & MCP Integration research — Structured error
categorization and retryability [exam task 2.2] (key), https://modelcontextprotocol.io/specification/2025-06-18/server/tools ; https://platform.claude.com/docs/en/api/errors ; https://code.claude.com/docs/en/errors

### Concept: Distinguishing access/permission failures from valid empty results
A tool or search that legitimately finds nothing must not be reported the
same way as a tool call that was blocked or failed. Anthropic's tool search
tool documentation states this explicitly: "A search that matches nothing
returns a `tool_search_tool_search_result` with an empty `tool_references`
array, not an error." That page's error-code taxonomy
(`invalid_tool_input`, `unavailable`, `too_many_requests`,
`execution_time_exceeded`) is reserved for genuine failures. Claude Code's
error reference draws the same line for file access: a permission `deny`
rule produces an explicit denial message ("File is covered by a Read deny
rule in your permission settings") — a distinct, reportable failure rather
than an absence of results.

**Example:** A `search_tickets` MCP tool that finds zero matching tickets
should return a successful, empty result list — not `isError: true` —
whereas a `search_tickets` call the caller lacks permission to run at all
should return a clearly labeled permission/access error so the caller
doesn't mistake "denied" for "nothing found."

**Source:** Tool Design & MCP Integration research — Distinguishing
access/permission failures from valid empty results [exam task 2.2] (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool ; https://code.claude.com/docs/en/errors

### Concept: MCP structured content and output schema
An MCP tool definition may include an optional `outputSchema` (a JSON
Schema for the tool's expected output — the same JSON Schema vocabulary
from Module 1). When provided, servers MUST return structured results
conforming to it in the result's `structuredContent` field, and clients
SHOULD validate against it; for backwards compatibility a tool returning
`structuredContent` SHOULD also serialize the same JSON into a
`TextContent` block. This lets clients and LLMs parse and validate tool
outputs reliably rather than relying on unstructured text.

**Example:** A `get_weather_data` tool declares an `outputSchema` requiring
`temperature`, `conditions`, and `humidity`; its response includes both a
`text` block with the JSON stringified and a `structuredContent` object
`{"temperature": 22.5, "conditions": "Partly cloudy", "humidity": 65}`.

**Source:** Tool Design & MCP Integration research — MCP structured content
and output schema (key), https://modelcontextprotocol.io/specification/2025-06-18/server/tools

### Concept: Tool annotations and trust boundary
MCP tool definitions may include optional `annotations` — properties
describing tool behavior for display/decision purposes. Because annotations
are supplied by the server itself, the specification warns that clients
MUST treat tool annotations as untrusted unless they come from a trusted
server, since a malicious or compromised server could mislabel a
destructive tool as safe.

**Example:** A file-deletion MCP tool might be annotated as read-only by a
malicious server; a well-designed client does not rely on that annotation
alone to skip a user confirmation prompt unless the server is explicitly
trusted.

**Source:** Tool Design & MCP Integration research — Tool annotations and
trust boundary (key), https://modelcontextprotocol.io/specification/2025-06-18/server/tools

---

## Lesson 3.4: MCP Architecture and Integration

### Concept: MCP architecture — Host, Client, Server, and primitives
MCP follows a client-server architecture: an MCP Host (an AI application
such as Claude Code or Claude Desktop) creates one MCP Client per MCP Server
it connects to, and each Client maintains a dedicated connection to its
Server. MCP separates a data layer (a JSON-RPC 2.0-based protocol defining
capability discovery and core primitives — Tools for AI-invokable actions,
Resources for contextual data, and Prompts for reusable interaction
templates) from a transport layer (stdio for local processes, or Streamable
HTTP for remote servers, supporting bearer tokens, API keys, and OAuth).
This is also the fuller treatment of the "Model Context Protocol"
prerequisite concept that Module 4 (Agentic Architecture) relies on for
task decomposition and enforcement patterns.

**Example:** When Claude Code connects to both a local filesystem MCP
server (over stdio) and a remote Sentry MCP server (over Streamable HTTP),
Claude Code (the Host) instantiates one MCP Client for the filesystem
server and another for Sentry, each maintaining its own connection.

**Source:** Merges Tool Design & MCP Integration's "MCP architecture"
(prerequisite), https://modelcontextprotocol.io/docs/learn/architecture ;
with Agentic Architecture & Orchestration's "Model Context Protocol (MCP)"
(prerequisite), https://code.claude.com/docs/en/agent-sdk/mcp

### Concept: JSON-RPC 2.0
JSON-RPC 2.0 is the underlying remote-procedure-call message format MCP
uses for all client-server communication: requests carry a `jsonrpc`
version, `id`, `method`, and `params`; responses carry either a `result` or
an `error` (with a numeric `code` and `message`); notifications are
one-way messages with no `id` and expect no response. Understanding this
format is necessary to distinguish an MCP protocol-level error from a
tool-execution-level error reported inside a successful result (Lesson
3.3).

**Example:** A request for a nonexistent tool returns a JSON-RPC error
object `{"jsonrpc": "2.0", "id": 3, "error": {"code": -32602, "message":
"Unknown tool: invalid_tool_name"}}` rather than a tool result.

**Source:** Tool Design & MCP Integration research — JSON-RPC 2.0
(prerequisite), https://modelcontextprotocol.io/specification/2025-06-18/server/tools

### Concept: Integrating MCP servers into workflows
Claude Code connects to MCP servers over multiple transports — stdio (local
processes, ideal for tools needing direct system access), HTTP (recommended
for remote servers, supports OAuth), SSE (deprecated in favor of HTTP), and
WebSocket (for servers that push events unprompted) — configured via `claude
mcp add --transport <type> <name> <url-or-command>`. Servers register at
three scopes: local (private, current project only, `~/.claude.json`),
project (team-shared via a version-controlled `.mcp.json`), and user
(private, across all of a user's projects). When the same server name is
defined at multiple scopes, Claude Code uses the single highest-precedence
definition (local > project > user > plugin-provided > claude.ai
connectors) rather than merging fields.

**Example:** A team adds a shared Jira MCP server with `claude mcp add
--transport http jira https://mcp.jira.example.com --scope project`, which
writes an entry to `.mcp.json` that is checked into version control so every
teammate gets the same tool once they approve the project-scoped server.

**Source:** Tool Design & MCP Integration research — Integrating MCP
servers into workflows (key), https://code.claude.com/docs/en/mcp

### Concept: Environment variable expansion in .mcp.json for credential management
Claude Code supports environment variable expansion inside `.mcp.json` so
teams can share one configuration file without hard-coding machine-specific
paths or secrets. Supported syntax: `${VAR}` expands to the value of
environment variable `VAR`, and `${VAR:-default}` expands to `VAR` if set,
otherwise falls back to `default`. Expansion applies in a server entry's
`command`, `args`, `env`, `url`, and `headers` fields. If a referenced
variable is unset and has no default, the config still loads: Claude Code
reports a missing-variable warning in `claude mcp list` output and leaves
the literal `${VAR}` text unexpanded. Separately, Claude Code protects a
fixed list of credential-like variable names (e.g. `ANTHROPIC_API_KEY`,
`AWS_BEARER_TOKEN_BEDROCK`, `HTTPS_PROXY`) by always reading them as empty
inside a remote server's `url`/`headers`, so a shared or plugin-provided
`.mcp.json` can never exfiltrate the local user's own credentials to a named
server.

**Example:** `{"mcpServers": {"api-server": {"type": "http", "url":
"${API_BASE_URL:-https://api.example.com}/mcp", "headers": {"Authorization":
"Bearer ${API_KEY}"}}}}` lets every teammate share the same committed
`.mcp.json` while each supplies their own `API_KEY` locally.

**Source:** Tool Design & MCP Integration research — Environment variable
expansion in .mcp.json [exam task 2.4] (key), https://code.claude.com/docs/en/mcp

### Concept: MCP resources as an addressable content catalog
Resources are one of MCP's three server-exposed primitives, alongside tools
and prompts: each resource is "uniquely identified by a URI" and represents
data a server wants to share for context — "files, database schemas, or
application-specific information." Clients discover the catalog with
`resources/list` and fetch a specific item with `resources/read`, without
needing the model to guess at and invoke a discovery tool first. Claude Code
exposes this to users directly: a resource can be referenced inline with an
`@server:resource` mention (e.g. `@github:repos/owner/repo/issues`), which
"fetches data from connected MCP servers" for inclusion in context. Because
a resource is fetched by a known, stable URI rather than searched for, this
pattern avoids the exploratory back-and-forth of a tool call whose job is
merely to locate content the server could already address directly.

**Example:** Instead of asking Claude to call a `search_issues` tool and
hope it finds the right ticket, a user references the ticket directly: "Can
you analyze @github:repos/owner/repo/issues and suggest a fix?", letting
Claude read the resource's contents via `resources/read` immediately.

**Source:** Tool Design & MCP Integration research — MCP resources as an
addressable content catalog [exam task 2.4] (key), https://modelcontextprotocol.io/specification/2025-06-18/server/resources ; https://code.claude.com/docs/en/common-workflows

### Concept: Dynamic tool discovery and updates in MCP workflows
MCP clients discover a server's tools via a `tools/list` request
(paginated) and invoke them via `tools/call`. Servers that declare the
`tools` capability with `listChanged: true` can send a
`notifications/tools/list_changed` message when their available tools
change; Claude Code automatically refreshes the tool list from that server
on receiving this notification, without requiring the user to disconnect
and reconnect, and — with tool search enabled — can list a newly connected
server's tools to Claude mid-turn.

**Example:** An MCP server that gains a new "create_invoice" tool after a
feature flag flips sends `notifications/tools/list_changed`; Claude Code
calls `tools/list` again and the new tool becomes available in the same
session.

**Source:** Tool Design & MCP Integration research — Dynamic tool discovery
and updates in MCP workflows (key), https://modelcontextprotocol.io/specification/2025-06-18/server/tools ; https://code.claude.com/docs/en/mcp

---

## Lesson 3.5: Distributing Tools and Selecting Built-in Tools

### Concept: Claude Code permission system (settings.json and CLI tool flags)
Building on Module 2's settings-file precedence and permission-modes
overview, the permission system is the mechanism by which Claude Code
controls which tools a session (or subagent) may use: CLI flags
(`--allowedTools`, `--disallowedTools`, `--tools`) and `settings.json`
`permissions.allow`/`permissions.deny` rules, which can be scoped to
specific paths (`Read(src/**)`), commands (`Bash(npm run *)`), or domains
(`WebFetch(domain:github.com)`). This is the mechanism underlying both
"restricting built-in tools" and subagent-level tool restriction, below.

**Example:** `{"permissions": {"deny": ["WebSearch", "Bash(rm *)"]}}` in
`settings.json` prevents any session using that configuration from
performing web searches or running `rm` commands, regardless of what a
subagent's own `tools` field allows.

**Source:** Tool Design & MCP Integration research — Claude Code permission
system (prerequisite), https://code.claude.com/docs/en/tools-reference

### Concept: Distributing tools across (sub)agents
Building on Module 2's general subagent notion, Claude Code subagents
restrict tool access through a `tools` field (allowlist) and/or a
`disallowedTools` field (denylist) in the subagent's frontmatter; if both
are set, `disallowedTools` is applied first and then `tools` is resolved
against what remains. Both fields accept MCP server-level patterns
(`mcp__<server>` or `mcp__<server>__*`) to grant or remove an entire
server's tools at once. Regardless of configuration, a fixed set of
sensitive tools (e.g., `AskUserQuestion`, `EndConversation`,
`ExitPlanMode` unless in plan mode) is always removed from subagents, and
background subagents are further restricted to a smaller built-in tool set
than foreground subagents. This lets an architect grant a read-only
research subagent only `Read, Grep, Glob` while a coding subagent also gets
`Write, Edit, Bash`.

**Example:** A `code-reviewer` subagent configured with `tools: Read, Glob,
Grep` can analyze code but cannot modify files, while a `coordinator`
subagent configured with `tools: Agent(worker, researcher), Read, Bash` can
only spawn the `worker` and `researcher` subagent types.

**Source:** Tool Design & MCP Integration research — Distributing tools
across (sub)agents (key), https://code.claude.com/docs/en/sub-agents

### Concept: Restricting built-in tools via Claude Code permissions
Which of Claude Code's built-in tools (Read, Write, Edit, Bash, PowerShell,
Glob, Grep, LSP, WebFetch, WebSearch, Agent, Monitor, Artifact,
NotebookEdit, task-tracking tools, AskUserQuestion,
EnterWorktree/ExitWorktree, EnterPlanMode/ExitPlanMode, SendMessage, Skill,
Workflow, and more) are active in a given session can be restricted via CLI
flags (`--allowedTools`, `--disallowedTools`, `--tools`) or via
path/command-scoped `settings.json` rules. This is the access-control layer
that complements — but is distinct from — choosing the functionally right
tool for a task (next concept): it governs what a session or subagent is
*permitted* to invoke at all, regardless of which tool would otherwise be
the best functional fit.

**Example:** A read-only auditing session is started with `claude
--allowedTools Read,Grep,Bash` so the agent can inspect code but never call
`Write` or `Edit`; a `settings.json` deny rule of `"Bash(rm *)"` blocks
destructive shell deletions regardless of what else is allowed.

**Source:** Tool Design & MCP Integration research — Restricting built-in
tools via Claude Code permissions (key), https://code.claude.com/docs/en/tools-reference

### Concept: Functional selection among Claude Code's file and search tools
Claude Code's built-in tools reference documents distinct, non-overlapping
functional purposes for its core file/search tools, and selecting the right
one for a task is itself a tested skill. Glob "finds files by name pattern"
(path pattern matching), while Grep searches file contents — "Where Glob
finds files by name, Grep finds lines inside them." Read "reads the
contents of files" (full-file reading, with line numbers, pagination, and
support for images/PDFs/notebooks). Write "creates or overwrites files" (a
full-file operation). Edit "makes targeted edits to specific files" (a
narrower, surgical modification via exact string replacement), distinct
from Write's whole-file replacement.

**Example:** To find which files mention a deprecated config key, use Grep
(content search); to find every `*.test.ts` file in a repo, use Glob (path
pattern matching); to view a whole file before deciding what to change, use
Read; to replace one function's body in place, use Edit; to regenerate an
entire generated file from scratch, use Write.

**Source:** Tool Design & MCP Integration research — Functional selection
among Claude Code's file and search tools [exam task 2.5] (key), https://code.claude.com/docs/en/tools-reference

### Concept: Edit tool's uniqueness constraint and recovery from non-unique matches
The Edit tool requires its `old_string` target to match exactly one location
in the file. Per Claude Code's tools reference: "`old_string` must appear
exactly once. When it appears more than once, Claude either supplies a
longer string with enough surrounding context to pin down one occurrence, or
sets `replace_all: true` to replace them all." This is the officially
documented, vendor-sourced recovery path when a targeted Edit call fails on
a non-unique match.

**Example:** An Edit call with `old_string: "return x"` fails because that
string appears five times in the file; Claude either expands `old_string`
to include several surrounding lines that occur only once, or reissues the
call with `replace_all: true` if every occurrence should legitimately
change.

**Source:** Tool Design & MCP Integration research — Edit tool's uniqueness
constraint and recovery [exam task 2.5] (key), https://code.claude.com/docs/en/tools-reference

### Concept: Read + Write as a fallback when Edit fails due to non-unique text matches
> **Sourcing note:** this is a distinct fallback strategy from the
> vendor-documented Edit recovery mechanism above (more context, or
> `replace_all: true`), sourced only to the official CCAR-F Exam Guide (Task
> Statement 2.5). It was not independently found in Claude Code's tools
> reference or other vendor documentation. Treat this as an exam-guide fact
> distinct from — not confirmed by — the vendor-documented mechanism above.

The exam guide names, as a Task 2.5 skill, "using Read + Write as a fallback
for reliable file modifications" specifically "when Edit fails due to
non-unique text matches." As stated by the guide, when a targeted Edit call
cannot be made to match a file uniquely, the fallback approach is to read
the file and write it back with the correction, rather than continuing to
adjust the Edit call.

**Example:** An Edit call fails because its target string is not unique in
the file; per the guide's named skill, the fallback is to use Read to
obtain the file's contents and Write to save the corrected version, rather
than continuing to adjust the Edit call.

**Source:** Claude Certified Architect – Foundations Exam Guide, Task
Statement 2.5 (exam-guide-only; not found in vendor documentation).
