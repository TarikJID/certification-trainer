# Module 8 — Context Management and Escalation Design

Domain: Domain 5 — Context Management & Reliability (15% of exam)
Covers Task Statements 5.1, 5.2.

**Prerequisites from earlier modules:** the Messages API request/response cycle
(Module 1, Lesson 1.1); multishot/few-shot prompting (Module 6, Lesson 4.2).

---

## Lesson 5.1 — Managing Conversation Context to Preserve Critical Information Across Long Interactions

**Maps to:** Task Statement 5.1: Manage conversation context to preserve critical
information across long interactions.

### Prerequisite concept: Attention budget

Attention in a transformer model is a finite resource: every additional token in
the context window depletes the "budget" of attention available to relate any
other pair of tokens, because self-attention scales as n² pairwise relationships
for n tokens. This is the mechanical reason context windows have practical, not
just nominal, limits — usable accuracy degrades before the token limit is
technically reached.

*Example:* Doubling the tokens in a prompt doesn't just cost twice the compute;
it spreads the model's capacity to relate any single token to the rest of the
input more thinly.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: prerequisite for context rot / lost-in-the-middle)

### Prerequisite concept: Full conversation history and the stateless Messages API — recap

Recall from Module 1, Lesson 1.1: the Messages API is stateless — every request
must include the complete message history the application wants Claude aware of.
This is why context management (trimming, summarizing, restructuring) happens
client-side rather than implicitly in the API.

*Example:* A multi-turn insurance quote flow where Claude has already collected
make, model, and year must re-send those facts in every subsequent request; if
the year is dropped, Claude will ask for it again.

*Source:* https://platform.claude.com/docs/en/build-with-claude/working-with-messages
(Teaches: 5.1-K4)

### Concept: Context rot and the "lost in the middle" effect

As the number of tokens in context grows, the model's ability to accurately
recall and use any given piece of information decreases — "context rot." One
manifestation is positional: models are most reliable at the beginning and end of
a long input, least reliable in the middle.

*Example:* A research agent aggregates ten subagent reports into one 50,000-token
context. A critical caveat in report #5 is statistically much more likely to be
dropped from the final answer than the same caveat in report #1 or #10.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.1-K2)

### Concept: Progressive summarization risk during compaction

Compaction — summarizing a conversation nearing its context limit and
reinitializing with that summary — trades completeness for space. A summarizer
optimizing for brevity will preferentially condense precise, low-frequency tokens
(exact numbers, percentages, dates, customer-stated expectations) into vague
paraphrases, because those details look individually unimportant even though
they're exactly what a downstream turn needs verbatim.

*Example:* "I need this resolved by Friday the 14th, and I was quoted $47.50"
gets compacted into "Customer requested timely resolution and discussed
pricing" — the date and dollar figure a later reply depends on are gone.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.1-K1)

### Concept: Tool-result context bloat

Tool outputs accumulate in context regardless of how much of that output the
agent actually needs, and are usually full raw payloads that consume tokens out
of proportion to their relevance. A 40-field order-lookup where only 5 fields
matter still costs the context window for all 40 on every subsequent turn unless
trimmed.

*Example:* An order-lookup tool returns shipping address, order history, loyalty
tier, SKU codes, and warehouse metadata for a customer asking only "where's my
refund" — the relevant fields are a small fraction of the tokens consumed.

*Source:* https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools;
https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.1-K3)

### Concept: Persistent "case facts" block outside summarized history

A technique for defending transactional facts against progressive-summarization
loss: extract amounts, dates, order numbers, statuses into a small structured
block included verbatim in every prompt, kept separate from — and never subject
to — compaction applied to the rest of the conversation.

*Example:* A support-agent prompt includes `<case_facts>{"order_id": "A-88213",
"refund_amount": "$47.50", "promised_by": "2026-09-19"}</case_facts>` on every
turn, independent of whatever summary of the rest of the conversation is
included.

*Source:* https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
(Teaches: 5.1-S1)

### Concept: Structured multi-issue context layering

