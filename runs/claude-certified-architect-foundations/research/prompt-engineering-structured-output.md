# Domain 4 — Prompt Engineering & Structured Output: Research

**Bullets received: 47 (22 Knowledge, 25 Skills). Bullets covered: 47/47.**
**Concepts in this file: 40 (32 key, 8 prerequisite).**

Source note: many bullets in this domain reproduce a specific scenario or worked
example (e.g. the code-review criteria wording, the `detected_pattern` /
`calculated_total` extraction fields, the self-review limitation) that is not
independently documented on Anthropic's product-docs site — it is the exam guide's
own illustrative content. Per the brief, the exam guide's own wording is an
acceptable official source on its own. Where a matching product-doc page exists
(prompting best practices, tool use, structured outputs, batch processing, extended
thinking) it is cited alongside or instead, and is not treated as senior to the guide.

---

## Prerequisite concepts

- Concept: JSON Schema fundamentals for tool/output definitions
  Type: prerequisite
  Teaches: (supports 4.3-K1, 4.3-K4, 4.3-S1, 4.3-S4, 4.3-S5, 4.3-S6)
  Definition: A JSON Schema describes the shape of a JSON object: a `type`, a
  `properties` map (each with its own `type`, optional `description`, and optional
  `enum`), a `required` array naming which properties must be present, and — for
  fields that may legitimately be absent — either omission from `required` or a
  `type` union such as `["string", "null"]` (or an `anyOf` with a `null` branch) to
  mark the field nullable. This vocabulary is what both Claude's tool `input_schema`
  and the Structured Outputs `json_schema` response format are built on.
  Example: `{"type":"object","properties":{"name":{"type":"string"},"phone":{"type":["string","null"]}},"required":["name"]}`
  makes `name` mandatory and `phone` optional-but-typed.
  Source: https://platform.claude.com/docs/en/build-with-claude/structured-outputs

