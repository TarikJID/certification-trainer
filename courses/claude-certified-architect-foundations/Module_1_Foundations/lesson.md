# Module 1: Foundations — Prompts, Tool Use, and the Context Window

This module has no single domain of its own on the exam. It gathers the
**prerequisite** concepts that the domain-researchers found underpinning
concepts in *every* one of the five CCAR-F domains — the Messages API shape,
how Claude calls tools, and what a context window is. Later modules build on
this module without re-explaining these basics.

---

## Lesson 1.1: The Messages API and Clear Prompting

### Concept: Messages API request structure
A Claude API request has a top-level `system` string (standing context, not
part of the back-and-forth) and a `messages` array where every entry has a
`role` of exactly `user` or `assistant` plus a `content` field. There is no
`system` role inside the `messages` array. This structure is what every other
API-level concept in this course sits on top of: tool-use blocks live inside
`assistant`/`user` message content, and structured-output configuration sits
alongside this same request.

**Example:** A request sets `system="You are a customer support agent. Reply
politely and concisely."` and `messages=[{"role": "user", "content": "What is
your return policy?"}]` — persona lives in `system`, the actual exchange lives
in `messages`.

**Source:** Prompt Engineering & Structured Output research — Messages API
request structure (prerequisite), https://platform.claude.com/docs/en/api/messages/create

### Concept: Messages API turn structure (conversation accumulation)
Each turn's input is all previous history plus the new message, and the
model's output becomes part of the input to the next turn — the conversation
only grows, never resets on its own within a session. Understanding this
progressive accumulation is necessary to understand why context grows, what
later mechanisms like compaction operate on, and where cache breakpoints can
be placed (Module 6).

**Example:** A three-turn chat sends the system prompt plus turns 1–2 as
input on turn 3; the assistant's turn-2 response is now part of turn 3's
input.

**Source:** Context Management & Reliability research — Messages API turn
structure (prerequisite), https://platform.claude.com/docs/en/build-with-claude/context-windows

### Concept: Being clear and direct
The foundational prompting principle: precise, explicit, unambiguous
instructions outperform vague or implicit ones. Anthropic frames this as
treating Claude "like a brilliant but new employee who lacks context on your
norms and workflows" — give contextual information, state exactly what you
want (including what the output should and should not contain), and provide
instructions as sequential numbered or bulleted steps. The documented "golden
rule": show the prompt to a colleague with minimal context and have them try
to follow it — if a human reader is confused, Claude likely will be too. This
principle underlies every more advanced technique this course covers
(multishot prompting, chain-of-thought, XML tag structuring, explicit review
criteria).

**Example:** A vague prompt "Write something about our new app" is rewritten
as "Write a 150-word marketing email for busy professionals announcing our
new productivity app. Output only the email body — no subject line, no
preamble, no explanation."

**Source:** Prompt Engineering & Structured Output research — Being clear and
direct (prerequisite), https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

