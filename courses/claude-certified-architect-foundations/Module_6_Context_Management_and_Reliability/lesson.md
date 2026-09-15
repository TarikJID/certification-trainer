# Module 6: Context Management & Reliability

Domain weight on the exam: 15%. This is the capstone module: it deepens
Module 1's context-window basics, and draws on the subagent/permission-mode
material from Modules 2 and 4, and the grounding/caching material from
Module 5.

Several concepts in this module are sourced solely to the official CCAR-F
Exam Guide because the researcher could not find vendor documentation
covering them, despite dedicated searching (third-party exam-prep
aggregators were deliberately excluded as a source). These are marked with
an explicit sourcing note wherever they appear — treat them as
certification-relevant facts, not confirmed product documentation.

---

## Lesson 6.1: Context Window Depth — Context Rot and Long-Context Recall

### Concept: Context window and context rot
Deepening Module 1's basic Context Window concept: the context window is all
the text (system prompt, every message including tool results/images/
documents, tool definitions, and the model's own output) the model can
reference when generating a response — a bounded "working memory," distinct
from the model's training data. As token count grows, model accuracy and
recall degrade, a phenomenon Anthropic calls "context rot": because
attention in a transformer creates pairwise relationships between tokens, a
larger context stretches the model's effective attention thinner, and
models have less training exposure to very long sequences. This means
curating what is in context is as important as how much space is available.
Context windows currently range up to 1M tokens depending on model; going
over the limit triggers either a 400 "prompt is too long" error (if input
alone exceeds the window) or, on newer models, a graceful `stop_reason:
"model_context_window_exceeded"`.

**Example:** A long customer-support chat accumulates dozens of tool calls
and file attachments; even though the 1M-token window isn't full, the model
starts missing details from early in the conversation because the "signal"
is diluted by irrelevant accumulated content — this is context rot, not
window overflow.

**Source:** Context Management & Reliability research — Context window and
context rot (key), https://platform.claude.com/docs/en/build-with-claude/context-windows ; https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

### Concept: Needle-in-a-haystack evaluation methodology
A test methodology for measuring long-context recall, where a specific
target fact ("the needle") is inserted somewhere inside a large body of
otherwise-unrelated text ("the haystack"), and the model is asked a
question that requires it to retrieve or use that inserted fact. This is
the methodology Anthropic used to measure and demonstrate the
isolated-sentence retrieval problem in Claude 2.1 (next concept), and is
foundational to understanding both context-rot claims above and the "lost
in the middle" effect below.

**Example:** Inserting a single invented sentence about a fictional fact
into the middle of a 100-page document and then asking the model to state
that fact verifies whether the model can retrieve information regardless of
its position in the document.

**Source:** Context Management & Reliability research — Needle-in-a-haystack
evaluation methodology (prerequisite), https://claude.com/blog/claude-2-1-prompting

### Concept: "Lost in the middle" positional recall degradation in long contexts
> **Sourcing note:** the exam guide names this effect directly (Domain 5,
> Task 5.1): "models reliably process information at the beginning and end
> of long inputs but may omit findings from middle sections." Anthropic's
> own documentation does not use the phrase "lost in the middle" verbatim,
> but documents a closely related, concretely measured version of the same
> class of failure (below). The exact phrase does not appear in the
> Anthropic blog post cited alongside it — treat the specific "lost in the
> middle" framing as an exam-guide fact, and the needle-in-a-haystack
> measurement as its closest vendor-documented corroboration, not a
> verbatim match.

Using the needle-in-a-haystack methodology above, Anthropic evaluated Claude
2.1's 200K-token context with a target sentence inserted into a long
document (e.g. a sentence about Dolores Park inserted into a collection of
Paul Graham essays); Claude only retrieved the isolated, out-of-place
sentence correctly 27% of the time — a failure Anthropic attributes to the
model's trained reluctance to assert isolated, seemingly-out-of-place
claims, not to positional attention decay specifically. Appending a single
instruction ("Here is the most relevant sentence in the context:") to the
start of the expected response raised accuracy to 98%. Separately,
Anthropic's long-context prompting guidance recommends placing long
documents near the top of the prompt (above the query/instructions),
reporting that moving the query to the end of the prompt can improve
response quality by up to 30% in tests — a placement-based mitigation
consistent with, though not framed as an explanation of, the
beginning/end-versus-middle effect the exam guide names.

**Example:** A 150-page contract is passed to Claude with the key
indemnification clause buried on page 80 (the "middle"); per the exam
guide's Task 5.1 framing, this middle-positioned clause is at higher risk of
being missed than clauses on page 1 or page 150 — documented mitigations
include restructuring the prompt (query at the end, documents first) and,
for isolated-fact retrieval, priming the response with an instruction to
state the most relevant sentence first.

**Source:** Claude Certified Architect – Foundations Exam Guide, Domain 5,
Task 5.1 (exam-guide framing) ; corroborating vendor sources:
https://claude.com/blog/claude-2-1-prompting ; https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/long-context-tips

### Concept: Context awareness (token budget injection)
On models that support it (Claude Sonnet 5, Sonnet 4.6, Sonnet 4.5, Haiku
4.5), the API automatically injects a `<budget:token_budget>` tag into the
system prompt showing the model its total context window, and after each
tool call injects a `<system_warning>` tag reporting tokens used and
remaining. This lets the model manage long-running, tool-heavy tasks
against its actual remaining capacity instead of guessing. It requires no
configuration. Models that don't receive these tags can instead be given an
explicit budget via the (beta) "task budgets" feature.

**Example:** During a 40-tool-call agentic coding session, the model sees
"Token usage: 150000/200000; 50000 remaining" after a tool result and
decides to wrap up and summarize its work rather than opening ten more
files.

**Source:** Context Management & Reliability research — Context awareness
(key), https://platform.claude.com/docs/en/build-with-claude/context-windows#context-awareness

---

## Lesson 6.2: Managing Context at Scale — Compaction, Editing, and Caching

### Concept: Server-side compaction
A beta API feature (`compact_20260112` strategy under
`context_management.edits`) that automatically summarizes older conversation
turns once input tokens cross a configured trigger (default 150,000,
minimum 50,000), inserting a `compaction` content block containing the
summary and instructing the API to drop everything before that block on
subsequent requests. Configuration options include a custom `trigger`,
`pause_after_compaction` (to let the caller inspect/edit the summary before
continuing — useful for enforcing a total token budget across many
compactions), and `instructions` (to replace the default summarization
prompt, e.g. to prioritize preserving code and technical decisions). It is
the primary mechanism recommended for long-running, multi-turn
conversations and agentic sessions that would otherwise hit the context
window limit. Module 4's `PreCompact` hook fires immediately before this
happens, giving an application a chance to archive the full transcript.

**Example:** An agent running for hours accumulates 300K tokens of tool
output; at the configured 150K-token trigger, the API auto-generates a
`<summary>` block preserving state, next steps, and learnings, and the
client appends that block so the next request continues with a much
smaller prompt.

**Source:** Context Management & Reliability research — Server-side
compaction (key), https://platform.claude.com/docs/en/build-with-claude/compaction

### Concept: Context editing (tool result clearing and thinking block clearing)
A more fine-grained, beta alternative/complement to compaction that lets a
caller selectively clear specific content instead of summarizing
everything. Two strategies: `clear_tool_uses_20250919` removes the oldest
tool call/result pairs once a trigger (input tokens or tool-use count) is
exceeded, keeping a configurable number of the most recent pairs,
optionally excluding specific tools, and replacing cleared content with a
placeholder; `clear_thinking_20251015` clears older extended-thinking
blocks (Module 1) while optionally preserving a configurable number of
recent ones (behavior differs by model class). Both require the
`context-management-2025-06-27` beta header (Module 1), are applied
server-side before Claude sees the prompt, and leave the caller's own local
conversation history untouched. The API reports exactly what was cleared
via a `context_management.applied_edits` field.

**Example:** An agentic research workflow with heavy web-search tool use
keeps only the last 3 tool results in context via
`clear_tool_uses_20250919`, dramatically cutting input-token cost on turn
50 without losing the model's ability to reason about its most recent
findings.

**Source:** Context Management & Reliability research — Context editing
(key), https://platform.claude.com/docs/en/build-with-claude/context-editing

### Concept: Prompt caching for long-running context
As introduced in Module 5 for batch workloads, prompt caching lets a caller
mark a prefix of a request (system prompt, tool definitions, long fixed
context) with `cache_control` so the API can reuse pre-computed state on a
later call instead of reprocessing it — cutting input-token cost by up to
~90% and latency by up to ~85% on cache hits. Caches default to a 5-minute
TTL (refreshed on each hit) or can be extended to 1 hour at 2x the write
cost. The cache only helps if the breakpoint is on content that is
byte-identical across requests, and it does not reduce what counts toward
the context window (Lesson 6.1) — only what a cache hit costs. In
multi-turn conversations (the specific relevance for long-running agentic
sessions, distinct from Module 5's batch-workload framing) the API supports
an automatic "lookback" that finds the longest previously-cached prefix.

**Example:** A coding agent with a large, static system prompt and tool
schema caches that prefix once; every subsequent turn in the session reads
that prefix from cache at 10% of the normal input price instead of repaying
full price for it on every request.

**Source:** Context Management & Reliability research — Prompt caching for
long-running context (key), https://platform.claude.com/docs/en/build-with-claude/prompt-caching ; https://platform.claude.com/docs/en/build-with-claude/context-windows

---

## Lesson 6.3: Long-Running Agents and Large Codebase Exploration

### Concept: Long-running agent harnesses (multi-session context handoff)
For tasks that span far more time or work than a single context window can
hold (e.g. multi-day software projects), Anthropic documents a two-agent
harness pattern: an initializer agent runs once to set up the environment,
expand the prompt into a structured requirements/feature list, and write a
boot script; a coding agent is then woken up repeatedly across many
separate sessions (Module 4's session-state mechanisms are what makes
"waking up" possible), each one reading git history and a progress file to
reconstruct state, making incremental progress on one feature, verifying it
end-to-end, and leaving a clean handoff (progress notes, updated
feature-pass/fail status, a commit) for the next session. This treats
context reset as unavoidable for very long jobs — summarization/compaction
alone is not enough — so the harness periodically does a full session
teardown and rebuild from a structured handoff artifact, analogous to
onboarding a new engineer.

**Example:** A project scoped to run over three days of Claude sessions has
each new session start by running `pwd`, reading `claude-progress.txt` and
the git log, picking the highest-priority failing feature from
`feature_list.json`, and verifying the dev server still works before
writing any new code.

**Source:** Context Management & Reliability research — Long-running agent
harnesses (key), https://anthropic.com/engineering/effective-harnesses-for-long-running-agents

### Concept: Context isolation via subagents for large codebase exploration
Because reading many files to understand an unfamiliar or large codebase
can flood the main conversation's context window, Claude Code and the
Agent SDK support delegating exploration to subagents — deepening Module
4's Coordinator–Subagent pattern and Module 2's `Explore` subagent —
including the built-in read-only `Explore` subagent, which runs on a
fast/cheap model and reads excerpts rather than whole files, in its own,
separate context window. Only the subagent's final, condensed summary
returns to the parent; every intermediate file read, grep result, or tool
call stays inside the subagent and never accumulates in the orchestrating
conversation. This gives context isolation, enables parallel exploration of
independent areas of a codebase, and lets a specialized subagent carry
tailored instructions/tool restrictions without adding noise to the main
agent's prompt.

**Example:** A `research-assistant` subagent explores dozens of files
across a monorepo to answer "how does the auth module handle token
refresh?"; the parent conversation only receives a two-paragraph summary,
not the dozens of files read to produce it.

**Source:** Context Management & Reliability research — Context isolation
via subagents for large codebase exploration (key), https://code.claude.com/docs/en/agent-sdk/subagents ; https://code.claude.com/docs/en/best-practices

### Concept: Structured note-taking (scratchpad files / agentic memory)
> **Sourcing note:** this concept combines an exam-guide-sourced pattern
> name with genuine vendor corroboration under a different name — both are
> cited below.

The exam guide (Domain 5, Task 5.4) names this pattern under "scratchpad
files," listing as Knowledge "the role of scratchpad files for persisting
key findings across context boundaries" and as a Skill "having agents
maintain scratchpad files recording key findings, referencing them for
subsequent questions to counteract context degradation." Anthropic's own
engineering documentation covers the same pattern under the name
"structured note-taking," also called "agentic memory": "a technique where
the agent regularly writes notes persisted to memory outside of the context
window." Anthropic gives two concrete examples — Claude Code creating a
to-do list, and a custom agent maintaining a NOTES.md file — and cites an
agent playing Pokémon that "maintains precise tallies across thousands of
game steps" and "develops maps of explored regions" by writing to
persistent notes. Anthropic ties this to a supporting product feature: the
memory tool, released in public beta alongside Claude Sonnet 4.5, which
"makes it easier to store and consult information outside the context
window through a file-based system," letting agents "build up knowledge
bases over time, maintain project state across sessions, and reference
previous work without keeping everything in context." This is distinct from
the two other large-scale concepts in this lesson: context isolation via
subagents delegates exploration to a separate agent with its own context
window; long-running harnesses tear down and rebuild an entire session
across multi-day projects; structured note-taking is the *in-session*
pattern of a single agent externalizing findings to a file as it goes,
specifically to counteract context degradation while exploring, without
delegating or resetting the session.

**Example:** While exploring an unfamiliar 500-file codebase in one long
session, an agent writes each significant finding ("auth is handled in
src/auth/session.ts via JWT refresh tokens") to a scratchpad/NOTES.md file
as it discovers it, then re-reads that file instead of re-deriving the same
facts from scratch when a later question in the same session depends on
them.

**Source:** Claude Certified Architect – Foundations Exam Guide, Domain 5,
Task 5.4 (pattern name: "scratchpad files") ; corroborating vendor source
(pattern name: "structured note-taking"/"agentic memory"):
https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

---

## Lesson 6.4: Error Propagation and Escalation to Humans

### Concept: Error propagation across multi-agent systems
When an agent delegates work to subagents (the orchestrator-worker
architecture from Modules 4 and 5), an error in a subagent does not
automatically surface as normal output to the parent: specifically, an API
error that ends a subagent early (e.g. a rate limit) is never delivered as
that subagent's result — the parent must handle its absence explicitly. At
larger scale, Anthropic's own production multi-agent research system
documents that small prompt/config changes can cause disproportionate,
cascading behavioral failures (e.g. an early version pathologically spawned
50 subagents for a simple query, or ran endless searches for information
that didn't exist), and that because agents are stateful and run for
extended periods, errors compound rather than resetting cleanly — the
chosen mitigation was checkpoint-based resumption (Module 4's session state
mechanisms) plus letting the model's own intelligence handle transient tool
failures gracefully, backed by full production tracing (without inspecting
conversation content) to diagnose failure patterns, and staged ("rainbow")
deployments so in-flight long-running agents aren't disrupted by an update.

**Example:** A lead research agent spawns five subagents to gather sources
in parallel; one subagent hits a rate-limit error and terminates silently
mid-task. The orchestrator must detect the missing/partial result (rather
than treating silence as "no relevant sources found") and decide whether to
retry, reassign, or proceed with reduced coverage.

**Source:** Context Management & Reliability research — Error propagation
across multi-agent systems (key), https://code.claude.com/docs/en/agent-sdk/subagents#what-subagents-inherit ; https://www.anthropic.com/engineering/built-multi-agent-research-system

### Concept: Escalation patterns and human review workflows (permission modes)
Deepening Module 2's permission modes and Module 4's enforcement/hooks
evaluation order: in Manual mode, the agent stops and asks the human before
most file-writing, command-execution, or network-reaching actions. In auto
mode (the default starting mode on paid plans), a separate classifier model
reviews actions instead of a human, using a two-stage design: a fast,
single-token "err on the side of blocking" filter, followed by a slower
chain-of-thought review only for actions the filter flagged, judging the
real-world effect of an action (including obfuscated/indirect operations)
rather than just its surface text. Rather than a numeric confidence
threshold, the shipped design escalates to a human based on outcome counts:
3 consecutive denials or 20 total denials in a session trigger human
escalation; short of that, a denied action is returned to the agent with an
instruction to find a safer path. Plan mode (Module 2) is a related,
task-scoped escalation pattern: it forces a read-only "explore and propose a
plan" phase and blocks all edits until the human explicitly approves the
plan.

**Example:** An autonomous agent's shell command is flagged by the
classifier as risky (installs an unfamiliar network tool); the action is
blocked and the agent is told to find another approach. After the agent
accumulates 3 consecutive blocked attempts down different paths, the
session escalates and asks the human directly instead of the classifier
deciding.

**Source:** Context Management & Reliability research — Escalation patterns
and human review workflows (key), https://code.claude.com/docs/en/permission-modes ; https://www.anthropic.com/engineering/claude-code-auto-mode

### Concept: Escalation triggers and anti-patterns for human handoff
> **Sourcing note:** sourced solely to the official CCAR-F Exam Guide
> (Domain 5, Task 5.2); no vendor documentation covering these specific
> business-process criteria was found. Distinct from the mechanics of the
> concept above.

The exam guide specifies criteria for when a system built on Claude should
hand a case off to a human — a business-logic/process-design concept about
*what should trigger* a handoff and *why naive proxies fail*, distinct from
the tool-permission escalation mechanics above (which cover *how* Claude
Code decides to pause an autonomous coding action). Appropriate escalation
triggers: "customer requests for a human, policy exceptions/gaps, and
inability to make meaningful progress." Two named anti-patterns to avoid:
"sentiment-based escalation and self-reported confidence scores are
unreliable proxies for actual case complexity"; and when a case is
ambiguous, the system should resolve it by "requesting additional
identifiers" from the user rather than picking a heuristic (guessing from
tone or a raw confidence number).

**Example:** A support agent detects a customer is being short/curt
(negative sentiment) but is still making progress resolving a
straightforward return request; sentiment alone should not trigger
escalation. Conversely, if the customer explicitly asks for a human, or the
request falls into a documented policy gap, the system should escalate
regardless of how "confident" or polite the exchange seemed — and if the
case is simply ambiguous (e.g. two accounts might match), the system should
ask the customer for another identifying detail rather than guessing.

**Source:** Claude Certified Architect – Foundations Exam Guide, Domain 5,
Task 5.2 (exam-guide-only; not found in vendor documentation).

---

## Lesson 6.5: Ambiguity, Confidence, and Provenance

### Concept: Ambiguity resolution patterns
Anthropic's prompting-best-practices documentation frames ambiguity handling
as an explicit, opposed design choice the system builder must make (a
direct application of Module 1's "being clear and direct" principle to
system-prompt design), illustrated by two named, complementary
system-prompt patterns for agentic/tool-using systems. `default_to_action`
makes the model proactive: when intent is unclear, it infers the most
useful likely action and proceeds — using its tools to discover missing
details rather than stopping to guess or ask.
`do_not_act_before_instructions` makes the model conservative: it does not
jump into implementation or change files unless clearly instructed, and,
when intent is ambiguous, defaults to providing information, research, and
recommendations rather than taking action. The documentation frames this as
necessary because Claude's newer models follow instructions precisely, so
without an explicit default the model may under- or over-act on an
ambiguous request — the fix is to pick and state one of these two behaviors
rather than leave it to the model's own inference. (Note: this source
documents the act-vs-infer/proceed-vs-report tension; it does not itself
use "ask a clarifying question" as the resolution mechanism, so that
specific framing is not attributed to this source.)

**Example:** A coding agent's system prompt includes a
`<do_not_act_before_instructions>` block; when asked "the login page looks
off, can you take a look?" it inspects the code and reports what it found
and recommends a fix, but does not edit any files, because the request
didn't clearly instruct it to make a change.

**Source:** Context Management & Reliability research — Ambiguity
resolution patterns (key), https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

### Concept: Confidence expression and calibration in human-review workflows
> **Sourcing note:** beyond the two documented mechanisms below, Anthropic's
> public documentation does not appear to define a single named "confidence
> calibration" framework for human-review workflows. Treat the term as an
> architectural pattern composed from these building blocks, not a single
> first-party feature.

Anthropic's documented practice for reducing hallucination and improving
reliability is to give the model explicit permission, in the prompt, to
express uncertainty or decline to answer rather than guessing when evidence
is insufficient — a direct fix for "the model makes up information" (Module
5's grounding techniques). This is distinct from an automated numeric
"confidence score" used to gate human review: Anthropic's own production
design for gating risky agent actions (the Claude Code auto-mode
classifier, Lesson 6.4) explicitly does not rely on a confidence threshold,
instead using escalation limits based on how many actions were denied (3
consecutive / 20 total) — because that proved more robust than trying to
calibrate a single confidence number.

**Example:** A document-QA agent is instructed "if the data is insufficient
to draw conclusions, say so rather than speculating" — when it can't find a
clear answer, it reports low confidence and routes the query to a human
reviewer instead of fabricating a number.

**Source:** Context Management & Reliability research — Confidence
expression and calibration (key), https://claude.com/blog/best-practices-for-prompt-engineering ; https://www.anthropic.com/engineering/claude-code-auto-mode

### Concept: Stratified random sampling and field-level confidence scores for validating extractions
> **Sourcing note:** sourced solely to the official CCAR-F Exam Guide
> (Domain 5, Task 5.5); no vendor documentation on this specific QA
> methodology was found.

The exam guide names a specific quality-assurance methodology for human
review workflows, distinct from the prompted-uncertainty and
denial-count mechanisms above: "stratified random sampling for measuring
error rates in high-confidence extractions and detecting novel error
patterns" — deliberately sampling across strata (e.g., by confidence band or
field type) rather than only reviewing low-confidence outputs, specifically
so a new failure mode affecting outputs rated high-confidence can still be
caught. Paired with "field-level confidence scores calibrated using labeled
validation sets" — assigning a confidence score per extracted field (not
just per document/response), calibrated (checked and adjusted) against a
human-labeled validation set so that, e.g., a field scored "90% confidence"
is empirically correct roughly 90% of the time.

**Example:** A system extracts twelve structured fields (name, date,
amount, etc.) from each of ten thousand invoices, assigning a separate
confidence score to each field. Instead of only spot-checking fields
flagged as low-confidence, a human reviewer also pulls a stratified random
sample of high-confidence extractions from each field type to check for a
new, systematic error the model's own confidence scoring doesn't yet
reflect; over time, the mapping from stated confidence to actual accuracy is
recalibrated against these labeled checks.

**Source:** Claude Certified Architect – Foundations Exam Guide, Domain 5,
Task 5.5 (exam-guide-only; not found in vendor documentation).

### Concept: Citations for information provenance
The Citations API feature lets a caller attach source documents (plain
text, PDF, or custom content blocks) to a request with `"citations":
{"enabled": true}`; the API chunks the documents into sentences and, when
Claude's response draws on them, returns citation objects pointing to the
exact supporting passage(s), including the source document and location.
Because `cited_text` doesn't count toward output tokens and citations are
guaranteed to be valid pointers into the supplied documents (unlike asking
the model to just quote sources itself in prose, per Module 5's grounding
techniques), this is the documented mechanism for preserving verifiable
provenance when a response synthesizes claims from one or more source
documents.

**Example:** A research assistant answers a question using three uploaded
reports; each sentence of its answer carries a citation back to the
specific report and passage it came from, so a reviewer can verify exactly
which source backs which claim.

**Source:** Context Management & Reliability research — Citations for
information provenance (key), https://platform.claude.com/docs/en/build-with-claude/citations

### Concept: Long-context document structuring for multi-source synthesis
Anthropic's long-context prompting guidance recommends specific structuring
for tasks that synthesize information from multiple long documents: place
documents near the top of the prompt (above instructions/query, consistent
with the "lost in the middle" mitigation from Lesson 6.1), wrap each in
`<document>` tags with `<source>` and other metadata subtags to disambiguate
provenance, and use a two-pass workflow where the model first extracts
relevant quotes into `<quotes>` tags before synthesizing a final answer
into a separate output tag — grounding the final answer in
explicitly-attributed extracted text rather than having the model
synthesize directly from an undifferentiated block of mixed sources. This
complements Citations (a first-class API feature, above) as a
prompting-level technique for provenance and quality when citations aren't
used or aren't sufficient by themselves.

**Example:** Given five vendor contracts to compare, the prompt wraps each
contract in its own `<document source="ContractA.pdf">` block, and
instructs the model to first pull the relevant liability clauses into
`<quotes>` tags (each tagged with its source contract) before writing the
comparison, so downstream readers can trace every stated fact back to a
specific contract.

**Source:** Context Management & Reliability research — Long-context
document structuring for multi-source synthesis (key), https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/long-context-tips ; https://www.anthropic.com/engineering/built-multi-agent-research-system
