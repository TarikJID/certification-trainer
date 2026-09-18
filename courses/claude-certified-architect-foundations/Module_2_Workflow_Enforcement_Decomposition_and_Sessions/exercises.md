# Module 2 Hands-On Exercise

## Scenario: Customer Support Agent with Guaranteed Compliance

You are hardening a customer-support agent that has tools `get_customer`,
`process_refund`, `escalate_to_human`, and MCP-backed tools `mcp__orders__lookup`
and `mcp__billing__lookup` (which report timestamps and statuses in inconsistent
formats).

### Part A — Enforcement

1. Design a `PreToolUse` hook (describe its match pattern, logic, and denial
   reason) that guarantees `process_refund` can never execute before
   `get_customer` has returned a verified customer ID — and explain, citing the
   permission evaluation order, why this guarantee holds even if the session is
   running in a permissive permission mode.
2. Design a second `PreToolUse` hook that blocks any `process_refund` call above
   $500 and redirects the agent toward `escalate_to_human`.
3. Explain why relying on a system-prompt instruction alone ("always verify
   identity before refunding") would not give you the same guarantee.

### Part B — Normalization

4. Design a `PostToolUse` hook that normalizes the outputs of
   `mcp__orders__lookup` and `mcp__billing__lookup` so that Claude always sees a
   single consistent date format and a single status vocabulary, regardless of
   which tool produced the result.

### Part C — Multi-concern handling and handoff

5. A customer writes: "My order arrived damaged AND I think I was double-charged
   — please help." Describe how the agent should decompose, investigate, and
   respond to this in one unified resolution.
6. That same case turns out to require human escalation (the damage claim
   involves a manufacturing defect outside policy). Write the structured handoff
   summary the agent should produce.

### Part D — Task decomposition and session management

7. The support team also wants a weekly batch job that reviews 15 open escalated
   tickets against the same 4 fixed review criteria. Should this use prompt
   chaining or dynamic adaptive decomposition? Justify your choice.
8. A support engineer resumes a named investigation session
   (`--resume billing-dispute-4471`) two days later, but the customer's account
   was updated by another team in the interim. What should the engineer do before
   asking Claude to continue, and why?

Write your answers as a short design document.
