# Domain 5 — Context Management & Reliability: Research

Bullets received: 53 (24 Knowledge, 29 Skills)
Bullets covered: 53
Concepts: 46 (39 key, 7 prerequisite)

Coverage note: several concepts teach more than one bullet ID where the underlying
technique is shared (e.g., the same escalate-immediately-on-request principle drives
5.2-K2, 5.2-S2 and 5.2-S3). Every one of the 53 bullet IDs appears in at least one
`Teaches:` line below; see the coverage index at the end of this file for the
bullet-by-bullet cross-check.

---

## Task Statement 5.1 — Manage conversation context to preserve critical information across long interactions

- Concept: Context rot and the "lost in the middle" effect
  Type: key
  Teaches: 5.1-K2
  Definition: As the number of tokens in a model's context window grows, its ability to accurately recall and use any given piece of information from that context decreases — a phenomenon Anthropic calls "context rot." One manifestation is positional: models are most reliable at using information near the beginning and end of a long input, and least reliable at using information buried in the middle, because transformer attention must model n² pairwise relationships across n tokens and every added token dilutes the "attention budget" available for any other token. For an agent, this means a finding correctly retrieved by a subagent early in a session can be effectively lost if it sits deep in the middle of an eventual synthesis input.
  Example: A research agent aggregates ten subagent reports into one 50,000-token context. A critical caveat located in report #5 (out of 10) is statistically much more likely to be dropped from the final answer than the same caveat in report #1 or report #10, even though nothing marked it as less important.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- Concept: Progressive summarization risk during compaction
  Type: key
  Teaches: 5.1-K1
  Definition: Compaction — summarizing a conversation nearing its context limit and reinitializing a new context window with that summary — trades completeness for space. The central risk is *what gets discarded*: a summarizer optimizing for brevity will preferentially condense precise, low-frequency tokens (exact numbers, percentages, dates, and specific customer-stated expectations) into vague paraphrases, because those details look individually unimportant even though they are exactly what a downstream turn needs verbatim. An overly aggressive compaction strategy risks losing critical nuance from a conversation.
  Example: A support conversation where the customer said "I need this resolved by Friday the 14th, and I was quoted $47.50 for the replacement" gets compacted into "Customer requested timely resolution and discussed pricing" — the date and dollar figure a later reply depends on are gone.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (compaction section); bullet wording from runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.1-K1)

- Concept: Tool-result context bloat
  Type: key
  Teaches: 5.1-K3
  Definition: Tool outputs returned into an agent's context accumulate regardless of how much of that output the agent actually needs, and because they are usually full raw payloads (API responses, database rows, file contents), they consume tokens out of proportion to their relevance to the task. A 40-field order-lookup response where only 5 fields matter to a return request still costs the context window for all 40 fields on every subsequent turn that keeps that tool result in history, unless it is trimmed or cleared.
  Example: An order-lookup tool returns shipping address, full order history, loyalty tier, internal SKU codes, and warehouse routing metadata for a customer asking only "where's my refund" — the refund-relevant fields (order ID, amount, status) are a small fraction of the tokens the raw tool result consumes.
  Source: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (tool-result clearing); https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- Concept: Full conversation history and the stateless Messages API
  Type: key
  Teaches: 5.1-K4
  Definition: The Claude Messages API is stateless: the server does not retain prior turns between requests. To maintain conversational coherence, every request must include the complete message history the application wants Claude to be aware of — omitting earlier turns causes Claude to lose track of what was already said, decided, or collected, even mid-topic. This is why context management (trimming, summarizing, restructuring) has to happen on the client side rather than being handled implicitly by the API.
  Example: A multi-turn insurance quote flow where Claude has already collected make, model, and year must re-send those facts (or their summary) in every subsequent request; if the year is dropped from the resent history, Claude will ask for it again or infer a wrong value.
  Source: https://platform.claude.com/docs/en/build-with-claude/working-with-messages

- Concept: Persistent "case facts" block outside summarized history
  Type: key
  Teaches: 5.1-S1
  Definition: A technique for defending transactional facts (amounts, dates, order numbers, statuses) against progressive-summarization loss: extract them into a small, structured block (e.g., a short JSON or key-value section) that is included verbatim in every prompt, kept separate from — and never subject to — the compaction/summarization pipeline applied to the rest of the conversation history. Because the block is regenerated or carried forward explicitly rather than summarized, its contents cannot be vaguened away.
  Example: A support agent prompt includes `<case_facts>{"order_id": "A-88213", "refund_amount": "$47.50", "promised_by": "2026-09-19"}</case_facts>` on every turn, independent of whatever summary of the chit-chat portion of the conversation is also included.
  Source: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (memory pattern for persisting knowledge outside the summarized transcript); bullet wording from context-management-reliability.md (5.1-S1)

