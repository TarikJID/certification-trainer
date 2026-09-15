# Module 6 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 6.1 — Context Window Depth

1. What is "context rot," and why does Anthropic attribute it to how
   transformer attention works, rather than to the context window being
   literally full?
2. What is the needle-in-a-haystack methodology, and what problem was it
   originally used to measure?
3. What accuracy did Claude 2.1 achieve on the original needle test, and
   what single change raised it to 98%?
4. Is "lost in the middle" a phrase found verbatim in Anthropic's own
   documentation, or is it sourced elsewhere? Where does the corroborating
   vendor evidence come from instead?
5. What does the `<system_warning>` tag report to the model, and when is it
   injected?

### Answers
1. Context rot is the degradation of model accuracy and recall as token
   count grows, even without hitting the window limit — attributed to
   attention creating pairwise token relationships that get stretched
   thinner as context grows, and to less training exposure to very long
   sequences.
2. A methodology where a target fact ("the needle") is inserted into a
   large body of unrelated text ("the haystack") and the model is asked to
   retrieve it; it was used to measure Claude 2.1's long-context recall.
3. 27% initially; appending "Here is the most relevant sentence in the
   context:" to the start of the expected response raised it to 98%.
4. It is sourced to the exam guide (Domain 5, Task 5.1), not found verbatim
   in Anthropic's own documentation. The corroborating vendor evidence is
   the needle-in-a-haystack measurement and the long-context prompting
   guidance's document-placement recommendation — related, but not framed
   identically.
5. It reports tokens used and remaining after a tool call; it's injected
   automatically after each tool call on supporting models.

---

## Quiz 6.2 — Managing Context at Scale

1. What is the default and minimum trigger (in input tokens) for server-side
   compaction?
2. What is the difference between `clear_tool_uses_20250919` and
   `clear_thinking_20251015`?
3. What beta header is required for context editing?
4. Roughly how much can prompt caching cut input-token cost, and what is
   required for a cache hit to occur?
5. Does prompt caching reduce what counts toward the context window, or
   only what a cache hit costs?

### Answers
1. Default 150,000 input tokens; minimum 50,000.
2. `clear_tool_uses_20250919` removes the oldest tool call/result pairs once
   a trigger is exceeded, keeping a configurable number of recent pairs;
   `clear_thinking_20251015` clears older extended-thinking blocks while
   optionally preserving a configurable number of recent ones.
3. `context-management-2025-06-27`.
4. Up to ~90%; the cache breakpoint must be on content that is
   byte-identical across requests.
5. Only what a cache hit costs — it does not reduce what counts toward the
   context window.

---

## Quiz 6.3 — Long-Running Agents and Large Codebase Exploration

1. What two agent roles make up Anthropic's documented long-running-agent
   harness pattern, and what does each do?
2. Why is summarization/compaction alone not treated as sufficient for
   multi-day jobs?
3. What is the key difference between "context isolation via subagents" and
   "structured note-taking," even though both counteract context problems?
4. What vendor-documented product feature supports structured note-taking
   outside the context window?
5. Under what exam-guide task and pattern name is structured note-taking
   also described?

### Answers
1. An initializer agent (runs once, sets up environment, expands the prompt
   into a structured requirements/feature list, writes a boot script) and a
   coding agent (woken up repeatedly across many sessions, reads git
   history and a progress file to reconstruct state, makes incremental
   progress, verifies it, leaves a handoff for the next session).
2. Because the harness pattern treats context reset as unavoidable for very
   long jobs, and periodically does a full session teardown and rebuild
   from a structured handoff artifact — summarization within a single
   session isn't designed to span that kind of multi-day reset.
3. Context isolation via subagents delegates exploration to a *separate
   agent* with its own context window; structured note-taking is an
   *in-session* pattern where a single agent externalizes findings to a
   file as it goes, without delegating or resetting the session.
4. The memory tool (released in public beta alongside Claude Sonnet 4.5),
   which stores and consults information outside the context window
   through a file-based system.
5. Domain 5, Task 5.4, under the name "scratchpad files."

---

## Quiz 6.4 — Error Propagation and Escalation to Humans

