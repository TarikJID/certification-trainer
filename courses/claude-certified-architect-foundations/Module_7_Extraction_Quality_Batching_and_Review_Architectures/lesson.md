# Module 7 — Extraction Quality, Batch Processing, and Review Architectures

Domain: Domain 4 — Prompt Engineering & Structured Output (20% of exam)
Covers Task Statements 4.4, 4.5, 4.6.

**Prerequisites from earlier modules:** structured error categorization (Module
3, Lesson 2.2); tool use with JSON schemas and semantic vs syntax errors (Module
6, Lesson 4.3); Messages API request/response basics (Module 1, Lesson 1.1).

---

## Lesson 4.4 — Validation, Retry, and Feedback Loops for Extraction Quality

**Maps to:** Task Statement 4.4: Implement validation, retry, and feedback loops
for extraction quality.

### Prerequisite concept: Structured/typed error results from the API

Claude API responses and batch results classify outcomes into typed categories
(e.g. `succeeded`, `errored`, `expired` for batch results; structured error
objects with a `type`). This typed-error pattern is the general mechanism
"retry with error feedback" builds on: a specific, structured error is something
you can append to a follow-up prompt or branch retry logic on, unlike an
undifferentiated failure. It echoes the structured error taxonomy you learned for
MCP tools in Module 3, Lesson 2.2.

*Example:* A batch result of `{"result":{"type":"errored","error":{"error":
{"type":"invalid_request_error"}}}}` tells the caller the request body needs
fixing before resubmission, distinct from a transient server error worth a plain
retry.

*Source:* https://platform.claude.com/docs/en/build-with-claude/batch-processing
(Teaches: prerequisite for 4.4-K1, K4, S1)

### Concept: Retry-with-error-feedback

When a validated extraction fails, the retry most likely to succeed appends the
*specific* validation errors found (not a generic "try again") to a follow-up
request, alongside the original document and the failed extraction, so the model
can see exactly what was wrong and self-correct against concrete feedback.

*Example:* A follow-up prompt includes the original invoice text, the first
extraction attempt, and "line_items sum to $210 but total is $250 — correct the
discrepancy," rather than simply re-asking the original question.

*Source:* exam-guide.txt (Domain 4, Task 4.4)
(Teaches: 4.4-K1, 4.4-S1)

### Concept: Limits of retry (absent information vs format errors)

Retry-with-feedback is only effective for errors caused by *how* the model
formatted or structured its answer. It is ineffective when the required
information simply does not exist in the source document provided — no amount of
retrying will conjure information the model was never given.

*Example:* A missing `due_date` because the invoice genuinely has no due date
will not be fixed by retrying; a missing `due_date` because the model put it in
the wrong nested object (a structural error) likely will be.

*Source:* exam-guide.txt (Domain 4, Task 4.4)
(Teaches: 4.4-K2, 4.4-S2)

### Concept: Feedback loop design with detected_pattern

Adding a `detected_pattern` field to each structured finding — recording which
specific code construct or textual pattern triggered it — turns individual
findings into data that can be aggregated later. Tracking which
`detected_pattern` values correlate with developer dismissals enables systematic
analysis of which patterns generate false positives.

*Example:* Logging `detected_pattern: "unchecked_optional_chaining"` on every
finding of that type lets a later analysis show it accounts for 60% of
dismissals, pointing directly at which category to refine or disable.

*Source:* exam-guide.txt (Domain 4, Task 4.4)
(Teaches: 4.4-K3, 4.4-S3)

### Concept: Semantic validation errors vs schema syntax errors

Schema syntax errors (malformed JSON, wrong field types, missing required
fields) are eliminated by enforcing extraction through tool use (Module 6, Lesson
4.3). Semantic validation errors are a distinct category tool use does *not*
eliminate: values individually well-typed but collectively inconsistent.

*Example:* A validator checking `sum(line_items.amount) == total` catches a
semantic error no JSON Schema constraint alone could catch, since both values
individually satisfy `type: number`.

*Source:* exam-guide.txt (Domain 4, Task 4.4)
(Teaches: 4.4-K4)

### Concept: Self-correction validation flows (calculated vs stated values, conflict flags)

A design pattern for catching semantic errors without external logic: have the
extraction schema itself request both a `stated_total` (what the document
claims) and a `calculated_total` (derived by the model from the line items it
also extracted), so a downstream comparison flags a discrepancy automatically. A
parallel pattern adds a `conflict_detected` boolean when the source document
itself contains internally inconsistent data.

*Example:* An invoice schema with both `stated_total: 250` and
`calculated_total: 210` lets a downstream check flag the invoice for human review
without a separate validation pass.