- Concept: Structured multi-issue context layering
  Type: key
  Teaches: 5.1-S2
  Definition: Extension of the case-facts pattern to sessions covering several distinct issues: rather than one flat fact block, each issue's structured data (order IDs, amounts, statuses relevant to that issue) is persisted in its own layer, so that resolving or updating one issue's facts doesn't require re-parsing a mixed summary and facts from one issue don't bleed into or get conflated with another's.
  Example: A single chat session where a customer asks about a late delivery (issue A: order #1001, status "in transit") and separately a billing discrepancy (issue B: invoice #55, amount $12 overcharge) — each issue keeps its own structured fact block so a reply about the delivery never accidentally cites the billing amount.
  Source: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (memory as persistent, addressable knowledge); bullet wording from context-management-reliability.md (5.1-S2)

- Concept: Tool-result clearing / trimming to relevant fields
  Type: key
  Teaches: 5.1-S3
  Definition: A context-management primitive that removes or replaces the bulk of a tool's raw output in the message history once it is no longer needed verbatim, either by trimming the output at insertion time to only the fields relevant to the task, or by later replacing old `tool_result` content with a placeholder while the surrounding `tool_use` record is preserved. This directly counters tool-result context bloat (5.1-K3) and is described as the cheapest form of compaction because it requires no additional model inference.
  Example: Before inserting an order-lookup result into context, the application strips it down to `{"order_id", "status", "amount", "return_eligible"}` and discards the other 35+ fields the API returned; or, several turns later, the API's `clear_tool_uses_20250919` context-management edit replaces the content of older `tool_result` blocks with `"[cleared to save context]"` while keeping the most recent N results intact.
  Source: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (tool-result clearing, `clear_tool_uses_20250919`)

- Concept: Positional structuring to counter position effects
  Type: key
  Teaches: 5.1-S4
  Definition: Because models are more reliable at the start and end of long inputs (see context rot / lost-in-the-middle), aggregated or synthesized inputs should place a short summary of key findings at the very beginning, and organize the detailed supporting material underneath with explicit section headers so the model can navigate to specific content deliberately rather than relying on serial attention across an undifferentiated block. Anthropic's own long-context guidance additionally recommends placing the query/instructions after the long content rather than before it, since queries placed at the end can measurably improve response quality on complex, multi-document inputs.
  Example: A synthesis prompt opens with `<key_findings>` summarizing the three most important conclusions in two sentences each, followed by `<detailed_results>` containing each subagent's full report under its own `<section title="...">` heading, with the actual analysis instruction placed after all of that content.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices (Long context prompting); https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- Concept: Subagent output metadata for downstream synthesis
  Type: key
  Teaches: 5.1-S5
  Definition: Requiring every subagent's structured output to carry metadata alongside its findings — dates, source locations, and methodological context (e.g., how a figure was derived, what population it covers) — so that a downstream synthesis agent, which typically only sees the subagent's condensed output and not the subagent's own exploration process, has enough context to interpret, compare, and correctly attribute those findings rather than treating them as free-floating facts.
  Example: A research subagent's output includes not just "median tenure: 3.2 years" but also `{"source": "2026 internal HR report", "collected": "2026-Q2", "method": "survey of active employees"}`, so the lead agent synthesizing several subagents' numbers can tell this figure apart from a differently-sourced one from a public salary survey.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system (structured findings and citation-agent pattern)

- Concept: Structured data over verbose reasoning for context-constrained downstream agents
  Type: key
  Teaches: 5.1-S6
  Definition: When an upstream agent's full output (raw content plus its chain of reasoning) would overwhelm a downstream agent's available context budget, the upstream agent should be modified to return a compact, structured payload instead — key facts, citations, and relevance scores — rather than verbose prose and reasoning chains. This mirrors the sub-agent architecture pattern where a subagent may do extensive exploratory work internally but returns only a condensed, distilled summary (often 1,000–2,000 tokens) to whichever agent consumes it.
  Example: Instead of a research subagent returning its full 8,000-token exploration transcript to a downstream report-writer agent with a small context budget, it returns `{"facts": [...], "citations": [...], "relevance": 0.9}` — a fraction of the size, with the same decision-relevant content.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (sub-agent architectures: "returns only a condensed, distilled summary of its work")

- Concept: Attention budget (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for context rot / lost-in-the-middle (5.1-K2)
  Definition: Attention in a transformer model is a finite resource: every additional token in the context window depletes the "budget" of attention available to relate any other pair of tokens, because self-attention scales as n² pairwise relationships for n tokens. This is the mechanical reason context windows have practical, not just nominal, limits — usable accuracy degrades well before the token limit is technically reached.
  Example: Doubling the number of tokens in a prompt does not just cost twice the compute; it also spreads the model's capacity to relate any single token to the rest of the input more thinly, which is why a 150K-token prompt is qualitatively harder for the model to use precisely than a 15K-token one, independent of the topic.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- Concept: Messages API request/response structure (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for full conversation history (5.1-K4) and tool-result clearing (5.1-S3)
  Definition: Each call to the Messages API sends a `messages` array of alternating `user` and `assistant` turns, where content is composed of typed blocks (`text`, `tool_use`, `tool_result`, etc.); the API has no memory of previous calls, so the array sent on a given request is the model's entire view of the conversation for that request. Understanding this structure is a precondition for reasoning about what "trimming," "clearing," or "summarizing" the history actually operates on.
  Example: A `tool_result` block must immediately follow the `tool_use` block it answers within the message array; the application, not the API, is responsible for assembling and, where needed, editing that array between calls.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls

---

## Task Statement 5.2 — Design effective escalation and ambiguity resolution patterns

- Concept: Escalation trigger taxonomy
  Type: key
  Teaches: 5.2-K1
  Definition: A well-designed agent escalates to a human under three distinct conditions, which should be evaluated independently rather than collapsed into one vague "if it's hard, escalate" rule: (1) the customer explicitly requests a human agent; (2) the request falls into a policy exception or a gap the agent's instructions don't address, not merely a case that is procedurally complex; and (3) the agent is unable to make meaningful progress (e.g., a tool keeps failing, or required information cannot be obtained). Complexity alone, in the absence of one of these three conditions, is not by itself a sufficient escalation trigger.
  Example: A refund request that requires five tool calls but resolves cleanly under existing policy should not escalate just because it took several steps; a refund request for a scenario the policy document never mentions should escalate even if it would otherwise be a single tool call.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.2-K1, exam guide bullet wording); supporting metric definition at https://platform.claude.com/docs/en/about-claude/use-case-guides/customer-support-chat ("Escalation efficiency")

- Concept: Immediate escalation on explicit request vs. offering resolution first
  Type: key
  Teaches: 5.2-K2, 5.2-S2, 5.2-S3
  Definition: Two different customer situations call for two different agent behaviors, and conflating them produces either an agent that escalates too eagerly or one that stalls a customer who has already asked for a human. When a customer explicitly and unambiguously demands a human agent, the agent should honor that immediately without first attempting its own investigation or resolution — investigating first after an explicit request reads as the agent overriding the customer's stated preference. When the issue itself is straightforward and within the agent's capability but the customer is frustrated, the better pattern is to acknowledge the frustration, offer to resolve it, and escalate only if the customer reiterates that they still want a human.
  Example: "I want to talk to a person" → escalate now, no further investigation. "This is so annoying, my order is three days late" (no explicit request for a human) → acknowledge the frustration, offer to check the order and resolve it, and escalate only if the customer says "no, I still want a person" after that offer.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.2-K2, 5.2-S2, 5.2-S3, exam guide bullet wording)

