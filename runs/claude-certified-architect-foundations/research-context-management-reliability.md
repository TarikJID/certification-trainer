# Research: Context Management & Reliability (CCAR-F Domain 5, 15%)

Bullets received: 53 (Task 5.1: 10, Task 5.2: 9, Task 5.3: 8, Task 5.4: 9, Task 5.5: 8, Task 5.6: 9)
Bullets covered: 53
Key concepts: 33
Prerequisite concepts: 11
Total concepts: 44
UNSOURCED concepts: 0

Precedence used throughout: Tier 1 = the archived exam guide itself (valid standalone, no corroboration required). Tier 2 = official Anthropic/Claude product documentation and engineering content (platform.claude.com/docs, code.claude.com/docs, platform.claude.com/cookbook, anthropic.com/engineering). Tier 3 = standards bodies/primary specifications (e.g., NIST). Tier 4 = named reputable secondary sources, used only when tiers 1-3 have nothing (none were needed in this domain).

Exam guide domain-5 citations reference `runs/claude-certified-architect-foundations/source-exam-guide.txt`, lines 675–813 ("Domain 5: Context Management & Reliability"), which is Tier 1 official certification material valid as a standalone source per the research brief. Where vendor/standards documentation independently corroborates or extends a bullet's content, both sources are cited, each at its correct tier.

---

## Task Statement 5.1 — Manage conversation context to preserve critical information across long interactions

### Key Concepts

- Concept: Context rot
  Type: key
  Teaches: 5.4-K1
  Definition: As the number of tokens in a model's context window grows, its ability to accurately recall and use information from that context degrades — not because the window is technically "full," but because transformer self-attention creates n² pairwise token relationships, so attention capacity is stretched thinner as sequence length increases. In practice this shows up as agents giving inconsistent answers, "forgetting" earlier findings, or falling back on generic "typical pattern" reasoning instead of citing specifics they discovered earlier in the session.
  Example: During a multi-hour codebase exploration session, an agent that correctly named the five specific classes implementing an interface in turn 3 starts saying "the classes typically follow the standard repository pattern" by turn 40, having lost the specific detail amid tens of thousands of accumulated tokens of exploration output.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents (tier 2, official Anthropic engineering blog)

- Concept: Lost-in-the-middle effect
  Type: key
  Teaches: 5.1-K2
  Definition: Language models process information placed at the very beginning and very end of a long input more reliably than information placed in the middle. When critical findings, facts, or figures are buried in the middle of a long aggregated context (e.g., page 40 of 80 concatenated tool outputs), the model is measurably more likely to omit or misstate them, even though the information was technically present in context.
  Example: A customer support agent reviewing a long transcript with the customer's refund amount stated in the middle of the conversation may quote the wrong figure, while amounts stated at the very start or the most recent message are recalled accurately.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-K2), tier 1; corroborated by https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "Long context prompting" section ("Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs"), tier 2

- Concept: Progressive summarization risk
  Type: key
  Teaches: 5.1-K1
  Definition: Each time a conversation or research thread is condensed into a summary, precise, high-signal details — exact numerical values, percentages, dates, and customer-stated expectations — are the most likely content to be smoothed into vague language (e.g., "the customer wanted a refund" instead of "$142.50 refund requested by 3/14"). Repeated summarization passes compound this loss, because a summary-of-a-summary starts from content that has already lost precision.
  Example: A three-turn summarization chain turns "customer paid $89.99 on March 3rd and was promised a refund within 5 business days" into "customer is waiting on a refund," discarding the amount, date, and SLA commitment that a downstream agent needs to resolve the case correctly.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-K1), tier 1; corroborated by https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, compaction section ("balancing recall... against precision... start by maximizing recall, then iterate toward precision"), tier 2

- Concept: Tool-result context accumulation and trimming
  Type: key
  Teaches: 5.1-K3, 5.1-S3
  Definition: Every tool call's raw result is appended to the conversation, and results (a full order record, an API response, a file read) frequently contain far more fields than are relevant to the task at hand. Left unmanaged, this accumulates tokens at a rate disproportionate to their usefulness — Anthropic gives the example that a tool has been called deep in message history, so there is little reason for the agent to keep re-reading its full raw result. The mitigation is to trim tool outputs to only the fields relevant to the current task before they persist in context, and/or to mechanically clear stale, re-fetchable tool results once they are no longer needed.
  Example: An order-lookup tool returns 40+ fields (shipping carrier, warehouse ID, promotional codes, internal SKUs); for a return request, only order number, item, purchase date, amount, and return-eligibility status are kept in the persisted context, with the rest discarded before the next turn.
  Source: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents ("once a tool has been called deep in the message history, why would the agent need to see the raw result again?... one of the safest lightest touch forms of compaction"), tier 2; mechanism detailed in https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools (`clear_tool_uses_20250919` context-management edit), tier 2