*Source:* exam-guide.txt (Domain 4, Task 4.4)
(Teaches: 4.4-S4)

**Quiz:** see `quiz.md`, Lesson 4.4.

---

## Lesson 4.5 — Designing Efficient Batch Processing Strategies

**Maps to:** Task Statement 4.5: Design efficient batch processing strategies.

### Prerequisite concept: Messages API request/response basics — recap

Recall from Module 1, Lesson 1.1: every Claude API call carries `model`,
`max_tokens`, a `messages` array, and optionally `system`, `tools`, and
`tool_choice`. The Message Batches API wraps many such request bodies (each
under a `params` object plus a `custom_id`) for asynchronous, discounted
processing.

*Example:* `{"model":"claude-opus-5","max_tokens":1024,"messages":[{"role":
"user","content":"Hello"}]}` is the same `params` shape used both directly
against `/v1/messages` and nested inside a batch request.

*Source:* https://platform.claude.com/docs/en/build-with-claude/batch-processing
(Teaches: prerequisite for 4.5-K1, K4, S1, S2, S3, S4)

### Concept: Message Batches API fundamentals

The Message Batches API processes large volumes of Messages requests
asynchronously at a 50% discount on input and output token pricing versus the
synchronous API. Most batches finish in under an hour, but there is no latency
SLA — a batch can take up to 24 hours, after which unprocessed requests expire.

*Example:* Submitting 50,000 document-classification requests as one batch costs
half what 50,000 individual synchronous calls would, but with no guarantee any of
them finish within, say, 10 minutes.

*Source:* https://platform.claude.com/docs/en/build-with-claude/batch-processing
(Teaches: 4.5-K1)

### Concept: Batch-appropriate workloads vs blocking workflows

Because the Batches API trades guaranteed low latency for cost savings, it fits
non-blocking, latency-tolerant workloads (overnight reports, weekly audits,
nightly test generation). It's the wrong choice for blocking workflows such as
pre-merge CI checks, where something is stalled waiting on the result — those
belong on the synchronous Messages API despite its higher per-token cost.

*Example:* A nightly job regenerating test suites can run entirely on the
Batches API; a pre-merge PR check that blocks a developer's merge button must use
the synchronous API so it returns within the CI step's timeout.

*Source:* exam-guide.txt (Domain 4, Task 4.5)
(Teaches: 4.5-K2, 4.5-S1)

### Concept: Batch API's lack of mid-request multi-turn tool calling

Each entry in a Message Batch is one independent Messages request, processed once
and returning one result; there is no mechanism to pause a batch entry
mid-processing, execute a client-side tool call it produced, and feed the
`tool_result` back into that same request before it completes. A full agentic
tool-use loop cannot happen within a single batch request.

*Example:* A workflow needing Claude to call a tool, inspect the result, and
decide a follow-up action cannot complete that loop inside one batch entry; each
such turn would need to be its own separate batch (or synchronous) request.

*Source:* exam-guide.txt (Domain 4, Task 4.5); request/response shape at
https://platform.claude.com/docs/en/build-with-claude/batch-processing
(Teaches: 4.5-K3)

### Concept: custom_id request/response correlation

Every batch request carries a caller-supplied `custom_id`, and every result is
tagged with that same `custom_id`. Because batch results can return in a
different order than submitted, `custom_id` is the only reliable way to match a
result back to its request — and what makes targeted failure handling possible:
only requests whose results are `errored` or `expired` need resubmission.

*Example:* Out of 10,000 submitted documents, 40 come back `errored` for
exceeding the context window; the caller looks up those 40 `custom_id`s, chunks
just those documents, and resubmits only that subset.

*Source:* https://platform.claude.com/docs/en/build-with-claude/batch-processing;
exam-guide.txt
(Teaches: 4.5-K4, 4.5-S3)

### Concept: Calculating batch submission frequency against an SLA

Because batch processing offers no latency guarantee up to its 24-hour cap,
meeting a tighter SLA requires backing the batch's worst-case processing time out
of the required turnaround and submitting on a schedule frequent enough to absorb
it. To guarantee results within N hours, batches must be submitted at an interval
no longer than (N − 24) hours.

*Example:* To guarantee a 30-hour SLA against a 24-hour maximum processing
window, batches must be submitted at least every 6 hours (the exam guide's own
worked figure of 4-hour windows is a stricter, safety-margined version).

*Source:* exam-guide.txt (Domain 4, Task 4.5); batch timing at
https://platform.claude.com/docs/en/build-with-claude/batch-processing
(Teaches: 4.5-S2)

### Concept: Prompt refinement on a sample before batch-scale submission