- Concept: Tool use round trip (tool_use / tool_result blocks)
  Type: prerequisite
  Teaches: (supports 4.3-K1, 4.3-K2, 4.3-S1, 4.3-S2, 4.3-S3)
  Definition: When a request includes a `tools` array, Claude's response can contain
  a `tool_use` content block naming a tool and giving its `input` object (populated
  according to that tool's `input_schema`). For client tools, the calling application
  reads this block, executes the corresponding logic, and — for a true multi-turn
  tool loop — sends a `tool_result` block back in a follow-up request so Claude can
  continue. For one-shot extraction use, the `tool_use.input` object itself *is* the
  structured output; no further turn is required.
  Example: A request with a `get_weather` tool returns
  `{"type":"tool_use","name":"get_weather","input":{"location":"San Francisco, CA"}}`;
  the caller reads `input` directly as the extracted structured data.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

- Concept: Messages API request/response basics
  Type: prerequisite
  Teaches: (supports 4.5-K1, 4.5-K4, 4.5-S1, 4.5-S2, 4.5-S3, 4.5-S4)
  Definition: Every Claude API call — synchronous or batched — is a Messages API
  request carrying `model`, `max_tokens`, a `messages` array, and optionally `system`,
  `tools`, and `tool_choice`. The synchronous Messages endpoint returns one response
  per request; the Message Batches API wraps many such request bodies (each under a
  `params` object plus a `custom_id`) for asynchronous, discounted processing.
  Understanding the plain synchronous shape is a prerequisite for reasoning about how
  batching changes (and doesn't change) that contract.
  Example: `{"model":"claude-opus-5","max_tokens":1024,"messages":[{"role":"user","content":"Hello"}]}`
  is the same `params` shape used both directly against `/v1/messages` and nested
  inside a batch request.
  Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing

- Concept: "Be clear and direct" — explicit instruction baseline
  Type: prerequisite
  Teaches: (supports 4.1-K1, 4.1-K2, 4.1-S1, 4.1-S3)
  Definition: Anthropic's general prompting guidance treats explicit, specific
  instructions as the baseline technique underneath every more advanced pattern
  (few-shot, structured output, criteria design): Claude should be treated as a
  capable but context-free new hire, and vague requests should be replaced with
  precise statements of the desired output and constraints. This is the general
  principle that domain-specific criteria design (Task 4.1) applies to the false
  positive problem.
  Example: The guide's own contrast — "flag comments only when claimed behavior
  contradicts actual code behavior" versus "check that comments are accurate" —
  is a direct instance of replacing a vague instruction with an explicit,
  checkable one.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Hallucination in LLM outputs
  Type: prerequisite
  Teaches: (supports 4.2-K4, 4.2-S4, 4.2-S5)
  Definition: Hallucination is Anthropic's own term for text a model generates that
  is factually incorrect or inconsistent with the source material it was given —
  for extraction tasks, this shows up as invented field values, fabricated numbers,
  or answers not actually supported by the document. Anthropic's guardrails guidance
  lists concrete mitigations: allowing the model to express uncertainty, grounding
  answers in direct quotes, verifying claims against the source, and restricting the
  model to only the provided material. Few-shot examples (the Task 4.2 skill) are a
  complementary mitigation: they show the model what correct, grounded extraction
  looks like for the exact document types it will see.
  Example: Given an invoice missing a due date, an ungrounded model may hallucinate
  a plausible-looking date; a hallucination-aware prompt instructs it to return
  `null` and cite why, rather than invent a value.
  Source: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations

- Concept: Multishot/few-shot prompting with `<example>` tags
  Type: prerequisite
  Teaches: (supports 4.2-K1, 4.2-K2, 4.2-K3, 4.2-S1, 4.2-S2, 4.2-S3)
  Definition: Anthropic's prompting guidance names "few-shot" or "multishot"
  prompting — including a small number of worked examples in the prompt — as one of
  the most reliable ways to steer output format, tone, and structure, more reliable
  than prose instructions alone. Effective examples are relevant (mirror the real use
  case), diverse (cover edge cases without introducing unintended patterns), and
  structured (wrapped in `<example>`/`<examples>` XML tags so Claude can tell them
  apart from instructions). The guidance recommends roughly 3-5 examples as a
  starting point.
  Example: Wrapping two worked code-review judgments in
  `<examples><example>...</example><example>...</example></examples>` before asking
  Claude to review a new diff.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Extended thinking (visible reasoning within a turn)
  Type: prerequisite
  Teaches: (supports 4.6-K1, 4.6-K2)
  Definition: Extended thinking lets Claude produce an internal reasoning process
  (thinking blocks) before its final answer, within the same request/response turn
  and the same conversational context as the content it is reasoning about. Because
  that reasoning happens inside the same session that produced the original output,
  it still has access to — and is anchored by — the model's own prior generation
  choices, which is exactly the property that limits its usefulness as a substitute
  for an independent reviewer (see 4.6-K1/K2 below).
  Example: Asking the same Claude session that just wrote a function to "think hard
  and double check your work" still reasons from inside the context where it already
  committed to its design choices.
  Source: https://platform.claude.com/docs/en/build-with-claude/extended-thinking

- Concept: Structured/typed error results from the API
  Type: prerequisite
  Teaches: (supports 4.4-K1, 4.4-K4, 4.4-S1)
  Definition: Claude API responses and batch results classify outcomes into typed
  categories (e.g. `succeeded`, `errored`, `expired` for batch results; structured
  error objects with a `type` such as `invalid_request_error`). This typed-error
  pattern is the general mechanism that "retry with error feedback" (4.4) builds on:
  a specific, structured error is something you can append to a follow-up prompt or
  branch retry logic on, unlike an undifferentiated failure.
  Example: A batch result of `{"result":{"type":"errored","error":{"error":{"type":"invalid_request_error"}}}}`
  tells the caller the request body itself needs fixing before resubmission, as
  distinct from a transient server error worth a plain retry.
  Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing

---

## Task Statement 4.1 — Design prompts with explicit criteria to improve precision and reduce false positives

- Concept: Explicit criteria over vague instructions
  Type: key
  Teaches: 4.1-K1, 4.1-S1
  Definition: Replacing a vague, judgment-dependent instruction with an explicit,
  checkable criterion improves precision because it gives the model (and a human
  auditor) a concrete test to apply, rather than an open-ended standard it must
  infer. The pattern generalizes: define, category by category, which issues to
  report (e.g. bugs, security) and which to explicitly skip (e.g. minor style,
  local team patterns) — a specific rule set, not a confidence threshold, is what
  narrows the false-positive surface.
  Example: The exam guide's own contrast: "flag comments only when claimed behavior
  contradicts actual code behavior" is an explicit, checkable criterion; "check that
  comments are accurate" is vague and leaves "accurate" undefined. A prompt built on
  the first produces consistently narrower, more defensible findings.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.1, lines 543-545, 552-553)

- Concept: Limits of confidence-based filtering language
  Type: key
  Teaches: 4.1-K2
  Definition: Generic qualifiers such as "be conservative" or "only report
  high-confidence findings" do not reliably improve precision, because they ask the
  model to self-estimate a confidence threshold rather than apply a defined rule —
  the model's internal notion of "high confidence" is not calibrated to the
  reviewer's actual tolerance for false positives. Specific categorical criteria
  (which categories to report, which to skip, and under what conditions) outperform
  confidence-based filtering because they replace an unobservable internal judgment
  with an externally specified, auditable rule.
  Example: Telling a code-review prompt to "be conservative" still lets it flag a
  stylistic nit it judges "high confidence"; telling it explicitly to skip "minor
  style and local patterns" removes that category outright regardless of the
  model's self-assessed confidence.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.1, lines 546-547)

- Concept: False positive rate and developer trust
  Type: key
  Teaches: 4.1-K3, 4.1-S2
  Definition: A high false-positive rate in one finding category erodes a
  developer's trust in the *whole* system, including categories where the tool is
  actually accurate — once a reviewer learns to distrust and skim past one noisy
  category, they tend to under-scrutinize adjacent, accurate categories too. The
  corresponding mitigation is to temporarily disable a category once its false
  positive rate is shown to be high, restoring trust in the remaining output, while
  the prompt for that disabled category is iterated on and re-enabled once improved.
  Example: If a "minor style" finding category is wrong 40% of the time, disabling
  it entirely (rather than leaving it in at reduced confidence) protects trust in
  the "security" and "bugs" categories that are accurate, until the style category's
  prompt is fixed and re-enabled.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.1, lines 548-549, 554-555); supporting production data (fewer than 1% of findings marked incorrect once verification/filtering is applied) at https://claude.com/blog/code-review

