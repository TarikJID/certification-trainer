# Module 3 — Tool Design & MCP Integration

Domain: Domain 2 — Tool Design & MCP Integration (18% of exam)
Covers Task Statements 2.1, 2.2, 2.3, 2.4, 2.5.

**Prerequisites from Module 1:** the Messages API request/response cycle and
`tool_use` blocks (Lesson 1.1); hub-and-spoke coordinator/subagent architecture
and context isolation (Lesson 1.2); MCP `isError` basics (Lesson 1.5).

---

## Lesson 2.1 — Designing Effective Tool Interfaces

**Maps to:** Task Statement 2.1: Design effective tool interfaces with clear
descriptions and boundaries.

### Prerequisite concept: Tool definition (name, description, input_schema)

In the Claude API, a tool is a JSON object with a `name`, a `description` (a
detailed plaintext explanation of what the tool does, when to use it, and how it
behaves), and an `input_schema` (a JSON Schema defining expected parameters).
These are passed in the `tools` parameter of a Messages API request; the API
folds them into a constructed system prompt so the model sees, at every turn,
what tools exist and what each is for.

*Example:* `{"name": "get_weather", "description": "Get the current weather in a
given location...", "input_schema": {"type": "object", "properties":
{"location": {"type": "string"}}, "required": ["location"]}}`.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 2.1-K1, 2.1-K2, 2.1-S1, 2.3-K4 — prerequisite)

### Concept: Tool description as the primary tool-selection signal

LLMs choose which tool to call based on the tool's name, description, and input
schema shown in context — there is no other channel by which the model learns
what a tool does. A minimal description provides too little information to
reliably discriminate between similar tools. Anthropic's guidance: write
descriptions of at least 3-4 sentences explaining what the tool does, when it
should and should not be used, what each parameter means, and any caveats — the
same amount of context you'd give a new employee.

*Example:* Anthropic's docs contrast a poor description ("Gets the stock price
for a ticker.") with a good one that states the exchange scope, output units, and
what the tool does *not* provide.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 2.1-K1, 2.1-K2, 2.1-S1)

### Concept: Tool description ambiguity/overlap and the rename/split fix

When two tools have near-identical descriptions (e.g. `analyze_content` vs
`analyze_document`), the model cannot reliably tell which applies and misroutes.
Two repair techniques: (a) rename a tool and rewrite its description to be
domain-specific (namespacing — grouping related tools under common prefixes helps
delineate boundaries); (b) split an overly generic tool into several
purpose-specific tools, each with its own narrow input/output contract.

*Example:* `extract_data_points`, `summarize_content`, and
`verify_claim_against_source` — three narrow tools replacing one overloaded
`analyze_document` tool.

*Source:* exam-guide.txt; https://www.anthropic.com/engineering/writing-tools-for-agents
(Teaches: 2.1-K3, 2.1-S2, 2.1-S3)

### Concept: System prompt keyword sensitivity and tool selection

Tool selection is influenced not only by tool descriptions but by the wording of
the surrounding system prompt. An instruction elsewhere in the system prompt that
shares vocabulary with a tool's name/description can create an unintended
association, biasing the model regardless of actual fit. The fix: review system
prompts specifically for keyword overlaps with tool names/descriptions.

*Example:* A system prompt repeatedly emphasizing "always analyze the document
thoroughly" can bias the model toward `analyze_document` even when
`verify_claim_against_source` is the better fit, simply due to the shared
keyword "analyze."

*Source:* exam-guide.txt; general principle corroborated by
https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 2.1-K4, 2.1-S4)

**Quiz:** see `quiz.md`, Lesson 2.1.

---

## Lesson 2.2 — Implementing Structured Error Responses for MCP Tools

**Maps to:** Task Statement 2.2: Implement structured error responses for MCP
tools.

### Prerequisite concept: MCP client-server architecture and protocol messages

MCP defines a client-server architecture in which an MCP server exposes tools,
resources, and prompts over a JSON-RPC 2.0 connection; a client (such as Claude
Code) discovers capabilities via list requests (`tools/list`, `resources/list`)
and invokes them via call/read requests (`tools/call`, `resources/read`).