- Concept: Unreliable complexity proxies — sentiment and self-reported confidence
  Type: key
  Teaches: 5.2-K3
  Definition: Two signals that look like reasonable escalation triggers but are not reliable proxies for actual case complexity: (1) sentiment analysis, because a customer can be highly frustrated about a trivially resolvable issue and calm about a genuinely complex one, so routing on tone alone both over-escalates easy cases and under-escalates hard ones; and (2) a model's own self-reported confidence score, because a model can be confidently wrong (over-confident on a case it is actually mishandling) or under-confident on a case it is in fact handling correctly, making self-assessment an unreliable substitute for grounding escalation in the concrete triggers in the escalation trigger taxonomy.
  Example: A politely worded message ("Would you mind checking why my refund policy exception wasn't honored per our verbal agreement with your manager last month?") is calm in tone but describes exactly the kind of policy-gap case that should escalate — sentiment analysis alone would miss it.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.2-K3, exam guide bullet wording)

- Concept: Clarification over heuristic disambiguation for multiple matches
  Type: key
  Teaches: 5.2-K4, 5.2-S5
  Definition: When a tool call intended to identify a specific customer or record returns multiple matches, the correct pattern is to ask the customer for an additional identifier (e.g., order number, email, ZIP code) to disambiguate rather than picking one match using a heuristic (most recent, alphabetically first, etc.). Heuristic selection risks silently acting on the wrong customer's data — a correctness and safety failure that clarification avoids entirely at the cost of one extra turn.
  Example: A lookup by name returns two "John Smith" accounts; instead of picking the first result, the agent asks "I found two accounts under that name — could you confirm the last four digits of the phone number on file?"
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.2-K4, 5.2-S5, exam guide bullet wording); analogous principle in https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview ("When required parameters are missing")

- Concept: Few-shot escalation criteria in the system prompt
  Type: key
  Teaches: 5.2-S1
  Definition: Rather than describing escalation policy only in abstract prose, adding concrete few-shot examples to the system prompt that demonstrate specific escalate-vs-resolve-autonomously decisions is one of the most reliable ways to steer a model's behavior consistently. Good examples for this purpose should be relevant (mirror the actual scenarios the agent will see), diverse (cover edge cases so the model doesn't overfit to one pattern), and clearly delimited (e.g., wrapped in `<example>` tags) so the model can distinguish them from ordinary instructions.
  Example: A system prompt includes `<example>` blocks showing (a) an explicit human request handled by immediate escalation, (b) a straightforward late-delivery question resolved autonomously, and (c) a competitor price-match request escalated because policy is silent on it — giving the model concrete precedent for all three trigger types rather than a single abstract rule.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices (Use examples effectively)

- Concept: Escalation on policy silence or ambiguity
  Type: key
  Teaches: 5.2-S4
  Definition: A specific application of the escalation trigger taxonomy's "policy exception/gap" condition: when a customer's request falls into territory the written policy simply doesn't address — as opposed to territory the policy explicitly covers and denies — the agent should escalate rather than improvise an answer, because silence is not permission and an incorrect improvised answer can create liability or an inconsistent precedent.
  Example: A store's policy explicitly covers price-matching against its own past sales but says nothing about matching a competitor's price. A customer asking for a competitor price match should be escalated, not denied or approved by the agent's own inference from the adjacent, but not identical, policy language.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.2-S4, exam guide bullet wording)