- Concept: Explicit severity criteria with concrete code examples
  Type: key
  Teaches: 4.1-S3
  Definition: Defining severity levels (e.g. critical/high/medium/low) purely by
  name invites inconsistent classification, because "critical" means different
  things to different reviewers and to the model across runs. Pairing each severity
  level with a concrete code example of an issue that belongs at that level gives
  the model a calibration anchor, producing more consistent classification across
  findings and across runs.
  Example: Instead of "high severity = serious bug," the prompt includes a snippet
  showing an unhandled null pointer that crashes a request path, labeled "high," so
  the model has a concrete reference point rather than an abstract label.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.1, lines 556-557)

---

## Task Statement 4.2 — Apply few-shot prompting to improve output consistency and quality

- Concept: Few-shot prompting for output consistency
  Type: key
  Teaches: 4.2-K1
  Definition: When detailed prose instructions alone produce inconsistently
  formatted or inconsistently actionable output, a small set of worked examples is
  the most effective technique for tightening that consistency — examples show,
  rather than describe, the exact shape and quality bar the output must meet, which
  is harder for a model to under- or over-generalize from than a prose rule.
  Example: A prompt that says "return findings with location, issue, severity, and
  a suggested fix" may still produce inconsistent phrasing across findings; adding
  two or three fully worked example findings in that exact shape locks the format in.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.2, lines 561-563); technique described generally at https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Few-shot examples for ambiguous-case handling
  Type: key
  Teaches: 4.2-K2, 4.2-S1
  Definition: Few-shot examples are especially effective for demonstrating how to
  resolve genuinely ambiguous cases — situations where more than one action is
  plausible (e.g. which of two overlapping tools to call, or whether a
  branch-level gap in test coverage counts as a finding). The skill is to construct
  a small number (2-4) of targeted examples for exactly these ambiguous scenarios,
  each one showing the reasoning for why the chosen action was preferred over the
  other plausible alternatives — not just the final answer, but the discriminating
  logic.
  Example: An example showing a request that could map to either `search_web` or
  `search_docs`, with a short explanation of why `search_docs` was chosen (the
  request named an internal document), teaches the model the discriminating signal
  rather than just one input/output pair.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.2, lines 564-565, 572-573)