*Example:* On connecting to a Jira MCP server, Claude Code sends `tools/list` and
receives back tool definitions for `create_issue` and `search_issues`, invoked
via `tools/call`.

*Source:* https://modelcontextprotocol.io/specification/2025-06-18/server/tools
(Teaches: 2.2-K1, 2.4-K1, 2.4-K3, 2.4-K4 — prerequisite)

### Concept: MCP isError flag — recap and structured taxonomy

Recall from Module 2 (Lesson 1.5) that MCP tool results carry a boolean `isError`
field, letting a failure flow through the normal result channel rather than as a
protocol-level error. This lesson builds the structured error design on top of
that mechanism.

*Source:* https://modelcontextprotocol.io/specification/2025-06-18/server/tools
(Teaches: 2.2-K1)

### Concept: Structured error categorization taxonomy (transient / validation / business / permission)

Treating all tool failures identically prevents the agent from choosing an
appropriate recovery action. Four categories recur: transient errors (timeouts —
usually safe to retry), validation errors (invalid input — retrying the same
input won't help), business errors (well-formed request violating policy — no
retry fixes this), and permission errors (needs escalation, not a retry).
Uniform messages like "Operation failed" force the agent to guess. The fix:
always return structured metadata — `errorCategory`, `isRetryable`, and a
human-readable description.

*Example:* A `process_refund` tool distinguishes a payment-gateway timeout
(`errorCategory: "transient"`, `isRetryable: true`) from a refund exceeding the
original purchase amount (`errorCategory: "business"`, `isRetryable: false`).

*Source:* exam-guide.txt; https://modelcontextprotocol.io/specification/2025-06-18/server/tools
(Teaches: 2.2-K2, 2.2-K3, 2.2-S1)

### Concept: Retryable vs non-retryable errors and structured retry metadata

A tool response should explicitly tell the agent whether retrying could plausibly
succeed. Transient errors are typically retryable; validation/business/permission
errors typically are not. `isRetryable: false` (or `retriable: false`) prevents
hopeless retry loops; pairing it with a customer-friendly explanation for
business-rule violations lets the agent communicate appropriately.

*Example:* `{ errorCategory: "business", isRetryable: false, retriable: false,
description: "Refunds over $500 require manager approval" }` — the agent explains
the limit rather than retrying.

*Source:* exam-guide.txt; https://modelcontextprotocol.io/specification/2025-06-18/server/tools
(Teaches: 2.2-K4, 2.2-S2)

### Concept: Local error recovery before escalation (subagent pattern)

In a coordinator-subagent architecture (recall Lesson 1.2), a subagent hitting a
transient failure should attempt local resolution first — retry, fall back to an
alternate source — rather than immediately surfacing every failure to the
coordinator. Only errors it genuinely cannot resolve locally should propagate up,
along with what was attempted and any partial results.

*Example:* A web-search subagent whose primary API times out retries once with a
shorter query, then falls back to a secondary source; only if both fail does it
report up with details of both attempts.

*Source:* exam-guide.txt; consistent with
https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 2.2-S3)

### Concept: Distinguishing access failures from valid empty results

A "no results" response is not necessarily a failure — it can be the correct
outcome of a well-formed query that legitimately matched nothing. This must be
reported differently from an access failure (timeout, unreachable backend),
because the two require opposite agent responses.

*Example:* `lookup_order("12345")` returning `{status: "success", results: []}`
means the search worked and no such order exists; a timeout returning
`{isError: true, errorCategory: "transient"}` means the search couldn't complete
at all.

*Source:* exam-guide.txt (recurs in Domain 5, Task 5.3)
(Teaches: 2.2-S4)

**Quiz:** see `quiz.md`, Lesson 2.2.

---

## Lesson 2.3 — Distributing Tools Across Agents and Configuring Tool Choice

**Maps to:** Task Statement 2.3: Distribute tools appropriately across agents and
configure tool choice.

### Prerequisite concept: Coordinator-subagent multi-agent architecture — recap

Recall from Lesson 1.2: in a hub-and-spoke system, a coordinator decomposes,
delegates, and aggregates, while subagents run with isolated context. This
lesson's tool-distribution decisions build directly on that division of labor.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 2.2-S3, 2.3-K2, 2.3-K3 — prerequisite)