- Concept: Complete conversation history for API coherence
  Type: key
  Teaches: 5.1-K4
  Definition: The Claude Messages API is stateless between requests — each call must include the full message history the model should be aware of, including prior `assistant` turns, `tool_use` blocks, and their matching `tool_result` blocks, in the exact order they occurred. Tool-result blocks must immediately follow their corresponding tool-use blocks; omitting messages or reordering them breaks the model's ability to maintain conversational coherence and can trigger explicit API validation errors.
  Example: An application drops an earlier `assistant` message containing a `tool_use` block while trying to save tokens; the next request now has an orphaned `tool_result` with no matching `tool_use`, and the API returns a 400 error rather than silently proceeding.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-K4), tier 1; corroborated by https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls ("Tool result blocks must immediately follow their corresponding tool use blocks in the message history"), tier 2

- Concept: Persistent "case facts" / structured issue-data layer
  Type: key
  Teaches: 5.1-S1, 5.1-S2
  Definition: Rather than letting transactional facts (amounts, dates, order numbers, statuses) live only inside prose history that is subject to summarization drift, the agent extracts them into a small, structured block (e.g., a JSON or key-value "case facts" object) that is included verbatim in every subsequent prompt, outside the summarized conversational history. For sessions covering multiple issues, each issue gets its own structured record in this layer so facts about issue A are never conflated with issue B.
  Example: A support session spanning three separate order problems maintains `case_facts = [{order_id: "A1002", amount: 42.10, status: "delayed"}, {order_id: "A1050", amount: 18.00, status: "refund_requested"}]`, injected fresh into every prompt regardless of how much the surrounding dialogue has been summarized.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-S1, 5.1-S2), tier 1; corroborated by structured note-taking guidance in https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, tier 2

- Concept: Position-based context structuring to mitigate lost-in-the-middle
  Type: key
  Teaches: 5.1-S4
  Definition: To counteract the lost-in-the-middle effect, key findings summaries are placed at the very beginning of an aggregated input (not buried after the raw detail), and the detailed supporting material that follows is broken into clearly labeled sections (e.g., XML tags or headers per source/topic) rather than one undifferentiated block. Anthropic's own long-context guidance additionally recommends placing the query/instructions at the end of the prompt, after the data, since recall of instructions is highest when they immediately precede generation.
  Example: A research synthesis prompt opens with a "Key Findings" section listing the three headline conclusions, followed by `<document index="1"><source>...</source><document_content>...</document_content></document>` blocks for each underlying source, with the actual analysis question placed after all documents.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "Long context prompting" section ("Put longform data at the top... Structure document content and metadata with XML tags"), tier 2

- Concept: Structured subagent output with provenance metadata
  Type: key
  Teaches: 5.1-S5
  Definition: When a subagent reports findings back to a coordinator or downstream agent, it is required to attach metadata alongside the substantive content — dates, source locations, and methodological context — rather than returning bare prose conclusions. This metadata is what lets a downstream synthesis step reconstruct where a claim came from and how reliable it is, without needing to re-run the original investigation.
  Example: Anthropic's multi-agent research system routes findings through a dedicated CitationAgent that processes the research report specifically to attach precise source locations to every claim, ensuring claims are properly attributed rather than left as unsourced prose.
  Source: https://www.anthropic.com/engineering/multi-agent-research-system, CitationAgent description, tier 2; requirement itself stated in runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-S5), tier 1

- Concept: Structured over verbose subagent output for context-constrained consumers
  Type: key
  Teaches: 5.1-S6
  Definition: When a downstream agent has a limited context budget, an upstream (producing) agent should be modified to return compact structured data — key facts, citations, relevance scores — instead of its full verbose content and reasoning chain. This shifts the compression decision upstream, where the producing agent has the full context needed to decide what matters, rather than forcing a token-starved downstream agent to do that filtering itself.
  Example: A search subagent that read 15 pages of documentation returns only `{claim: "rate limit is 50 rps", source: "docs.example.com/limits", relevance: 0.9}` rather than pasting the full page text and its own step-by-step reasoning into the coordinator's context.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.1-S6), tier 1; corroborated by "each subagent... explores extensively... but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens)" in https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, tier 2

---

## Task Statement 5.2 — Design effective escalation and ambiguity resolution patterns

### Key Concepts

- Concept: Escalation trigger taxonomy
  Type: key
  Teaches: 5.2-K1, 5.2-S2, 5.2-S4
  Definition: Appropriate triggers for escalating a case to a human agent are: (1) the customer explicitly requests a human, (2) the request falls into a policy exception or a gap where policy is silent or ambiguous about the specific situation, and (3) the agent is unable to make meaningful progress on the issue. This is a narrower and more precise set of triggers than "escalate whenever a case seems complex" — complexity alone is not, by itself, a valid trigger. When policy simply doesn't address a customer's specific request (e.g., a competitor price-match request when policy only covers the company's own-site price adjustments), that silence is itself a trigger to escalate rather than to improvise an answer.
  Example: A customer asks the support agent to match a competitor's advertised price. The store's policy document only describes matching the company's own past prices, saying nothing about competitors. Rather than deciding one way or the other, the agent escalates the request to a human because policy is silent on this specific case.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.2-K1, 5.2-S2, 5.2-S4), tier 1; corroborated by human-in-the-loop guardrail guidance ("pause for human feedback at checkpoints or when encountering blockers") in https://www.anthropic.com/engineering/building-effective-agents, tier 2

- Concept: Escalate-immediately vs. offer-to-resolve distinction
  Type: key
  Teaches: 5.2-K2, 5.2-S3
  Definition: When a customer explicitly demands a human agent, the system should honor that request immediately without first attempting to investigate or resolve the issue itself — investigating first, however well-intentioned, ignores the customer's stated preference. By contrast, when a customer expresses frustration but has not explicitly demanded escalation, and the issue is within the agent's capability to resolve, the correct pattern is to acknowledge the frustration while still offering to resolve the issue, escalating only if the customer reiterates their preference for a human after that offer.
  Example: A frustrated customer writes "this is ridiculous, fix my order now" (no explicit human request) — the agent acknowledges the frustration and offers to resolve the order issue directly. A different customer writes "I want to speak to a person" — the agent transfers immediately, without first trying to solve the problem.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.2-K2, 5.2-S3), tier 1

