# Module 9 — Error Propagation and Large Codebase Context

Domain: Domain 5 — Context Management & Reliability (15% of exam)
Covers Task Statements 5.3, 5.4.

**Prerequisites from earlier modules:** hub-and-spoke coordinator/subagent
architecture and context isolation (Module 1, Lesson 1.2); structured error
categorization and MCP `isError` (Module 3, Lesson 2.2); Explore subagent (Module
5, Lesson 3.4).

---

## Lesson 5.3 — Error Propagation Strategies Across Multi-Agent Systems

**Maps to:** Task Statement 5.3: Implement error propagation strategies across
multi-agent systems.

### Prerequisite concept: tool_result / is_error mechanics

The Claude API's mechanism for reporting a tool execution problem back to the
model: a `tool_result` block with `is_error: true` and a `content` string
describing the failure. This is the concrete, API-level building block that
"structured error context" and "access failure vs. empty result" reasoning is
built on when a multi-agent system's subagents are themselves implemented as
tool-using Claude agents — the same underlying pattern as the MCP `isError` flag
you learned in Module 3, Lesson 2.2.

*Example:* `{"type": "tool_result", "tool_use_id": "toolu_01", "content":
"ConnectionError: weather service unavailable (HTTP 500)", "is_error": true}`.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
(Teaches: prerequisite for 5.3-K1, S1, K2, S2)

### Prerequisite concept: Orchestrator-worker (lead agent / subagent) architecture — recap

Recall from Module 1, Lesson 1.2: a lead ("coordinator") agent analyzes a task,
delegates to specialized subagents operating with their own context windows, and
compiles their findings. Error-propagation design only makes sense against this
architecture: it's the coordinator's job to make recovery decisions, which is
why subagents need to hand it structured, decision-useful information.

*Example:* A lead research agent spawns three subagents to investigate different
aspects of a question in parallel, then synthesizes their (possibly partially
failed) findings into one final report.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: prerequisite for 5.3-K1, K4, S1, S3, S4)

### Concept: Structured error context for coordinator recovery

When a subagent or tool call fails, returning a structured error payload — the
failure type, what was attempted, any partial results already obtained, and
possible alternative approaches — gives the coordinator enough information to
make an intelligent recovery decision instead of reacting to a bare failure
signal.

*Example:* A search subagent that times out returns
`{"failure_type": "timeout", "attempted": "search internal KB for 'refund policy
EU'", "partial_results": ["found 2 of estimated 5 relevant docs"],
"alternatives": ["retry with narrower query", "fall back to public help
center"]}` rather than just `"search failed"`.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls;
https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.3-K1, 5.3-S1)

### Concept: Distinguishing access failures from valid empty results — multi-agent framing

A query that fails to execute (timeout, service unavailable) and a query that
executes successfully but legitimately finds nothing are fundamentally different
outcomes that must be reported differently — the same principle you learned for
MCP tools in Module 3, Lesson 2.2, applied here to subagent-to-coordinator
reporting.

*Example:* A customer-database lookup that times out reports
`{"status": "access_failure", "reason": "timeout"}`; the same lookup completing
with zero matches reports `{"status": "success", "result_count": 0}` — both look
like "nothing came back," but only one is worth retrying.

*Source:* exam-guide.txt (Domain 5, Task 5.3); https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
(Teaches: 5.3-K2, 5.3-S2)

### Concept: Generic error statuses as a context-hiding anti-pattern

An error report like "search unavailable" discards exactly the details (what
failed, what was tried, what's recoverable) that would let a coordinator act
intelligently. Write instructive error messages that state what went wrong and
what should be tried next.

*Example:* "search unavailable" tells the coordinator nothing about whether to
retry, wait, or use a different source. "Rate limit exceeded on internal search
API; retry after 60s, or use the cached index as a fallback" tells it exactly
what options exist.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
(Teaches: 5.3-K3)

### Concept: Anti-patterns — silent suppression and full-workflow termination

Two opposite but equally damaging mishandlings of subagent failure: (1) silently
suppressing an error by returning an empty result as if it were a successful,
valid empty result; (2) terminating the entire workflow the moment any single
subagent fails. The graceful middle path is targeted recovery: let the
coordinator know a specific subagent is failing and let it adapt.

*Example:* If a market-data subagent fails while three others succeed, silently
returning `{}` misleads the report; halting the whole task wastes the three
completed subagents' work. The better response is a report including the three
findings plus an explicit note that market data is unavailable.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.3-K4)

### Concept: Local recovery before propagation

Subagents should attempt to resolve transient failures themselves (retry, an
alternate query, a fallback source) before escalating to the coordinator, and
only propagate once local recovery has been exhausted — at which point the
propagated error should still include what was attempted and any partial
results, echoing the "local error recovery within subagents" pattern from Module
3, Lesson 2.2.

*Example:* A subagent whose first search returns a connection error retries once
with backoff and, if that also fails, tries a narrower query before finally
reporting up that both attempts failed, along with whatever partial data either
returned.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.3-S3)

### Concept: Coverage annotations in synthesis output

A synthesis agent combining output from multiple subagents (some possibly
partially failed) should explicitly annotate which parts of its output are
well-supported and which topic areas have coverage gaps due to unavailable
sources — the synthesis-level counterpart of not silently suppressing errors: the
gap is disclosed instead of smoothed over.

*Example:* A market research report ends with a "Coverage notes" section stating
that pricing and competitor data are well-supported by three independent
sources, while regulatory information could not be retrieved.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.3-S4)