- Concept: System-prompt role and behavior framing (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for few-shot escalation criteria (5.2-S1)
  Definition: Setting an explicit role and behavioral frame in the system prompt ("You are a support agent for X, and your job includes knowing when to hand off to a human") establishes the context within which few-shot examples and escalation rules are interpreted; even a single clear sentence of role-framing measurably focuses a model's tone and behavior for a given use case before any examples are added.
  Example: `IDENTITY = "You are Eva, a support assistant for Acme... you can resolve straightforward issues yourself and know when to bring in a human."` sets the frame that the few-shot escalation examples then make concrete.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices (Give Claude a role)

---

## Task Statement 5.3 — Implement error propagation strategies across multi-agent systems

- Concept: Structured error context for coordinator recovery
  Type: key
  Teaches: 5.3-K1, 5.3-S1
  Definition: When a subagent or tool call fails, returning a structured error payload — the failure type, what was attempted, any partial results already obtained, and possible alternative approaches — gives the coordinating agent enough information to make an intelligent recovery decision (retry, try an alternative source, proceed with partial data, or surface the gap to the user) instead of reacting to a bare failure signal. This mirrors the API-level `tool_result` mechanism, where `is_error: true` is paired with a descriptive `content` string rather than a generic failure flag, specifically so the calling model can decide what to do next instead of guessing.
  Example: A search subagent that times out returns `{"failure_type": "timeout", "attempted": "search internal KB for 'refund policy EU'", "partial_results": ["found 2 of estimated 5 relevant docs"], "alternatives": ["retry with narrower query", "fall back to public help center"]}` rather than just `"search failed"`.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls (Handling errors with is_error); https://www.anthropic.com/engineering/built-multi-agent-research-system (stateful error compounding, deterministic safeguards)

- Concept: Distinguishing access failures from valid empty results
  Type: key
  Teaches: 5.3-K2, 5.3-S2
  Definition: A query that fails to execute (timeout, service unavailable, authentication error) and a query that executes successfully but legitimately finds nothing are fundamentally different outcomes that must be reported differently. An access failure is something the coordinator may want to retry or route around; a valid empty result is a completed, trustworthy answer that happens to be "no matches" and retrying it will not change the outcome. Collapsing both into the same "no results" response removes the information the coordinator needs to choose the right next action.
  Example: A customer database lookup that times out after 30 seconds should report `{"status": "access_failure", "reason": "timeout"}`; the same lookup that completes and finds zero matching accounts should report `{"status": "success", "result_count": 0}` — both look like "nothing came back," but only one is worth retrying.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.3-K2, exam guide bullet wording); https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls (is_error semantics)

