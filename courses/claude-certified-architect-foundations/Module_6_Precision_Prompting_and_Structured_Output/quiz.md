# Module 6 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 4.1 (Explicit Criteria and False Positives)

**Q1.** Contrast "check that comments are accurate" with "flag comments only when
claimed behavior contradicts actual code behavior." Which is more effective and
why?

<details><summary>ANSWER</summary>
The second is more effective — it's an explicit, checkable criterion, while the
first is vague and leaves "accurate" undefined. Explicit criteria give the model
(and a human auditor) a concrete test to apply, rather than an open-ended
standard it must infer.
</details>

**Q2.** Why doesn't telling a review prompt to "only report high-confidence
findings" reliably reduce false positives?

<details><summary>ANSWER</summary>
It asks the model to self-estimate a confidence threshold rather than apply a
defined rule, and the model's internal notion of "high confidence" isn't
calibrated to the reviewer's actual tolerance. Specific categorical criteria
(which categories to report vs. skip) outperform confidence-based filtering.
</details>

**Q3.** A "minor style" finding category is wrong 40% of the time. Why might this
hurt trust in the "security" category too, and what's the recommended
mitigation?

<details><summary>ANSWER</summary>
A high false-positive rate in one category erodes trust in the whole system —
once a reviewer learns to distrust and skim past one noisy category, they tend to
under-scrutinize adjacent, accurate categories too. Mitigation: temporarily
disable the high-false-positive category to restore trust in the rest, while its
prompt is iterated on.
</details>

**Q4.** Why pair severity level names (critical/high/medium/low) with concrete
code examples?

<details><summary>ANSWER</summary>
Defining severity purely by name invites inconsistent classification because
terms like "critical" mean different things across reviewers and runs. A
concrete example anchors each level to a specific reference point, producing
more consistent classification.
</details>

---

## Quiz — Lesson 4.2 (Few-Shot Prompting)

**Q1.** A prompt describes the desired output format in detailed prose but output
is still inconsistently formatted. What technique should you apply, and why is it
more effective than more prose?

<details><summary>ANSWER</summary>
Few-shot prompting — add a small set of worked examples. Examples show, rather
than describe, the exact shape and quality bar output must meet, which is harder
for a model to under- or over-generalize from than a prose rule.
</details>

**Q2.** For a tool-selection task where a request could plausibly map to either
of two tools, what should a good few-shot example include beyond just the
correct answer?

<details><summary>ANSWER</summary>
The reasoning for why the chosen action was preferred over the other plausible
alternative — not just the final input/output pair — so the model learns the
discriminating signal, not just a lookup-table case.
</details>

**Q3.** Why does showing both an acceptable pattern example and a genuinely buggy
(but superficially similar) pattern example help the model generalize to novel,
unseen inputs?

<details><summary>ANSWER</summary>
It teaches the model the underlying judgment / distinguishing feature between the
two, rather than a lookup table of specific cases — this lets it correctly judge
a third, unseen pattern by applying the same distinguishing logic.
</details>

**Q4.** How do few-shot examples reduce hallucination specifically in extraction
tasks, and give an example of the kind of hard case they address.

<details><summary>ANSWER</summary>
They show concrete instances of correct handling for hard cases that otherwise
invite fabrication — e.g., informal measurements like "~2 cups" correctly
normalized, or a citation embedded mid-sentence correctly attributed — preventing
the model from fabricating a precise value or returning an empty field for a
format it hasn't seen described in prose.
</details>

---

## Quiz — Lesson 4.3 (Structured Output via Tool Use and JSON Schema)

**Q1.** Why does defining an extraction tool with a JSON Schema `input_schema`
and having Claude call it produce more reliable structured output than asking
Claude to "return the data as JSON" in prose?

<details><summary>ANSWER</summary>
The model's tool call must conform to the declared parameter shape, which
eliminates the class of errors from asking the model to emit raw JSON as free
text (missing quotes, trailing commas, truncated braces) — the structured data is
read directly from the `tool_use` block's `input`.
</details>

**Q2.** An invoice extraction is schema-valid: every field has the right type,
`total` is present. But the three `line_items` amounts sum to $210 while `total`
reads $250. Did tool-use-enforced structured output fail here? Explain.

<details><summary>ANSWER</summary>
No schema violation occurred — this is a semantic error, not a syntax error.
Strict JSON schemas via tool use eliminate syntax errors but cannot by themselves
prevent values that are individually well-formed but collectively wrong.
</details>

**Q3.** A `document_type` field needs to handle document types not anticipated at
design time. How should the schema be designed?

<details><summary>ANSWER</summary>
Use an enum with an escape-hatch `"other"` value paired with a free-text `detail`
field (e.g. `document_type_detail`), plus an explicit `"unclear"` value for
genuinely ambiguous cases — giving the model a truthful option instead of forcing
a best-guess match to an existing enum value.
</details>

**Q4.** Multiple extraction tools exist (`extract_invoice`, `extract_receipt`,
`extract_purchase_order`) and you don't know the incoming document's type ahead
of time. Which `tool_choice` setting guarantees a structured call while still
letting the model pick the right schema?

<details><summary>ANSWER</summary>
`tool_choice: {"type": "any"}` — it forces Claude to call one of the available
tools rather than returning conversational text, while still letting the model
choose which tool (schema) best fits the document.
</details>

**Q5.** A `customer.company` field is marked `required`, but many source
documents (personal purchases) have no company name. What happens, and how
should the schema be fixed?

<details><summary>ANSWER</summary>
The model is forced to either fail or fabricate a company name to satisfy the
required constraint. Fix: mark the field optional and nullable (not in
`required`, typed to accept `null`), so the model can correctly return `null`
when the information legitimately isn't present.
</details>