Extension of the case-facts pattern to sessions covering several distinct
issues: each issue's structured data is persisted in its own layer, so resolving
one issue's facts doesn't require re-parsing a mixed summary and facts from one
issue don't bleed into another's.

*Example:* A single chat session where a customer asks about a late delivery
(issue A) and separately a billing discrepancy (issue B) keeps each issue's own
structured fact block so a reply about the delivery never cites the billing
amount.

*Source:* https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
(Teaches: 5.1-S2)

### Concept: Tool-result clearing / trimming to relevant fields

A context-management primitive that removes or replaces the bulk of a tool's raw
output once it's no longer needed verbatim, either by trimming at insertion time
to only relevant fields, or by later replacing old `tool_result` content with a
placeholder. This directly counters tool-result context bloat and is the
cheapest form of compaction since it requires no additional model inference.

*Example:* Before inserting an order-lookup result into context, the application
strips it down to `{"order_id", "status", "amount", "return_eligible"}` and
discards the other 35+ fields returned.

*Source:* https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
(Teaches: 5.1-S3)

### Concept: Positional structuring to counter position effects

Because models are more reliable at the start and end of long inputs, aggregated
inputs should place a short summary of key findings at the very beginning, and
organize detailed material underneath with explicit section headers. Long-context
guidance additionally recommends placing the query/instructions after the long
content rather than before it.

*Example:* A synthesis prompt opens with `<key_findings>` summarizing the three
most important conclusions, followed by `<detailed_results>` under section
headers, with the actual analysis instruction placed after all of that content.

*Source:* https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices;
https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.1-S4)

### Concept: Subagent output metadata for downstream synthesis

Requiring every subagent's structured output to carry metadata alongside its
findings — dates, source locations, methodological context — so a downstream
synthesis agent, which typically only sees the subagent's condensed output, has
enough context to interpret and correctly attribute findings.

*Example:* A research subagent's output includes not just "median tenure: 3.2
years" but also `{"source": "2026 internal HR report", "collected": "2026-Q2",
"method": "survey of active employees"}`.

*Source:* https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.1-S5)

### Concept: Structured data over verbose reasoning for context-constrained downstream agents

When an upstream agent's full output would overwhelm a downstream agent's
context budget, the upstream agent should return a compact, structured payload —
key facts, citations, relevance scores — rather than verbose prose and reasoning
chains.

*Example:* Instead of a research subagent returning its full 8,000-token
exploration transcript, it returns `{"facts": [...], "citations": [...],
"relevance": 0.9}` — a fraction of the size, with the same decision-relevant
content.

*Source:* https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
(Teaches: 5.1-S6)

**Quiz:** see `quiz.md`, Lesson 5.1.

---

## Lesson 5.2 — Designing Effective Escalation and Ambiguity Resolution Patterns

**Maps to:** Task Statement 5.2: Design effective escalation and ambiguity
resolution patterns.

### Prerequisite concept: System-prompt role and behavior framing

Setting an explicit role and behavioral frame in the system prompt establishes
the context within which few-shot examples and escalation rules are interpreted;
even a single clear sentence of role-framing measurably focuses a model's tone
and behavior before any examples are added.

*Example:* `IDENTITY = "You are Eva, a support assistant for Acme... you can
resolve straightforward issues yourself and know when to bring in a human."`
sets the frame that few-shot escalation examples then make concrete.

*Source:* https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
(Teaches: prerequisite for few-shot escalation criteria)

### Concept: Escalation trigger taxonomy

A well-designed agent escalates under three distinct conditions, evaluated
independently rather than collapsed into one vague "if it's hard, escalate"
rule: (1) the customer explicitly requests a human; (2) the request falls into a
policy exception or gap, not merely a procedurally complex case; (3) the agent is
unable to make meaningful progress. Complexity alone, absent one of these three
conditions, is not by itself a sufficient trigger.

*Example:* A refund requiring five tool calls that resolves cleanly under policy
should not escalate just because it took several steps; a refund scenario the
policy never mentions should escalate even if it would otherwise be one tool
call.

