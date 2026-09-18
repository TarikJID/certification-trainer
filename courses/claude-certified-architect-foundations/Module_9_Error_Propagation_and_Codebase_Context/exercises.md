# Module 9 Hands-On Exercise

## Scenario: A Resilient Research Coordinator Exploring a Large Monorepo

You're building a coordinator that spawns subagents to (a) research external
market data via several APIs and (b) explore a 2,000-file monorepo to plan a
large refactor. Both are long-running, multi-agent tasks where failures and
context limits are expected.

### Part A — Error propagation

1. Design the structured error payload a market-data subagent should return when
   one of its three data sources times out but the other two succeed. Include
   failure type, what was attempted, partial results, and alternatives.
2. The coordinator receives that structured error. Walk through its decision
   process: does it retry, proceed with partial data, or escalate further? What
   information from the payload drives that decision?
3. A different subagent's database query returns zero rows. Show the two
   different response shapes this subagent should use to distinguish "the query
   failed" from "the query succeeded and found nothing," and explain why
   conflating them would be dangerous here.
4. Identify which of the two anti-patterns (silent suppression, full-workflow
   termination) a naive implementation of this coordinator might fall into if a
   single data source is unreachable, and describe the graceful alternative.
5. Write the "Coverage notes" section of a final market report where one of
   three sources failed.

### Part B — Codebase context management

6. Design a scratchpad file structure the monorepo-exploration agent should
   maintain across a multi-day refactor-planning effort. Show an example entry.
7. The exploration proceeds in two phases: "map module structure" then "trace
   dependency chains for the payment module." Describe how you'd hand off from
   phase 1 to phase 2 without either losing phase 1's findings or flooding phase
   2's subagents with phase 1's full raw exploration transcript.
8. Design a manifest-based crash recovery scheme: what does each subagent export,
   where, and what does the coordinator do on resume after a crash mid-exploration?
9. Forty tool calls into the exploration, you notice the agent describing a
   class's behavior in generic terms rather than citing what it found earlier.
   What's happening, and what are your two options to address it (one
   in-session, one architectural)?

Write your answers as a short reliability design document.