- Concept: Unreliable complexity proxies (sentiment and self-reported confidence)
  Type: key
  Teaches: 5.2-K3
  Definition: Two commonly proposed heuristics for deciding when to escalate — the customer's emotional sentiment and the model's own self-reported confidence score — are unreliable proxies for actual case complexity. A customer can be highly upset about a simple, easily-resolved issue, and calm about a genuinely complex, policy-ambiguous one. Likewise, a model can express high self-reported confidence while still being wrong, because self-reported confidence reflects the model's fluency in producing an answer, not a calibrated measure of correctness. Effective escalation criteria must therefore be grounded in the structural properties of the case itself (explicit request, policy gap, inability to progress) rather than in sentiment or self-assessed confidence.
  Example: A calmly worded message ("Can you match this competitor's price? Not urgent.") may be a genuinely hard, policy-ambiguous case that should escalate, while an angrily worded message about a late delivery may be a simple, well-covered case the agent can resolve outright — sentiment alone would route both incorrectly if used as the escalation trigger.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.2-K3), tier 1

- Concept: Multiple-match disambiguation
  Type: key
  Teaches: 5.2-K4, 5.2-S5
  Definition: When a lookup tool returns more than one matching customer record (e.g., two customers with the same name), the agent must not use a heuristic to silently pick one (such as "most recent order" or "first result") — doing so risks acting on the wrong customer's account. Instead, the agent should be instructed to ask the customer for an additional identifying detail (e.g., order number, billing zip code, email) to disambiguate before proceeding.
  Example: A lookup for "John Smith" returns two customer records. Rather than guessing based on which account was more recently active, the agent responds: "I found multiple accounts under that name — could you confirm your order number or the email on the account?"
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.2-K4, 5.2-S5), tier 1

- Concept: Few-shot escalation criteria embedded in the system prompt
  Type: key
  Teaches: 5.2-S1
  Definition: Explicit escalation criteria are made concrete and consistent by adding a small set of worked few-shot examples to the system prompt, each demonstrating a scenario and the correct decision (escalate vs. resolve autonomously) with the reasoning behind it. This lets the model generalize the underlying judgment to novel, unseen scenarios rather than only matching the literal cases given as examples.
  Example: A system prompt includes: "`<example>` Customer: 'I've called three times about this and nothing's fixed.' → Escalate: repeated unresolved contact signals inability to make progress. `</example>`" alongside 2–3 other worked examples covering policy gaps and explicit requests.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.2-S1), tier 1; few-shot technique itself documented in https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "Use examples effectively" section, tier 2

---

## Task Statement 5.3 — Implement error propagation strategies across multi-agent systems

### Key Concepts

- Concept: Structured error context for coordinator recovery
  Type: key
  Teaches: 5.3-K1, 5.3-S1
  Definition: When a subagent or tool call fails, returning a bare, generic status is not enough for a coordinator to make a good recovery decision. Instead, the failure should be reported as structured context: the failure type (e.g., timeout, rate limit, not-found), what was attempted (the query or action), any partial results already obtained, and potential alternative approaches. This gives the coordinator (or the model consuming a tool_result) the information needed to decide whether to retry, try an alternative, work from partial data, or surface the failure to a human.
  Example: Instead of returning `"error": "search unavailable"`, a subagent returns `{"failure_type": "timeout", "attempted": "search vendor DB for SKU-1234", "partial_results": ["found in cache from 2 days ago"], "alternatives": ["retry after 30s", "use cached result with staleness warning"]}`.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.3-K1, 5.3-S1), tier 1; corroborated by tool-error guidance ("Write instructive error messages. Instead of generic errors like 'failed,' include what went wrong and what Claude should try next") in https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls, tier 2

