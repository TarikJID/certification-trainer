# Module 3 Hands-On Exercise

## Redesign a poorly designed tool set

You inherit a tool set for a customer-support agent with the following four
tools, each with a one-line description:

- `getData(id)` — "Gets data."
- `updateData(id, data)` — "Updates data."
- `sendMsg(user, text)` — "Sends a message to a user."
- `sendChannelMsg(channel, text)` — "Sends a message to a channel."

1. **Diagnose.** For each tool, identify which Lesson 3.1/3.2 failure
   mode(s) it exhibits (ambiguous/overlapping description, too generic, poor
   description quality, namespacing/consolidation opportunity).

2. **Redesign.** Rewrite the tool set: give each tool a name, a 3-4 sentence
   description following the "new hire" standard, and an `input_schema`.
   Decide whether `sendMsg`/`sendChannelMsg` should be consolidated into one
   tool with an `action`/`target_type` parameter, or kept separate with
   better namespacing (e.g. `notify_user` / `notify_channel`) — justify your
   choice using Lesson 3.2's consolidation-vs-namespacing guidance.

3. **Error contract.** For your redesigned `updateData`-equivalent tool,
   write three example `tool_result` responses: one transient/retryable
   failure, one validation failure, and one business-logic failure — each
   correctly categorized per Lesson 3.3, with an instructive error message
   (not a bare "failed").

4. **MCP framing.** Assume this tool set is actually exposed by an MCP
   server named `support`. Write out what the fully namespaced MCP tool
   names would look like (`mcp__<server>__<tool>`), and specify an
   `outputSchema` for your update tool's result.

5. **Distribution.** You have a `triage` subagent (read-only) and a
   `resolver` subagent (can act). Specify each subagent's `tools`/
   `disallowedTools` frontmatter so that `triage` can only read data and
   `resolver` can additionally update it and send messages, without either
   subagent gaining tools outside its role.

### What a strong answer includes
- Correct identification that `getData`/`updateData` are too generic (no
  clear contract) and under-described, and that `sendMsg`/`sendChannelMsg`
  are an overlapping-name misrouting risk.
- Descriptions that state what the tool does, when to use it, and what each
  parameter means, matching the "new hire" standard, not just longer prose.
- Error responses using `is_error: true` with categorized, instructive
  messages, correctly mapped to transient/validation/business-logic.
- Correct MCP naming convention `mcp__support__<tool>` and a schema-shaped
  `outputSchema`.
- A `disallowedTools`-then-`tools` resolution that leaves `triage` strictly
  read-only.