- Concept: Generic error statuses as a context-hiding anti-pattern
  Type: key
  Teaches: 5.3-K3
  Definition: An error report like "search unavailable" is technically informative but practically useless to a coordinator, because it discards exactly the details (what failed, what was tried, what's recoverable) that would let the coordinator act intelligently rather than merely give up or retry blindly. Anthropic's own tool-error guidance makes the same point at the API level: write instructive error messages that state what went wrong and what should be tried next, rather than generic strings — the richer message is what lets the calling model recover or adapt without guessing.
  Example: "search unavailable" tells the coordinator nothing about whether to retry, wait, or use a different source. "Rate limit exceeded on internal search API; retry after 60s, or use the cached index as a fallback" tells it exactly what options exist.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls ("Write instructive error messages")

- Concept: Anti-patterns — silent suppression and full-workflow termination
  Type: key
  Teaches: 5.3-K4
  Definition: Two opposite but equally damaging ways of mishandling subagent failure: (1) silently suppressing an error by returning an empty result as if it were a successful, valid empty result — which hides a real failure from the coordinator and can lead it to present incomplete findings as complete; and (2) terminating the entire multi-agent workflow the moment any single subagent fails — which is needlessly fragile, since most workflows can still deliver a partial, honestly-labeled answer even when one branch failed. The graceful middle path is targeted recovery: let the coordinator know a specific tool or subagent is failing and let it adapt, rather than either hiding the failure or stopping everything.
  Example: If a market-data subagent fails while three other subagents succeed, silently returning `{}` for the market-data section (as if there was simply no data) misleads the final report; halting the whole research task because of that one failure wastes the three subagents' completed work. The better response is a report that includes the three findings plus an explicit note that market data is unavailable.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system (graceful degradation: "letting the agent know when a tool is failing and letting it adapt works surprisingly well"); bullet wording from context-management-reliability.md (5.3-K4)

- Concept: Local recovery before propagation
  Type: key
  Teaches: 5.3-S3
  Definition: Subagents should attempt to resolve transient failures themselves (retry logic, an alternate query, a fallback source) before escalating the failure up to the coordinator, and should only propagate an error once local recovery has been exhausted — at which point the propagated error should still include what was attempted and any partial results gathered, so the coordinator isn't starting recovery from zero. This combines the adaptability of an agent that can retry and adjust with deterministic safeguards (retry logic, checkpoints) that don't depend on the model noticing the failure on its own.
  Example: A subagent whose first search query returns a connection error retries once with backoff and, if that also fails, tries a narrower query before finally reporting up to the coordinator that both the retry and the narrower query failed, along with whatever partial data either attempt returned.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system ("combines the adaptability of AI agents built on Claude with deterministic safeguards like retry logic and regular checkpoints")

- Concept: Coverage annotations in synthesis output
  Type: key
  Teaches: 5.3-S4
  Definition: A synthesis agent combining output from multiple subagents (some of which may have partially failed or hit unavailable sources) should explicitly annotate which parts of its output are well-supported by successful, complete subagent findings and which topic areas have coverage gaps due to unavailable sources or failed calls — rather than presenting a uniformly confident-looking report that silently omits what wasn't found. This is the synthesis-level counterpart of not silently suppressing errors: the gap is disclosed instead of smoothed over.
  Example: A market research report ends with a "Coverage notes" section stating that pricing and competitor data are well-supported by three independent sources, while regulatory information could not be retrieved because the regulatory-filings subagent's source was unavailable.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system (LLM-judge rubric dimension "completeness: are all requested aspects covered?"); bullet wording from context-management-reliability.md (5.3-S4)

- Concept: tool_result / is_error mechanics (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for structured error context (5.3-K1, S1) and access-failure/empty-result distinction (5.3-K2, S2)
  Definition: The Claude API's mechanism for reporting a tool execution problem back to the model: a `tool_result` block with `is_error: true` and a `content` string describing the failure. This is the concrete, API-level building block that "structured error context" and "access failure vs. empty result" reasoning is built on top of when a multi-agent system's subagents are themselves implemented as tool-using Claude agents.
  Example: `{"type": "tool_result", "tool_use_id": "toolu_01", "content": "ConnectionError: weather service unavailable (HTTP 500)", "is_error": true}`.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls

- Concept: Orchestrator-worker (lead agent / subagent) architecture (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for coordinator recovery decisions (5.3-K1, K4, S1, S3, S4)
  Definition: The architectural pattern in which a lead ("coordinator") agent analyzes a task, develops a strategy, and delegates to specialized subagents that operate — often in parallel — each with their own context window, before the coordinator compiles their findings. Error-propagation design only makes sense against this architecture: it's the coordinator's job to make recovery decisions, which is why subagents need to hand it structured, decision-useful information rather than opaque pass/fail signals.
  Example: A lead research agent spawns three subagents to investigate different aspects of a question in parallel, then synthesizes their (possibly partially failed) findings into one final report.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system (orchestrator-worker pattern)

---

## Task Statement 5.4 — Manage context effectively in large codebase exploration

- Concept: Context degradation in extended exploration sessions
  Type: key
  Teaches: 5.4-K1
  Definition: In long-running codebase-exploration sessions, the same context-rot dynamics that affect any long context show up concretely: the model starts giving inconsistent answers about the codebase, and it begins referencing "typical patterns" or generic assumptions about how code like this usually works instead of the specific classes, functions, or files it actually discovered earlier in the session. This is a practical symptom of the model losing reliable access to details buried deep in a growing history, and the standard mitigation is to start a fresh session or subagent for a new task rather than letting one session run indefinitely.
  Example: Forty tool calls into a codebase exploration, the model is asked "does the RefundProcessor class validate currency codes?" and instead of citing the actual class it inspected earlier, it answers with a generic statement about how refund processors "typically" handle currency validation — a sign the earlier, specific finding has rotted out of effective context.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (context rot); bullet wording from context-management-reliability.md (5.4-K1); supporting guidance at https://claude.com/blog/using-claude-code-session-management-and-1m-context ("when you start a new task, you should also start a new session")

- Concept: Scratchpad / progress files for cross-boundary persistence
  Type: key
  Teaches: 5.4-K2, 5.4-S2
  Definition: A technique for counteracting context degradation: agents write notes recording key findings to a file that persists outside the context window (a "scratchpad" or progress log), and reference that file for subsequent questions rather than relying on the model's in-context memory of earlier discoveries. Because the file is external, its contents survive compaction, session restarts, and context-window boundaries in a way that in-context memory does not.
  Example: During a multi-session refactor, the agent maintains a `claude-progress.txt` recording "Discovered: `PaymentGateway` in `src/billing/gateway.py` handles all currency conversion; `RefundProcessor` does NOT validate currency, relies on gateway" — and re-reads that file at the start of the next session or before answering a related question, instead of re-deriving or guessing the answer.
  Source: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents (progress-log pattern: "Read the progress notes file and git commit logs"); https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (structured note-taking)

- Concept: Subagent delegation to isolate verbose exploration output
  Type: key
  Teaches: 5.4-K3, 5.4-S1
  Definition: Rather than having the main agent perform exhaustive exploration itself (accumulating every file read, grep result, and log line in its own context), the main agent spawns subagents to investigate narrow, specific questions ("find all test files," "trace refund flow dependencies"). Each subagent gets its own fresh, isolated context window — it doesn't see the main conversation's history — does whatever exploration it needs, and returns only a condensed summary. The main agent's context stays reserved for high-level coordination instead of being flooded with exploration noise.
  Example: Instead of the main agent grepping through hundreds of files itself, it spawns a subagent with the task "find every place `RefundProcessor` is instantiated and summarize the call sites in under 200 words," and only that summary — not the raw grep output — returns to the main conversation.
  Source: https://code.claude.com/docs/en/sub-agents ("Each subagent starts with a fresh, isolated context window... the subagent does that work in its own context and returns only the summary")

- Concept: Structured state exports and manifests for crash recovery
  Type: key
  Teaches: 5.4-K4, 5.4-S4
  Definition: A design for surviving crashes or context-window truncation in long-running, multi-agent work: each agent exports its state (plan, findings so far, what's been done) to a known, structured location rather than keeping it only in its own volatile context. On resume, the coordinator loads a manifest referencing those exports and injects the relevant state back into each agent's prompt, so work resumes from where it stopped instead of restarting from scratch. Anthropic's own research-agent system uses this pattern explicitly: it saves its plan to persistent memory specifically because context exceeding 200,000 tokens gets truncated, and the plan needs to survive that truncation.
  Example: A lead agent spawns subagents to explore a large monorepo, and each subagent writes its findings to `state/subagent-2-findings.json`; a coordinator-level `manifest.json` lists which subagents have completed and where their state lives, so if the process crashes mid-run, the coordinator can reload the manifest and re-inject each completed subagent's findings without re-running it.
  Source: https://www.anthropic.com/engineering/built-multi-agent-research-system ("saving its plan to Memory to persist the context, since if the context window exceeds 200,000 tokens it will be truncated"); https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents (structured feature-list state plus git history)

- Concept: Phase summarization before spawning next-phase subagents
  Type: key
  Teaches: 5.4-S3
  Definition: When codebase exploration proceeds in phases (e.g., "map the module structure" then "trace the refund flow"), summarizing the key findings of one phase and injecting that summary into the initial context of the next phase's subagents avoids two failure modes at once: it prevents the next phase from starting with no relevant context (having to rediscover what the previous phase already found), and it prevents the next phase's subagents from inheriting the first phase's full, bloated exploration history.
  Example: After a "map the module structure" phase produces a long exploration transcript, the coordinator distills it into a five-bullet summary of the module layout and passes only that summary — not the full transcript — as starting context to the subagents spawned for the "trace the refund flow" phase.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (sub-agent architectures: condensed summaries handed off between phases); https://www.anthropic.com/engineering/built-multi-agent-research-system (lead agent decomposing work into sequenced subagent tasks)

- Concept: Using /compact to reduce context usage mid-session
  Type: key
  Teaches: 5.4-S5
  Definition: `/compact` is a Claude Code command that asks the model to summarize the conversation so far and replaces the message history with that summary, freeing up context space consumed by verbose discovery output (file reads, tool results, exploration back-and-forth) accumulated during extended exploration. It can be steered with an instruction describing what to keep (e.g., "focus on the auth refactor, drop the test debugging"), and using it proactively — before context is nearly exhausted — gives more control over what survives than waiting for an automatic compaction to trigger.
  Example: Partway through a long codebase-exploration session whose context is filling with verbose `grep`/`read` output, the developer runs `/compact focus on what we learned about the refund flow, drop the test-file listing` to shrink the context while explicitly preserving the discovery that matters for the next step.
  Source: https://claude.com/blog/using-claude-code-session-management-and-1m-context ("Compact asks the model to summarize the conversation so far, then replaces the history with that summary"); https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (compaction primitive)

- Concept: Subagent context isolation mechanics (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for subagent delegation (5.4-K3, S1)
  Definition: A Claude Code subagent, when spawned, starts with a fresh context window that does not include the parent conversation's history, previously invoked skills, or previously read files — it receives only its own system prompt, a delegation message describing the task, and (unless omitted) project configuration like CLAUDE.md and git status. This isolation is the structural reason delegating exploration to a subagent keeps verbose output out of the main agent's context rather than merely relocating the same bloat.
  Example: A subagent tasked with "find all test files" cannot see that the main agent already read `src/billing/gateway.py` earlier in the session — it starts clean and must be told anything from the parent context it actually needs.
  Source: https://code.claude.com/docs/en/sub-agents

---

## Task Statement 5.5 — Design human review workflows and confidence calibration

- Concept: Aggregate accuracy metrics masking segment-level failures
  Type: key
  Teaches: 5.5-K1, 5.5-S2
  Definition: A single aggregate accuracy figure across an entire extraction pipeline (e.g., "97% overall") can be produced by uniformly good performance or by a mix of near-perfect performance on most document types/fields and much worse performance concentrated in a specific segment — and the aggregate number alone cannot distinguish the two. Because errors in practice tend to cluster in specific document types or specific fields rather than spreading evenly, validating accuracy broken down by document type and field, rather than relying on the single aggregate figure, is necessary before deciding a pipeline is safe to run with reduced human review.
  Example: An extraction pipeline reporting 97% overall accuracy could be masking the fact that invoices are extracted at 99.5% accuracy while handwritten receipts — a small fraction of total volume — are extracted at 60% accuracy; the aggregate figure looks safe while the receipt segment is not.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.5-K1, 5.5-S2, exam guide bullet wording)

- Concept: Stratified random sampling for error-rate measurement
  Type: key
  Teaches: 5.5-K2, 5.5-S1
  Definition: A sampling method for measuring true error rates and catching novel error patterns: rather than reviewing only low-confidence extractions (which is where errors are expected and already being caught), draw a representative random sample from each stratum — document type, confidence band, field type — including high-confidence extractions, and have humans verify it. Sampling only the low-confidence tail misses the case where the model develops a new failure mode that happens to produce high, but wrong, confidence; only sampling across all strata, including the high-confidence ones, can detect that.
  Example: A monthly QA process randomly samples 2% of extractions from each of {invoices, receipts, contracts} × {high, medium confidence} bands and has a human verify each sampled extraction, rather than reviewing only the extractions the model itself flagged as low-confidence.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.5-K2, 5.5-S1, exam guide bullet wording)

- Concept: Field-level confidence scores calibrated against labeled validation sets
  Type: key
  Teaches: 5.5-K3, 5.5-S3
  Definition: Having the model output a confidence score per extracted field (rather than one confidence score for the whole document) allows review effort to be routed at the granularity where errors actually occur. Because a raw model-reported confidence number is not automatically a calibrated probability (a 0.9 confidence score is not guaranteed to mean 90% correctness, and the true correctness rate at a given confidence level can differ by field), the score must be calibrated: compare model confidence to actual correctness on a labeled validation set, per field, and set review thresholds based on that measured relationship rather than on the raw score.
  Example: On a labeled validation set, extractions with a self-reported 0.9 confidence for the "invoice date" field turn out to be correct 96% of the time, while 0.9-confidence extractions for the "line-item total" field are only correct 82% of the time — so the review threshold for line-item totals is set higher than for dates, even though the model reported the same raw confidence number for both.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.5-K3, 5.5-S3, exam guide bullet wording)

- Concept: Validating accuracy by document type and field segment before automating
  Type: key
  Teaches: 5.5-K4
  Definition: Before reducing or removing human review for high-confidence extractions, accuracy must be validated broken down by document type and field segment — not just in aggregate — specifically because the aggregate-metric masking problem (5.5-K1) means a pipeline that looks safe overall can still be unsafe for a specific segment. This validation step is what justifies automating away review for the segments that check out, while keeping review in place for segments that haven't been separately verified.
  Example: Before turning off human review for "high-confidence" extractions across the board, the team runs a per-segment accuracy analysis and finds that high-confidence extractions from scanned (as opposed to digital) PDFs are meaningfully less accurate — so review stays in place for scanned documents even as it's removed for digital ones.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.5-K4, exam guide bullet wording)

- Concept: Confidence- and ambiguity-based routing to human review
  Type: key
  Teaches: 5.5-S4
  Definition: Extractions should be routed to human review when either the model's calibrated field-level confidence is low, or the source documents themselves are ambiguous or contain contradictory information — two different reasons an extraction might be untrustworthy that both warrant a human look. Because reviewer capacity is limited, routing should prioritize which extractions most need attention (lowest confidence, most contradictory sources) rather than treating all flagged items as equal priority. This complements the general hallucination-reduction pattern of allowing a model to express uncertainty rather than forcing a confident-looking answer it isn't sure of.
  Example: An extraction where the model has low confidence on the contract's termination date, and where two pages of the contract state different dates, is routed to a human reviewer ahead of one that is merely low-confidence on a field where the source text is unambiguous.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.5-S4, exam guide bullet wording); https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations (allowing the model to flag uncertainty instead of guessing)

---

## Task Statement 5.6 — Preserve information provenance and handle uncertainty in multi-source synthesis

- Concept: Source attribution loss during summarization
  Type: key
  Teaches: 5.6-K1
  Definition: When findings from multiple sources are compressed during a summarization step, the mapping between each specific claim and the source it came from is one of the first things lost, because a summarizer optimizing for brevity naturally merges similar-sounding claims from different sources into one generalized statement, discarding which source said what. Once that mapping is gone, it typically cannot be reconstructed from the summary alone, which is why attribution needs to be preserved structurally rather than recovered after the fact.
  Example: Two sources report slightly different figures for the same statistic; after an unstructured summarization pass, the report says only "reports suggest values in this range" with no way to tell a reader which source said which number.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.6-K1, exam guide bullet wording); supporting mechanism at https://platform.claude.com/cookbook/misc-using-citations

- Concept: Structured claim-source mappings preserved through synthesis
  Type: key
  Teaches: 5.6-K2, 5.6-S1
  Definition: The fix for source-attribution loss: require every subagent to output its findings as structured claim-source mappings (the claim text, the source URL or document name, and the relevant excerpt) rather than free-form prose, and require the downstream synthesis agent to preserve and merge those mappings — not just the claims — as it combines findings from multiple subagents. The Claude API's own citations feature implements this structurally: each cited span in a response carries a `citations` array linking the exact cited text back to a specific source document, so the claim-to-source link survives as machine-checkable structure rather than depending on prose staying accurate.
  Example: A subagent's structured output is `{"claim": "Adoption grew 40% YoY", "source": "2026 Industry Report, p.12", "excerpt": "...adoption increased by 40% year over year..."}`, and the synthesis agent's final report keeps that same claim-source pairing intact (e.g., as a numbered citation) instead of collapsing it into unsourced prose.
  Source: https://platform.claude.com/cookbook/misc-using-citations (structured citations mapping cited text to source documents)

- Concept: Annotating conflicting statistics rather than arbitrarily selecting one
  Type: key
  Teaches: 5.6-K3, 5.6-S3
  Definition: When two or more credible sources report different values for the same fact, the correct handling is not to silently pick one (which discards real information and can be wrong) but to preserve both values with their source attribution, explicitly annotated as conflicting, and let the coordinator or a downstream decision-maker decide how to reconcile them (e.g., preferring the more recent or more authoritative source, or presenting both to the end user) rather than have an intermediate agent make that call invisibly.
  Example: Source A reports a market size of $4.2B (dated 2025) and Source B reports $5.1B (dated 2026); the document-analysis output keeps both figures with their sources and dates and flags them as conflicting, rather than the analysis agent quietly reporting only one number.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.6-K3, 5.6-S3, exam guide bullet wording)