- Concept: Access failure vs. valid empty result
  Type: key
  Teaches: 5.3-K2, 5.3-S2
  Definition: An error report must distinguish between two structurally different situations that a naive implementation could conflate: an access failure (the query could not be executed at all — e.g., a timeout, an unreachable service — which may warrant a retry) versus a valid empty result (the query executed successfully and simply found nothing matching — which is a legitimate, final answer, not a failure). Collapsing these into one signal (e.g., both returning "no results") prevents the coordinator from making the correct decision — retrying a genuine access failure, or accepting a genuine empty result instead of wastefully retrying it.
  Example: A search for "customer emails matching 'urgent refund'" returns zero matches because none exist — that is a valid empty result and should be reported as `{"status": "success", "matches": 0}`. A search that times out mid-query should instead be reported as `{"status": "access_failure", "reason": "timeout"}`, since retrying might succeed where accepting "zero matches" would be wrong.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.3-K2, 5.3-S2), tier 1

- Concept: Error-suppression and workflow-termination anti-patterns
  Type: key
  Teaches: 5.3-K3, 5.3-K4
  Definition: Two opposite but equally damaging anti-patterns in multi-agent error handling: (1) silently suppressing errors by returning an empty result as if it were a successful "no matches" outcome, which hides the failure from the coordinator entirely and can cause downstream decisions to be made on false premises; and (2) terminating the entire workflow the moment any single subagent or tool call fails, which is overly brittle and discards potentially valuable partial results from every other part of the system that succeeded. Generic, uninformative error statuses (e.g., "search unavailable") sit adjacent to this problem: they technically report a failure but strip out the detail the coordinator needs to respond intelligently.
  Example: A multi-agent research pipeline where one of five subagents fails to reach a source should not (a) quietly report "0 findings" as if that source had nothing to say, nor (b) abort the entire research task and discard the four subagents that succeeded — it should report the one failure with context and let the coordinator decide whether to proceed with four-fifths coverage, retry, or note the gap.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.3-K3, 5.3-K4), tier 1; corroborated by "minor system failures can be catastrophic for agents" and the practice of resuming from checkpoints rather than restarting from scratch, in https://www.anthropic.com/engineering/multi-agent-research-system, tier 2

- Concept: Local recovery before propagation
  Type: key
  Teaches: 5.3-S3
  Definition: Subagents should attempt to resolve transient failures themselves (e.g., retry a timed-out call once or twice) rather than immediately bubbling every hiccup up to the coordinator. Only errors the subagent genuinely cannot resolve locally should be propagated upward — and when they are propagated, the report should still include what was attempted and any partial results obtained during the local recovery attempts, so the coordinator isn't starting from zero.
  Example: A subagent's API call times out; it retries once automatically and succeeds, so nothing is escalated. A second call fails on both the original attempt and the retry; only then does the subagent propagate a structured failure to the coordinator, including the fact that a retry was already attempted and its own result.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.3-S3), tier 1; corroborated by "letting the agent know when a tool is failing and letting it adapt works surprisingly well" combined with retry logic and checkpoints, in https://www.anthropic.com/engineering/multi-agent-research-system, tier 2

- Concept: Coverage annotation in synthesis output
  Type: key
  Teaches: 5.3-S4
  Definition: When a coordinator assembles a final synthesis from multiple subagents, some of which may have partially failed or returned incomplete data, the output should be explicitly annotated to indicate which findings are well-supported (drawn from complete, successful subagent runs) versus which topic areas have coverage gaps because a source was unavailable or a subagent failed. This keeps the downstream consumer (human or another agent) from mistaking a gap for a confirmed absence of information.
  Example: A synthesized report includes a note: "Pricing data: well-supported (3 independent sources agree). Competitor roadmap: coverage gap — the vendor's investor-relations subagent could not reach its source; this section reflects only secondary reporting."
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.3-S4), tier 1

---

## Task Statement 5.4 — Manage context effectively in large codebase exploration

### Key Concepts

- Concept: Scratchpad files for cross-boundary persistence
  Type: key
  Teaches: 5.4-K2, 5.4-S2
  Definition: To counteract context degradation over an extended exploration session, agents maintain scratchpad files — plain files on disk recording key findings as they are discovered — and explicitly reference those files for subsequent questions rather than relying purely on what remains salient in the live conversation context. Because the scratchpad persists outside the model's context window, it survives context boundaries (compaction, session restarts, or agent handoffs) that would otherwise cause the finding to be lost or misremembered.
  Example: While tracing a refund flow across a codebase, an agent appends to `findings.md`: "RefundProcessor.calculate() at src/billing/refund.py:142 handles partial refunds; does NOT handle currency conversion — see follow-up needed." A later exploration phase reads this file first instead of re-deriving the same fact.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.4-K2, 5.4-S2), tier 1; corroborated by state-management guidance on unstructured progress notes ("Freeform progress notes work well for tracking general progress and context") in https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "State management best practices" section, tier 2

- Concept: Subagent delegation for context isolation during exploration
  Type: key
  Teaches: 5.4-K3, 5.4-S1
  Definition: Rather than having the main agent read every file and accumulate all raw exploration output directly into its own context, specific investigation questions ("find all test files," "trace refund flow dependencies") are delegated to subagents, each of which runs in its own isolated context window. The subagent can explore as verbosely as it needs to internally, but only its condensed summary returns to the main conversation, so the main agent's context stays available for high-level coordination rather than being consumed by raw discovery output.
  Example: Claude Code's built-in Explore subagent is dispatched with read-only tools to search a codebase; its full grep/read trail stays inside its own context, and the main session receives only "Found 12 database-related files across /db, /models, /migrations."
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.4-K3, 5.4-S1), tier 1; corroborated by "Claude delegates to Explore when it needs to search or understand a codebase without making changes. This keeps exploration results out of your main conversation context" in https://code.claude.com/docs/en/sub-agents, tier 2