### Concept: Tool-set size and selection reliability

Every tool available to an agent adds to the decision the model must make at each
turn. As the number of available tools grows, especially when several are
similar, the chance of choosing incorrectly rises — a direct cost of decision
complexity. Giving an agent a small, focused set (illustratively 4-5 rather than
18) measurably improves selection reliability.

*Example:* A synthesis subagent given all 18 of a system's tools selects
incorrectly more often than one scoped to the 4-5 tools its role actually needs.

*Source:* exam-guide.txt; consistent with
https://www.anthropic.com/engineering/writing-tools-for-agents
(Teaches: 2.3-K1)

### Concept: Tool misuse outside an agent's specialization

When an agent has access to a tool outside its specialization, it will sometimes
attempt to use it anyway, because the model reasons locally about what might help
with the current subtask, not about the system's overall division of labor. The
design response: restrict each subagent's tool set to only the tools relevant to
its role.

*Example:* A synthesis agent responsible only for combining findings is given the
tool set `{format_report, cite_source}` — with no `web_search` — so it cannot
attempt its own searches.

*Source:* exam-guide.txt; consistent with
https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 2.3-K2, 2.3-S1)

### Concept: Scoped tool access with limited cross-role tools

Keep tool sets scoped to the role by default, but deliberately add a small number
of narrow, constrained cross-role tools for specific high-frequency needs, while
routing anything more complex through the coordinator. A related tactic: replace
an overly generic tool with a narrower, purpose-built one that enforces its own
constraints.

*Example:* A synthesis agent gets a scoped `verify_fact` tool for spot-checking
claims but not a general `web_search`. Separately, `fetch_url` is replaced with
`load_document`, which rejects non-document URLs outright.

*Source:* exam-guide.txt; https://www.anthropic.com/engineering/writing-tools-for-agents
(Teaches: 2.3-K3, 2.3-S2, 2.3-S3)

### Concept: tool_choice configuration (auto / any / tool / none)

`tool_choice` controls whether and how Claude must use tools. `auto` (default)
lets Claude decide whether to call a tool or respond in text. `any` forces a tool
call but leaves the choice of which tool up to the model. `tool` (`{"type":
"tool", "name": "..."}`) forces a specific named tool. `none` prevents any tool
call. Forcing a tool causes the API to prefill the assistant turn, so Claude
won't emit explanatory text before the forced `tool_use` block.

*Example:* A pipeline forces `extract_metadata` first, then switches to `any` on
the next turn so Claude must call one of several enrichment tools next.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use
(Teaches: 2.3-K4, 2.3-S4, 2.3-S5)

**Quiz:** see `quiz.md`, Lesson 2.3.

---

## Lesson 2.4 — Integrating MCP Servers into Claude Code and Agent Workflows

**Maps to:** Task Statement 2.4: Integrate MCP servers into Claude Code and agent
workflows.

### Concept: MCP server scoping (project vs user vs local)

Project scope stores the server definition in `.mcp.json` at the project root;
because it's checked into version control, every team member gets the same
shared configuration — right for team-wide tooling. User scope stores it in
`~/.claude.json`, available across that developer's projects but private to
them — appropriate for personal/experimental servers. When defined at more than
one scope, Claude Code connects using the highest-precedence definition rather
than merging fields.

*Example:* A team adds a shared Jira MCP server to `.mcp.json` and commits it, while
one developer separately registers an experimental server in their own
`~/.claude.json`.

*Source:* https://code.claude.com/docs/en/mcp; exam-guide.txt
(Teaches: 2.4-K1, 2.4-S1, 2.4-S2)

### Concept: Environment variable expansion in .mcp.json

Because project-scoped `.mcp.json` files are committed to version control, they
cannot safely contain literal secrets. Claude Code supports `${VAR}` (expands to
the environment variable) and `${VAR:-default}` (falls back to a default) inside
`command`, `args`, `env`, `url`, and `headers` fields. This lets a team commit one
shared config while each developer supplies their own credential locally.

*Example:* `"headers": {"Authorization": "Bearer ${GITHUB_TOKEN}"}` lets every
team member authenticate with their own token without it appearing in the repo.

