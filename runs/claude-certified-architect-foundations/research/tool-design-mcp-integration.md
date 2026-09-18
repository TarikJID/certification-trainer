# Domain 2 Research: Tool Design & MCP Integration

Bullets received: 43 (20 Knowledge, 23 Skills)
Bullets covered: 43
Concepts: 29 (24 key, 5 prerequisite)

---

## Key Concepts

- Concept: Tool description as the primary tool-selection signal
  Type: key
  Teaches: 2.1-K1, 2.1-K2, 2.1-S1
  Definition: LLMs choose which tool to call based on the tool's name, description, and input schema shown to it in context — there is no other channel by which the model learns what a tool does. A minimal description that only states the tool's name and a one-line summary of what it does provides too little information for the model to reliably discriminate between similar tools, especially when several tools address overlapping tasks. Anthropic's own guidance is to write descriptions of at least 3–4 sentences that explain what the tool does, when it should and should not be used, what each parameter means, and any caveats/limitations — the same amount of context you would give a new employee. Including concrete example queries, expected input formats, and edge cases in the description (or via the optional `input_examples` field) further reduces the ambiguity a model must resolve at call time.
  Example: Anthropic's docs contrast a poor description ("Gets the stock price for a ticker.") with a good one that states the exchange scope, output units, and what the tool does *not* provide — the good version leaves far fewer open questions for the model.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (Best practices for tool definitions); exam guide 2.1-K1, 2.1-K2.