- Concept: Structured state export and crash-recovery manifest
  Type: key
  Teaches: 5.4-K4, 5.4-S4
  Definition: For long-running, multi-agent exploration or build workflows that may crash or be interrupted, each agent exports its state (what it found, what it completed, what remains) to a known, structured location on disk. A coordinator that resumes after a crash loads a manifest — an index of what each agent exported and where — and injects the relevant state back into each agent's prompt on restart, so work does not have to be redone from scratch.
  Example: Agent A crashes after indexing 60% of a repository's modules. On restart, the coordinator reads `manifest.json`, sees Agent A's last checkpoint recorded 340 of 560 modules indexed with their file paths, and resumes Agent A from module 341 instead of re-scanning the whole repository.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.4-K4, 5.4-S4), tier 1; corroborated by structured JSON state-tracking guidance ("Use structured formats for state data... to help Claude understand schema requirements") in https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "State management best practices" section, tier 2

- Concept: Phase-summary injection before spawning the next subagent wave
  Type: key
  Teaches: 5.4-S3
  Definition: When an exploration task is broken into sequential phases, each handled by its own subagent(s), the key findings from one phase are summarized before the next phase's subagents are spawned, and that summary is injected into the initial context of the new subagents. This prevents each new phase from starting completely blind, while still avoiding the cost of forwarding the full raw output of the previous phase.
  Example: Phase 1 subagents map out a codebase's module structure; before Phase 2 subagents are spawned to trace a specific bug, the coordinator injects a two-paragraph summary of the module map into their initial prompts rather than the full raw file listings Phase 1 produced.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.4-S3), tier 1; corroborated by the general pattern of spawning fresh subagents with condensed prior context rather than carrying forward full history, in https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, tier 2

- Concept: /compact for extended exploration sessions
  Type: key
  Teaches: 5.4-S5
  Definition: Claude Code's `/compact` command summarizes the conversation so far — including verbose discovery output accumulated during exploration — and replaces it with a compressed version, freeing token budget while retaining awareness of what happened earlier in the session. It can be run manually (optionally with instructions on what to preserve, e.g., "focus on the refund flow findings") or triggers automatically as the context window approaches its limit.
  Example: After several hours of codebase exploration filled with grep and file-read output, a developer runs `/compact focus on the authentication bug investigation` before continuing, so the session keeps working with a condensed summary instead of hitting the context limit mid-task.
  Source: https://code.claude.com/docs/en/context-window, tier 2; usage pattern also named directly in runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.4-S5), tier 1

---

## Task Statement 5.5 — Design human review workflows and confidence calibration

### Key Concepts

- Concept: Aggregate-metric masking and segment-level validation
  Type: key
  Teaches: 5.5-K1, 5.5-K4, 5.5-S2
  Definition: A single aggregate accuracy figure (e.g., "97% overall accuracy") can conceal badly underperforming subsets of the data — a specific document type, or a specific field — because errors concentrated in a small but important segment get averaged away by strong performance elsewhere. Before reducing human review or automating high-confidence extractions, accuracy must be validated broken out by document type and by field, confirming that every segment independently meets the bar, not just the aggregate.
  Example: An extraction pipeline reports 97% overall field accuracy, but a breakdown by document type shows handwritten invoices are only 81% accurate while typed invoices are 99.5% — the aggregate figure alone would have hidden the fact that automating without review is unsafe specifically for handwritten invoices.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.5-K1, 5.5-K4, 5.5-S2), tier 1

- Concept: Stratified random sampling for error-rate measurement
  Type: key
  Teaches: 5.5-K2, 5.5-S1
  Definition: Stratified random sampling segments a population into homogeneous groups (strata) — such as document type or extraction field — and draws random samples from each stratum independently, rather than sampling the population as a whole. Applied to a production extraction pipeline, this means periodically sampling from within the pool of "high-confidence" (i.e., not routed to review) extractions, stratified by document type or field, to measure the true error rate in what is otherwise never manually checked, and to catch novel error patterns the model has started making that a confidence score alone wouldn't flag.
  Example: A pipeline auto-accepts extractions above a 0.9 confidence threshold. To make sure that threshold is still safe, the team draws a random 2% sample from within each document-type stratum of auto-accepted extractions every week and has a human verify them, rather than only reviewing the low-confidence items that were already flagged.
  Source: https://csrc.nist.gov/glossary/term/stratified_sampling ("the process of segmenting a population across levels of some factors to minimize variability within those segments," sourced to NIST SP 800-55v1 / NIST Handbook 151), tier 3 (standards body); application to extraction pipelines stated in runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.5-K2, 5.5-S1), tier 1

