# Module 10 — Human Review, Confidence Calibration, and Information Provenance

Domain: Domain 5 — Context Management & Reliability (15% of exam)
Covers Task Statements 5.5, 5.6.

**Prerequisites from earlier modules:** hallucination in LLM outputs (Module 6,
Lesson 4.2); structured claim/output design (Module 6, Lesson 4.3; Module 1,
Lesson 1.3 — Structured Data Formats); coverage annotations in synthesis (Module
9, Lesson 5.3).

---

## Lesson 5.5 — Designing Human Review Workflows and Confidence Calibration

**Maps to:** Task Statement 5.5: Design human review workflows and confidence
calibration.

### Concept: Aggregate accuracy metrics masking segment-level failures

A single aggregate accuracy figure across an entire pipeline (e.g. "97%
overall") can be produced by uniformly good performance or by a mix of
near-perfect performance on most segments and much worse performance
concentrated in a specific one — and the aggregate number alone cannot
distinguish the two. Because errors tend to cluster in specific document types or
fields, validating accuracy broken down by document type and field is necessary
before deciding a pipeline is safe to run with reduced human review.

*Example:* A pipeline reporting 97% overall accuracy could be masking that
invoices are extracted at 99.5% while handwritten receipts — a small fraction of
volume — are extracted at 60%; the aggregate figure looks safe while the receipt
segment is not.

*Source:* exam-guide.txt (Domain 5, Task 5.5)
(Teaches: 5.5-K1, 5.5-S2)

### Concept: Stratified random sampling for error-rate measurement

Rather than reviewing only low-confidence extractions (where errors are expected
and already being caught), draw a representative random sample from each
stratum — document type, confidence band, field type — including high-confidence
extractions, and have humans verify it. Sampling only the low-confidence tail
misses the case where the model develops a new failure mode that happens to
produce high, but wrong, confidence.

*Example:* A monthly QA process randomly samples 2% of extractions from each of
{invoices, receipts, contracts} × {high, medium confidence} bands and has a human
verify each, rather than reviewing only model-flagged low-confidence extractions.

*Source:* exam-guide.txt (Domain 5, Task 5.5)
(Teaches: 5.5-K2, 5.5-S1)

### Concept: Field-level confidence scores calibrated against labeled validation sets

Having the model output a confidence score per extracted field, rather than one
score for the whole document, allows review effort to be routed at the
granularity where errors actually occur. A raw model-reported confidence number
is not automatically a calibrated probability, so the score must be calibrated:
compare model confidence to actual correctness on a labeled validation set, per
field, and set review thresholds based on that measured relationship.

*Example:* On a labeled validation set, 0.9-confidence extractions for "invoice
date" are correct 96% of the time, while 0.9-confidence extractions for
"line-item total" are only correct 82% of the time — so the review threshold for
line-item totals is set higher than for dates, despite the same raw confidence
number.

*Source:* exam-guide.txt (Domain 5, Task 5.5)
(Teaches: 5.5-K3, 5.5-S3)

### Concept: Validating accuracy by document type and field segment before automating

Before reducing or removing human review for high-confidence extractions,
accuracy must be validated broken down by document type and field segment — not
just in aggregate — because the aggregate-metric masking problem means a
pipeline that looks safe overall can still be unsafe for a specific segment. This
justifies automating away review for segments that check out while keeping
review for segments not yet separately verified.

*Example:* Before turning off human review for "high-confidence" extractions
across the board, a per-segment analysis finds scanned PDFs are meaningfully less
accurate — so review stays in place for scanned documents even as it's removed
for digital ones.

*Source:* exam-guide.txt (Domain 5, Task 5.5)
(Teaches: 5.5-K4)

### Concept: Confidence- and ambiguity-based routing to human review

Extractions should be routed to human review when either the model's calibrated
field-level confidence is low, or the source documents themselves are ambiguous
or contain contradictory information. Because reviewer capacity is limited,
routing should prioritize which extractions most need attention rather than
treating all flagged items as equal priority — complementing the general
hallucination-reduction pattern of allowing a model to express uncertainty.

*Example:* An extraction where the model has low confidence on a contract's
termination date, and two pages of the contract state different dates, is routed
ahead of one that is merely low-confidence on an otherwise unambiguous field.

*Source:* exam-guide.txt (Domain 5, Task 5.5); https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
(Teaches: 5.5-S4)

**Quiz:** see `quiz.md`, Lesson 5.5.

---

## Lesson 5.6 — Preserving Information Provenance and Handling Uncertainty in Multi-Source Synthesis

**Maps to:** Task Statement 5.6: Preserve information provenance and handle
uncertainty in multi-source synthesis.

### Prerequisite concept: Claude API citations feature

An API feature where documents are passed with `citations: {"enabled": true}`,
after which any claim in Claude's response that draws on a specific document is
annotated with a `citations` array identifying the exact cited text and the
source document — and the feature will not fabricate citations to documents not
actually provided. This is the concrete API mechanism structured claim-source
mapping and temporal-metadata handling are built on when Claude itself performs
the synthesis.

*Example:* Passing an order-tracking document with `"citations": {"enabled":
True}` causes any generated sentence drawn from that document to carry a citation
object naming the document title and the exact source span.

*Source:* https://platform.claude.com/cookbook/misc-using-citations
(Teaches: prerequisite for 5.6-K2, S1, K4, S4)

### Concept: Source attribution loss during summarization

When findings from multiple sources are compressed during summarization, the
mapping between each specific claim and the source it came from is one of the
first things lost, because a summarizer optimizing for brevity naturally merges
similar-sounding claims from different sources into one generalized statement.
Once lost, this mapping typically cannot be reconstructed from the summary
alone.

*Example:* Two sources report slightly different figures for the same statistic;
after unstructured summarization, the report says only "reports suggest values in
this range" with no way to tell which source said which number.

*Source:* exam-guide.txt (Domain 5, Task 5.6); supporting mechanism at
https://platform.claude.com/cookbook/misc-using-citations
(Teaches: 5.6-K1)

### Concept: Structured claim-source mappings preserved through synthesis

The fix for attribution loss: require every subagent to output findings as
structured claim-source mappings (claim text, source URL/document name, relevant
excerpt) rather than free-form prose, and require the downstream synthesis agent
to preserve and merge those mappings — not just the claims — as it combines
findings. The Claude API's citations feature implements this structurally,
linking each cited span back to a specific source document.

*Example:* A subagent's structured output is `{"claim": "Adoption grew 40%
YoY", "source": "2026 Industry Report, p.12", "excerpt": "...adoption increased
by 40% year over year..."}`, and the synthesis agent's final report keeps that
pairing intact rather than collapsing it into unsourced prose.

*Source:* https://platform.claude.com/cookbook/misc-using-citations
(Teaches: 5.6-K2, 5.6-S1)

### Concept: Annotating conflicting statistics rather than arbitrarily selecting one

When two or more credible sources report different values for the same fact, the
correct handling is not to silently pick one — which discards information and can
be wrong — but to preserve both values with their source attribution, explicitly
annotated as conflicting, and let the coordinator or a downstream decision-maker
decide how to reconcile them.

*Example:* Source A reports a market size of $4.2B (dated 2025) and Source B
reports $5.1B (dated 2026); the document-analysis output keeps both figures with
their sources and dates and flags them as conflicting, rather than reporting only
one number.

*Source:* exam-guide.txt (Domain 5, Task 5.6)
(Teaches: 5.6-K3, 5.6-S3)

### Concept: Temporal metadata to prevent false contradictions

Requiring subagents to include the publication or data-collection date of each
source allows a downstream synthesis step to correctly interpret differences
between sources as temporal change rather than misreading them as a
contradiction. Without a date attached, an old figure and a new figure look
identical to a genuinely conflicting pair. The citations mechanism supports this
via a document's `context` field, which can carry metadata like publication date.

*Example:* Source A (2023) reports headcount as 500; Source B (2026) reports
1,200. With dates attached, this reads as growth over three years; without
dates, it would misleadingly look like two sources disagreeing about the same
current number.

*Source:* exam-guide.txt (Domain 5, Task 5.6); https://platform.claude.com/cookbook/misc-using-citations
(Teaches: 5.6-K4, 5.6-S4)

### Concept: Distinguishing well-established from contested findings in report structure

A synthesis report should be structured with explicit sections separating
findings that are well-established (consistently supported across multiple
credible sources) from findings that are contested (sources disagree, or only
one lower-confidence source supports it), preserving each source's own
characterization and methodological context rather than flattening everything
into equally-confident-sounding prose.

*Example:* A report has a "Well-established findings" section (backed by three
independent sources using comparable methodology) and a separate "Contested /
single-source findings" section, rather than presenting both kinds of claim with
the same tone and certainty.

*Source:* exam-guide.txt (Domain 5, Task 5.6); https://www.anthropic.com/engineering/built-multi-agent-research-system
(Teaches: 5.6-S2)

### Concept: Rendering content types appropriately rather than uniformly

A multi-source synthesis output should render different kinds of content in the
format that content is naturally structured for — financial data as tables,
news-style findings as prose, technical findings as structured lists — rather
than forcing every content type through one uniform format, which typically both
loses information the natural format would have preserved and reads worse.

*Example:* A synthesis report presents quarterly revenue figures as an actual
comparison table, presents the general market narrative as flowing prose, and
presents technical compliance requirements as a numbered list.

*Source:* exam-guide.txt (Domain 5, Task 5.6)
(Teaches: 5.6-S5)

**Quiz:** see `quiz.md`, Lesson 5.6.

---

## Module 10 summary of bullets taught

Lesson 5.5: 5.5-K1, 5.5-K2, 5.5-K3, 5.5-K4, 5.5-S1, 5.5-S2, 5.5-S3, 5.5-S4
Lesson 5.6: 5.6-K1, 5.6-K2, 5.6-K3, 5.6-K4, 5.6-S1, 5.6-S2, 5.6-S3, 5.6-S4, 5.6-S5
