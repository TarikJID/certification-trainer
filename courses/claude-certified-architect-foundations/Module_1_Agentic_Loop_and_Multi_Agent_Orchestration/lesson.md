# Module 1 — The Agentic Loop and Multi-Agent Orchestration

Domain: Domain 1 — Agentic Architecture & Orchestration (27% of exam)
Covers Task Statements 1.1, 1.2, 1.3.

This module builds the foundation the rest of the course depends on: how a single
agentic loop actually runs, and how a coordinator agent orchestrates multiple
subagents on top of that loop. Every later module (tool design, Claude Code
workflows, structured output, context management) assumes you understand the
mechanics taught here.

---

## Lesson 1.1 — Designing and Implementing Agentic Loops

**Maps to:** Task Statement 1.1: Design and implement agentic loops for autonomous
task execution.

### Prerequisite concept: The Claude Messages API request/response cycle and `tool_use` blocks

The Claude API is stateless: each call to the Messages endpoint sends the full
conversation so far (system prompt, prior turns, tool definitions) and receives a
response with one or more content blocks. When Claude wants to use a tool, the
response includes a `tool_use` content block naming the tool and its input
arguments, alongside a top-level `stop_reason` field. This request/response
mechanic is the substrate the agentic loop is built on: the loop is a program that
keeps calling this endpoint, feeding tool outputs back in, until the model stops
requesting tools.

*Example:* A Messages API call with `messages=[{"role":"user","content":"What's the
weather in Boston?"}]` and a `get_weather` tool defined returns a response with
`stop_reason: "tool_use"` and a content block
`{"type":"tool_use","name":"get_weather","input":{"location":"Boston"}}`.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/build-a-tool-using-agent
(Teaches: 1.1-K1, 1.1-K2, 1.1-S1, 1.1-S2 — prerequisite)

### Concept: The Agentic Loop Lifecycle (`stop_reason`-driven control flow)

The agentic loop is the cycle that powers Claude Code and the Agent SDK: send a
request to Claude, let it respond with text, request tool calls, or both; execute
any requested tools; feed the results back in; and repeat. The loop's continuation
decision is driven entirely by `stop_reason`: `"tool_use"` means Claude wants to
call a tool and the loop must execute it and continue; `"end_turn"` means Claude
produced a final response with no further tool calls and the loop terminates.

*Example:* For "Fix the failing tests in auth.ts," turn 1 has Claude call
`Bash(npm test)` (`stop_reason: "tool_use"`); turn 2 it calls `Read`; turn 3 it
calls `Edit` then re-runs tests; the final turn produces text only with
`stop_reason: "end_turn"`, and the loop stops.

*Source:* exam-guide.txt (Task 1.1 Knowledge bullet); corroborated by
https://code.claude.com/docs/en/agent-sdk/agent-loop
(Teaches: 1.1-K1, 1.1-S1)

### Concept: Appending Tool Results to Conversation History

After each tool executes, its result is added to the conversation as a new message
(a `tool_result` block tied to the originating `tool_use` block's ID) and included
in the next request sent to Claude. Nothing about a tool's outcome is available to
Claude for its next decision unless it has been explicitly appended to the history
sent on the following API call.

*Example:* After `Bash(npm test)` returns "3 failing tests," that output becomes
part of the history in the next call to Claude, letting Claude's next turn reason
about which file to read based on the specific failures reported.

*Source:* https://code.claude.com/docs/en/agent-sdk/agent-loop
(Teaches: 1.1-K2, 1.1-S2)

### Concept: Model-Driven Decision-Making vs Pre-configured Decision Trees

An agentic loop lets Claude decide, turn by turn, which tool to call next based on
everything in context — model-driven (agent) behavior, distinct from a hard-coded
workflow where the sequence of steps is authored in advance and the LLM only fills
in sub-steps. Anthropic's own distinction: workflows orchestrate LLMs and tools
through predefined code paths; agents let the LLM dynamically direct its own
process and tool usage.

*Example:* A hard-coded decision tree always calls `get_customer` then
`process_refund` on any refund request. An agentic loop instead lets Claude decide
— based on the conversation — whether to call `lookup_order` first, ask a
clarifying question, or escalate.

*Source:* https://www.anthropic.com/research/building-effective-agents
(Teaches: 1.1-K3)

### Concept: Agentic Loop Termination Anti-Patterns

Three implementation mistakes undermine a correctly designed loop: (1) parsing the
assistant's natural-language text for phrases like "I'm done" to decide whether to
stop — unreliable because free text is not a structured signal; (2) treating an
arbitrary iteration cap as the *primary* stopping mechanism rather than a safety
backstop; and (3) checking for the presence of assistant text content as a proxy
for completion, which fails because Claude can emit explanatory text alongside a
`tool_use` block in the same turn. The correct mechanism is the structured
`stop_reason` field.

*Example:* An implementation that stops as soon as a response contains any text
would terminate prematurely on a turn where Claude writes "Let me check the logs
first" while also requesting a `Read` call in the same response — `stop_reason` is
still `"tool_use"`, so the loop should have continued.

*Source:* exam-guide.txt (Task 1.1 Skills bullet); corroborated by
https://code.claude.com/docs/en/agent-sdk/agent-loop
(Teaches: 1.1-S3)

**Quiz:** see `quiz.md`, Lesson 1.1.

---

## Lesson 1.2 — Orchestrating Multi-Agent Systems with Coordinator-Subagent Patterns

**Maps to:** Task Statement 1.2: Orchestrate multi-agent systems with
coordinator-subagent patterns.

### Concept: Hub-and-Spoke Coordinator Architecture

In a hub-and-spoke (orchestrator-worker) design, one coordinator ("lead") agent
sits at the hub and every subagent is a spoke that communicates only with the
coordinator — subagents never talk to each other directly. All inter-subagent
communication, error handling, and information routing passes through the
coordinator, giving it full observability and consistent error handling at the
cost of becoming a bottleneck if not designed to delegate effectively.

*Example:* Anthropic's Research feature uses a lead agent that spawns web-search
and document-analysis subagents; those subagents never communicate with each
other — each reports findings back only to the lead agent.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.2-K1, 1.2-S4)

