# Module 8 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 5.1 (Managing Conversation Context)

**Q1.** A 10-report synthesis puts a critical caveat in report #5 of 10. Why is
that caveat statistically more likely to be dropped than one in report #1 or #10?

<details><summary>ANSWER</summary>
The "lost in the middle" effect: models are most reliable at using information
near the beginning and end of a long input, and least reliable at using
information buried in the middle, because self-attention's capacity to relate any
token to the rest of the input is finite and dilutes as context grows.
</details>

**Q2.** A customer said "I need this resolved by Friday the 14th, and I was
quoted $47.50." After compaction, the summary reads "Customer requested timely
resolution and discussed pricing." What went wrong, and what's the fix?

<details><summary>ANSWER</summary>
Progressive summarization risk: a summarizer optimizing for brevity
preferentially condenses precise, low-frequency tokens (exact dates, dollar
amounts) into vague paraphrases. Fix: extract such transactional facts into a
persistent "case facts" block included verbatim in every prompt, outside the
summarized/compacted history.
</details>

**Q3.** An order-lookup tool returns 40 fields but only 5 are relevant to a
return request. What should happen to the other 35, and why does the Messages
API's statelessness make this the application's responsibility rather than the
API's?

<details><summary>ANSWER</summary>
The other 35 fields should be trimmed/cleared before or after insertion into
context (tool-result clearing) since they consume tokens disproportionate to
their relevance. Because the Messages API is stateless and simply echoes back
whatever's in the request each time, any trimming, summarizing, or restructuring
of history must happen client-side — the API does not do it implicitly.
</details>

**Q4.** How should a synthesis prompt aggregating several subagents' reports be
structured to counter position effects?

<details><summary>ANSWER</summary>
Place a short summary of key findings at the very beginning, organize detailed
supporting material underneath with explicit section headers, and place the
actual query/analysis instruction after the long content rather than before it.
</details>

**Q5.** A downstream synthesis agent has a small context budget. Should an
upstream research subagent hand it its full 8,000-token exploration transcript
including reasoning chains, or something else? Why?

<details><summary>ANSWER</summary>
Something else — a compact, structured payload (key facts, citations, relevance
scores) rather than verbose prose and reasoning chains. This mirrors the
sub-agent pattern of returning only a condensed, distilled summary of work, so
the downstream agent gets the decision-relevant content without exceeding its
context budget.
</details>

---

## Quiz — Lesson 5.2 (Escalation and Ambiguity Resolution)

**Q1.** A refund request takes five tool calls but resolves cleanly under
existing policy. Should the agent escalate just because it was complex? Why or
why not?

<details><summary>ANSWER</summary>
No. Complexity alone, in the absence of an explicit human request, a policy
exception/gap, or an inability to make progress, is not by itself a sufficient
escalation trigger under the escalation trigger taxonomy.
</details>

**Q2.** A customer explicitly says "I want to talk to a person." What should the
agent do, and what should it NOT do first?

<details><summary>ANSWER</summary>
Honor the request immediately and escalate without first attempting its own
investigation or resolution. Investigating first would read as the agent
overriding the customer's stated preference.
</details>

**Q3.** Why are sentiment analysis and the model's own self-reported confidence
score both unreliable as the primary basis for an escalation decision?

<details><summary>ANSWER</summary>
Sentiment: a customer can be highly frustrated about a trivially resolvable
issue and calm about a genuinely complex one, so tone doesn't track actual
complexity. Self-reported confidence: a model can be confidently wrong or
under-confident on a case it's actually handling correctly, so its own estimate
isn't a reliable proxy either.
</details>

**Q4.** A customer lookup by name returns two matching accounts. What should the
agent do, and what should it avoid doing?

<details><summary>ANSWER</summary>
Ask the customer for an additional identifier (order number, email, ZIP code) to
disambiguate. It should avoid picking one match using a heuristic (e.g. most
recent), because that risks silently acting on the wrong customer's data.
</details>

**Q5.** A store's policy covers price-matching its own past sales but says
nothing about matching a competitor's price. A customer asks for a competitor
price match. What should the agent do, and why is "the policy doesn't forbid it,
so I'll approve it" the wrong reasoning?

<details><summary>ANSWER</summary>
Escalate. This falls under "policy silence/ambiguity" — when written policy
simply doesn't address a request, silence is not permission, and an incorrect
improvised answer can create liability or an inconsistent precedent. The agent
should not infer approval from adjacent-but-not-identical policy language.
</details>