- Concept: Few-shot generalization to novel patterns
  Type: key
  Teaches: 4.2-K3, 4.2-S3
  Definition: Well-chosen few-shot examples teach the model the *underlying
  judgment* behind a decision, not merely a lookup table of the specific cases
  shown — this lets it generalize that judgment to novel inputs it has not seen an
  example of, rather than only matching pre-specified cases verbatim. Providing
  examples that distinguish acceptable patterns from genuine issues (rather than
  only "here is a bug" examples) is what enables this generalization while also
  reducing false positives, because the model learns the boundary, not just one side
  of it.
  Example: Showing one example of an intentional, idiomatic short-circuit pattern
  (acceptable) alongside one example of a superficially similar but genuinely buggy
  pattern (an issue) teaches the model the distinguishing feature, so it can judge a
  third, unseen pattern correctly.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.2, lines 566-567, 576-577)

- Concept: Few-shot examples reducing extraction hallucination
  Type: key
  Teaches: 4.2-K4, 4.2-S4, 4.2-S5
  Definition: In extraction tasks, few-shot examples reduce hallucination by
  showing the model concrete instances of correct handling for the hard cases that
  otherwise invite fabrication: informal or non-standard measurements, and documents
  whose structure varies (inline citations versus a separate bibliography,
  methodology described in its own section versus embedded inline). The same
  technique addresses a related failure mode — empty or null extraction of fields
  that are actually present but expressed unusually — by including examples where
  the correct extraction is demonstrated for that varied formatting.
  Example: An extraction prompt paired with one example showing "~2 cups" correctly
  normalized to a quantity, and one example showing a citation embedded mid-sentence
  correctly attributed, prevents the model from either fabricating a precise value
  or returning an empty citation field for a format it hasn't seen described in
  prose instructions.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.2, lines 568-569, 578-581); general hallucination-reduction rationale at https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations

- Concept: Demonstrating desired output format via examples
  Type: key
  Teaches: 4.2-S2
  Definition: Including few-shot examples that show the specific desired output
  fields (e.g. location, issue, severity, suggested fix) in fully worked-out form is
  a distinct skill from choosing *which* ambiguous cases to exemplify — it targets
  structural consistency of the output shape itself, ensuring every finding is
  reported with the same fields in the same form regardless of which issue it
  describes.
  Example: Every example finding in the prompt includes all four fields —
  `location`, `issue`, `severity`, `suggested_fix` — even when one of them is
  trivial, so the model never learns to omit a field when a finding seems simple.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.2, lines 574-575)

---

## Task Statement 4.3 — Enforce structured output using tool use and JSON schemas

- Concept: Tool use with JSON schemas for guaranteed structured output
  Type: key
  Teaches: 4.3-K1, 4.3-S1
  Definition: Defining an extraction "tool" whose `input_schema` is the JSON Schema
  of the data you want, and having Claude call that tool, is the most reliable way
  to get schema-compliant structured output — the model's tool call must conform to
  the declared parameter shape, which eliminates the class of errors that comes from
  asking the model to emit raw JSON as free text (missing quotes, trailing commas,
  truncated braces). The structured data is then read directly from the `tool_use`
  block's `input` object rather than parsed out of prose.
  Example: A tool `extract_invoice` with `input_schema` requiring `vendor`,
  `line_items`, and `total`; the caller reads `response.content` for the
  `tool_use` block and takes `.input` directly as the parsed invoice, with no
  `JSON.parse()` step of free-form text needed.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools ; https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

- Concept: tool_choice options (auto / any / forced tool)
  Type: key
  Teaches: 4.3-K2
  Definition: `tool_choice` controls whether and how Claude must use a tool.
  `{"type":"auto"}` (the default) lets Claude decide whether to call a tool or
  respond in text. `{"type":"any"}` requires Claude to call *some* tool from the
  provided set, but does not pick which one. `{"type":"tool","name":"..."}` forces
  a specific, named tool to be called. `{"type":"none"}` prevents any tool call.
  This distinction matters for structured-output reliability: `auto` can still
  return plain text instead of the structured call you need, while `any` and
  `tool` guarantee a tool call happens.
  Example: A support-triage prompt using `tool_choice: "auto"` may sometimes answer
  in prose instead of calling `classify_ticket`; switching to `tool_choice: "any"`
  guarantees some structured classification tool is called every time.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools ; runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 587-589)