### Concept: System Prompt and Role Prompting
The system prompt shapes how Claude behaves across a whole conversation or
agent run — role, capabilities, constraints, response style — set separately
from the per-turn `messages`. Giving Claude a role ("role prompting") is
described as the most powerful way to use system prompts: even one sentence
("You are a data scientist specializing in customer insight analysis for
Fortune 500 companies") measurably changes response character and quality.
Task-specific instructions are generally kept in the user turn, with role and
standing behavior in the system prompt. In the Agent SDK specifically, a
system prompt can come from a minimal SDK default, the `claude_code` preset
(the full CLI prompt, optionally extended), or a fully custom string — and
this is the mechanism by which each subagent later gets its own distinct
persona (Module 4).

**Example:** `system: "You are AcmeBot, the enterprise-grade AI assistant for
AcmeTechCo... maintain a professional, concise tone"` makes every reply in
the conversation consistently reflect that persona and its constraints.

**Source:** Merges two research entries — Agentic Architecture &
Orchestration's "System Prompt" (prerequisite),
https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts ; and
Prompt Engineering & Structured Output's "System prompts and role prompting"
(key), https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

---

## Lesson 1.2: Tool Use and the Agentic Loop

### Concept: Tool Use (Function Calling)
The foundational capability that lets Claude call functions ("tools")
defined by the developer (client tools, executed in the calling application)
or provided by Anthropic (server tools, executed on Anthropic's
infrastructure). The developer specifies what operations are available and
the shape of their inputs (an `input_schema`); Claude decides when and how to
call them based on the request and each tool's description, returning a
`tool_use` content block naming the tool and a JSON object of arguments; the
calling application (for client tools) executes the operation and returns a
`tool_result` block, after which Claude continues. The default `tool_choice`
of `auto` lets Claude decide per turn whether to call a tool at all. This
single request/execute/respond round trip is the primitive that every agent
loop in this course repeats turn after turn, and is a prerequisite for
understanding hooks, subagents, and structured outputs alike.

**Example:** A `get_weather` tool is registered with an `input_schema`
requiring a `location` string. Asked "What's the weather in San Francisco?",
Claude responds with `stop_reason: "tool_use"` and a `tool_use` block
`{"name": "get_weather", "input": {"location": "San Francisco, CA"}}`; the
application looks up the weather and returns it as a `tool_result`, after
which Claude produces the final text answer.

**Source:** This single concept is stated in five separate research entries
across four domains, all describing the same round trip: Agentic Architecture
& Orchestration (prerequisite), Prompt Engineering & Structured Output (key
and prerequisite), Tool Design & MCP Integration (prerequisite), Context
Management & Reliability (prerequisite). Canonical source:
https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

### Concept: JSON Schema fundamentals, and its use in tool inputs
JSON Schema is a vocabulary for annotating and validating the structure of
JSON documents — property names, data types, required properties, enumerated
values, and nested/object structures. It underlies both the `input_schema`
used to define tools and (as covered later) the `output_config.format`
schema used for structured API output. MCP's `inputSchema`/`outputSchema`
fields (Module 3) are the same JSON Schema vocabulary applied to MCP tool
definitions.

**Example:** `{"type": "object", "properties": {"location": {"type":
"string"}}, "required": ["location"]}` declares that a valid document must be
an object with a required string `location` property.

**Source:** Merges Prompt Engineering & Structured Output's "JSON Schema
fundamentals" (prerequisite), https://json-schema.org/ ; and Tool Design &
MCP Integration's "JSON Schema for tool inputs" (prerequisite),
https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

### Concept: Tool definition structure (name, description, input_schema)
A user-defined tool passed to the Claude API is an object with `name`
(matching `^[a-zA-Z0-9_-]{1,128}$`), `description` (a detailed plaintext
explanation of what the tool does, when it should be used, and how it
behaves), `input_schema` (a JSON Schema object defining expected
parameters), and an optional `input_examples` array of schema-valid example
inputs. This is what Claude reads to decide whether and how to call the
tool. Module 3 covers how to write *good* descriptions and how to structure
a whole tool set; this lesson only covers the basic shape.

**Example:** A `get_weather` tool is defined with `name: "get_weather"`, a
description explaining it returns current weather for a location, and an
`input_schema` requiring a `location` string and an optional `unit` enum of
`celsius`/`fahrenheit`.

**Source:** Tool Design & MCP Integration research — Tool definition
structure (key), https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

### Concept: The Agentic Loop ("tools in a loop")
The general pattern underlying every agent in the Claude ecosystem: the model
autonomously and repeatedly selects and invokes tools, incorporates their
results, and decides whether to continue calling tools or to stop and
respond in plain text — without a human approving every single step. This
general pattern is deepened twice later in the course: Module 2 covers how
Claude Code's own built-in tools (Bash, Read, Edit) fit this loop
interactively, and Module 4 covers the Agent SDK's fully formalized version
of the loop (turns, `max_turns`, `ResultMessage`).

**Example:** A coding agent loops: read file → propose edit → run tests →
read failure output → propose another edit, continuing until tests pass or a
stop condition is reached.

**Source:** Context Management & Reliability research — Agentic loop ("tools
in a loop") (prerequisite), https://www.anthropic.com/engineering/built-multi-agent-research-system

---

## Lesson 1.3: The Context Window, Tokens, and Extended Thinking

### Concept: The Context Window
The total amount of information available to the model during a session —
it does not reset between turns. It accumulates the system prompt, tool
definitions, conversation history, tool inputs, and tool outputs across
every turn. Content that stays the same across turns (system prompt, tool
definitions, project instructions) is automatically prompt-cached. When
accumulated context approaches its limit, the SDK/API has mechanisms to
manage it automatically or on request — covered in depth in Module 6.

**Example:** A long session that reads many large files and runs verbose
Bash commands can consume thousands of tokens per turn; once accumulated
context nears the model's limit, the SDK can emit a compaction event rather
than truncating silently.

**Source:** Agentic Architecture & Orchestration research — The Context
Window (prerequisite), https://code.claude.com/docs/en/agent-sdk/agent-loop

### Concept: Tokens and token counting
Tokens are the base unit the API measures context, cost, and limits in.
Every part of a request — system prompt, messages, tool results, tool
definitions, images/documents — and the model's own output is measured in
tokens. A token-counting endpoint lets a caller estimate usage before sending
a request, including previewing the effect of context-management edits
(Module 6).

**Example:** Before sending a 40-document batch to be summarized, a
developer calls the token-counting endpoint to confirm the request fits
under the model's context window.

**Source:** Context Management & Reliability research — Tokens and token
counting (prerequisite), https://platform.claude.com/docs/en/build-with-claude/context-windows

### Concept: Extended thinking (thinking blocks)
A beta/optional mode where the model produces a visible `thinking` block of
reasoning before its final answer. Thinking tokens are billed as output
tokens and, depending on model, either persist in context across turns or
are automatically stripped by the API. This distinction matters later for
context-editing strategies that specifically target thinking blocks (Module
6).

**Example:** With extended thinking enabled and a 4096-token thinking
budget, a model reasons step-by-step about a hard math problem in a
`thinking` block before producing its final text answer.

**Source:** Context Management & Reliability research — Extended thinking
(prerequisite), https://platform.claude.com/docs/en/build-with-claude/context-windows

### Concept: Beta headers and API versioning
Several advanced features covered later in this course (server-side
compaction, context editing) ship as opt-in betas gated behind an
`anthropic-beta` header value (e.g. `compact-2026-01-12`,
`context-management-2025-06-27`). These are versioned, opt-in capabilities —
not default behavior — and a request that omits the header silently does not
get the feature.

**Example:** A request must include `betas=["context-management-2025-06-27"]`
for the API to honor a `context_management.edits` configuration; omitting it
means the edits are silently not applied.

**Source:** Context Management & Reliability research — Beta headers and API
versioning (prerequisite), https://platform.claude.com/docs/en/build-with-claude/context-editing
