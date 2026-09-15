# Module 5 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 5.1 — Defining Success and Core Prompting Techniques

1. What does "SMART" stand for in the context of success criteria, and give
   one example of a vague goal turned into a SMART one.
2. How many examples does Anthropic recommend for multishot prompting, and
   what tags should wrap them?
3. What are the three increasing levels of structure for chain-of-thought
   prompting?
4. Name two of the four benefits of using XML tags to structure a prompt.

### Answers
1. Specific, Measurable, Achievable, Relevant, Time-bound (SMART, applied
   here as specific/measurable/achievable/relevant). Example: "good
   performance" becomes "F1 score ≥ 0.85."
2. 3–5 diverse, relevant examples, wrapped in `<example>`/`<examples>` tags.
3. A basic prompt ("Think step-by-step"), a guided prompt outlining
   specific steps, and a structured prompt separating reasoning and answer
   with XML tags (`<thinking>`/`<answer>`).
4. Any two of: clarity, accuracy, flexibility, parseability.

---

## Quiz 5.2 — Increasing Consistency and Reducing False Positives

1. Name three of the five documented techniques for increasing output
   consistency.
2. According to Anthropic's Code Review default prompt, what stated reason
   justifies not flagging low-confidence or subjective findings?
3. What is a "verification bar" in the context of review criteria?
4. What specific comparative claim about "be conservative"-style
   instructions is flagged as unverified in this course, and why?
5. Name three grounding techniques for reducing hallucinations.

### Answers
1. Any three of: precisely specify output format; prefill the response (not
   on newest models); constrain with concrete examples; ground in a fixed
   knowledge base; chain prompts into smaller subtasks.
2. "False positives erode trust and waste reviewer time."
3. A requirement that a finding be backed by evidence — e.g. a `file:line`
   citation supporting a claim about behavior — before it can be posted.
4. The claim that generic instructions like "be conservative" or "only
   report high-confidence findings" do NOT improve precision relative to
   specific categorical criteria. It's flagged as unverified because it was
   found only on a disqualified third-party aggregator site, not in
   Anthropic's official documentation.
5. Any three of: allowing "I don't know" instead of guessing; extracting
   word-for-word quotes before answering; requiring citations and
   retracting unsupported claims; chain-of-thought verification; best-of-N
   verification; iterative self-verification; restricting Claude to only
   provided documents.

---

## Quiz 5.3 — Structured Outputs and Tool-Call Reliability

1. What is the difference between JSON Outputs (`output_config.format`) and
   strict tool use (`strict: true`)?
2. Why does JSON Outputs not require a retry loop?
3. Name two JSON Schema features that structured outputs do NOT support.
4. How many times will Claude typically retry an invalid tool call before
   giving up, and what feature eliminates this retry need at the source?

### Answers
1. JSON Outputs constrains Claude's final text response to be valid JSON
   matching a schema; strict tool use guarantees a tool call's name and
   input parameters exactly match the tool's `input_schema`. They can be
   combined.
2. Because schema compliance is enforced through constrained decoding — a
   compiled grammar blocks invalid tokens during generation, so the output
   is guaranteed valid the first time.
3. Any two of: recursive schemas, most numeric length constraints, most
   string length constraints.
4. 2–3 times; `strict: true` eliminates the need by guaranteeing
   schema-conformant inputs from the start.

---

## Quiz 5.4 — Batch Processing and Prompt Caching for Scale

1. What are the size limits on a single Message Batch?
2. What discount does the Batches API give versus standard pricing?
3. Why are cache hits on Batches API requests "best-effort," and what is
   the recommended mitigation?
4. Why is the 1-hour cache preferred over the 5-minute cache for batch
   workloads specifically?

### Answers
1. 100,000 requests or 256 MB, whichever is reached first.
2. 50% off standard API pricing on input tokens, output tokens, and special
   tokens.
3. Because asynchronous batch requests can be processed concurrently and in
   any order, so there's no guarantee later requests land while an earlier
   cache write is still warm. Mitigation: send a single priming request
   with the shared prefix and a 1-hour cache block first, then submit the
   rest of the batch once that completes.
4. Because batch completion commonly falls in the 5-minute-to-1-hour range,
   so the shorter 5-minute cache would likely expire before most of the
   batch's requests land.

---

## Quiz 5.5 — Multi-Instance Review Architectures

1. Name the four basic agentic workflow building blocks.
2. What are the three multi-instance review architectures, and in one
   sentence each, what does each do?
3. When is evaluator-optimizer NOT an effective pattern to use?
4. Is human oversight still recommended even with a multi-instance voting
   architecture in place?

### Answers
1. The augmented LLM, prompt chaining, routing, and parallelization by
   sectioning.
2. Evaluator-optimizer: one call generates, a second evaluates and gives
   feedback in a loop. Orchestrator-workers: a central LLM dynamically
   decomposes a task and delegates to specialized workers, then synthesizes
   results. Parallelization by voting: the same task runs through multiple
   independent calls and results are combined via voting/consensus.
3. When first-attempt quality already suffices, or when evaluation criteria
   are subjective or unclear.
4. Yes — human oversight is still recommended as a final check across all
   of these architectures.