- Concept: Semantic vs syntax errors in structured output
  Type: key
  Teaches: 4.3-K3
  Definition: Strict JSON schemas enforced via tool use eliminate *syntax* errors —
  the output is always valid, well-typed JSON matching the schema — but they cannot
  by themselves prevent *semantic* errors, where the values are individually
  well-formed but collectively wrong: line items that don't sum to the stated total,
  or a value placed in a structurally valid but semantically incorrect field. Schema
  compliance and factual/logical correctness are separate guarantees, and only the
  first is automatic from tool use.
  Example: An invoice extraction can produce a perfectly schema-valid JSON object
  where the three `line_items` amounts sum to $210 but the `total` field reads $250
  — no schema violation occurred, but the extraction is wrong.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 590-591)

- Concept: Schema design — required/optional fields and enum + "other" pattern
  Type: key
  Teaches: 4.3-K4, 4.3-S5
  Definition: Two schema design choices materially affect extraction quality.
  First, whether a field is listed in `required` determines whether the model is
  forced to supply a value even when the source doesn't contain one. Second, for
  categorical (`enum`) fields, adding an escape-hatch value such as `"other"`
  (paired with a free-text `detail` field) plus an explicit `"unclear"` value for
  genuinely ambiguous cases keeps the schema extensible to categories not
  anticipated at design time, and gives the model a truthful option instead of
  forcing a best-guess match to an existing enum value.
  Example: A `document_type` enum of `["invoice","receipt","other"]` with a paired
  `document_type_detail` string lets the model correctly tag a purchase order as
  `"other"` with `detail: "purchase order"` instead of miscategorizing it as
  `"invoice"`.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 592-593, 604-605); enum/nullable schema syntax at https://platform.claude.com/docs/en/build-with-claude/structured-outputs

- Concept: tool_choice "any" for guaranteed extraction when schema is unknown
  Type: key
  Teaches: 4.3-S2
  Definition: When multiple extraction schemas (tools) exist and the incoming
  document type is not known ahead of time, setting `tool_choice: {"type":"any"}`
  guarantees the model calls one of the available extraction tools rather than
  returning conversational text, while still letting the model pick which schema
  best fits the document.
  Example: With `extract_invoice`, `extract_receipt`, and `extract_purchase_order`
  all offered and `tool_choice: "any"` set, an unclassified incoming document is
  guaranteed to produce a structured call to whichever of the three tools the model
  judges to fit.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools ; runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 598-599)

- Concept: Forced tool selection for step ordering
  Type: key
  Teaches: 4.3-S3
  Definition: Setting `tool_choice: {"type":"tool","name":"extract_metadata"}`
  forces that specific tool to be called on this turn, which is useful for
  guaranteeing a particular extraction step runs before dependent enrichment steps
  — the pipeline can rely on `extract_metadata` having executed rather than
  hoping the model chose to call it in the desired order under `auto`.
  Example: Forcing `extract_metadata` on the first turn of a document pipeline
  guarantees document type and author are known before a second turn (running under
  `tool_choice: "auto"`) decides which enrichment tools to call next.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 600-601); mechanism at https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: Optional/nullable fields prevent fabrication
  Type: key
  Teaches: 4.3-S4
  Definition: When a source document may not contain a given piece of information,
  designing that schema field as optional (not in `required`) and nullable (typed
  to accept `null`, e.g. `["string","null"]`) prevents the model from having to
  invent a plausible-looking value just to satisfy a `required` constraint it
  cannot legitimately fill. This directly targets a fabrication failure mode caused
  by schema design rather than model behavior alone.
  Example: A `customer.company` field marked required against a personal-purchase
  receipt with no company name forces the model to either fail or fabricate a
  company; marking it optional/nullable lets it correctly return `null`.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 602-603); nullable-field syntax at https://platform.claude.com/docs/en/build-with-claude/structured-outputs

- Concept: Format normalization rules alongside strict schemas
  Type: key
  Teaches: 4.3-S6
  Definition: A strict output schema constrains the *shape* of the data but not
  necessarily its internal formatting consistency (e.g. date formats, unit
  conventions, casing) when source documents themselves are formatted
  inconsistently. Including explicit normalization rules in the prompt text
  alongside the schema — telling the model how to normalize dates, currencies, or
  measurement units it encounters — closes that gap, so schema-valid output is also
  internally consistent output.
  Example: A schema requires `date` as a string, but source invoices use both
  `MM/DD/YYYY` and `DD-Mon-YYYY`; a prompt rule instructing "normalize all dates to
  ISO 8601 (YYYY-MM-DD)" ensures the schema-valid `date` field is also consistently
  formatted across documents.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.3, lines 606-607)