- Concept: Temporal metadata to prevent false contradictions
  Type: key
  Teaches: 5.6-K4, 5.6-S4
  Definition: Requiring subagents to include the publication or data-collection date of each source in their structured output allows a downstream synthesis step to correctly interpret differences between sources as temporal change (the underlying fact genuinely changed between when Source A and Source B were collected) rather than misreading them as a contradiction between two disagreeing sources describing the same point in time. Without a date attached, an old figure and a new figure look identical to a genuinely conflicting pair of figures.
  Example: Source A (published 2023) reports a company's headcount as 500; Source B (published 2026) reports 1,200. With dates attached, this reads as growth over three years, not a factual dispute; without dates, it would misleadingly look like two sources disagreeing about the same current number. The citations mechanism supports this via a document's `context` field, which can carry metadata such as publication date that Claude uses to interpret — without directly citing — the document.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.6-K4, 5.6-S4, exam guide bullet wording); https://platform.claude.com/cookbook/misc-using-citations (the `context` field for source metadata such as publication date)

- Concept: Distinguishing well-established from contested findings in report structure
  Type: key
  Teaches: 5.6-S2
  Definition: A synthesis report should be structured with explicit sections separating findings that are well-established (consistently supported across multiple credible sources) from findings that are contested (sources disagree, or only one lower-confidence source supports it), and should preserve each source's own characterization and methodological context rather than flattening everything into equally-confident-sounding prose. This gives a reader the same information a careful human analyst would have flagged, instead of hiding the difference in confidence behind uniform phrasing.
  Example: A report has a "Well-established findings" section (backed by three independent sources using comparable methodology) and a separate "Contested / single-source findings" section (one source, or sources using notably different methodologies), rather than presenting both kinds of claim with the same tone and certainty.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.6-S2, exam guide bullet wording); https://www.anthropic.com/engineering/built-multi-agent-research-system (LLM-judge rubric dimension "source quality")

