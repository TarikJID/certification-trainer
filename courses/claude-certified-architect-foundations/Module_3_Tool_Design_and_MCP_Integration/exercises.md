# Module 3 Hands-On Exercise

## Scenario: Redesigning a Document-Processing Toolset

Your team has a synthesis agent with the following 18 tools available:
`fetch_url`, `analyze_content`, `analyze_document`, `web_search`, `db_query`,
`send_email`, `create_ticket`, `format_report`, `cite_source`, `verify_fact`, and
8 more spanning unrelated integrations. Two of the tools —
`analyze_content` and `analyze_document` — have nearly identical one-line
descriptions, and the agent frequently calls the wrong one. Separately, the
system's MCP server for the internal knowledge base returns errors as a flat
string: `"Operation failed"` for every failure type.

### Part A — Tool interface design

1. Rewrite `analyze_content`/`analyze_document` following the rename/split
   guidance from Lesson 2.1. Produce at least two replacement tool definitions
   (name, 3-4 sentence description, and a rough input schema) with no functional
   overlap.
2. Identify one phrase you'd check the system prompt for that could bias
   selection toward one of your new tools for the wrong reason, and rewrite it.

### Part B — Tool distribution

3. This synthesis agent should only combine already-gathered findings. Propose a
   scoped tool set (4-5 tools) for it, explain what you removed and why, and
   identify one narrow cross-role tool (if any) worth keeping for a
   high-frequency need.
4. Design a `tool_choice` configuration for a pipeline step that must always run
   `extract_metadata` first, followed by a step where the model should be
   guaranteed to call *some* enrichment tool but you don't care which.

### Part C — Structured MCP errors

5. Redesign the internal knowledge base MCP server's error response format.
   Produce example JSON responses for: a timeout, an invalid query parameter, a
   policy-restricted document request, and a permission failure — each using the
   `errorCategory` / `isRetryable` / description pattern.
6. A subagent using this knowledge base hits a timeout. Walk through what it
   should try locally before reporting to the coordinator, and what the eventual
   report to the coordinator should contain if local recovery fails.

### Part D — MCP integration and built-in tools

7. The team wants the internal knowledge base MCP server shared with the whole
   team, and one engineer wants to experiment with a second, personal MCP server
   for a note-taking tool. Specify the scope and file location for each, and show
   how the shared server's API token would be referenced without being committed
   to the repo.
8. A developer needs to find every caller of a function `sendNotification` that
   is also re-exported as `notify` and `pushAlert` elsewhere in the codebase.
   Describe the exact sequence of built-in tool calls (which tool, in what order)
   they should make.

Write your answers as a short design document.