- Concept: Tool description ambiguity/overlap and the rename/split fix
  Type: key
  Teaches: 2.1-K3, 2.1-S2, 2.1-S3
  Definition: When two tools have near-identical descriptions (e.g., `analyze_content` vs `analyze_document`), the model cannot reliably tell which one applies to a given request and misroutes — calling the wrong tool or alternating unpredictably between them. Anthropic's own tool-design guidance names this exact failure mode directly: "When tools overlap in function or have a vague purpose, agents can get confused about which ones to use." Two concrete repair techniques address it: (a) rename a tool and rewrite its description to be domain-specific instead of generic (e.g., `analyze_content` → `extract_web_results`, scoped explicitly to web content) so its name/description no longer overlaps with a neighboring tool — Anthropic describes this general approach as namespacing, i.e. "grouping related tools under common prefixes," which "can help delineate boundaries between lots of tools"; and (b) split an overly generic tool (`analyze_document`) into several purpose-specific tools (`extract_data_points`, `summarize_content`, `verify_claim_against_source`), each with its own narrow, well-defined input/output contract, instead of cramming multiple use cases behind one ambiguous interface.
  Example: `extract_data_points` takes a document and a field list and returns typed values; `summarize_content` takes a document and a target length and returns prose; `verify_claim_against_source` takes a claim and a document and returns a boolean plus a supporting excerpt — three narrow tools replacing one overloaded `analyze_document` tool.
  Source: exam guide 2.1-K3, 2.1-S2, 2.1-S3 (task statement's own wording, including the analyze_content/analyze_document and split-tool examples); https://www.anthropic.com/engineering/writing-tools-for-agents ("Namespacing your tools" and "Choosing the right tools for agents" sections, quoted above).

- Concept: System prompt keyword sensitivity and tool selection
  Type: key
  Teaches: 2.1-K4, 2.1-S4
  Definition: Tool selection is influenced not only by the tool descriptions themselves but by the wording of the surrounding system prompt. Because the model reasons over the whole context at once, an instruction elsewhere in the system prompt that happens to share vocabulary with a tool's name or description can create an unintended association, biasing the model toward (or away from) a tool regardless of whether it is actually the best fit for the request — undermining even a well-written tool description. The fix is to review system prompts specifically for keyword overlaps with tool names/descriptions and adjust the wording so instructions do not accidentally steer tool choice.
  Example: A system prompt that repeatedly emphasizes "always analyze the document thoroughly" can bias the model toward calling a tool literally named `analyze_document` even when a more specific tool (`verify_claim_against_source`) is the better fit, simply because the instruction and the tool name share the keyword "analyze."
  Source: exam guide 2.1-K4, 2.1-S4 (task statement's own wording — the exam guide is the only source describing this exact system-prompt/tool-name interaction; the general principle that tool descriptions steer selection is corroborated by https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools).

- Concept: MCP isError flag
  Type: key
  Teaches: 2.2-K1
  Definition: MCP tool results are JSON-RPC responses containing a `content` array plus an `isError` boolean field. When a tool's execution fails (an API call failed, invalid business state, etc.), the server returns `isError: true` along with `content` describing the failure, rather than raising a protocol-level JSON-RPC error. This lets the failure flow through the normal result channel so the calling agent (the LLM) can see the error content and decide how to react — retry, ask the user, try a different tool. This is distinct from protocol-level JSON-RPC errors (e.g., code `-32602` invalid params, `-32601` method not found), which are reserved for infrastructure-level problems like an unknown tool name, not business-logic failures inside an otherwise-working tool call.
  Example: `{ "result": { "content": [{"type":"text","text":"Failed to fetch weather data: API rate limit exceeded"}], "isError": true } }` tells the agent the tool ran but failed, versus a JSON-RPC `error` object, which would mean the call itself was malformed.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools (Error Handling section).

- Concept: Structured error categorization taxonomy (transient / validation / business / permission)
  Type: key
  Teaches: 2.2-K2, 2.2-K3, 2.2-S1
  Definition: Not all tool failures are the same, and treating them identically prevents the agent from choosing an appropriate recovery action. Four practically distinct categories recur in production tool design: transient errors (timeouts, temporary service unavailability — usually safe to retry), validation errors (the caller supplied invalid input — retrying the same input will not help, but correcting it might), business errors (the request is well-formed but violates a policy or business rule — no retry fixes this; it needs different handling such as informing the user), and permission errors (the caller lacks authorization — needs escalation or different credentials, not a retry). Returning a uniform generic message like "Operation failed" collapses these categories, forcing the agent to guess whether it should retry, apologize, or escalate. The fix is to always return structured error metadata — an `errorCategory` field (e.g., "transient" | "validation" | "business" | "permission"), an `isRetryable` boolean, and a human-readable description — so the agent can make an evidence-based recovery decision instead of guessing.
  Example: A `process_refund` tool distinguishes a payment-gateway timeout (`errorCategory: "transient"`, `isRetryable: true`) from a refund request that exceeds the customer's original purchase amount (`errorCategory: "business"`, `isRetryable: false`, description: "Refund amount exceeds original purchase").
  Source: exam guide 2.2-K2, 2.2-K3, 2.2-S1 (task statement's own wording); general result/error structure per https://modelcontextprotocol.io/specification/2025-06-18/server/tools.

- Concept: Retryable vs non-retryable errors and structured retry metadata
  Type: key
  Teaches: 2.2-K4, 2.2-S2
  Definition: Beyond categorizing an error's type, a tool response should explicitly tell the agent whether retrying the same call could plausibly succeed. Transient errors are typically retryable; validation, business, and permission errors typically are not, because the failure is a property of the request or state rather than a temporary condition — retrying an unchanged, invalid request just wastes a turn. Returning explicit metadata such as `isRetryable: false` (or `retriable: false`) prevents the agent from looping on hopeless retries. For business-rule violations specifically, pairing `retriable: false` with a customer-friendly, non-technical explanation lets the agent communicate the outcome to the end user appropriately instead of surfacing an opaque failure or blindly retrying.
  Example: A `process_refund` tool that rejects a refund above the policy threshold returns `{ errorCategory: "business", isRetryable: false, retriable: false, description: "Refunds over $500 require manager approval" }`, so the agent explains the limit to the customer rather than retrying.
  Source: exam guide 2.2-K4, 2.2-S2 (task statement's own wording); MCP `isError`/content pattern at https://modelcontextprotocol.io/specification/2025-06-18/server/tools.

- Concept: Local error recovery before escalation (subagent pattern)
  Type: key
  Teaches: 2.2-S3
  Definition: In a coordinator–subagent architecture, a subagent that hits a transient failure (e.g., one search source times out) should attempt to resolve it locally first — retry, fall back to an alternate source, or otherwise recover on its own — rather than immediately surfacing every failure to the coordinator. Only errors the subagent genuinely cannot resolve locally should be propagated upward, and when they are, the subagent should include what was attempted and any partial results obtained so far, not just a bare failure signal. This keeps the coordinator's attention focused on decisions that actually require it, with enough context to decide whether to proceed with partial coverage, re-delegate, or escalate further.
  Example: A web-search subagent whose primary search API times out retries once with a shorter query, then falls back to a secondary source; only if both fail does it report to the coordinator: "search source A timed out twice, source B returned no results for 'Q3 revenue', partial results: [X]".
  Source: exam guide 2.2-S3 (task statement's own wording); consistent with the graceful-adaptation/checkpoint-recovery philosophy in https://www.anthropic.com/engineering/built-multi-agent-research-system.

- Concept: Distinguishing access failures from valid empty results
  Type: key
  Teaches: 2.2-S4
  Definition: A tool call that returns "no results" is not necessarily a failure — it can be the correct, successful outcome of a well-formed query that legitimately matched nothing (e.g., a customer search that correctly finds no account for a given email). This must be reported differently from an access failure such as a timeout or an unreachable backend, because the two require opposite agent responses: an access failure may warrant a retry or an escalation decision, while a valid empty result should be treated as a completed, trustworthy answer and acted on accordingly. Collapsing both cases into the same "no data" response forces the agent to guess which situation it is in, risking either needless retries or false reassurance.
  Example: `lookup_order("12345")` returning `{ status: "success", results: [] }` means the search worked and no such order exists; `lookup_order` timing out and returning `{ isError: true, errorCategory: "transient" }` means the search could not be completed at all — the agent should treat these very differently.
  Source: exam guide 2.2-S4 (task statement's own wording; the same distinction recurs verbatim in the exam guide's Domain 5, Task Statement 5.3).

- Concept: Tool-set size and selection reliability
  Type: key
  Teaches: 2.3-K1
  Definition: Every tool available to an agent adds to the decision the model must make at each turn: which of these N tools (if any) best matches the current need. As the number of available tools grows, especially when several are similar or only occasionally relevant, the chance of the model choosing incorrectly rises — a direct cost of decision complexity, not a bug in any individual tool description. The corollary is that giving an agent a small, focused set of tools appropriate to its role (illustratively, 4–5 rather than 18) measurably improves selection reliability, because there is simply less to discriminate between. This is one reason multi-agent architectures assign different, narrower toolsets to each specialized subagent rather than giving every subagent the full tool inventory.
  Example: A synthesis subagent given all 18 of a system's tools (including several near-duplicate search and lookup tools meant for other roles) selects incorrectly more often than one scoped down to the 4–5 tools its role actually needs (e.g., `read_document`, `cite_source`, `format_report`, `verify_fact`).
  Source: exam guide 2.3-K1 (task statement's own wording); consistent with Anthropic's guidance that "more tools don't always lead to better outcomes" and that too many/overlapping tools "distract agents from pursuing efficient strategies," https://www.anthropic.com/engineering/writing-tools-for-agents.

- Concept: Tool misuse outside an agent's specialization
  Type: key
  Teaches: 2.3-K2, 2.3-S1
  Definition: When an agent has access to a tool outside the task it is specialized for, it will sometimes attempt to use that tool anyway, even when a different agent is better positioned to handle that kind of request — because the model reasons locally about what might help with the current subtask, not about the system's overall division of labor. This produces wasted effort, duplicated work, and inconsistent results (e.g., a synthesis agent that should only combine already-gathered findings instead fires off its own new web searches). The design response is to restrict each subagent's tool set to only the tools relevant to its role, removing the opportunity for cross-specialization misuse at the source rather than relying on prompting alone to prevent it.
  Example: A synthesis agent responsible only for combining findings into a report is given the tool set `{format_report, cite_source}` — with no `web_search` tool — so it cannot attempt its own searches even if a gap in the findings tempts it to.
  Source: exam guide 2.3-K2, 2.3-S1 (task statement's own wording); consistent with the specialized-subagent scoping described in https://www.anthropic.com/engineering/built-multi-agent-research-system ("an objective, an output format, guidance on the tools and sources to use, and clear task boundaries").

- Concept: Scoped tool access with limited cross-role tools
  Type: key
  Teaches: 2.3-K3, 2.3-S2, 2.3-S3
  Definition: The general design principle is to give each agent only the tools its role needs, but real workflows sometimes have high-frequency cross-role needs that would be wasteful to route through a full escalation every time. The refinement: keep the tool set scoped to the role by default, but deliberately add a small number of narrow, constrained cross-role tools for those specific high-frequency needs, while routing anything more complex through the coordinator rather than expanding the agent's general-purpose tool access. A related tactic is replacing an overly generic, broadly-scoped tool with a narrower, purpose-built one that enforces its own constraints — e.g., replacing a generic `fetch_url` (which can retrieve anything) with `load_document` (which validates that the URL actually points to a document before fetching).
  Example: A synthesis agent is given a scoped `verify_fact` tool (checks a single claim against a source) for its high-frequency need to spot-check claims, but does not get a general `web_search` tool; anything beyond a simple fact check routes back through the coordinator. Separately, a research agent's `fetch_url` tool is replaced with `load_document`, which rejects non-document URLs outright.
  Source: exam guide 2.3-K3, 2.3-S2, 2.3-S3 (task statement's own wording); consistent with scoped-tool guidance in https://www.anthropic.com/engineering/writing-tools-for-agents and subagent task-boundary guidance in https://www.anthropic.com/engineering/built-multi-agent-research-system.

- Concept: tool_choice configuration (auto / any / tool / none)
  Type: key
  Teaches: 2.3-K4, 2.3-S4, 2.3-S5
  Definition: The Claude API's `tool_choice` parameter controls whether and how Claude must use the tools provided in a request. `auto` (the default when tools are supplied) lets Claude decide whether to call a tool at all or respond in text. `any` forces Claude to call one of the provided tools, but leaves the choice of which tool up to the model — useful to guarantee a tool call happens without dictating which tool. `tool` (specified as `{"type": "tool", "name": "..."}`) forces a single, specific named tool to be called, useful for ensuring a particular step (e.g., an extraction or metadata step) always runs first in a pipeline; subsequent steps are then driven in follow-up turns with `tool_choice` reset. `none` prevents any tool call. Forcing a tool (`any` or `tool`) causes the API to prefill the assistant turn, so Claude will not emit explanatory text before the forced `tool_use` block.
  Example: A pipeline forces `extract_metadata` first with `tool_choice: {"type": "tool", "name": "extract_metadata"}`, then on the next turn switches to `tool_choice: "any"` so Claude must call one of several enrichment tools next, guaranteeing structured tool use throughout instead of ever falling back to a plain conversational reply.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use (Controlling Claude's output / Forcing tool use).

- Concept: MCP server scoping (project vs user vs local)
  Type: key
  Teaches: 2.4-K1, 2.4-S1, 2.4-S2
  Definition: Claude Code lets MCP servers be registered at different scopes that control who sees them and how they are shared. Project scope stores the server definition in a `.mcp.json` file at the project root; because that file is checked into version control, every team member who clones the repo automatically gets the same shared server configuration — the right choice for team-wide tooling everyone needs (e.g., a shared issue tracker integration). User scope stores the server definition in `~/.claude.json` outside any single project and is available across all of that developer's projects but is private to them — appropriate for personal or experimental servers a developer is trying without imposing them on teammates. When a server is defined at more than one scope, Claude Code connects using the highest-precedence definition rather than merging fields across scopes.
  Example: A team adds a shared Jira MCP server to `.mcp.json` and commits it so every engineer gets it automatically, while one developer separately registers an experimental, internal-only MCP server in their own `~/.claude.json` that nobody else's session will see.
  Source: https://code.claude.com/docs/en/mcp (Configuration Scopes; Scope Hierarchy and Precedence); exam guide 2.4-K1, 2.4-S1, 2.4-S2.

- Concept: Environment variable expansion in .mcp.json
  Type: key
  Teaches: 2.4-K2
  Definition: Because project-scoped `.mcp.json` files are committed to version control, they cannot safely contain literal secrets (API tokens, credentials). Claude Code supports variable-substitution syntax — `${VAR}` (expands to the environment variable `VAR`) and `${VAR:-default}` (falls back to a default if `VAR` is unset) — inside the `command`, `args`, `env`, `url`, and `headers` fields of an MCP server definition. This lets a team commit one shared config file whose structure everyone uses, while each developer supplies their own credential value locally through their shell environment rather than through the file itself. If a referenced variable is unset and has no default, Claude Code still loads the config but reports a warning and uses the unexpanded `${VAR}` text literally.
  Example: `"headers": {"Authorization": "Bearer ${GITHUB_TOKEN}"}` in a committed `.mcp.json` lets every team member authenticate with their own personal GitHub token (set locally as the `GITHUB_TOKEN` environment variable) without that token ever appearing in the repository.
  Source: https://code.claude.com/docs/en/mcp (Environment Variable Expansion Syntax); exam guide 2.4-K2.

- Concept: Tools from connected MCP servers available together to the agent
  Type: key
  Teaches: 2.4-K3
  Definition: Claude Code connects to every MCP server configured across the applicable scopes and lists each connected server's tools; once connected, tools from all connected servers are combined into a single set the agent can choose from in a given turn — the agent is not restricted to one server's tools at a time and does not need to explicitly switch between servers. Connection itself is not necessarily synchronous at startup: Claude Code's own docs describe servers that can connect lazily (a cached remote server can show a status like "connects on first use," and Claude Code "connects the server the first time Claude calls one of the server's tools") or that can still be connecting in the background when a request comes in — in which case "Claude waits for that server before continuing" (via the tool-search mechanism, or a `WaitForMcpServers` tool) rather than proceeding with only a partial tool set. The practical effect the exam guide's bullet describes — that tools from all configured servers are available simultaneously to the agent — holds from the agent's point of view: by the time Claude acts on a turn that needs a given server's tools, that server's tools are present alongside every other connected server's tools rather than gated behind an explicit per-server selection step.
  Example: A developer with both a Jira MCP server (project scope) and a personal database MCP server (user scope) configured sees tools from both — e.g., `mcp__jira__create_issue` and `mcp__postgres__query` — available together once both have connected, without activating one server over the other; if the database server is still connecting when Claude's first request needs one of its tools, Claude Code waits for that connection before proceeding rather than acting without it.
  Source: exam guide 2.4-K3 (task statement's own wording); https://code.claude.com/docs/en/mcp ("Tool availability" under "Scale with MCP tool search," and "Server status detail," describing connection timing and wait behavior); MCP `tools/list` protocol behavior at https://modelcontextprotocol.io/specification/2025-06-18/server/tools.

- Concept: MCP resources for content catalogs
  Type: key
  Teaches: 2.4-K4, 2.4-S5
  Definition: In addition to tools (invokable actions), MCP servers can expose resources — read-only, URI-identified content such as files, database schemas, or application-specific data — that a client can list (`resources/list`) and read (`resources/read`) directly, without the agent needing to guess at and issue a sequence of exploratory tool calls to discover what is available. Exposing catalog-like data as a resource rather than requiring it to be fetched via tool calls is valuable when the agent benefits from seeing it up front — e.g., a list of open issue summaries, a documentation hierarchy, or a set of database table schemas — reducing wasted round trips spent probing.
  Example: A database MCP server exposes its table schemas as MCP resources (so the agent sees the full schema catalog up front) while exposing `query` and `insert_record` as tools (the actions performed against that schema) — letting the agent plan a correct query without first calling a "list tables" tool and then a "describe table" tool for each candidate.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/resources (Resources overview; Data Types); exam guide 2.4-K4, 2.4-S5.

- Concept: Tool description quality vs built-in tool preference
  Type: key
  Teaches: 2.4-S3
  Definition: When an MCP tool's description is thin or generic compared to the rich, well-documented descriptions of Claude Code's built-in tools (like Grep), the agent will tend to default to the built-in tool even when the MCP tool is actually more capable for the task. Because tool descriptions are the model's only window into what a tool does, an MCP tool that under-explains its capabilities and outputs effectively loses the "competition" for selection against a familiar, well-documented built-in alternative, even if it would produce a better result. The fix follows the same principle as general tool-description quality: enhance the MCP tool's description to explicitly explain its capabilities and what it returns in enough detail that the agent can recognize when it is the stronger choice.
  Example: A generic `search_code` MCP tool described only as "search the codebase" loses out to the built-in Grep in an agent's tool selection even when `search_code` performs semantic, symbol-aware search that would find matches Grep's plain-text pattern matching misses; rewriting its description to say "performs semantic, type-aware symbol search across the codebase, finding usages Grep's plain-text search cannot" corrects the selection.
  Source: exam guide 2.4-S3 (task statement's own wording); consistent with the tool-description-quality principle in https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools and https://www.anthropic.com/engineering/writing-tools-for-agents.

- Concept: Community MCP servers vs custom implementations
  Type: key
  Teaches: 2.4-S4
  Definition: For standard, widely-used integrations (e.g., Jira, Slack, GitHub, a common database), an existing, actively-maintained community or vendor-provided MCP server typically already implements the protocol correctly and covers common use cases, so building and maintaining a custom server for the same integration duplicates effort without a corresponding benefit. Custom MCP servers are better reserved for genuinely team-specific workflows — internal systems, proprietary APIs, or business logic no community server could reasonably cover — where the value of a purpose-built integration (e.g., enforcing your own rate limits, caching, or internal auth) outweighs the cost of owning a server. The Model Context Protocol project itself maintains a public repository distinguishing reference/official integrations from community-contributed servers, reflecting this build-vs-reuse tradeoff at the ecosystem level.
  Example: A team adopts the existing community-maintained Jira MCP server rather than writing their own Jira integration, but builds a custom MCP server for their internal, proprietary order-management system because no community server exists for it.
  Source: exam guide 2.4-S4 (task statement's own wording); https://github.com/modelcontextprotocol/servers (official MCP servers repository distinguishing reference implementations, official integrations, and community servers).

- Concept: Grep for content search
  Type: key
  Teaches: 2.5-K1, 2.5-S1
  Definition: Grep is Claude Code's built-in tool for searching the contents of files for a text or regex pattern across a codebase (built on ripgrep, so it uses regex syntax rather than POSIX grep semantics, and respects `.gitignore` by default). It is the right tool whenever the task is "find text/patterns inside files" — function names, error message strings, import statements, or any other content-level pattern — as opposed to finding files by name. It supports multiple output modes (file paths only, matching lines with line numbers, or match counts) and can be scoped by glob pattern or language type.
  Example: To find every caller of a function `processPayment()` across a codebase, or to locate every place a specific error message string appears, Grep is the correct tool — Glob (which only matches file names) cannot search inside file contents.
  Source: https://code.claude.com/docs/en/tools-reference (Grep Tool); exam guide 2.5-K1, 2.5-S1.

- Concept: Glob for file path pattern matching
  Type: key
  Teaches: 2.5-K2, 2.5-S2
  Definition: Glob is Claude Code's built-in tool for finding files by matching their path/name against a glob pattern (e.g., `**/*.test.tsx`, `src/**/*.ts`, `*.{json,yaml}`) — it operates purely on file paths and names, not on file contents. It is the right tool whenever the task is "find files that match this naming or extension pattern" rather than "find text inside files." Results are sorted by modification time and capped at a fixed number of matches per call.
  Example: Finding every React test file in a project uses Glob with the pattern `**/*.test.tsx`; this returns a list of matching file paths without inspecting what is inside any of them, which is what's needed before deciding which ones to open with Read.
  Source: https://code.claude.com/docs/en/tools-reference (Glob Tool); exam guide 2.5-K2, 2.5-S2.

- Concept: Read/Write for full file operations vs Edit for targeted modification
  Type: key
  Teaches: 2.5-K3
  Definition: Claude Code provides three complementary file-manipulation tools with distinct contracts. Read retrieves the full contents of a file (with line numbers attached) given an absolute path, and is typically a prerequisite before editing. Write creates a new file or completely overwrites an existing file's contents — appropriate when generating a new file or when a change is broad enough that replacing the whole file is simpler/safer than a series of targeted edits. Edit makes a targeted, surgical modification by requiring an exact `old_string` that must match the file's current content character-for-character and must occur exactly once in the file (uniqueness is enforced); it then replaces that exact match with `new_string`. Edit is preferred for small, precise modifications because it verifies the anchor text before changing anything, reducing the risk of an unintended change elsewhere in the file.
  Example: Adding a single new function to an existing file is a job for Edit (locate a unique anchor point and insert around it); generating a brand-new configuration file from scratch is a job for Write; inspecting a file's current contents before either operation is a job for Read.
  Source: https://code.claude.com/docs/en/tools-reference (Read/Write/Edit Tool sections); exam guide 2.5-K3.

- Concept: Edit failure fallback to Read + Write
  Type: key
  Teaches: 2.5-K4, 2.5-S3
  Definition: Edit's uniqueness requirement (its `old_string` must appear exactly once in the file) means it can fail on files where the target text occurs multiple times or where surrounding context is ambiguous. When that happens, the reliable fallback is to Read the entire file to get full context, then use Write to save back a reconstructed version of the file with the intended change applied — this sidesteps the uniqueness constraint entirely because Write replaces the whole file rather than needing to isolate one match. The alternative within Edit itself is to supply a longer `old_string` with enough surrounding context to make it unique, or set `replace_all: true` if every occurrence should genuinely change identically; Read+Write is the correct fallback specifically when neither of those fits.
  Example: A configuration file contains the string `timeout: 30` in five different, unrelated sections; Edit fails because `old_string: "timeout: 30"` is non-unique, so the agent reads the full file, decides which specific occurrence to change based on surrounding context, and writes back the complete file with only that one instance modified.
  Source: https://code.claude.com/docs/en/tools-reference (Edit Tool — Handling Non-Unique Matches; Fallback Strategy); exam guide 2.5-K4, 2.5-S3.

- Concept: Incremental codebase exploration strategy (Grep-then-Read)
  Type: key
  Teaches: 2.5-S4
  Definition: Rather than reading every file in a codebase upfront (slow, and it consumes a large amount of context budget on mostly-irrelevant content), an effective exploration strategy starts narrow and expands only as needed: use Grep to search for entry points — a function name, an API route, a class definition — directly relevant to the question at hand, then use Read on just the specific files Grep surfaces to follow imports and trace how control/data flows between them, repeating this narrow-search-then-targeted-read cycle as understanding builds. This keeps context usage proportional to what is actually needed to answer the question, instead of front-loading the entire codebase into context regardless of relevance.
  Example: To understand how session refresh works in an unfamiliar codebase, first Grep for "refreshToken" or "session" to find the handful of files that mention it, then Read only those files (and whatever they import) rather than reading the entire `src/` directory tree.
  Source: exam guide 2.5-S4 (task statement's own wording); consistent with the context-window-management guidance ("Claude's context window fills up fast... performance degrades as it fills") in https://code.claude.com/docs/en/best-practices.

- Concept: Tracing function usage across wrapper modules
  Type: key
  Teaches: 2.5-S5
  Definition: When a function is re-exported or wrapped by intermediate modules (a common pattern in larger codebases — e.g., a service function wrapped by an index/barrel file, then re-wrapped by a framework adapter), a single Grep for the original function's name will miss call sites that only reference it under a re-exported or aliased name. The reliable technique is a two-step search: first identify all the names a given piece of functionality is exported or re-exported under (by reading the module and its wrapper/index files), then run a Grep search for each of those names across the codebase to find every actual call site, rather than assuming a single name-based search captures full usage.
  Example: A `validateUser` function defined in `auth/core.ts` is re-exported as `checkAuth` from `auth/index.ts` and re-wrapped as `requireAuth` in `middleware/auth.ts`; finding all real usages requires Grep-ing for `validateUser`, `checkAuth`, and `requireAuth` in turn, not just the original name.
  Source: exam guide 2.5-S5 (task statement's own wording); consistent with the Grep (content) vs Glob (path) tool distinction in https://code.claude.com/docs/en/tools-reference.

---

## Prerequisites

- Concept: Tool definition (name, description, input_schema)
  Type: prerequisite
  Teaches: 2.1-K1, 2.1-K2, 2.1-S1, 2.3-K4
  Definition: In the Claude API, a tool is defined as a JSON object with a `name` (matching `^[a-zA-Z0-9_-]{1,128}$`), a `description` (a detailed plaintext explanation of what the tool does, when to use it, and how it behaves), and an `input_schema` (a JSON Schema object defining the tool's expected parameters). These definitions are passed in the `tools` parameter of a Messages API request; the API folds them into a constructed system prompt so the model can see, at every turn, what tools exist and what each is for. Every technique for designing better tool interfaces — writing clearer descriptions, renaming tools, splitting overloaded tools, or configuring `tool_choice` — operates on this basic three-part definition structure.
  Example: `{"name": "get_weather", "description": "Get the current weather in a given location...", "input_schema": {"type": "object", "properties": {"location": {"type": "string"}}, "required": ["location"]}}` is a complete tool definition.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: Agentic tool-use loop (tool_use / tool_result exchange)
  Type: prerequisite
  Teaches: 2.2-K1, 2.3-K4, 2.4-K3
  Definition: A single Claude API tool-use interaction is a cycle: the model responds with a `tool_use` content block naming a tool and its input; the calling application executes that tool and sends the result back as a `tool_result` content block in the next request; the model then reasons over that result to decide what to do next (call another tool, or respond in text). Structured error metadata (`isError`, `errorCategory`, and so on) and MCP tool results only matter because they are read and acted on inside this loop — a tool's response is fed directly back to the model as the next piece of context it reasons over, not shown to a human first.
  Example: Claude emits a `tool_use` block calling `process_refund`; the application executes it, gets back an error, and sends a `tool_result` block containing the structured error metadata; Claude's next turn reasons over that error content to decide whether to retry, explain the failure, or escalate.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use

- Concept: MCP client-server architecture and protocol messages
  Type: prerequisite
  Teaches: 2.2-K1, 2.4-K1, 2.4-K3, 2.4-K4
  Definition: The Model Context Protocol defines a client-server architecture in which an MCP server exposes capabilities — tools, resources, and prompts — over a JSON-RPC 2.0 connection, and a client (such as Claude Code) discovers those capabilities via list requests (`tools/list`, `resources/list`) and invokes them via call/read requests (`tools/call`, `resources/read`). A server declares which capabilities it supports (e.g., a `tools` capability, optionally with `listChanged` notifications) so the client knows what discovery/invocation flows are available. Understanding this discovery-then-invoke protocol shape is necessary before reasoning about how MCP servers get configured into Claude Code, how their tools get discovered, or how their results (including errors) are structured.
  Example: On connecting to a Jira MCP server, Claude Code sends a `tools/list` request and receives back a list of tool definitions (`name`, `description`, `inputSchema`) for tools like `create_issue` and `search_issues`, which it can then invoke via `tools/call`.
  Source: https://modelcontextprotocol.io/specification/2025-06-18/server/tools

- Concept: Coordinator-subagent multi-agent architecture
  Type: prerequisite
  Teaches: 2.2-S3, 2.3-K2, 2.3-K3
  Definition: In a hub-and-spoke multi-agent system, a coordinator (lead) agent decomposes a task, delegates pieces of it to specialized subagents, and aggregates their results; subagents run with their own isolated context and do not automatically share memory with the coordinator or with each other. Because each subagent is specialized for a narrower slice of the overall task, decisions about which tools each subagent needs, how errors should flow between a subagent and the coordinator, and how much a subagent should try to resolve locally before escalating are all downstream of this basic division of labor.
  Example: A research system's coordinator delegates "find recent statistics on X" to a web-search subagent and "summarize this PDF" to a document-analysis subagent; each subagent works in its own context and reports its findings (or an unresolved error) back to the coordinator rather than to each other directly.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system

- Concept: Claude Code as an agentic coding environment with built-in tools
  Type: prerequisite
  Teaches: 2.5-K1, 2.5-K2, 2.5-K3
  Definition: Claude Code is an agentic command-line/IDE tool that can read files, run commands, and make changes autonomously, backed by a fixed set of built-in tools (Read, Write, Edit, Bash, Grep, Glob) alongside any configured MCP tools. Because context window space is limited and performance degrades as it fills, which built-in tool is used for a given step (searching content vs. finding files vs. viewing vs. changing a file) has a direct, practical effect on how much of the context budget a task consumes and how reliably it succeeds — this is the backdrop against which built-in tool selection criteria (Grep vs Glob vs Read vs Write vs Edit) make sense.
  Example: A developer exploring an unfamiliar repository benefits from Claude Code's built-in tools being distinct and purpose-specific (Grep for content, Glob for filenames) rather than a single do-everything "search" tool, because it lets exploration stay narrow and context-efficient.
  Source: https://code.claude.com/docs/en/best-practices
