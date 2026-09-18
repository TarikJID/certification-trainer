# Module 1 Hands-On Exercise

## Scenario: Building a Research Coordinator

You are designing a coordinator agent for a research assistant product. The
coordinator has access to a `web_search` subagent and a `document_analysis`
subagent, and must produce a synthesized report for the query: "Compare the
regulatory approaches to AI in the EU, US, and China over the past two years."

### Part A — Loop design

1. Describe, turn by turn, what `stop_reason` values you would expect to see as
   the coordinator works through this query, and what the loop does at each one.
2. Identify one termination anti-pattern (from Lesson 1.1) that a naive
   implementation might fall into if it tried to detect "the coordinator is done"
   by scanning the coordinator's text output for a phrase like "Report complete."
   Explain why it's unreliable and what the loop should check instead.

### Part B — Decomposition and delegation

3. Write a task decomposition for this query that assigns non-overlapping scopes
   to three subagent invocations (you may spawn more than one `web_search`
   instance). For each, write a goal-oriented (not procedural) subagent prompt
   that includes: an objective, an expected output format, guidance on which
   sources to use, and clear task boundaries.
4. Explain, referencing Subagent Context Isolation, exactly what information you
   must include directly in each subagent's prompt versus what you can assume the
   subagent already knows.
5. Show how you would emit these subagent invocations to run in parallel rather
   than serially, and estimate (qualitatively) the latency benefit.

### Part C — Synthesis and refinement

6. After the three subagents report back, your first synthesis draft has strong
   EU and US coverage but almost nothing on China. Using the Iterative Refinement
   Loop concept, describe the coordinator's next action in detail (what it
   spawns, what prompt it gives the new subagent, and when it re-invokes
   synthesis).
7. Explain how you would pass the three subagents' findings into the synthesis
   subagent's prompt so that content and source attribution are not lost —
   referencing the Structured Data Formats concept.

### Part D — Reflection

8. If your coordinator's `allowedTools` configuration omitted `"Agent"`, what
   would happen when it tried to execute your Part B decomposition? Why?

Write your answers as a short design document (prose plus example prompts is
fine — this does not need to be runnable code).