---

## Task Statement 4.4 — Implement validation, retry, and feedback loops for extraction quality

- Concept: Retry-with-error-feedback
  Type: key
  Teaches: 4.4-K1, 4.4-S1
  Definition: When a validated extraction fails, the retry that is most likely to
  succeed is one that appends the *specific* validation errors found (not a generic
  "try again") to a follow-up request, alongside the original source document and
  the failed extraction itself, so the model can see exactly what was wrong and
  self-correct against that concrete feedback rather than guessing what changed.
  Example: A follow-up prompt includes the original invoice text, the first
  extraction attempt, and the message "line_items sum to $210 but total is $250 —
  correct the discrepancy," rather than simply re-asking the original extraction
  question.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.4, lines 611-613, 621-623)

- Concept: Limits of retry (absent information vs format errors)
  Type: key
  Teaches: 4.4-K2, 4.4-S2
  Definition: Retry-with-feedback is only effective for errors caused by *how* the
  model formatted or structured its answer — those are fixable by giving the model
  another attempt with guidance. It is ineffective when the underlying problem is
  that the required information simply does not exist in the source document
  provided (e.g. it lives only in a separate document not included in context); no
  amount of retrying will conjure information the model was never given. Recognizing
  which category a given validation failure falls into determines whether a retry is
  worth attempting at all.
  Example: A missing `due_date` because the invoice text genuinely has no due date
  will not be fixed by retrying; a missing `due_date` because the model put it in
  the wrong nested object (a structural error) likely will be fixed by a retry with
  the validation error attached.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.4, lines 614-615, 624-625)

- Concept: Feedback loop design with detected_pattern
  Type: key
  Teaches: 4.4-K3, 4.4-S3
  Definition: Adding a `detected_pattern` field to each structured finding —
  recording which specific code construct or textual pattern triggered that finding
  — turns individual findings into data that can be aggregated later. When
  developers dismiss findings, tracking which `detected_pattern` values correlate
  with dismissals enables systematic analysis of which patterns are generating false
  positives, closing the loop from individual review outputs back into prompt
  improvement.
  Example: Logging `detected_pattern: "unchecked_optional_chaining"` on every
  finding of that type lets a later analysis show that pattern accounts for 60% of
  developer dismissals, pointing directly at which category to refine or disable.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.4, lines 616-617, 626-627)

- Concept: Semantic validation errors vs schema syntax errors
  Type: key
  Teaches: 4.4-K4
  Definition: Schema syntax errors (malformed JSON, wrong field types, missing
  required fields) are eliminated by enforcing extraction through tool use, as
  covered under Task 4.3. Semantic validation errors are a distinct category that
  tool use does *not* eliminate: values that are individually well-typed but
  collectively inconsistent, such as line items that don't sum to the stated total,
  or a value placed in a structurally valid but wrong field. Validation logic in
  Task 4.4 targets this second, semantic category specifically.
  Example: A validator that checks `sum(line_items.amount) == total` catches a
  semantic error that no JSON Schema constraint alone could catch, since both
  values individually satisfy the schema's `type: number` requirement.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.4, lines 618-619)

- Concept: Self-correction validation flows (calculated vs stated values, conflict flags)
  Type: key
  Teaches: 4.4-S4
  Definition: A validation-flow design pattern for catching semantic errors without
  external logic: have the extraction schema itself request both a `stated_total`
  (what the document claims) and a `calculated_total` (derived by the model from the
  line items it also extracted), so a downstream comparison of the two flags a
  discrepancy automatically. A parallel pattern adds a boolean field like
  `conflict_detected` when the source document itself contains internally
  inconsistent data, making that inconsistency visible in the structured output
  rather than silently resolved one way or the other.
  Example: An invoice schema with both `stated_total: 250` and
  `calculated_total: 210` (derived from summing extracted line items) lets a
  downstream check flag the invoice for human review without needing a separate
  validation pass outside the extraction call.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.4, lines 628-629)

---

## Task Statement 4.5 — Design efficient batch processing strategies