*Source:* https://code.claude.com/docs/en/mcp; exam-guide.txt
(Teaches: 2.4-K2)

### Concept: Tools from connected MCP servers available together to the agent

Claude Code connects to every MCP server configured across applicable scopes; once
connected, tools from all connected servers combine into a single set the agent
can choose from — not gated behind an explicit per-server selection step. If a
server is still connecting when a request needs its tools, Claude Code waits for
that connection rather than proceeding with only a partial tool set.

*Example:* A developer with both a Jira server (project scope) and a personal
database server (user scope) sees tools from both — `mcp__jira__create_issue`
and `mcp__postgres__query` — available together once both have connected.

*Source:* exam-guide.txt; https://code.claude.com/docs/en/mcp;
https://modelcontextprotocol.io/specification/2025-06-18/server/tools
(Teaches: 2.4-K3)

### Concept: MCP resources for content catalogs

Beyond tools, MCP servers can expose resources — read-only, URI-identified
content such as files, database schemas, or application-specific data — that a
client can list and read directly, without the agent needing to guess at a
sequence of exploratory tool calls. Valuable when the agent benefits from seeing
catalog-like data up front.

*Example:* A database MCP server exposes its table schemas as resources (seen up
front) while exposing `query` and `insert_record` as tools — letting the agent
plan a correct query without first calling "list tables" then "describe table."

*Source:* https://modelcontextprotocol.io/specification/2025-06-18/server/resources;
exam-guide.txt
(Teaches: 2.4-K4, 2.4-S5)

### Concept: Tool description quality vs built-in tool preference

When an MCP tool's description is thin compared to the rich descriptions of
Claude Code's built-in tools (like Grep), the agent will tend to default to the
built-in tool even when the MCP tool is more capable. The fix: enhance the MCP
tool's description to explicitly explain its capabilities and outputs in enough
detail that the agent recognizes when it's the stronger choice.

*Example:* A generic `search_code` MCP tool described only as "search the
codebase" loses out to built-in Grep even when it performs semantic, symbol-aware
search Grep's plain-text matching would miss; rewriting the description corrects
selection.

*Source:* exam-guide.txt; consistent with
https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 2.4-S3)

### Concept: Community MCP servers vs custom implementations

For standard, widely-used integrations, an existing community/vendor-maintained
MCP server typically already implements the protocol correctly, so building a
custom server for the same integration duplicates effort. Custom servers are
better reserved for genuinely team-specific workflows — internal systems,
proprietary APIs — where the value of a purpose-built integration outweighs the
cost of owning a server.

*Example:* A team adopts the existing community Jira MCP server rather than
writing their own, but builds a custom server for their internal, proprietary
order-management system.

*Source:* exam-guide.txt; https://github.com/modelcontextprotocol/servers
(Teaches: 2.4-S4)

**Quiz:** see `quiz.md`, Lesson 2.4.

---

## Lesson 2.5 — Selecting and Applying Built-in Tools (Read, Write, Edit, Bash, Grep, Glob)

**Maps to:** Task Statement 2.5: Select and apply built-in tools effectively.

### Prerequisite concept: Claude Code as an agentic coding environment with built-in tools

Claude Code is an agentic command-line/IDE tool that can read files, run
commands, and make changes autonomously, backed by a fixed set of built-in tools
(Read, Write, Edit, Bash, Grep, Glob) alongside any configured MCP tools. Because
context window space is limited and performance degrades as it fills, which
built-in tool is used for a given step has a direct effect on context budget and
reliability.

*Example:* A developer exploring an unfamiliar repository benefits from distinct,
purpose-specific tools (Grep for content, Glob for filenames) rather than one
do-everything "search" tool.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 2.5-K1, 2.5-K2, 2.5-K3 — prerequisite)

### Concept: Grep for content search

Grep searches file contents for a text/regex pattern across a codebase (built on
ripgrep; respects `.gitignore` by default). It's the right tool for "find
text/patterns inside files" — function names, error messages, import statements —
as opposed to finding files by name.

*Example:* Finding every caller of `processPayment()` or every occurrence of a
specific error string — Glob cannot search inside file contents.

*Source:* https://code.claude.com/docs/en/tools-reference; exam-guide.txt
(Teaches: 2.5-K1, 2.5-S1)

