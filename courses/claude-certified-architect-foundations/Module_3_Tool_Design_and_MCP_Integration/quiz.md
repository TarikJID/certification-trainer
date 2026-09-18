# Module 3 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 2.1 (Tool Interfaces and Descriptions)

**Q1.** Why does a minimal tool description (name + one-line summary) cause
unreliable tool selection when several similar tools exist?

<details><summary>ANSWER</summary>
The tool description is the primary — really the only — signal the model uses to
choose between tools. A minimal description gives too little information to
discriminate between similar tools, especially when purposes overlap.
</details>

**Q2.** `analyze_content` and `analyze_document` have near-identical
descriptions and the agent keeps misrouting between them. Name the two repair
techniques from this lesson and give an example of each.

<details><summary>ANSWER</summary>
(a) Rename and rewrite the description to be domain-specific (e.g.
`analyze_content` → `extract_web_results`, scoped to web content). (b) Split an
overly generic tool into purpose-specific tools with narrow contracts (e.g.
splitting `analyze_document` into `extract_data_points`, `summarize_content`,
`verify_claim_against_source`).
</details>

**Q3.** A system prompt says "always analyze the document thoroughly." Why might
this bias tool selection even if every tool description is well written?

<details><summary>ANSWER</summary>
Tool selection is influenced by the whole context, not just tool descriptions.
The word "analyze" in the system prompt shares vocabulary with a tool named
`analyze_document`, creating an unintended keyword association that can steer
selection toward that tool even when a differently-named tool is the better fit.
</details>

---

## Quiz — Lesson 2.2 (Structured MCP Error Responses)

**Q1.** Name the four error categories this lesson describes, and state whether
each is typically retryable.

<details><summary>ANSWER</summary>
Transient (usually retryable), validation (not retryable as-is — needs corrected
input), business (not retryable — a policy rule, not a temporary condition),
permission (not retryable — needs escalation/different credentials).
</details>

**Q2.** Why is a generic "Operation failed" response worse than one carrying
`errorCategory`, `isRetryable`, and a description?

<details><summary>ANSWER</summary>
A uniform message collapses all failure types into one, forcing the agent to
guess whether to retry, apologize, or escalate. Structured metadata lets the
agent make an evidence-based recovery decision instead.
</details>

**Q3.** A search subagent's primary API times out. What should it do before
reporting anything to the coordinator, and what must it include if it does end up
reporting?

<details><summary>ANSWER</summary>
It should attempt local recovery first (retry, fall back to an alternate source)
rather than immediately escalating. If it ultimately can't resolve the failure
locally, it should propagate to the coordinator along with what was attempted and
any partial results — not just a bare failure signal.
</details>

**Q4.** `lookup_order("12345")` returns `{status: "success", results: []}`. Is
this a failure? Contrast it with what a timeout response should look like.

<details><summary>ANSWER</summary>
No — this is a valid empty result: the query executed successfully and
legitimately found nothing. A timeout is an access failure and should be reported
differently, e.g. `{isError: true, errorCategory: "transient"}`, since the two
require opposite agent responses (trust the empty result vs. consider retry).
</details>

---

## Quiz — Lesson 2.3 (Tool Distribution and tool_choice)

**Q1.** Why does giving an agent 18 tools instead of 4-5 degrade tool selection
reliability, even if none of the 18 tools is individually poorly designed?

<details><summary>ANSWER</summary>
Every additional tool adds to the decision the model must make each turn — more
tools (especially similar ones) increases decision complexity and the chance of
choosing incorrectly, independent of any single tool's description quality.
</details>

**Q2.** A synthesis agent, which should only combine already-gathered findings,
keeps firing off its own web searches when it has a `web_search` tool available.
What's the fix?

<details><summary>ANSWER</summary>
Restrict the synthesis agent's tool set to only the tools relevant to its role
(e.g. `format_report`, `cite_source`), removing `web_search` entirely so
cross-specialization misuse isn't possible — rather than relying on prompting
alone to prevent it.
</details>

**Q3.** Explain the difference between `tool_choice: "auto"`, `"any"`, and
`{"type": "tool", "name": "..."}`, and give a scenario where you'd want to force
a specific named tool.

<details><summary>ANSWER</summary>
`auto` lets Claude decide whether to call a tool at all or respond in text.
`any` forces some tool call but leaves the choice of which tool to Claude.
`{"type":"tool","name":"..."}` forces one specific tool. You'd force a named tool
to guarantee a particular step (e.g. `extract_metadata`) always runs first in a
pipeline before other steps.
</details>

