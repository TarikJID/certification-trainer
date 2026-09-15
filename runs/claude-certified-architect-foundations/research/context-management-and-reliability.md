# Domain: Context Management & Reliability (15% of the exam)

Covers: managing conversation context across long interactions; designing escalation and ambiguity resolution patterns; implementing error propagation strategies across multi-agent systems; managing context in large codebase exploration; designing human review workflows with confidence calibration; and preserving information provenance in multi-source synthesis.

Note on sourcing: the official exam guide PDF could not be parsed as text by the available fetch tool (it returned raw compressed PDF stream data), so scoping below is based on the domain description supplied by the orchestrator plus the authoritative vendor documentation families named in the task (docs.claude.com / platform.claude.com, code.claude.com/docs, and anthropic.com/engineering). Where a specific sub-topic named in the domain description (e.g. "confidence calibration") does not have a dedicated, named treatment in Anthropic's own documentation, this is flagged explicitly rather than filled in from memory.

---

## Key Concepts

- Concept: Context window and context rot
  Type: key
  Definition: The context window is all the text (system prompt, every message including tool results/images/documents, tool definitions, and the model's own output) that a model can reference when generating a response — a bounded "working memory," distinct from the model's training data. As token count grows, model accuracy and recall degrade, a phenomenon Anthropic calls "context rot": because attention in a transformer creates pairwise relationships between tokens, a larger context stretches the model's effective attention thinner, and models have less training exposure to very long sequences. This means curating what is in context is as important as how much space is available. Context windows currently range up to 1M tokens depending on model; going over the limit triggers either a 400 "prompt is too long" error (if input alone exceeds the window) or, on newer models, a graceful `stop_reason: "model_context_window_exceeded"`.
  Example: A long customer-support chat accumulates dozens of tool calls and file attachments; even though the 1M-token window isn't full, the model starts missing details from early in the conversation because the "signal" is diluted by irrelevant accumulated content — this is context rot, not window overflow.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows ; https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- Concept: Context awareness (token budget injection)
  Type: key
  Definition: On models that support it (Claude Sonnet 5, Sonnet 4.6, Sonnet 4.5, Haiku 4.5), the API automatically injects a `<budget:token_budget>` tag into the system prompt showing the model its total context window, and after each tool call injects a `<system_warning>` tag reporting tokens used and remaining. This lets the model manage long-running, tool-heavy tasks against its actual remaining capacity instead of guessing. It requires no configuration — the tags are inserted automatically. Models that don't receive these tags can instead be given an explicit budget via the (beta) "task budgets" feature.
  Example: During a 40-tool-call agentic coding session, the model sees "Token usage: 150000/200000; 50000 remaining" after a tool result and decides to wrap up and summarize its work rather than opening ten more files.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows#context-awareness

- Concept: Server-side compaction
  Type: key
  Definition: Compaction is a beta API feature (`compact_20260112` strategy under `context_management.edits`) that automatically summarizes older conversation turns once input tokens cross a configured trigger (default 150,000, minimum 50,000), inserting a `compaction` content block containing the summary and instructing the API to drop everything before that block on subsequent requests. Configuration options include a custom `trigger`, `pause_after_compaction` (to let the caller inspect/edit the summary before continuing — useful for enforcing a total token budget across many compactions), and `instructions` (to replace the default summarization prompt, e.g. to prioritize preserving code and technical decisions). It is the primary mechanism recommended for long-running, multi-turn conversations and agentic sessions that would otherwise hit the context window limit.
  Example: An agent running for hours accumulates 300K tokens of tool output; at the configured 150K-token trigger, the API auto-generates a `<summary>` block preserving state, next steps, and learnings, and the client appends that block so the next request continues with a much smaller prompt.
  Source: https://platform.claude.com/docs/en/build-with-claude/compaction

- Concept: Context editing (tool result clearing and thinking block clearing)
  Type: key
  Definition: A more fine-grained, beta alternative/complement to compaction that lets a caller selectively clear specific content instead of summarizing everything. Two strategies exist: `clear_tool_uses_20250919` removes the oldest tool call/result pairs once a trigger (input tokens or tool-use count) is exceeded, keeping a configurable number of the most recent pairs, optionally excluding specific tools, and replacing cleared content with a placeholder; `clear_thinking_20251015` clears older extended-thinking blocks while optionally preserving a configurable number of recent ones (behavior differs by model class). Both require the `context-management-2025-06-27` beta header, are applied server-side before Claude sees the prompt, and leave the caller's own local conversation history untouched. The API reports exactly what was cleared via a `context_management.applied_edits` field.
  Example: An agentic research workflow with heavy web-search tool use keeps only the last 3 tool results in context via `clear_tool_uses_20250919`, dramatically cutting input-token cost on turn 50 without losing the model's ability to reason about its most recent findings.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-editing

- Concept: Prompt caching for long-running context
  Type: key
  Definition: Prompt caching lets a caller mark a prefix of a request (system prompt, tool definitions, long fixed context) with `cache_control` so the API can reuse the pre-computed state on a later call instead of reprocessing it, cutting input-token cost by up to ~90% and latency by up to ~85% on cache hits. Caches default to a 5-minute TTL (refreshed on each hit) or can be extended to 1 hour at 2x the write cost. The cache only helps if the breakpoint is on content that is byte-identical across requests, and it does not reduce what counts toward the context window — only what a cache hit costs. In multi-turn conversations the API supports an automatic "lookback" that finds the longest previously-cached prefix.
  Example: A coding agent with a large, static system prompt and tool schema caches that prefix once; every subsequent turn in the session reads that prefix from cache at 10% of the normal input price instead of repaying full price for it on every request.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-caching ; https://platform.claude.com/docs/en/build-with-claude/context-windows

- Concept: Long-running agent harnesses (multi-session context handoff)
  Type: key
  Definition: For tasks that span far more time or work than a single context window can hold (e.g. multi-day software projects), Anthropic documents a two-agent harness pattern: an initializer agent runs once to set up the environment, expand the prompt into a structured requirements/feature list, and write a boot script; a coding agent is then woken up repeatedly across many separate sessions, each one reading git history and a progress file to reconstruct state, making incremental progress on one feature, verifying it end-to-end, and leaving a clean handoff (progress notes, updated feature-pass/fail status, a commit) for the next session. This treats context reset as unavoidable for very long jobs — summarization/compaction is not enough by itself — so the harness periodically does a full session teardown and rebuild from a structured handoff artifact, analogous to onboarding a new engineer.
  Example: A project scoped to run over three days of Claude sessions has each new session start by running `pwd`, reading `claude-progress.txt` and the git log, picking the highest-priority failing feature from `feature_list.json`, and verifying the dev server still works before writing any new code.
  Source: https://anthropic.com/engineering/effective-harnesses-for-long-running-agents

- Concept: Context isolation via subagents for large codebase exploration
  Type: key
  Definition: Because reading many files to understand an unfamiliar or large codebase can flood the main conversation's context window, Claude Code and the Agent SDK support delegating exploration to subagents (including the built-in read-only `Explore` subagent, which runs on a fast/cheap model and reads excerpts rather than whole files) that run in their own, separate context window. Only the subagent's final, condensed summary returns to the parent; every intermediate file read, grep result, or tool call stays inside the subagent and never accumulates in the orchestrating conversation. This gives context isolation, enables parallel exploration of independent areas of a codebase, and lets a specialized subagent carry tailored instructions/tool restrictions without adding noise to the main agent's prompt.
  Example: A `research-assistant` subagent explores dozens of files across a monorepo to answer "how does the auth module handle token refresh?"; the parent conversation only receives a two-paragraph summary, not the dozens of files that were read to produce it.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents ; https://code.claude.com/docs/en/best-practices

- Concept: Error propagation across multi-agent systems
  Type: key
  Definition: When an agent delegates work to subagents (orchestrator-worker architecture), an error in a subagent does not automatically surface as normal output to the parent: specifically, an API error that ends a subagent early (e.g. a rate limit) is never delivered as that subagent's result — the parent must handle its absence explicitly. At larger scale, Anthropic's own production multi-agent research system documents that small prompt/config changes can cause disproportionate, cascading behavioral failures (e.g. an early version pathologically spawned 50 subagents for a simple query, or ran endless searches for information that didn't exist), and that because agents are stateful and run for extended periods, errors compound rather than resetting cleanly — the chosen mitigation was checkpoint-based resumption plus letting the model's own intelligence handle transient tool failures gracefully, backed by full production tracing (without inspecting conversation content) to diagnose failure patterns, and staged ("rainbow") deployments so in-flight long-running agents aren't disrupted by an update.
  Example: A lead research agent spawns five subagents to gather sources in parallel; one subagent hits a rate-limit error and terminates silently mid-task. The orchestrator must detect the missing/partial result (rather than treating silence as "no relevant sources found") and decide whether to retry, reassign, or proceed with reduced coverage.
  Source: https://code.claude.com/docs/en/agent-sdk/subagents#what-subagents-inherit ; https://www.anthropic.com/engineering/built-multi-agent-research-system

- Concept: Escalation patterns and human review workflows (permission modes)
  Type: key
  Definition: Claude Code's permission modes govern whether the model acts autonomously or pauses for a human. In Manual mode, the agent stops and asks the human before most file-writing, command-execution, or network-reaching actions. In auto mode (the default starting mode on paid plans), a separate classifier model reviews actions instead of a human, using a two-stage design: a fast, single-token "err on the side of blocking" filter, followed by a slower chain-of-thought review only for actions the filter flagged, judging the real-world effect of an action (including obfuscated/indirect operations) rather than just its surface text. Rather than a numeric confidence threshold, the shipped design escalates to a human based on outcome counts: 3 consecutive denials or 20 total denials in a session trigger human escalation; short of that, a denied action is returned to the agent with an instruction to find a safer path. Plan mode is a related, task-scoped escalation pattern: it forces a read-only "explore and propose a plan" phase and blocks all edits until the human explicitly approves the plan, at which point permissions are restored.
  Example: An autonomous agent's shell command is flagged by the classifier as risky (installs an unfamiliar network tool); the action is blocked and the agent is told to find another approach. After the agent accumulates 3 consecutive blocked attempts down different paths, the session escalates and asks the human directly instead of the classifier deciding.
  Source: https://code.claude.com/docs/en/permission-modes ; https://www.anthropic.com/engineering/claude-code-auto-mode

- Concept: Ambiguity resolution patterns
  Type: key
  Definition: Anthropic's own prompt-engineering guidance frames ambiguity handling as a design choice the system builder must make explicit, since the model will not reliably infer the right behavior on its own: for interactive applications, the most robust approach is to instruct the model to turn back to the user with a clarifying question rather than silently guessing when a task is under-specified. Anthropic's guidance further distinguishes "task is unclear/unanswerable" (ask at least one concise question resolving the most important uncertainty) from "some information is missing but the task is still answerable" (state an assumption explicitly and proceed) — avoiding a blanket "never ask questions" rule, which instead should be scoped to when a task is genuinely ambiguous versus merely under-specified.
  Example: A prompt asks an agent to "clean up the marketing files," which could mean deleting, reorganizing, or reformatting. Per this pattern, the agent should ask which of these is intended rather than picking one silently — versus a task like "summarize this document" where a missing detail (e.g. desired length) can be assumed and stated ("I'll produce a 3-paragraph summary unless you'd prefer otherwise") without stopping to ask.
  Source: https://claude.com/blog/best-practices-for-prompt-engineering ; https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Confidence expression and calibration in human-review workflows
  Type: key
  Definition: Anthropic's documented practice for reducing hallucination and improving reliability is to give the model explicit permission, in the prompt, to express uncertainty or decline to answer rather than guessing when evidence is insufficient — this is presented as a direct fix for "the model makes up information." This is distinct from, and should not be conflated with, an automated numeric "confidence score" used to gate human review: Anthropic's own production design for gating risky agent actions (the Claude Code auto-mode classifier) explicitly does not rely on a confidence threshold, instead using escalation limits based on how many actions were denied (3 consecutive / 20 total) to decide when to hand off to a human, because that proved more robust than trying to calibrate a single confidence number. Caveat: beyond these two documented mechanisms (prompted uncertainty expression, and denial-count-based escalation), Anthropic's public documentation does not appear to define a single named "confidence calibration" framework for human-review workflows; treat the term as an architectural pattern to be composed from these documented building blocks rather than a single first-party feature.
  Example: A document-QA agent is instructed "if the data is insufficient to draw conclusions, say so rather than speculating" — when it can't find a clear answer in the source material, it reports low confidence and routes the query to a human reviewer instead of fabricating a number.
  Source: https://claude.com/blog/best-practices-for-prompt-engineering ; https://www.anthropic.com/engineering/claude-code-auto-mode
  Unsourced aspect: a distinct, officially-named "confidence calibration" methodology for human review workflows was not found in Anthropic's vendor documentation; the definition above is composed from two separately documented mechanisms rather than one authoritative source describing "confidence calibration" as such.

- Concept: Citations for information provenance
  Type: key
  Definition: The Citations API feature lets a caller attach source documents (plain text, PDF, or custom content blocks) to a request with `"citations": {"enabled": true}`; the API chunks the documents into sentences and, when Claude's response draws on them, returns citation objects pointing to the exact supporting passage(s), including the source document and location. Because `cited_text` doesn't count toward output tokens and citations are guaranteed to be valid pointers into the supplied documents (unlike asking the model to just quote sources itself in prose), this is the documented mechanism for preserving verifiable provenance when a response synthesizes claims from one or more source documents.
  Example: A research assistant answers a question using three uploaded reports; each sentence of its answer carries a citation back to the specific report and passage it came from, so a reviewer can verify (or a UI can display) exactly which source backs which claim.
  Source: https://platform.claude.com/docs/en/build-with-claude/citations

- Concept: Long-context document structuring for multi-source synthesis
  Type: key
  Definition: Anthropic's long-context prompting guidance recommends specific structuring for tasks that synthesize information from multiple long documents: place documents near the top of the prompt (above instructions/query), wrap each in `<document>` tags with `<source>` and other metadata subtags to disambiguate provenance, and use a two-pass workflow where the model first extracts relevant quotes into `<quotes>` tags before synthesizing a final answer into a separate output tag — grounding the final answer in explicitly-attributed extracted text rather than having the model synthesize directly from an undifferentiated block of mixed sources. This complements Citations (which is a first-class API feature) as a prompting-level technique for provenance and quality when citations aren't used or aren't sufficient by themselves.
  Example: Given five vendor contracts to compare, the prompt wraps each contract in its own `<document source="ContractA.pdf">` block, and instructs the model to first pull the relevant liability clauses into `<quotes>` tags (each tagged with its source contract) before writing the comparison, so downstream readers can trace every stated fact back to a specific contract.
  Source: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/long-context-tips ; https://www.anthropic.com/engineering/built-multi-agent-research-system

---

## Prerequisite Concepts

- Concept: Tokens and token counting
  Type: prerequisite
  Definition: The base unit the API measures context, cost, and limits in. Every part of a request (system prompt, messages, tool results, tool definitions, images/documents) and the model's own output is measured in tokens; the token-counting endpoint lets a caller estimate usage before sending a request, including previewing the effect of context management edits.
  Example: Before sending a 40-document batch to be summarized, a developer calls the token-counting endpoint to confirm the request will fit under the model's context window, and again with `clear_tool_uses_20250919` configured to see the post-edit token count.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows

- Concept: Messages API turn structure (system prompt, user/assistant turns)
  Type: prerequisite
  Definition: The Messages API represents a conversation as an ordered list of user/assistant turns plus an optional system prompt; each turn's input phase includes all previous history plus the new message, and its output becomes input to the next turn. Understanding this progressive accumulation is necessary to understand why context grows, what compaction/context editing operate on, and where cache breakpoints can be placed.
  Example: A three-turn chat sends the system prompt plus turns 1–2 as input on turn 3; the assistant's turn-2 response is now part of turn 3's input.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows

- Concept: Tool use (tool-calling loop)
  Type: prerequisite
  Definition: The pattern where the model emits a `tool_use` block, the caller executes the tool and returns a `tool_result` block, and the model continues from there — potentially many times per turn. Tool results and tool definitions both count toward the context window, which is why tool-result clearing, subagent-based isolation, and error-propagation handling all center on this loop.
  Example: An agent calls a `web_search` tool, receives search results as a `tool_result`, and uses them to decide whether to call the tool again or produce a final answer.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows#the-context-window-with-thinking-and-tool-use

- Concept: Extended thinking (thinking blocks)
  Type: prerequisite
  Definition: A beta/optional mode where the model produces a visible `thinking` block of reasoning before its final answer. Thinking tokens are billed as output tokens and, depending on model, either persist in context across turns or are automatically stripped by the API. This distinction is the basis for the `clear_thinking_20251015` context-editing strategy and for context-window accounting during tool use.
  Example: With extended thinking enabled and a 4096-token thinking budget, a model reasons step-by-step about a hard math problem in a `thinking` block before producing its final text answer.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-windows#the-context-window-with-thinking

- Concept: Beta headers and API versioning
  Type: prerequisite
  Definition: Several context-management features (compaction, context editing) are shipped as opt-in betas gated behind an `anthropic-beta` header value (e.g. `compact-2026-01-12`, `context-management-2025-06-27`). Understanding that these are versioned, opt-in capabilities — not default behavior — is necessary before configuring them.
  Example: A request must include `betas=["context-management-2025-06-27"]` for the API to honor a `context_management.edits` configuration; omitting it means the edits are silently not applied.
  Source: https://platform.claude.com/docs/en/build-with-claude/context-editing

- Concept: Agentic loop ("tools in a loop")
  Type: prerequisite
  Definition: The general pattern underlying agents in the Claude ecosystem: an LLM autonomously and repeatedly selects and invokes tools, incorporates their results, and decides whether to continue or stop, without a human in the loop for every step. This is the foundation for subagents, long-running harnesses, and permission-mode escalation, all of which exist to manage or supervise this loop.
  Example: A coding agent loops: read file → propose edit → run tests → read failure output → propose another edit, continuing until tests pass or a stop condition is reached.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system

- Concept: Subagent / orchestrator-worker architecture basics
  Type: prerequisite
  Definition: The architectural pattern where a lead ("orchestrator") agent decomposes a task and delegates independent pieces to separate "worker" subagents, each with its own isolated context, then integrates their results. This is the base pattern that error-propagation strategies, provenance/citation passes, and large-codebase exploration subagents are all built on top of.
  Example: A lead research agent decomposes "compare pricing models across five competitors" into five parallel subagent tasks, one per competitor, then merges their findings into one report.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system ; https://code.claude.com/docs/en/agent-sdk/subagents

- Concept: Prompt engineering fundamentals (clarity and explicit instruction)
  Type: prerequisite
  Definition: The baseline practice of stating exactly what is wanted rather than assuming the model will infer intent — including specifying format, constraints, and what to do in edge cases. This baseline is the precondition for both ambiguity-resolution instructions (telling the model when to ask vs. assume) and uncertainty-expression instructions (telling the model when it may decline/hedge).
  Example: Instead of "summarize this," a prompt states "summarize this in 3 bullet points, and note if any point can't be verified from the source text."
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Retrieval / document-grounding basics
  Type: prerequisite
  Definition: The general practice of supplying external source documents in the prompt/context (rather than relying purely on model memory) so answers can be grounded in and traced back to specific text. This is the precondition for both the Citations API feature and long-context document-structuring techniques, both of which exist to make that grounding verifiable.
  Example: A support agent is given the actual product manual as a document in context rather than being asked to answer from general knowledge, so its answers can be checked against the manual.
  Source: https://platform.claude.com/docs/en/build-with-claude/citations

- Concept: Permission/approval model for agent actions
  Type: prerequisite
  Definition: The general concept that an autonomous agent's actions (file writes, shell commands, network calls) can be gated behind an approval step of some kind, before considering the specific escalation mechanics (manual approval, classifier-based auto mode, or plan-mode two-phase approval) built on top of it.
  Example: Before any permission mode is configured, the base fact a learner needs is simply that Claude Code *can* be made to pause before an action with a side effect, rather than always executing immediately.
  Source: https://code.claude.com/docs/en/permission-modes

---

## Sources Cited

- https://platform.claude.com/docs/en/build-with-claude/context-windows
- https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- https://platform.claude.com/docs/en/build-with-claude/compaction
- https://platform.claude.com/docs/en/build-with-claude/context-editing
- https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- https://anthropic.com/engineering/effective-harnesses-for-long-running-agents
- https://code.claude.com/docs/en/agent-sdk/subagents
- https://code.claude.com/docs/en/best-practices
- https://www.anthropic.com/engineering/built-multi-agent-research-system
- https://code.claude.com/docs/en/permission-modes
- https://www.anthropic.com/engineering/claude-code-auto-mode
- https://claude.com/blog/best-practices-for-prompt-engineering
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- https://platform.claude.com/docs/en/build-with-claude/citations
- https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/long-context-tips