Because a failed or low-quality large-scale batch run is expensive to detect and
re-run, the efficient sequencing is to iterate on prompt quality against a small
synchronous sample first — catching format issues, false positives, and edge-case
failures cheaply — before submitting the full volume as a batch.

*Example:* Testing an extraction prompt synchronously against 20 representative
documents, fixing issues that surface, and only then submitting the remaining
50,000 documents as a single batch.

*Source:* exam-guide.txt (Domain 4, Task 4.5)
(Teaches: 4.5-S4)

**Quiz:** see `quiz.md`, Lesson 4.5.

---

## Lesson 4.6 — Designing Multi-Instance and Multi-Pass Review Architectures

**Maps to:** Task Statement 4.6: Design multi-instance and multi-pass review
architectures.

### Prerequisite concept: Extended thinking (visible reasoning within a turn)

Extended thinking lets Claude produce an internal reasoning process (thinking
blocks) before its final answer, within the same request/response turn and the
same conversational context as the content it is reasoning about. Because that
reasoning happens inside the same session that produced the original output, it
still has access to — and is anchored by — the model's own prior generation
choices.

*Example:* Asking the same Claude session that just wrote a function to "think
hard and double check your work" still reasons from inside the context where it
already committed to its design choices.

*Source:* https://platform.claude.com/docs/en/build-with-claude/extended-thinking
(Teaches: prerequisite for 4.6-K1, K2)

### Concept: Self-review limitations (retained reasoning context)

A Claude instance that generated output and is then asked, in the same session,
to review it retains the reasoning context and assumptions that led to its
original choices. That retained context makes it structurally less likely to
question its own decisions than a reviewer encountering the output fresh.

*Example:* A session that wrote an authentication check and is asked "review
your code for bugs" in the same conversation is anchored on the design choices it
already made.

*Source:* exam-guide.txt (Domain 4, Task 4.6)
(Teaches: 4.6-K1)

### Concept: Independent review instances outperform self-review

A second, independent Claude instance — one that never saw the generating
instance's reasoning trace, only the final output — catches subtle issues more
effectively than self-review instructions, and more effectively than extended
thinking within the same session (which still reasons from inside the same
generation context). The skill: route generated output through a second, freshly
started Claude call for verification.

*Example:* An automated code-review pipeline spawns a fresh review instance
containing only the diff and review criteria (no prior conversation), rather than
asking the generating session to "now review what you just wrote."

*Source:* exam-guide.txt (Domain 4, Task 4.6); independent-agent verification in
production at https://claude.com/blog/code-review
(Teaches: 4.6-K2, 4.6-S1)

### Concept: Multi-pass review — per-file and cross-file passes

For reviews spanning many files, one pass across the entire change set risks
attention dilution and contradictory findings across parts of the same review.
Splitting into focused per-file passes for local issues, plus a separate
cross-file integration pass for issues that only exist across file boundaries,
keeps each pass's attention concentrated on a scope it can cover well. This is
the same avoiding-attention-dilution logic you saw applied to code review task
decomposition in Module 2, Lesson 1.6.

*Example:* A 40-file pull request gets 40 focused single-file passes plus one
additional pass whose entire job is tracing data flow between changed files.

*Source:* exam-guide.txt (Domain 4, Task 4.6)
(Teaches: 4.6-K3, 4.6-S2)

### Concept: Confidence self-reporting for calibrated review routing

Having a verification pass report a confidence value alongside each finding —
rather than only a binary flag/no-flag — enables calibrated routing downstream:
high-confidence findings surfaced directly, low-confidence findings routed to an
additional check instead of being either silently dropped or surfaced with equal
authority.

*Example:* A verification pass returns `{"finding": "...", "confidence": 0.9}`
for a clear-cut security issue and `{"finding": "...", "confidence": 0.4}` for a
borderline style concern; the pipeline auto-posts the first and routes the second
to a secondary check.

*Source:* exam-guide.txt (Domain 4, Task 4.6)
(Teaches: 4.6-S3)

**Quiz:** see `quiz.md`, Lesson 4.6.

---

## Module 7 summary of bullets taught

Lesson 4.4: 4.4-K1, 4.4-K2, 4.4-K3, 4.4-K4, 4.4-S1, 4.4-S2, 4.4-S3, 4.4-S4
Lesson 4.5: 4.5-K1, 4.5-K2, 4.5-K3, 4.5-K4, 4.5-S1, 4.5-S2, 4.5-S3, 4.5-S4
Lesson 4.6: 4.6-K1, 4.6-K2, 4.6-K3, 4.6-S1, 4.6-S2, 4.6-S3
