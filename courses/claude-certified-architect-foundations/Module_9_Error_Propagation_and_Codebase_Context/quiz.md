# Module 9 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 5.3 (Error Propagation Across Multi-Agent Systems)

**Q1.** A subagent's search times out. Instead of reporting "search failed,"
what should it report, and why does that help the coordinator?

<details><summary>ANSWER</summary>
A structured error payload including failure type, what was attempted, any
partial results, and possible alternatives. This gives the coordinator enough
information to make an intelligent recovery decision (retry, try an alternative,
proceed with partial data) instead of reacting to a bare failure signal.
</details>

**Q2.** Why must "access failure" and "valid empty result" be reported
differently by a subagent, even though both might look like "nothing came back"?

<details><summary>ANSWER</summary>
They require opposite coordinator responses: an access failure (timeout,
unreachable backend) may warrant a retry or escalation decision, while a valid
empty result is a completed, trustworthy answer that retrying won't change.
Collapsing both into one response removes the information the coordinator needs
to choose correctly.
</details>

**Q3.** A market-data subagent fails while three others succeed. Name the two
anti-patterns a coordinator might fall into, and describe the better response.

<details><summary>ANSWER</summary>
Anti-pattern 1: silently suppressing the error (returning `{}` as if it were a
valid empty result), which misleads the report. Anti-pattern 2: terminating the
entire workflow because of the one failure, wasting the three subagents'
completed work. Better: include the three successful findings plus an explicit
note that market data is unavailable — targeted recovery, not suppression or
full termination.
</details>

**Q4.** Should a subagent report every transient failure straight to the
coordinator? What should it do first, and what must it include if it does end up
escalating?

<details><summary>ANSWER</summary>
No — it should attempt local recovery first (retry, alternate query, fallback
source). Only once local recovery is exhausted should it propagate to the
coordinator, and the propagated error should include what was attempted and any
partial results gathered, not just a bare failure.
</details>

**Q5.** Why is "search unavailable" a worse error message than "Rate limit
exceeded on internal search API; retry after 60s, or use the cached index as a
fallback"?

<details><summary>ANSWER</summary>
The generic message discards exactly the details (what failed, what was tried,
what's recoverable) that would let the coordinator act intelligently — it can
only guess whether to retry, wait, or switch sources. The instructive message
states what went wrong and what to try next, enabling an informed decision.
</details>

---

## Quiz — Lesson 5.4 (Context in Large Codebase Exploration)

**Q1.** Forty tool calls into an exploration session, the model answers a
question about a specific class with a generic statement about how such classes
"typically" behave, rather than citing what it actually found earlier. What is
this a symptom of, and what's the standard mitigation?

<details><summary>ANSWER</summary>
Context degradation (a codebase-exploration manifestation of context rot) — the
earlier, specific finding has effectively "rotted" out of reliable reach in a
long, growing history. The standard mitigation is to start a fresh session or
subagent for a new task rather than letting one session run indefinitely.
</details>

**Q2.** What's the purpose of a scratchpad/progress file during a multi-session
refactor, and why does it survive problems that in-context memory doesn't?

<details><summary>ANSWER</summary>
It records key findings outside the context window, so the agent can reference
it for subsequent questions instead of relying on in-context memory. Because the
file is external, its contents survive compaction, session restarts, and
context-window boundaries — none of which in-context memory survives.
</details>

**Q3.** Instead of having the main agent grep through hundreds of files itself,
what should it do, and what specifically stays out of the main agent's context as
a result?

<details><summary>ANSWER</summary>
Spawn a subagent to investigate the narrow question (e.g. "find every place
`RefundProcessor` is instantiated"). The subagent's own isolated context absorbs
the raw exploration output (file reads, grep results); only its condensed
summary returns to the main agent's context.
</details>

**Q4.** Why does Anthropic's research-agent system save its plan to persistent
memory (a structured state export) rather than keeping it only in the agent's own
context?

<details><summary>ANSWER</summary>
Because context exceeding 200,000 tokens gets truncated, and the plan needs to
survive that truncation. Exporting state to a known location (with a
coordinator-level manifest referencing it) lets work resume from where it
stopped, rather than restarting from scratch, if a crash or truncation occurs.
</details>

**Q5.** What does `/compact` do, and why is running it proactively (before
context is nearly exhausted) better than waiting for automatic compaction?

<details><summary>ANSWER</summary>
`/compact` asks the model to summarize the conversation so far and replaces the
message history with that summary, freeing space consumed by verbose discovery
output. Running it proactively — optionally steered with an instruction about
what to keep — gives more control over what survives than waiting for an
automatic compaction trigger to decide for you.
</details>
