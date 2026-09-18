# Module 7 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 4.4 (Validation, Retry, and Feedback Loops)

**Q1.** An extraction fails validation. What should a retry request include to
maximize the chance of success, and why does simply re-asking the same question
perform worse?

<details><summary>ANSWER</summary>
It should append the *specific* validation errors found, alongside the original
document and the failed extraction, so the model sees exactly what was wrong.
Simply re-asking gives the model no concrete feedback to correct against, so it
may repeat the same mistake.
</details>

**Q2.** A `due_date` field keeps coming back empty for a specific invoice. Before
retrying, what should you check, and why?

<details><summary>ANSWER</summary>
Whether the due date actually exists anywhere in the source document provided.
If it's genuinely absent from the source, retrying is ineffective — no amount of
retrying conjures information the model was never given. Retry only helps for
format/structural errors, not missing source information.
</details>

**Q3.** What does a `detected_pattern` field enable that a plain "issue found /
no issue found" flag does not?

<details><summary>ANSWER</summary>
It records which specific code construct or pattern triggered each finding,
turning individual findings into aggregable data. Tracking which
`detected_pattern` values correlate with developer dismissals enables systematic
analysis of which patterns generate false positives — pointing at what to refine.
</details>

**Q4.** An invoice schema includes both `stated_total` and `calculated_total`.
What failure mode does this design catch, and how, without any extra validation
pass?

<details><summary>ANSWER</summary>
It catches semantic errors like line items not summing to the stated total. The
model extracts `stated_total` (what the document claims) and derives
`calculated_total` from the line items it also extracted; a downstream
comparison of the two automatically flags a discrepancy.
</details>

---

## Quiz — Lesson 4.5 (Batch Processing Strategies)

**Q1.** A pre-merge CI check needs Claude's review result within the pipeline's
2-minute timeout. Should this use the Batches API? Why or why not?

<details><summary>ANSWER</summary>
No — the Batches API gives no latency SLA (up to 24 hours) and is meant for
non-blocking, latency-tolerant workloads. A pre-merge check is a blocking
workflow and must use the synchronous Messages API despite its higher per-token
cost.
</details>

**Q2.** Why can't a workflow that needs Claude to call a tool, see the result,
and decide a follow-up action run inside a single batch request?

<details><summary>ANSWER</summary>
Each batch entry is one independent request processed once, returning one
result — there's no mechanism to pause an entry mid-processing, execute a
client-side tool call, and feed the result back into that same request before it
completes. A full agentic tool-use loop requires multiple turns.
</details>

**Q3.** Out of 10,000 submitted batch documents, 40 come back `errored`. How do
you identify and handle exactly those 40, and what field makes this possible?

<details><summary>ANSWER</summary>
`custom_id` — every batch request/result pair is tagged with the caller-supplied
`custom_id`. Look up the 40 `custom_id`s of the `errored` results, apply any
needed fix (e.g. chunking oversized documents), and resubmit only that subset
rather than the whole batch.
</details>

**Q4.** You need to guarantee results within a 30-hour SLA using a batch API with
a 24-hour maximum processing window. How often must you submit batches at
minimum, and what's the general formula?

<details><summary>ANSWER</summary>
At least every 6 hours. General formula: to guarantee results within N hours,
submit batches at an interval no longer than (N − 24) hours, since any individual
request might sit for the full 24-hour window in the worst case.
</details>

**Q5.** Why should you test an extraction prompt synchronously on a small sample
before submitting 50,000 documents as one batch?

<details><summary>ANSWER</summary>
A failed or low-quality large-scale batch run is expensive to detect and re-run.
Iterating on prompt quality against a small sample first catches format issues,
false positives, and edge-case failures cheaply, maximizing the batch's
first-pass success rate.
</details>

---

## Quiz — Lesson 4.6 (Multi-Instance and Multi-Pass Review)

**Q1.** Why is a Claude session less likely to catch its own mistakes when asked
to review code it just wrote, in the same session?

<details><summary>ANSWER</summary>
The session retains the reasoning context and assumptions that led to its
original choices, making it structurally less likely to question its own
decisions than a reviewer encountering the output fresh — it tends to re-justify
what it already committed to.
</details>

**Q2.** Would asking the generating session to use extended thinking to "double
check its work" solve the self-review limitation? Why or why not?

<details><summary>ANSWER</summary>
No — extended thinking still happens within the same request/response turn and
the same conversational context as the original generation, so it's still
anchored by the model's own prior generation choices. It's not a substitute for
an independent reviewer with no memory of that reasoning.
</details>

**Q3.** For a 40-file pull request, what review architecture avoids both
attention dilution and missing cross-file issues?

<details><summary>ANSWER</summary>
40 focused single-file passes (each looking only at that file's local logic) plus
one separate cross-file integration pass dedicated to tracing how data flows
between the changed files — rather than one pass attempting both at once.
</details>

**Q4.** What's the benefit of having a verification pass report a confidence
score alongside each finding, rather than a plain flag/no-flag?

<details><summary>ANSWER</summary>
It enables calibrated routing: high-confidence findings can be surfaced directly,
while low-confidence findings can be routed to an additional check (another pass
or a human) instead of being silently dropped or given the same authority as a
high-confidence finding.
</details>