- Concept: Message Batches API fundamentals
  Type: key
  Teaches: 4.5-K1
  Definition: The Message Batches API processes large volumes of Messages requests
  asynchronously at a 50% discount on both input and output token pricing versus the
  synchronous API. Most batches finish in under an hour, but the API gives no
  latency SLA — a batch can take up to 24 hours, after which any requests still
  unprocessed expire. This combination (discount, asynchrony, no latency guarantee)
  is what makes it suited to some workloads and unsuited to others (see next
  concept).
  Example: Submitting 50,000 document-classification requests as one batch costs
  half what 50,000 individual synchronous calls would, but the caller cannot assume
  any of them will be done within, say, 10 minutes.
  Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing

- Concept: Batch-appropriate workloads vs blocking workflows
  Type: key
  Teaches: 4.5-K2, 4.5-S1
  Definition: Because the Batches API trades guaranteed low latency for cost
  savings, it fits non-blocking, latency-tolerant workloads — overnight reports,
  weekly audits, nightly test generation — where nothing downstream is waiting on an
  individual response within seconds or minutes. It is the wrong choice for blocking
  workflows such as pre-merge CI checks, where a human or pipeline step is stalled
  waiting on the result; those belong on the synchronous Messages API despite its
  higher per-token cost.
  Example: A nightly job that regenerates test suites for every repository can run
  entirely on the Batches API; a pre-merge PR check that blocks a developer's merge
  button must use the synchronous API so it returns within the CI step's timeout.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.5, lines 636-638, 644-645)

- Concept: Batch API's lack of mid-request multi-turn tool calling
  Type: key
  Teaches: 4.5-K3
  Definition: Each entry in a Message Batch is one independent Messages request,
  processed once and returning one result; there is no mechanism to pause a
  batch entry mid-processing, execute a client-side tool call it produced, and feed
  the `tool_result` back into that same request before it completes. Consequently a
  full agentic tool-use loop (Claude calls a tool, your code executes it and returns
  a result, Claude continues) cannot happen within a single batch request — only
  patterns that fit in one request/one response (e.g. server-side tools that resolve
  within Anthropic's infrastructure, or a single client tool call captured as the
  batch entry's whole output) are compatible with batching as-is.
  Example: A workflow that needs Claude to call a tool, inspect the result, and then
  decide on a follow-up action cannot complete that loop inside one batch entry;
  each such turn would need to be its own separate batch (or synchronous) request.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.5, lines 639-640); request/response shape supporting this reasoning at https://platform.claude.com/docs/en/build-with-claude/batch-processing

- Concept: custom_id request/response correlation
  Type: key
  Teaches: 4.5-K4, 4.5-S3
  Definition: Every batch request carries a caller-supplied `custom_id`, and every
  result in the batch's output is tagged with that same `custom_id`. Because batch
  results can return in a different order than the requests were submitted in, and
  are not otherwise indexed, `custom_id` is the only reliable way to match a result
  back to the request that produced it. This same field is what makes targeted
  failure handling possible: when a batch finishes, only the requests whose results
  are `errored` or `expired` (identified by their `custom_id`) need to be
  resubmitted — with any needed fix, such as chunking a document that exceeded the
  context limit — rather than resubmitting the entire batch.
  Example: Out of 10,000 submitted documents, 40 come back `errored` because they
  exceeded the context window; the caller looks up those 40 `custom_id`s, chunks
  just those documents, and resubmits only that subset as a new (much smaller)
  batch.
  Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing ; runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.5, lines 641, 648-649)

- Concept: Calculating batch submission frequency against an SLA
  Type: key
  Teaches: 4.5-S2
  Definition: Because batch processing offers no latency guarantee up to its
  24-hour cap, meeting an end-to-end SLA that is tighter than "sometime tomorrow"
  requires backing the batch's worst-case processing time out of the required
  turnaround and submitting on a schedule frequent enough to absorb it. Concretely,
  to guarantee results within N hours of a request arriving, batches must be
  submitted at an interval no longer than (N − 24) hours, since any individual
  request might sit for the full 24-hour window before being picked up in the worst
  case.
  Example: To guarantee a 30-hour SLA against a batch API with a 24-hour maximum
  processing window, batches must be submitted at least every 6 hours — the exam
  guide's own worked figure (4-hour windows) is a stricter, safety-margined version
  of this same calculation.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.5, line 646-647); batch timing behavior at https://platform.claude.com/docs/en/build-with-claude/batch-processing