- Concept: Rendering content types appropriately rather than uniformly
  Type: key
  Teaches: 5.6-S5
  Definition: A multi-source synthesis output should render different kinds of content in the format that content is naturally structured for — financial data as tables, news-style findings as prose, technical findings as structured lists — rather than forcing every content type through one uniform format (e.g., converting everything to paragraphs, or everything to bullet lists), which typically both loses information the natural format would have preserved (e.g., row/column alignment in tabular data) and reads worse to the eventual consumer of the report.
  Example: A synthesis report presents quarterly revenue figures from several sources as an actual comparison table, presents the general market narrative as flowing prose, and presents a list of specific technical compliance requirements as a numbered list — rather than converting all three into one uniform block of prose.
  Source: runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md (5.6-S5, exam guide bullet wording)

- Concept: Claude API citations feature (prerequisite)
  Type: prerequisite
  Teaches: prerequisite for structured claim-source mappings (5.6-K2, S1) and temporal metadata (5.6-K4, S4)
  Definition: An API feature where documents are passed with `citations: {"enabled": true}`, after which any claim in Claude's response that draws on a specific document is annotated with a `citations` array identifying the exact cited text and the source document — and the feature will not fabricate citations to documents or locations that were not actually provided. This is the concrete API mechanism that structured claim-source mapping and temporal-metadata handling are built on when Claude itself performs the synthesis.
  Example: Passing an order-tracking document with `"citations": {"enabled": True}` causes any generated sentence drawn from that document to carry a citation object naming the document title and the exact source span, rather than an unattributed claim.
  Source: https://platform.claude.com/cookbook/misc-using-citations