- Concept: Field-level confidence calibration with labeled validation sets
  Type: key
  Teaches: 5.5-K3, 5.5-S3
  Definition: Rather than a single document-level confidence score, the model outputs a confidence value per extracted field. Because a model's stated confidence is not automatically well-calibrated (a field marked "90% confident" is not guaranteed to be correct 90% of the time), the review threshold applied to that confidence score must itself be calibrated against a labeled validation set — i.e., empirically checking, on data with known correct answers, what confidence level actually corresponds to what real-world accuracy, and setting the human-review cutoff based on that measured relationship rather than the raw score.
  Example: A model reports 0.85 confidence on extracted "total_amount" fields; validation against 500 labeled invoices shows that fields at 0.85 confidence are actually correct only 88% of the time, while fields at 0.95 confidence are correct 99.5% of the time — so the review threshold is set at 0.95, not 0.85, based on this calibration exercise.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.5-K3, 5.5-S3), tier 1; corroborated by the schema-level confidence-field pattern (`"confidence": {"type": "number"}`) documented in https://platform.claude.com/docs/en/build-with-claude/structured-outputs, "Classification" example, tier 2

- Concept: Confidence-based human review routing
  Type: key
  Teaches: 5.5-S4
  Definition: Extractions are routed to human review based on two conditions: low model confidence (below the calibrated threshold), or ambiguous/contradictory source documents (where even a confident extraction may be extracting the wrong thing because the source itself is unclear). Because reviewer capacity is limited, routing decisions should prioritize which flagged cases are reviewed first rather than treating every flagged item identically, so scarce reviewer time is spent where it has the most impact.
  Example: Of 1,000 daily extractions, 40 fall below the confidence threshold and 15 come from documents where two source fields disagree; both groups are routed to a review queue, with the contradictory-source cases prioritized first since they represent a higher-certainty error signal than a merely low-confidence score.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.5-S4), tier 1; corroborated by human-in-the-loop checkpoint guidance in https://www.anthropic.com/engineering/building-effective-agents, tier 2

---

## Task Statement 5.6 — Preserve information provenance and handle uncertainty in multi-source synthesis

### Key Concepts

- Concept: Structured claim-source mapping preservation
  Type: key
  Teaches: 5.6-K1, 5.6-K2, 5.6-S1
  Definition: Source attribution (which specific document, URL, or excerpt a claim came from) is easily lost the moment findings are compressed by a summarization step, if that step only preserves the claim's content and not its origin. The fix is for subagents to output claims paired with structured source metadata — source URLs, document names, and the relevant excerpt supporting the claim — and for every downstream agent in the synthesis chain (including any summarization step) to explicitly preserve and merge these claim-source mappings rather than only merging the prose conclusions.
  Example: A research subagent's output includes `{"claim": "adoption grew 40% in 2025", "source_url": "industry-report.example.com/2025", "excerpt": "...year-over-year adoption grew 40%..."}` rather than the bare sentence "adoption grew 40% in 2025," so the fact and its provenance survive being merged with four other subagents' findings.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.6-K1, 5.6-K2, 5.6-S1), tier 1; corroborated by the dedicated CitationAgent that identifies "specific locations for citations" to ensure "all claims are properly attributed to their sources" in https://www.anthropic.com/engineering/multi-agent-research-system, tier 2, and by the Citations API feature, which returns per-claim source location, URL, and cited text grounding a response in its documents, documented at https://platform.claude.com/docs/en/build-with-claude/citations, tier 2

- Concept: Conflict annotation instead of arbitrary reconciliation
  Type: key
  Teaches: 5.6-K3, 5.6-S3
  Definition: When credible sources report conflicting statistics on the same question, the synthesis process should not arbitrarily pick one value and discard the other. Instead, both (or all) conflicting values are preserved with their source attribution and the conflict is explicitly annotated. Reconciliation, if it happens at all, is a deliberate decision made by a coordinator with visibility into both values and their sources — not an implicit, invisible choice buried inside an earlier compression step.
  Example: Source A reports "market size: $4.2B (2024)" and Source B reports "market size: $3.8B (2024)." The document-analysis stage outputs both figures with their sources annotated as conflicting, rather than silently reporting only one of them as if there were no disagreement; the coordinator then decides how to present or reconcile the discrepancy before final synthesis.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.6-K3, 5.6-S3), tier 1

- Concept: Temporal metadata requirement to prevent false contradictions
  Type: key
  Teaches: 5.6-K4, 5.6-S4
  Definition: Two figures that appear to contradict each other can simply reflect different points in time rather than a genuine disagreement between sources. Requiring every subagent to include the publication date or data-collection date alongside each structured finding lets the synthesis step correctly recognize "$4.2B (2024 report)" and "$5.1B (2026 report)" as a trend over time rather than an unresolved conflict between sources.
  Example: Without dates, "unemployment rate: 4.1%" and "unemployment rate: 3.6%" look contradictory. With required dates attached — "4.1% (reported January 2025)" and "3.6% (reported August 2026)" — the synthesis correctly reads this as a trend rather than flagging it as a source conflict.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.6-K4, 5.6-S4), tier 1

