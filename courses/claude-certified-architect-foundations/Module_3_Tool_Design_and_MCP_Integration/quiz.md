# Module 3 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 3.1 — Writing Effective Tool Descriptions

1. What does Anthropic say is "by far the most important factor in tool
   performance"?
2. Why is `notification-send-user` next to `notification-send-channel` a
   risky pair of tool names?
3. Contrast the two claims about system prompt wording and tool selection
   from this lesson: one is vendor-documented, one is exam-guide-only. What
   does each claim, and which is which?

### Answers
1. Extremely detailed tool descriptions.
2. Their near-identical names/purposes make them easy for Claude to
   conflate, risking a user notification being sent when a channel
   notification was intended, or vice versa — the "ambiguous/overlapping
   descriptions" failure mode.
3. The vendor-documented claim (platform.claude.com) is that system prompt
   wording moves the boundary of *whether a tool gets called at all* (e.g.
   "always call a tool first" vs. "use your judgment"). The exam-guide-only
   claim (Task 2.1) is stronger and more specific: that keyword-sensitive
   wording can create an *unintended association with a particular tool*,
   potentially overriding a well-written tool description — this claim is
   not corroborated by vendor documentation.

---

## Quiz 3.2 — Structuring the Tool Set

1. What is the fix for a tool that is "too generic," per Anthropic's
   guidance?
2. Roughly how many tokens can a typical five-server MCP setup consume in
   tool definitions before any work is done, and around what tool count
   does selection accuracy start to degrade?
3. What are the four `tool_choice` values and what does each do?
4. What does `strict: true` add when combined with a forced `tool_choice`?

### Answers
1. Split it into a few purpose-specific tools with clear, distinct
   purposes and well-defined input/output contracts, rather than one
   broad, do-everything tool.
2. ~55k tokens; selection accuracy degrades past roughly 30-50 available
   tools.
3. `auto` (Claude decides whether to call any tool, the default);
   `any` (Claude must call one of the provided tools, but picks which);
   `tool` (forces one specific named tool); `none` (no tool use allowed,
   the default when no tools are given).
4. It guarantees the forced call's arguments conform exactly to the tool's
   schema, in addition to guaranteeing that a call happens at all.

---

## Quiz 3.3 — Error Handling for Tools and MCP

1. What field, set to `true`, signals a client-tool execution failure in a
   `tool_result` block?
2. How does MCP distinguish a protocol-level error from a tool-execution
   error?
3. Name the four-way error categorization the exam guide names, and give
   one Claude API/MCP concept that corresponds to each.
4. Why must a "no results found" search response NOT be reported as
   `isError: true`?

### Answers
1. `is_error`.
2. Protocol errors are standard JSON-RPC errors (e.g., code -32602)
   returned when the request itself is malformed or the tool doesn't
   exist; tool execution errors are reported inside a successful JSON-RPC
   response as a `CallToolResult` with `isError: true`.
3. Transient/retryable (Claude API `rate_limit_error`/`overloaded_error`,
   retried automatically with backoff), validation
   (`invalid_request_error`/MCP "invalid input data"), permission
   (`permission_error`), and business-logic (MCP "business logic errors,"
   e.g. a refund exceeding a policy limit).
4. Because it is a legitimate, successful outcome (nothing matched), not a
   failure — conflating the two would make the caller unable to
   distinguish "denied/broken" from "nothing found."

---

## Quiz 3.4 — MCP Architecture and Integration

1. What are MCP's three server-exposed primitives?
2. What does an MCP Host do when it connects to two different MCP servers?
3. What does `${API_KEY}` syntax do inside `.mcp.json`, and what happens if
   `API_KEY` is unset with no default given?
4. Why does referencing `@github:repos/owner/repo/issues` avoid the
   "exploratory back-and-forth" of a search tool call?
5. What message does a server send when its available tools change, and
   what does Claude Code do in response?

### Answers
1. Tools (AI-invokable actions), Resources (contextual data), and Prompts
   (reusable interaction templates).
2. It creates one MCP Client per Server, each maintaining its own dedicated
   connection.
3. It expands to the value of the `API_KEY` environment variable at
   runtime, letting a shared config file avoid hard-coded secrets. If
   unset with no default, the config still loads — Claude Code reports a
   missing-variable warning in `claude mcp list` and leaves the literal
   `${API_KEY}` text unexpanded.
4. Because a resource is fetched by a known, stable URI via `resources/read`
   rather than searched for — the server can already address the content
   directly, with no need for the model to first invoke a tool to locate
   it.
5. `notifications/tools/list_changed`; Claude Code automatically calls
   `tools/list` again to refresh the tool list, without requiring
   disconnect/reconnect.

---

## Quiz 3.5 — Distributing Tools and Selecting Built-in Tools

1. If a subagent's frontmatter sets both `tools` and `disallowedTools`,
   which is applied first?
2. Name two tools that are always removed from a subagent regardless of
   configuration.
3. What is the functional difference between Grep and Glob?
4. What is the functional difference between Write and Edit?
5. When Edit's `old_string` matches more than one location, what two
   vendor-documented recovery options does Claude have — and what is the
   distinct, exam-guide-only fallback mentioned in this lesson?

### Answers
1. `disallowedTools` is applied first, and then `tools` is resolved
   against what remains.
2. Any two of: `AskUserQuestion`, `EndConversation`, `ExitPlanMode` (unless
   in plan mode).
3. Glob finds files by name/path pattern; Grep searches the contents of
   files for a pattern — "Where Glob finds files by name, Grep finds lines
   inside them."
4. Write creates or overwrites an entire file's contents; Edit makes a
   narrower, surgical, exact-string-replacement change to part of a file.
5. Vendor-documented: supply a longer `old_string` with enough surrounding
   context to make it unique, or set `replace_all: true`. The distinct,
   exam-guide-only fallback (Task 2.5, not vendor-corroborated) is to use
   Read to get the file's contents and Write to save a corrected version
   instead.