**Q4.** What is the middle-ground pattern between "give every agent every tool"
and "give every agent only its narrowest role-specific tools"?

<details><summary>ANSWER</summary>
Scoped tool access with limited cross-role tools: keep the default tool set
scoped to the role, but add a small number of narrow, constrained cross-role
tools for specific high-frequency needs, routing anything more complex through
the coordinator instead of broadly expanding tool access.
</details>

---

## Quiz — Lesson 2.4 (MCP Server Integration)

**Q1.** A team wants a shared GitHub MCP server available to every teammate via
version control, and a developer separately wants to try an experimental server
just for themselves. Which scopes should each use?

<details><summary>ANSWER</summary>
The team's shared server goes in project-scoped `.mcp.json` (committed to version
control). The developer's experimental server goes in user-scoped
`~/.claude.json` (private to them, applies across their own projects).
</details>

**Q2.** Why shouldn't a committed `.mcp.json` contain a literal API token, and
what syntax solves this?

<details><summary>ANSWER</summary>
Because the file is checked into version control, a literal secret would be
exposed to everyone with repo access. Environment variable expansion (`${VAR}`,
or `${VAR:-default}`) lets the file reference a variable name while each
developer supplies their own value locally.
</details>

**Q3.** A generic MCP tool `search_code` is described only as "search the
codebase" and the agent keeps using the built-in Grep instead, even though
`search_code` does more capable semantic search. What's the fix?

<details><summary>ANSWER</summary>
Enhance the MCP tool's description to explicitly explain its capabilities and
what it returns in enough detail that the agent recognizes it as the stronger
choice — tool descriptions are the model's only window into a tool's
capabilities, so an under-described MCP tool loses to a well-documented built-in.
</details>

**Q4.** When should a team build a custom MCP server instead of adopting an
existing community one?

<details><summary>ANSWER</summary>
When the integration is genuinely team-specific — internal systems, proprietary
APIs, business logic no community server could reasonably cover — where the
value of a purpose-built integration outweighs the cost of owning a server. For
standard integrations (e.g. Jira), an existing community/vendor server should be
adopted instead.
</details>

**Q5.** What are MCP resources, and why would a database MCP server expose table
schemas as resources rather than requiring a tool call to fetch them?

<details><summary>ANSWER</summary>
Resources are read-only, URI-identified content a client can list and read
directly, without exploratory tool calls. Exposing table schemas as resources
lets the agent see the full schema catalog up front and plan a correct query
immediately, instead of first calling "list tables" then "describe table" for
each candidate.
</details>

---

## Quiz — Lesson 2.5 (Built-in Tools: Read, Write, Edit, Bash, Grep, Glob)

**Q1.** You need to find every place the string "DEPRECATED_API_KEY" appears in a
codebase. Which built-in tool, and why not the alternative?

<details><summary>ANSWER</summary>
Grep — it searches file contents for a pattern. Glob would only match files by
name/path pattern and cannot search inside file contents.
</details>

**Q2.** `Edit` fails because its `old_string` matches five places in a file.
What's the reliable fallback, and why does it work when Edit doesn't?

<details><summary>ANSWER</summary>
Read the full file, then Write back a reconstructed version with the intended
change applied. It works because Write replaces the whole file rather than
needing to isolate one unique match, sidestepping Edit's uniqueness constraint.
</details>

**Q3.** Describe the "Grep-then-Read" incremental exploration strategy and why
it's preferable to reading every file in a codebase upfront.

<details><summary>ANSWER</summary>
Start narrow: Grep for entry points directly relevant to the question, then Read
only the specific files Grep surfaces to follow imports and trace flow, repeating
as understanding builds. This keeps context usage proportional to what's actually
needed, instead of front-loading the entire codebase regardless of relevance.
</details>

**Q4.** A function `validateUser` is re-exported as `checkAuth` and re-wrapped as
`requireAuth` elsewhere. Why would a single Grep for `validateUser` under-count
its usages, and what's the correct technique?

<details><summary>ANSWER</summary>
Call sites that reference the function only under its re-exported/aliased names
won't match a Grep for the original name. The correct technique: first identify
all the names the functionality is exported/re-exported under, then Grep for
each name in turn across the codebase.
</details>