---

## Coverage index (bullet ID → concept)

5.1-K1 → Progressive summarization risk during compaction
5.1-K2 → Context rot and the "lost in the middle" effect
5.1-K3 → Tool-result context bloat
5.1-K4 → Full conversation history and the stateless Messages API
5.1-S1 → Persistent "case facts" block outside summarized history
5.1-S2 → Structured multi-issue context layering
5.1-S3 → Tool-result clearing / trimming to relevant fields
5.1-S4 → Positional structuring to counter position effects
5.1-S5 → Subagent output metadata for downstream synthesis
5.1-S6 → Structured data over verbose reasoning for context-constrained downstream agents

5.2-K1 → Escalation trigger taxonomy
5.2-K2 → Immediate escalation on explicit request vs. offering resolution first
5.2-K3 → Unreliable complexity proxies — sentiment and self-reported confidence
5.2-K4 → Clarification over heuristic disambiguation for multiple matches
5.2-S1 → Few-shot escalation criteria in the system prompt
5.2-S2 → Immediate escalation on explicit request vs. offering resolution first
5.2-S3 → Immediate escalation on explicit request vs. offering resolution first
5.2-S4 → Escalation on policy silence or ambiguity
5.2-S5 → Clarification over heuristic disambiguation for multiple matches

5.3-K1 → Structured error context for coordinator recovery
5.3-K2 → Distinguishing access failures from valid empty results
5.3-K3 → Generic error statuses as a context-hiding anti-pattern
5.3-K4 → Anti-patterns — silent suppression and full-workflow termination
5.3-S1 → Structured error context for coordinator recovery
5.3-S2 → Distinguishing access failures from valid empty results
5.3-S3 → Local recovery before propagation
5.3-S4 → Coverage annotations in synthesis output

5.4-K1 → Context degradation in extended exploration sessions
5.4-K2 → Scratchpad / progress files for cross-boundary persistence
5.4-K3 → Subagent delegation to isolate verbose exploration output
5.4-K4 → Structured state exports and manifests for crash recovery
5.4-S1 → Subagent delegation to isolate verbose exploration output
5.4-S2 → Scratchpad / progress files for cross-boundary persistence
5.4-S3 → Phase summarization before spawning next-phase subagents
5.4-S4 → Structured state exports and manifests for crash recovery
5.4-S5 → Using /compact to reduce context usage mid-session

5.5-K1 → Aggregate accuracy metrics masking segment-level failures
5.5-K2 → Stratified random sampling for error-rate measurement
5.5-K3 → Field-level confidence scores calibrated against labeled validation sets
5.5-K4 → Validating accuracy by document type and field segment before automating
5.5-S1 → Stratified random sampling for error-rate measurement
5.5-S2 → Aggregate accuracy metrics masking segment-level failures
5.5-S3 → Field-level confidence scores calibrated against labeled validation sets
5.5-S4 → Confidence- and ambiguity-based routing to human review

5.6-K1 → Source attribution loss during summarization
5.6-K2 → Structured claim-source mappings preserved through synthesis
5.6-K3 → Annotating conflicting statistics rather than arbitrarily selecting one
5.6-K4 → Temporal metadata to prevent false contradictions
5.6-S1 → Structured claim-source mappings preserved through synthesis
5.6-S2 → Distinguishing well-established from contested findings in report structure
5.6-S3 → Annotating conflicting statistics rather than arbitrarily selecting one
5.6-S4 → Temporal metadata to prevent false contradictions
5.6-S5 → Rendering content types appropriately rather than uniformly