- Concept: Distinguishing well-established from contested findings in synthesis structure
  Type: key
  Teaches: 5.6-S2
  Definition: A synthesis report is structured with explicit sections separating well-established findings (supported by multiple independent, credible sources) from contested ones (where sources disagree, or evidence is thin), while preserving how each original source characterized its own claim and any methodological context that affects how much weight the claim should carry (e.g., sample size, survey methodology, self-reported vs. measured data).
  Example: A synthesized market report has a "Well-Established" section (three independent analyst firms agree adoption grew 30–40% in 2025) and a separate "Contested" section (one source's 60% figure is based on a small self-reported survey, flagged with that methodological caveat rather than presented as equally weighted).
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.6-S2), tier 1; corroborated by evaluation guidance to prefer "primary sources over lower-quality secondary sources" and assess source quality during synthesis, in https://www.anthropic.com/engineering/multi-agent-research-system, tier 2

- Concept: Content-type-appropriate rendering in synthesis output
  Type: key
  Teaches: 5.6-S5
  Definition: Different kinds of synthesized content are rendered in the format best suited to that content type — financial data as tables, news-style findings as prose, and technical findings as structured lists — rather than flattening every content type into one uniform presentation format (e.g., forcing tabular financial data into prose paragraphs, or forcing narrative news content into a rigid table).
  Example: A combined report on a company presents quarterly revenue figures in a table, recent news coverage as narrative prose paragraphs, and a list of technical product changes as a structured bulleted list — rather than describing the revenue table in prose or tabulating the news coverage.
  Source: runs/claude-certified-architect-foundations/source-exam-guide.txt (Domain 5, Task 5.6-S5), tier 1

---

## Prerequisite Concepts

- Concept: Context window as a finite, shared resource
  Type: prerequisite
  Teaches: 5.1-K2, 5.1-K3, 5.4-K1
  Definition: A model's context window is a hard limit on the total tokens (system prompt, tool schemas, conversation history, tool results) available to it in a single request. Every category of content that loads into a session — system prompt, memory files, tool definitions, conversation turns, tool results — competes for this same finite budget, and different parts of the pipeline load automatically before the user's own content ever arrives.
  Example: Claude Code's context-window visualization shows a session's window opening already partially consumed by the system prompt (~4,200 tokens), auto-loaded memory (~680 tokens), and environment info (~280 tokens) before any user conversation begins.
  Source: https://code.claude.com/docs/en/context-window, tier 2

- Concept: The tool-use loop (tool_use / tool_result blocks)
  Type: prerequisite
  Teaches: 5.1-K4, 5.3-K1, 5.3-S1
  Definition: Claude's agentic tool use follows a defined request/response cycle: Claude responds with `stop_reason: "tool_use"` and one or more `tool_use` content blocks naming a tool and its input; the calling application executes the tool and sends back a `user` message containing a matching `tool_result` block (referencing the `tool_use_id`), optionally marked `is_error: true` with a descriptive error message if execution failed; Claude then continues the conversation using that result.
  Example: Claude emits `{"type": "tool_use", "id": "toolu_01", "name": "get_order", "input": {"order_id": "A1002"}}`; the application executes the lookup and replies with `{"type": "tool_result", "tool_use_id": "toolu_01", "content": "..."}`, or `{"is_error": true, "content": "Rate limit exceeded. Retry after 60 seconds."}` if the call failed.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls, tier 2

- Concept: System prompts for role and behavior configuration
  Type: prerequisite
  Teaches: 5.2-S1, 5.2-K1
  Definition: The `system` parameter of a Claude API request sets persistent role, tone, and behavioral instructions that apply across the whole conversation, distinct from the turn-by-turn `messages`. Setting a role, even in a single sentence, measurably focuses the model's behavior for a given use case, and is the natural place to add durable policy such as escalation criteria.
  Example: `"system": "You are Eva, a friendly and knowledgeable AI assistant for Acme Insurance Company..."` establishes identity and scope once, rather than being restated in every user turn.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "Give Claude a role" section, tier 2

- Concept: Few-shot (multishot) prompting
  Type: prerequisite
  Teaches: 5.2-S1
  Definition: Providing a small number (typically 3–5) of worked examples, wrapped in `<example>` tags, is one of the most reliable ways to steer a model's output format, tone, and — critically for ambiguous judgment calls — its decision-making pattern. Effective examples are relevant to the real use case, diverse enough to cover edge cases, and structured so the model can distinguish example content from instructions.
  Example: A prompt includes three `<example>` blocks, each showing a customer message and the correct escalate-or-resolve decision with a one-line rationale, so the model can generalize the underlying judgment to a fourth, unseen scenario.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices, "Use examples effectively" section, tier 2