### Concept: Subagent Context Isolation

A subagent's context window starts fresh: it does not automatically inherit the
coordinator's conversation history, prior tool calls, or tool results. The only
channel from coordinator to subagent is the prompt string passed at invocation
(plus its own system prompt/`AgentDefinition` and, unless disabled, project
`CLAUDE.md`). This is deliberate — it lets subagents explore verbose content
without any of that bulk accumulating in the coordinator's own context.

*Example:* A `research-assistant` subagent can read 40 source documents while
investigating a topic; none of those documents' raw content enters the
coordinator's context — only the subagent's final summary does.

*Source:* https://code.claude.com/docs/en/agent-sdk/subagents
(Teaches: 1.2-K2)

### Concept: Coordinator Responsibilities — Decomposition, Delegation, Aggregation, Dynamic Subagent Selection

The coordinator handles the whole lifecycle of a request: decomposition into
subtasks, delegation to the right subagent(s), and aggregation of subagent outputs
into a coherent final result. A well-designed coordinator also analyzes each
query's complexity and decides which subagents are actually needed, rather than
always routing through the full pipeline.

*Example:* A coordinator receiving "What is the capital of France?" invokes no
subagents and answers directly; a coordinator receiving "Compare the semiconductor
supply chains of three countries" spawns three or more subagents.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.2-K3, 1.2-S1)

### Concept: Risks of Overly Narrow Task Decomposition

When a coordinator gives subagents insufficiently scoped task boundaries, the
result is duplicated work, coverage gaps, or subagents that fail to find necessary
information — because each subagent, operating in isolation, cannot see what
others are doing. Vague assignments are the direct cause; the fix is giving every
subagent an explicit objective, output format, tool/source guidance, and task
boundaries.

*Example:* A lead agent split "semiconductor shortage" into three vaguely-scoped
subagents; one investigated the 2021 automotive chip crisis while two others
redundantly investigated 2025 supply chains, leaving other angles uncovered.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.2-K4)

### Concept: Partitioning Research Scope Across Subagents to Minimize Duplication

To avoid the duplication risk above, a coordinator assigns each subagent a
distinct, non-overlapping slice of the problem — a specific subtopic, source type,
or time period — rather than letting multiple subagents independently pursue the
same broad question. Effective partitioning requires an explicit objective,
expected output format, tool/source guidance, and clear scope boundaries.

*Example:* Instead of three subagents each told to "research the semiconductor
shortage," the coordinator assigns one to "2021 automotive chip crisis root
causes," one to "2024-2026 supply chain diversification," and one to "policy
responses (CHIPS Act and equivalents)."

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.2-S2)

### Concept: Iterative Refinement Loop (Coordinator Re-delegation)

After subagents report back and a synthesis step produces a draft, the coordinator
evaluates that output for gaps — and, if coverage is insufficient, re-delegates:
spawning additional search/analysis subagents with targeted queries, then
re-invoking synthesis. This repeats until the coordinator judges coverage
sufficient.

*Example:* A first synthesis pass on "AI regulation trends" lacks EU-specific
detail; the coordinator spawns a new subagent scoped to "EU AI Act provisions" and
re-runs synthesis once it reports back.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.2-S3)

**Quiz:** see `quiz.md`, Lesson 1.2.

---

## Lesson 1.3 — Configuring Subagent Invocation, Context Passing, and Spawning

**Maps to:** Task Statement 1.3: Configure subagent invocation, context passing,
and spawning.

### Concept: The Task/Agent Tool and the `allowedTools` Requirement

Subagents are spawned through a dedicated tool — named `Task` in the exam guide's
terminology and the `system:init` tools list, surfaced as `"Agent"` in `tool_use`
blocks on current SDK versions (older versions emit `"Task"` in both places; code
detecting subagent invocation should match either name). For a coordinator to
invoke subagents at all, this tool must be present in its `allowedTools`
configuration.

*Example:* `ClaudeAgentOptions(allowed_tools=["Read", "Grep", "Glob", "Agent"],
agents={...})` — omitting `"Agent"` from that list means Claude can never invoke
any defined subagent even though they exist in `agents`.