- Concept: Prompt refinement on a sample before batch-scale submission
  Type: key
  Teaches: 4.5-S4
  Definition: Because a batch's discount and throughput make each individual
  request cheap but a failed or low-quality large-scale run expensive to detect and
  re-run, the efficient sequencing is to iterate on prompt quality against a small
  synchronous sample first — catching format issues, false positives, and edge-case
  failures cheaply and quickly — before submitting the full volume as a batch. This
  maximizes the first-pass success rate of the batch run and minimizes costly
  iterative resubmission at scale.
  Example: Testing an extraction prompt synchronously against 20 representative
  documents, fixing the schema and prompt issues that surface, and only then
  submitting the remaining 50,000 documents as a single batch — rather than
  discovering a systematic extraction bug only after the full batch completes.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.5, line 650)

---

## Task Statement 4.6 — Design multi-instance and multi-pass review architectures

- Concept: Self-review limitations (retained reasoning context)
  Type: key
  Teaches: 4.6-K1
  Definition: A Claude instance that generated a piece of output (code, an
  extraction, an analysis) and is then asked, in the same session, to review that
  output retains the reasoning context and assumptions that led to its original
  choices. That retained context makes it structurally less likely to question its
  own decisions than a reviewer encountering the output fresh — the model tends to
  re-justify what it already committed to rather than re-derive an independent
  judgment.
  Example: A Claude session that wrote an authentication check and is then asked
  "review your code for bugs" in the same conversation is anchored on the design
  choices it already made, and is less likely to notice a subtle flaw in an
  assumption it built into the original implementation.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.6, lines 655-656)

- Concept: Independent review instances outperform self-review
  Type: key
  Teaches: 4.6-K2, 4.6-S1
  Definition: A second, independent Claude instance — one that never saw the
  generating instance's reasoning trace or intermediate decisions, only the final
  output to review — catches subtle issues more effectively than instructing the
  same session to self-review, and more effectively than relying on extended
  thinking within that same session (extended thinking still reasons from inside
  the same generation context; see the extended-thinking prerequisite above). The
  corresponding skill is architectural: route generated output through a second,
  freshly-started Claude call for verification, rather than a self-review
  instruction appended to the generating session.
  Example: An automated code-review pipeline spawns a fresh review instance,
  containing only the diff and review criteria (no prior conversation), rather than
  asking the generating session to "now review what you just wrote."
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.6, lines 657-658, 663-664); independent-agent verification in production at https://claude.com/blog/code-review

- Concept: Multi-pass review — per-file and cross-file passes
  Type: key
  Teaches: 4.6-K3, 4.6-S2
  Definition: For reviews spanning many files, running one review pass across the
  entire change set at once risks attention dilution (the model spreads its
  attention thin across too much content) and contradictory findings across
  different parts of the same review. Splitting the work into focused per-file
  passes for local, single-file issues, plus a separate cross-file integration pass
  dedicated to issues that only exist across file boundaries (e.g. data flow between
  a changed function and its callers elsewhere), keeps each pass's attention
  concentrated on a scope it can actually cover well.
  Example: A 40-file pull request is reviewed with 40 focused single-file passes
  (each looking only at that file's local logic) plus one additional pass whose
  entire job is tracing how data flows between the changed files — rather than one
  pass attempting both at once.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.6, lines 659-660, 665-666)

- Concept: Confidence self-reporting for calibrated review routing
  Type: key
  Teaches: 4.6-S3
  Definition: Having a verification pass report a confidence value alongside each
  finding — rather than only a binary flag/no-flag — enables calibrated routing of
  findings downstream: high-confidence findings can be surfaced directly, while
  low-confidence findings can be routed to an additional check (another pass, or a
  human) instead of being either silently dropped or surfaced with the same
  authority as a high-confidence one.
  Example: A verification pass returns `{"finding": "...", "confidence": 0.9}` for
  a clear-cut security issue and `{"finding": "...", "confidence": 0.4}` for a
  borderline style concern; the pipeline auto-posts the first and routes the second
  to a secondary check before posting.
  Source: runs/claude-certified-architect-foundations/sources/exam-guide.txt (Domain 4, Task 4.6, lines 667-668)