- Concept: Subagents as isolated context windows
  Type: prerequisite
  Teaches: 5.1-S6, 5.4-K3, 5.4-S1, 5.4-S3
  Definition: A subagent is a specialized assistant that runs in its own separate context window, with its own system prompt and (optionally restricted) tool access, and does not inherit the parent conversation's history. It receives only a delegation message describing its task, does its work in isolation, and returns a result to the parent — meaning verbose intermediate work never touches the parent's context budget.
  Example: Claude Code's built-in Explore subagent is invoked with read-only tools (Read, Grep, Glob); it does not see the main session's prior conversation, does its own file searching, and returns a short summary such as "Found 12 database-related files."
  Source: https://code.claude.com/docs/en/sub-agents, tier 2

- Concept: Orchestrator-worker multi-agent pattern
  Type: prerequisite
  Teaches: 5.3-S3, 5.3-S4, 5.6-S1, 5.6-S2
  Definition: A workflow pattern in which a central (orchestrator/lead) LLM dynamically breaks a task into subtasks it could not fully predict in advance, delegates each subtask to a worker (subagent) LLM, and then synthesizes the workers' results into a final output. This differs from a fixed parallelization pipeline because the orchestrator determines subtasks dynamically based on the specific input, and is the structural pattern underlying error propagation and multi-source synthesis in agentic systems.
  Example: A lead research agent decomposes "compare three cloud providers' pricing" into three parallel worker subagents, one per provider, each returning structured findings that the lead agent then synthesizes into a single comparison.
  Source: https://www.anthropic.com/engineering/building-effective-agents, "Orchestrator-workers" section, tier 2

- Concept: The memory tool for cross-session persistence
  Type: prerequisite
  Teaches: 5.4-K2, 5.4-S2
  Definition: An Anthropic-provided client tool (`memory_20250818`) that lets an agent create, read, update, delete, and rename files in a persistent directory across sessions, giving it a structured place to write notes that survive a context reset. The agent decides what to record, and reads its own memory files back on demand rather than needing everything re-loaded into the live context window — the same underlying mechanism scratchpad-file patterns rely on.
  Example: An agent researching a company across two separate sessions writes findings to `/memories/company_x.md` in session one, and opens that file at the start of session two instead of re-deriving everything from scratch.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool, tier 2

- Concept: Structured outputs and JSON-schema-constrained extraction
  Type: prerequisite
  Teaches: 5.1-S1, 5.5-K3, 5.5-S3
  Definition: Claude can be constrained to always return output conforming to a specified JSON schema — including fields such as a numeric `confidence` value alongside extracted content — using either the Structured Outputs feature or `tool_use` with a JSON schema. Constrained decoding restricts what tokens the model can generate at each step so the output literally cannot violate the schema, which is the mechanism that makes structured "case facts" blocks and per-field confidence scores reliably parseable.
  Example: A classification schema defines `{"category": "string", "confidence": "number", "tags": ["string"], "sentiment": "string"}` as required fields with `additionalProperties: false`, guaranteeing every response includes a numeric confidence value in a fixed location.
  Source: https://platform.claude.com/docs/en/build-with-claude/structured-outputs, tier 2

- Concept: Context-management primitives (compaction and tool-result clearing)
  Type: prerequisite
  Teaches: 5.1-K3, 5.4-S5
  Definition: The Claude API exposes explicit, configurable `context_management` edits: `compact_20260112`, which replaces conversation history with a model-generated high-fidelity summary once a token threshold is crossed, and `clear_tool_uses_20250919`, which mechanically (with no inference cost) replaces old tool_result content with a placeholder while preserving the record that the call was made. Claude Code's `/compact` slash command and its automatic compaction behavior are the interactive-session expression of the same underlying idea.
  Example: A long-running agent configures `clear_tool_uses_20250919` to trigger at 100K input tokens (keeping the 6 most recent tool results) and `compact_20260112` to trigger at 200K tokens as a fallback, reducing peak context from 335K to under 170K tokens in Anthropic's own benchmark.
  Source: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools, tier 2

- Concept: Citations feature for document-grounded attribution
  Type: prerequisite
  Teaches: 5.6-K1, 5.6-S1
  Definition: An Anthropic API feature that, when enabled on a document passed to Claude, returns detailed citations alongside generated claims — the exact passage and source document that supports each part of the response — allowing an application to verify and surface exactly which source backs which statement, rather than trusting an unattributed summary.
  Example: Asking "What color is the grass and sky?" against a document with citations enabled returns not just "The grass is green and the sky is blue" but a citation block pointing to the exact source sentence for each color claim.
  Source: https://platform.claude.com/docs/en/build-with-claude/citations, tier 2

- Concept: Human-in-the-loop checkpoints and guardrails in agentic systems
  Type: prerequisite
  Teaches: 5.2-K1, 5.5-S4
  Definition: A design pattern for agentic systems in which the agent pauses to request human feedback at defined checkpoints or when it encounters a blocker it cannot resolve, combined with stopping conditions (such as a maximum number of iterations) to keep the system from compounding errors autonomously. This is the general architectural basis underlying both customer-support escalation triggers and confidence-based routing to human document reviewers.
  Example: A coding agent is configured to pause and request human approval before any destructive or hard-to-reverse action (force-push, dropping a database table), rather than proceeding autonomously through every step of a long task.
  Source: https://www.anthropic.com/engineering/building-effective-agents, tier 2