1. What happens to a subagent's result if it hits an API error (e.g. a rate
   limit) partway through its task — is it delivered as a normal (if
   partial) result?
2. What two-stage design does the Claude Code auto-mode classifier use to
   review actions?
3. What numeric thresholds trigger human escalation in Claude Code's
   auto-mode design, and what happens short of reaching them?
4. What is "case/ticket triage and routing," in general terms, and why is
   it logically prior to the question of when a case should escalate to a
   human?
5. According to the exam guide, name the three appropriate escalation
   triggers for a human handoff.
6. What two things does the exam guide name as unreliable proxies for
   actual case complexity, and what should the system do instead when a
   case is simply ambiguous?

### Answers
1. No — an error that ends a subagent early is never delivered as that
   subagent's result; the parent must detect and explicitly handle its
   absence.
2. A fast, single-token "err on the side of blocking" filter, followed by a
   slower chain-of-thought review only for actions the filter flagged.
3. 3 consecutive denials or 20 total denials in a session; short of that, a
   denied action is returned to the agent with an instruction to find a
   safer path.
4. The general support-operations concept of a case or ticket being
   classified and routed to a handler (automated or human) based on its
   characteristics. It's logically prior because a case must already be
   classified/routed somewhere before a decision about escalating that
   routing to a human specifically can even be asked.
5. Customer requests for a human, policy exceptions/gaps, and inability to
   make meaningful progress.
6. Sentiment-based escalation and self-reported confidence scores; for an
   ambiguous case, the system should request additional identifiers from
   the user rather than guessing from a heuristic.

---

## Quiz 6.5 — Ambiguity, Confidence, and Provenance

1. Contrast `default_to_action` and `do_not_act_before_instructions`.
2. Why is "confidence calibration" flagged as not a single named,
   first-party Anthropic framework, even though it's a real, sourced
   concept in this course?
3. What is "statistical sampling basics" (populations, strata, sample
   validity), and why does "stratified random sampling" as a QA technique
   not make sense without it?
4. What is "stratified random sampling" designed to catch that
   spot-checking only low-confidence outputs would miss?
5. What is "retrieval / document-grounding basics," and how is it more
   basic than Module 5's grounding techniques (e.g. extracting quotes to
   reduce hallucination)?
6. What does the Citations API guarantee that asking the model to quote its
   own sources in prose does not?
7. In multi-document synthesis, where should documents be placed relative
   to the query, and why does this matter given Lesson 6.1?

### Answers
1. `default_to_action` makes the model proactive — infer the likely
   intended action and proceed, using tools to fill in missing details.
   `do_not_act_before_instructions` makes it conservative — report and
   recommend rather than act, unless clearly instructed to make a change.
2. Because it's composed from two separately documented mechanisms
   (prompted uncertainty expression, and denial-count-based escalation)
   rather than described by Anthropic as one unified, named methodology.
3. The general statistical concept of drawing a sample from a population,
   and specifically of dividing the population into strata (subgroups,
   e.g. by confidence band) and sampling from each stratum rather than the
   population as one undifferentiated group. Stratified random sampling as
   a QA technique is just this general concept applied with "confidence
   band" as the stratifying variable — it doesn't make sense without first
   understanding what a stratum is and why sampling by stratum differs from
   plain random sampling.
4. A new, systematic error affecting outputs the model itself rated as
   high-confidence — since a review process that only checks low-confidence
   outputs would never sample those.
5. It's the general practice of supplying external source documents in the
   prompt/context so answers can be grounded in and traced back to
   specific text. It's more basic than Module 5's quote-extraction
   technique because that technique is a specific method built on top of
   already having source documents in context — you have to be supplying
   documents at all before you can ask the model to extract quotes from
   them.
6. That citations are guaranteed valid pointers into the supplied documents
   — the model quoting itself in prose carries no such guarantee.
7. Above the instructions/query (near the top of the prompt); this matters
   because it's consistent with the "lost in the middle"/beginning-and-end
   recall pattern from Lesson 6.1 — content at the very end (the query) and
   the very beginning (the documents) is more reliably processed.