### Concept: Glob for file path pattern matching

Glob finds files by matching their path/name against a glob pattern (e.g.
`**/*.test.tsx`) — it operates purely on paths/names, not contents. Right tool
for "find files matching this naming/extension pattern." Results are sorted by
modification time and capped at a fixed number of matches.

*Example:* Finding every React test file uses Glob with `**/*.test.tsx`,
returning matching paths without inspecting contents.

*Source:* https://code.claude.com/docs/en/tools-reference; exam-guide.txt
(Teaches: 2.5-K2, 2.5-S2)

### Concept: Read/Write for full file operations vs Edit for targeted modification

Read retrieves full file contents (with line numbers) given an absolute path,
typically a prerequisite before editing. Write creates or completely overwrites a
file — appropriate for new files or broad changes. Edit makes a targeted
modification requiring an exact `old_string` that must match uniquely; it then
replaces that match with `new_string`. Edit is preferred for small, precise
modifications because it verifies the anchor text first.

*Example:* Adding one new function is a job for Edit; generating a brand-new
config file from scratch is a job for Write; inspecting a file first is a job for
Read.

*Source:* https://code.claude.com/docs/en/tools-reference; exam-guide.txt
(Teaches: 2.5-K3)

### Concept: Edit failure fallback to Read + Write

Edit's uniqueness requirement means it can fail when the target text occurs
multiple times. The reliable fallback: Read the entire file for full context,
then Write back a reconstructed version with the intended change — sidestepping
the uniqueness constraint entirely since Write replaces the whole file.

*Example:* A config file has `timeout: 30` in five unrelated sections; Edit fails
on non-uniqueness, so the agent reads the full file, decides which occurrence to
change, and writes back the complete file with only that instance modified.

*Source:* https://code.claude.com/docs/en/tools-reference; exam-guide.txt
(Teaches: 2.5-K4, 2.5-S3)

### Concept: Incremental codebase exploration strategy (Grep-then-Read)

Rather than reading every file upfront, start narrow and expand only as needed:
Grep for entry points directly relevant to the question, then Read just the
specific files Grep surfaces to follow imports and trace flow, repeating this
narrow-search-then-targeted-read cycle as understanding builds.

*Example:* To understand session refresh, first Grep for "refreshToken" or
"session," then Read only the files that mention it, rather than the entire
`src/` tree.

*Source:* exam-guide.txt; consistent with https://code.claude.com/docs/en/best-practices
(Teaches: 2.5-S4)

### Concept: Tracing function usage across wrapper modules

When a function is re-exported or wrapped by intermediate modules, a single Grep
for the original name will miss call sites referencing it under a re-exported or
aliased name. The reliable technique: first identify all the names a piece of
functionality is exported/re-exported under, then Grep for each of those names in
turn.

*Example:* `validateUser` (defined in `auth/core.ts`) is re-exported as
`checkAuth` and re-wrapped as `requireAuth`; finding all usages requires Grep-ing
all three names.

*Source:* exam-guide.txt; consistent with https://code.claude.com/docs/en/tools-reference
(Teaches: 2.5-S5)

**Quiz:** see `quiz.md`, Lesson 2.5.

---

## Module 3 summary of bullets taught

Lesson 2.1: 2.1-K1, 2.1-K2, 2.1-K3, 2.1-K4, 2.1-S1, 2.1-S2, 2.1-S3, 2.1-S4
Lesson 2.2: 2.2-K1, 2.2-K2, 2.2-K3, 2.2-K4, 2.2-S1, 2.2-S2, 2.2-S3, 2.2-S4
Lesson 2.3: 2.3-K1, 2.3-K2, 2.3-K3, 2.3-K4, 2.3-S1, 2.3-S2, 2.3-S3, 2.3-S4, 2.3-S5
Lesson 2.4: 2.4-K1, 2.4-K2, 2.4-K3, 2.4-K4, 2.4-S1, 2.4-S2, 2.4-S3, 2.4-S4, 2.4-S5
Lesson 2.5: 2.5-K1, 2.5-K2, 2.5-K3, 2.5-K4, 2.5-S1, 2.5-S2, 2.5-S3, 2.5-S4, 2.5-S5