**Quiz:** see `quiz.md`, Lesson 5.3.

---

## Lesson 5.4 — Managing Context Effectively in Large Codebase Exploration

**Maps to:** Task Statement 5.4: Manage context effectively in large codebase
exploration.

### Prerequisite concept: Subagent context isolation mechanics — recap

Recall from Module 1, Lesson 1.2 and Module 5, Lesson 3.4: a subagent starts with
a fresh context window that does not include the parent conversation's history,
previously invoked skills, or previously read files — it receives only its own
system prompt, a delegation message, and (unless omitted) project configuration.

*Example:* A subagent tasked with "find all test files" cannot see that the main
agent already read `src/billing/gateway.py` earlier in the session — it starts
clean and must be told anything it actually needs.

*Source:* https://code.claude.com/docs/en/sub-agents
(Teaches: prerequisite for subagent delegation, 5.4-K3, S1)

### Concept: Context degradation in extended exploration sessions

In long-running codebase-exploration sessions, the same context-rot dynamics
from Lesson 5.1 show up concretely: the model starts giving inconsistent
answers, and begins referencing "typical patterns" instead of the specific
classes/functions/files it actually discovered earlier. The standard mitigation
is to start a fresh session or subagent for a new task rather than letting one
session run indefinitely.

*Example:* Forty tool calls into a codebase exploration, asked "does
RefundProcessor validate currency codes?", the model answers with a generic
statement about how refund processors "typically" handle validation instead of
citing the actual class it inspected earlier — a sign the earlier finding has
rotted out of effective context.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents;
supporting guidance at https://claude.com/blog/using-claude-code-session-management-and-1m-context
(Teaches: 5.4-K1)

### Concept: Scratchpad / progress files for cross-boundary persistence

A technique for counteracting context degradation: agents write notes recording
key findings to a file that persists outside the context window, and reference
that file for subsequent questions rather than relying on in-context memory.
Because the file is external, its contents survive compaction, session restarts,
and context-window boundaries.

*Example:* During a multi-session refactor, the agent maintains
`claude-progress.txt` recording "`RefundProcessor` does NOT validate currency,
relies on gateway" — and re-reads it at the start of the next session instead of
re-deriving or guessing.

*Source:* https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents;
https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.4-K2, 5.4-S2)

### Concept: Subagent delegation to isolate verbose exploration output

Rather than the main agent performing exhaustive exploration itself (accumulating
every file read and grep result in its own context), the main agent spawns
subagents to investigate narrow, specific questions. Each subagent gets its own
fresh, isolated context window, does whatever exploration it needs, and returns
only a condensed summary — keeping the main agent's context reserved for
high-level coordination.

*Example:* Instead of the main agent grepping hundreds of files itself, it
spawns a subagent with "find every place `RefundProcessor` is instantiated and
summarize the call sites in under 200 words," and only that summary returns.

*Source:* https://code.claude.com/docs/en/sub-agents
(Teaches: 5.4-K3, 5.4-S1)

### Concept: Structured state exports and manifests for crash recovery

A design for surviving crashes or context-window truncation in long-running,
multi-agent work: each agent exports its state (plan, findings, what's been
done) to a known, structured location. On resume, the coordinator loads a
manifest referencing those exports and injects relevant state back into each
agent's prompt. Anthropic's own research-agent system saves its plan to
persistent memory specifically because context exceeding 200,000 tokens gets
truncated.

*Example:* A lead agent spawns subagents to explore a large monorepo, each
writing findings to `state/subagent-2-findings.json`; a coordinator-level
`manifest.json` lists which subagents completed and where their state lives, so a
crash mid-run can be recovered without re-running completed subagents.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system;
https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
(Teaches: 5.4-K4, 5.4-S4)

### Concept: Phase summarization before spawning next-phase subagents

When codebase exploration proceeds in phases, summarizing the key findings of
one phase and injecting that summary into the initial context of the next
phase's subagents avoids two failure modes: the next phase starting with no
relevant context, and the next phase's subagents inheriting the first phase's
full, bloated exploration history.

*Example:* After a "map the module structure" phase produces a long exploration
transcript, the coordinator distills it into a five-bullet summary and passes
only that summary as starting context to the subagents spawned for the "trace
the refund flow" phase.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents;
https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.4-S3)

### Concept: Using /compact to reduce context usage mid-session

`/compact` asks the model to summarize the conversation so far and replaces the
message history with that summary, freeing up context space consumed by verbose
discovery output. It can be steered with an instruction describing what to keep,
and using it proactively — before context is nearly exhausted — gives more
control over what survives than waiting for automatic compaction.

*Example:* Partway through a long codebase-exploration session, the developer
runs `/compact focus on what we learned about the refund flow, drop the
test-file listing`.

*Source:* https://claude.com/blog/using-claude-code-session-management-and-1m-context;
https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
(Teaches: 5.4-S5)

**Quiz:** see `quiz.md`, Lesson 5.4.

---

## Module 9 summary of bullets taught

Lesson 5.3: 5.3-K1, 5.3-K2, 5.3-K3, 5.3-K4, 5.3-S1, 5.3-S2, 5.3-S3, 5.3-S4
Lesson 5.4: 5.4-K1, 5.4-K2, 5.4-K3, 5.4-K4, 5.4-S1, 5.4-S2, 5.4-S3, 5.4-S4, 5.4-S5