*Source:* https://code.claude.com/docs/en/agent-sdk/subagents; naming corroborated
by exam-guide.txt
(Teaches: 1.3-K1)

### Concept: Explicit Context Passing to Subagents

Because a subagent's context starts fresh (Subagent Context Isolation, Lesson
1.2), any information a subagent needs — prior findings, file paths, error
messages, decisions already made — must be written directly into the prompt
string passed to the `Task`/`Agent` call. There is no automatic inheritance and no
shared memory between separate subagent invocations.

*Example:* When invoking a synthesis subagent, the coordinator's prompt includes
the full text of the web-search subagent's findings and the document-analysis
subagent's findings pasted directly in — not a pointer like "see the earlier
results."

*Source:* https://code.claude.com/docs/en/agent-sdk/subagents
(Teaches: 1.3-K2, 1.3-S1)

### Concept: `AgentDefinition` Configuration

Subagents defined programmatically are configured through an `AgentDefinition` (or
its filesystem equivalent, a markdown file in `.claude/agents/`) with, at minimum,
a `description` (used to decide when to invoke this subagent automatically) and a
`prompt` (the subagent's own system prompt). Optional fields include `tools`
(restricting which tools it can use), `model`, and others (`disallowedTools`,
`skills`, `maxTurns`, `background`).

*Example:* `AgentDefinition(description="Expert code review specialist...",
prompt="You are a code review specialist...", tools=["Read","Grep","Glob"],
model="sonnet")` defines a read-only reviewer subagent Claude invokes
automatically when a task matches its description.

*Source:* https://code.claude.com/docs/en/agent-sdk/subagents
(Teaches: 1.3-K3)

### Concept: Session Forking for Divergent Exploration (`fork_session`)

Forking creates a new, independent session that starts as a copy of an existing
session's full history from that shared baseline; the original session's history
is left unchanged, so both can be resumed and extended separately. This is the
mechanism for exploring divergent approaches from a common analysis point.

*Example:* After a session has fully analyzed an authentication module, forking it
lets one branch explore "how would this look with JWT?" while the original is
separately resumed to explore "how would this look with OAuth2?"

*Source:* https://code.claude.com/docs/en/agent-sdk/sessions
(Teaches: 1.3-K4 — full skill coverage continues in Lesson 1.7)

### Concept: Structured Data Formats Separating Content from Metadata

When subagents pass findings up to the coordinator or across to another subagent,
mixing narrative content and attribution metadata (source URL, document name,
page number) into a single unstructured blob makes attribution easy to lose during
later summarization. The fix: require subagents to output findings as a claim plus
an evidence excerpt plus a separate source field, so content and metadata travel
together but distinctly.

*Example:* Anthropic's research system routes findings through a dedicated
CitationAgent that pairs the research report and documents to identify the source
location for every claim — possible only because each finding was captured with
claim and source as separate, structured fields.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.3-S2)

### Concept: Parallel Subagent Spawning in a Single Turn

A coordinator can spawn multiple subagents concurrently by emitting several
`Task`/`Agent` calls within a single response, rather than issuing one call,
waiting, then issuing the next. Independent subtasks running in parallel finish in
the time of the slowest one rather than the sum of all. Anthropic's research
system spawns 3-5 subagents in parallel and reports up to 90% research-time
reduction for complex queries versus serial execution.

*Example:* A coordinator investigating "compare React, Vue, and Svelte adoption
trends" emits three `Task` calls in one turn — one per framework.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system;
corroborated by https://code.claude.com/docs/en/agent-sdk/agent-loop
(Teaches: 1.3-S3)

### Concept: Goal-Oriented Coordinator Prompts (vs Step-by-Step Procedural Instructions)

Coordinator prompts for subagents should specify what a good outcome looks like —
an objective, an output format, and quality criteria — rather than a rigid,
literal sequence of steps. Declarative instructions let a subagent adapt to what
it discovers; procedural instructions break as soon as reality diverges. The
contract still needs to be concrete: an objective, output format, tool/source
guidance, and clear task boundaries.

*Example:* "Investigate whether Company X has faced regulatory action in the past
3 years; report as {date, regulator, outcome, source} entries; only use primary
regulatory filings and major news outlets" is goal-oriented. "First search for
'Company X lawsuit', then open the third result..." is procedural and brittle.

*Source:* https://www.anthropic.com/engineering/multi-agent-research-system
(Teaches: 1.3-S4)

**Quiz:** see `quiz.md`, Lesson 1.3.

---

## Module 1 summary of bullets taught

Lesson 1.1: 1.1-K1, 1.1-K2, 1.1-K3, 1.1-S1, 1.1-S2, 1.1-S3
Lesson 1.2: 1.2-K1, 1.2-K2, 1.2-K3, 1.2-K4, 1.2-S1, 1.2-S2, 1.2-S3, 1.2-S4
Lesson 1.3: 1.3-K1, 1.3-K2, 1.3-K3, 1.3-K4, 1.3-S1, 1.3-S2, 1.3-S3, 1.3-S4