*Source:* exam-guide.txt (Domain 5, Task 5.2); supporting metric at
https://platform.claude.com/docs/en/about-claude/use-case-guides/customer-support-chat
(Teaches: 5.2-K1)

### Concept: Immediate escalation on explicit request vs. offering resolution first

Two different situations call for two different behaviors. When a customer
explicitly demands a human, honor that immediately without first attempting
investigation — investigating first reads as overriding the customer's stated
preference. When the issue itself is straightforward but the customer is
frustrated (no explicit human request), acknowledge the frustration, offer to
resolve it, and escalate only if the customer reiterates they still want a human.

*Example:* "I want to talk to a person" → escalate now. "This is so annoying, my
order is three days late" (no explicit request) → acknowledge, offer to check and
resolve, escalate only if the customer insists afterward.

*Source:* exam-guide.txt (Domain 5, Task 5.2)
(Teaches: 5.2-K2, 5.2-S2, 5.2-S3)

### Concept: Unreliable complexity proxies — sentiment and self-reported confidence

Two signals that look like reasonable escalation triggers but aren't: (1)
sentiment analysis — a customer can be frustrated about a trivial issue and calm
about a genuinely complex one; (2) a model's own self-reported confidence — it
can be confidently wrong or under-confident on a case it's handling correctly.

*Example:* A politely worded message asking about a policy exception per a
verbal agreement with a manager is calm in tone but describes exactly the kind of
policy-gap case that should escalate — sentiment alone would miss it.

*Source:* exam-guide.txt (Domain 5, Task 5.2)
(Teaches: 5.2-K3)

### Concept: Clarification over heuristic disambiguation for multiple matches

When a tool call intended to identify a specific customer or record returns
multiple matches, the correct pattern is to ask for an additional identifier
rather than picking one match using a heuristic (most recent, alphabetically
first). Heuristic selection risks silently acting on the wrong customer's data.

*Example:* A lookup by name returns two "John Smith" accounts; instead of picking
the first, the agent asks "I found two accounts under that name — could you
confirm the last four digits of the phone number on file?"

*Source:* exam-guide.txt (Domain 5, Task 5.2); analogous principle at
https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
(Teaches: 5.2-K4, 5.2-S5)

### Concept: Few-shot escalation criteria in the system prompt

Rather than describing escalation policy only in abstract prose, adding concrete
few-shot examples to the system prompt that demonstrate specific
escalate-vs-resolve-autonomously decisions is one of the most reliable ways to
steer behavior consistently — building directly on the few-shot technique from
Module 6, Lesson 4.2.

*Example:* A system prompt includes `<example>` blocks showing (a) an explicit
human request handled by immediate escalation, (b) a straightforward late-
delivery question resolved autonomously, and (c) a competitor price-match
request escalated because policy is silent on it.

*Source:* https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
(Teaches: 5.2-S1)

### Concept: Escalation on policy silence or ambiguity

A specific application of the escalation trigger taxonomy's "policy exception/
gap" condition: when a request falls into territory the written policy simply
doesn't address, the agent should escalate rather than improvise, because silence
is not permission and an incorrect improvised answer can create liability or an
inconsistent precedent.

*Example:* A store's policy covers price-matching against its own past sales but
says nothing about competitor prices. A customer asking for a competitor price
match should be escalated, not denied or approved by inference.

*Source:* exam-guide.txt (Domain 5, Task 5.2)
(Teaches: 5.2-S4)

**Quiz:** see `quiz.md`, Lesson 5.2.

---

## Module 8 summary of bullets taught

Lesson 5.1: 5.1-K1, 5.1-K2, 5.1-K3, 5.1-K4, 5.1-S1, 5.1-S2, 5.1-S3, 5.1-S4,
5.1-S5, 5.1-S6
Lesson 5.2: 5.2-K1, 5.2-K2, 5.2-K3, 5.2-K4, 5.2-S1, 5.2-S2, 5.2-S3, 5.2-S4, 5.2-S5
