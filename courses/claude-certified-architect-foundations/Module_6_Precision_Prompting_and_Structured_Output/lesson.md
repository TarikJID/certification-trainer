# Module 6 — Precision Prompting and Structured Output Enforcement

Domain: Domain 4 — Prompt Engineering & Structured Output (20% of exam)
Covers Task Statements 4.1, 4.2, 4.3.

**Prerequisites from earlier modules:** the Messages API request/response cycle
and `tool_use` blocks (Module 1, Lesson 1.1); tool definitions
(`name`/`description`/`input_schema`) and `tool_choice` (Module 3, Lessons 2.1 and
2.3); JSON Schema basics (Module 5, Lesson 3.6).

---

## Lesson 4.1 — Designing Prompts with Explicit Criteria to Reduce False Positives

**Maps to:** Task Statement 4.1: Design prompts with explicit criteria to improve
precision and reduce false positives.

### Prerequisite concept: "Be clear and direct" — explicit instruction baseline

Anthropic's general prompting guidance treats explicit, specific instructions as
the baseline technique underneath every more advanced pattern: Claude should be
treated as a capable but context-free new hire, and vague requests should be
replaced with precise statements of the desired output and constraints. This
lesson applies that general principle to the false-positive problem specifically.

*Example:* The guide's own contrast — "flag comments only when claimed behavior
contradicts actual code behavior" versus "check that comments are accurate" — is
a direct instance of replacing a vague instruction with an explicit, checkable
one.

*Source:* https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
(Teaches: prerequisite for explicit criteria over vague instructions)

### Concept: Explicit criteria over vague instructions

Replacing a vague, judgment-dependent instruction with an explicit, checkable
criterion improves precision because it gives the model a concrete test to apply,
rather than an open-ended standard it must infer. The pattern generalizes: define,
category by category, which issues to report and which to explicitly skip — a
specific rule set, not a confidence threshold, narrows the false-positive
surface.

*Example:* "Flag comments only when claimed behavior contradicts actual code
behavior" is explicit and checkable; "check that comments are accurate" is vague
and leaves "accurate" undefined.

*Source:* exam-guide.txt (Domain 4, Task 4.1)
(Teaches: 4.1-K1, 4.1-S1)

### Concept: Limits of confidence-based filtering language

Generic qualifiers such as "be conservative" or "only report high-confidence
findings" do not reliably improve precision, because they ask the model to
self-estimate a confidence threshold rather than apply a defined rule — the
model's internal notion of "high confidence" is not calibrated to the reviewer's
actual tolerance for false positives. Specific categorical criteria outperform
confidence-based filtering.

*Example:* Telling a prompt to "be conservative" still lets it flag a stylistic
nit it judges "high confidence"; telling it explicitly to skip "minor style and
local patterns" removes that category outright.

*Source:* exam-guide.txt (Domain 4, Task 4.1)
(Teaches: 4.1-K2)

### Concept: False positive rate and developer trust

A high false-positive rate in one finding category erodes trust in the *whole*
system, including categories where the tool is actually accurate — once a
reviewer learns to distrust one noisy category, they tend to under-scrutinize
adjacent, accurate categories too. Mitigation: temporarily disable a category
once its false-positive rate is shown to be high, restoring trust in the
remaining output, while that category's prompt is iterated on.

*Example:* If a "minor style" finding category is wrong 40% of the time,
disabling it entirely protects trust in the "security" and "bugs" categories
until it's fixed and re-enabled.

*Source:* exam-guide.txt (Domain 4, Task 4.1); supporting production data at
https://claude.com/blog/code-review
(Teaches: 4.1-K3, 4.1-S2)

### Concept: Explicit severity criteria with concrete code examples

Defining severity levels purely by name invites inconsistent classification,
because "critical" means different things to different reviewers and to the
model across runs. Pairing each severity level with a concrete code example of an
issue that belongs at that level gives the model a calibration anchor.

*Example:* Instead of "high severity = serious bug," include a snippet showing an
unhandled null pointer that crashes a request path, labeled "high."

*Source:* exam-guide.txt (Domain 4, Task 4.1)
(Teaches: 4.1-S3)

**Quiz:** see `quiz.md`, Lesson 4.1.

---

## Lesson 4.2 — Applying Few-Shot Prompting to Improve Output Consistency and Quality

**Maps to:** Task Statement 4.2: Apply few-shot prompting to improve output
consistency and quality.

### Prerequisite concept: Multishot/few-shot prompting with `<example>` tags

Anthropic's prompting guidance names few-shot ("multishot") prompting —
including a small number of worked examples in the prompt — as one of the most
reliable ways to steer output format, tone, and structure, more reliable than
prose instructions alone. Effective examples are relevant, diverse, and
structured (wrapped in `<example>`/`<examples>` tags). Recommended starting
point: roughly 3-5 examples.

*Example:* Wrapping two worked code-review judgments in
`<examples><example>...</example><example>...</example></examples>` before
asking Claude to review a new diff.

*Source:* https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
(Teaches: prerequisite for 4.2-K1, K2, K3, S1, S2, S3)

### Prerequisite concept: Hallucination in LLM outputs

Hallucination is text a model generates that is factually incorrect or
inconsistent with the source material given — for extraction, this shows up as
invented field values or answers not actually supported by the document.
Mitigations include allowing the model to express uncertainty, grounding answers
in direct quotes, and verifying claims against the source. Few-shot examples are
a complementary mitigation.

*Example:* Given an invoice missing a due date, an ungrounded model may
hallucinate a plausible-looking date; a hallucination-aware prompt instructs it
to return `null` instead.

*Source:* https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
(Teaches: prerequisite for 4.2-K4, S4, S5)

### Concept: Few-shot prompting for output consistency

When detailed prose instructions alone produce inconsistently formatted or
actionable output, a small set of worked examples is the most effective way to
tighten consistency — examples show, rather than describe, the exact shape and
quality bar output must meet.

*Example:* A prompt saying "return findings with location, issue, severity, and a
suggested fix" may still produce inconsistent phrasing; adding two or three fully
worked example findings in that exact shape locks the format in.

*Source:* exam-guide.txt (Domain 4, Task 4.2)
(Teaches: 4.2-K1)

### Concept: Few-shot examples for ambiguous-case handling

Few-shot examples are especially effective for demonstrating how to resolve
genuinely ambiguous cases — situations where more than one action is plausible.
The skill: construct 2-4 targeted examples for exactly these scenarios, each
showing the reasoning for why the chosen action was preferred over other
plausible alternatives.

*Example:* An example showing a request that could map to either `search_web` or
`search_docs`, with a short explanation of why `search_docs` was chosen, teaches
the discriminating signal.

*Source:* exam-guide.txt (Domain 4, Task 4.2)
(Teaches: 4.2-K2, 4.2-S1)

### Concept: Few-shot generalization to novel patterns

Well-chosen few-shot examples teach the model the underlying judgment behind a
decision, not merely a lookup table of the cases shown — letting it generalize to
novel inputs. Providing examples that distinguish acceptable patterns from
genuine issues (not only "here is a bug" examples) enables this generalization
while also reducing false positives.

*Example:* Showing one intentional, idiomatic short-circuit pattern (acceptable)
alongside a superficially similar but genuinely buggy pattern (an issue) teaches
the distinguishing feature.

*Source:* exam-guide.txt (Domain 4, Task 4.2)
(Teaches: 4.2-K3, 4.2-S3)

### Concept: Few-shot examples reducing extraction hallucination

In extraction tasks, few-shot examples reduce hallucination by showing concrete
instances of correct handling for hard cases that otherwise invite fabrication:
informal measurements, and documents whose structure varies. The same technique
addresses empty/null extraction of fields that are actually present but expressed
unusually.

*Example:* An extraction prompt with one example showing "~2 cups" correctly
normalized, and one showing a citation embedded mid-sentence correctly
attributed, prevents fabricating a precise value or returning an empty field.

*Source:* exam-guide.txt (Domain 4, Task 4.2)
(Teaches: 4.2-K4, 4.2-S4, 4.2-S5)

### Concept: Demonstrating desired output format via examples

Including few-shot examples that show the specific desired output fields (e.g.
location, issue, severity, suggested fix) in fully worked-out form is a distinct
skill from choosing which ambiguous cases to exemplify — it targets structural
consistency of the output shape itself.

*Example:* Every example finding in the prompt includes all four fields, even
when one is trivial, so the model never learns to omit a field when a finding
seems simple.

*Source:* exam-guide.txt (Domain 4, Task 4.2)
(Teaches: 4.2-S2)

**Quiz:** see `quiz.md`, Lesson 4.2.

---

## Lesson 4.3 — Enforcing Structured Output Using Tool Use and JSON Schemas

**Maps to:** Task Statement 4.3: Enforce structured output using tool use and
JSON schemas.

### Prerequisite concept: JSON Schema fundamentals for tool/output definitions — recap

Recall from Module 5, Lesson 3.6: a JSON Schema describes the shape of a JSON
object — `type`, a `properties` map, a `required` array, and a `type` union
(e.g. `["string", "null"]`) or `anyOf` with a null branch to mark a field
nullable. This vocabulary is what Claude's tool `input_schema` is built on.

*Example:* `{"type":"object","properties":{"name":{"type":"string"},"phone":
{"type":["string","null"]}},"required":["name"]}` makes `name` mandatory and
`phone` optional-but-typed.

*Source:* https://platform.claude.com/docs/en/build-with-claude/structured-outputs
(Teaches: 4.3-K1, K4, S1, S4, S5, S6 — prerequisite)

### Prerequisite concept: Tool use round trip (tool_use / tool_result blocks) — recap

Recall from Module 1, Lesson 1.1 and Module 3, Lesson 2.1: Claude's response can
contain a `tool_use` block naming a tool and its `input` object. For one-shot
extraction, the `tool_use.input` object itself *is* the structured output — no
further turn required.

*Example:* A request with a `get_weather` tool returns
`{"type":"tool_use","name":"get_weather","input":{"location":"San Francisco,
CA"}}`; the caller reads `input` directly as extracted structured data.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
(Teaches: 4.3-K1, K2, S1, S2, S3 — prerequisite)

### Concept: Tool use with JSON schemas for guaranteed structured output

Defining an extraction "tool" whose `input_schema` is the JSON Schema of the data
you want, and having Claude call that tool, is the most reliable way to get
schema-compliant structured output — the model's tool call must conform to the
declared parameter shape, eliminating the class of errors from asking the model
to emit raw JSON as free text.

*Example:* A tool `extract_invoice` with `input_schema` requiring `vendor`,
`line_items`, and `total`; the caller reads the `tool_use` block's `input`
directly as the parsed invoice, no `JSON.parse()` of free text needed.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 4.3-K1, 4.3-S1)

### Concept: tool_choice options (auto / any / forced tool) — recap and extraction framing

Recall from Module 3, Lesson 2.3: `auto` lets Claude decide whether to call a
tool; `any` requires calling some tool from the provided set; `{"type":"tool",
"name":"..."}` forces a specific tool; `none` prevents any call. For structured
output specifically: `auto` can still return plain text instead of the
structured call you need, while `any` and `tool` guarantee a call happens.

*Example:* A support-triage prompt using `tool_choice: "auto"` may sometimes
answer in prose instead of calling `classify_ticket`; switching to `"any"`
guarantees some structured classification tool is called every time.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 4.3-K2)

### Concept: Semantic vs syntax errors in structured output

Strict JSON schemas enforced via tool use eliminate *syntax* errors — output is
always valid, well-typed JSON matching the schema — but cannot by themselves
prevent *semantic* errors, where values are individually well-formed but
collectively wrong (e.g. line items that don't sum to the stated total). Schema
compliance and factual/logical correctness are separate guarantees.

*Example:* An invoice extraction can be perfectly schema-valid with `line_items`
summing to $210 while `total` reads $250 — no schema violation, but the
extraction is wrong.

*Source:* exam-guide.txt (Domain 4, Task 4.3)
(Teaches: 4.3-K3)

### Concept: Schema design — required/optional fields and enum + "other" pattern

Two schema design choices materially affect extraction quality. First, whether a
field is `required` determines whether the model is forced to supply a value even
when the source doesn't contain one. Second, for `enum` fields, adding an
escape-hatch value like `"other"` (paired with a free-text `detail` field) plus an
explicit `"unclear"` value keeps the schema extensible.

*Example:* A `document_type` enum of `["invoice","receipt","other"]` with a
paired `document_type_detail` string lets the model correctly tag a purchase
order as `"other"` with `detail: "purchase order"`.

*Source:* exam-guide.txt (Domain 4, Task 4.3); enum/nullable syntax at
https://platform.claude.com/docs/en/build-with-claude/structured-outputs
(Teaches: 4.3-K4, 4.3-S5)

### Concept: tool_choice "any" for guaranteed extraction when schema is unknown

When multiple extraction schemas (tools) exist and the incoming document type is
unknown, setting `tool_choice: {"type":"any"}` guarantees the model calls one of
the available extraction tools rather than returning text, while still letting it
pick which schema fits.

*Example:* With `extract_invoice`, `extract_receipt`, and
`extract_purchase_order` all offered and `tool_choice: "any"` set, an
unclassified document is guaranteed to produce a structured call to whichever
tool the model judges fits.

*Source:* https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools;
exam-guide.txt
(Teaches: 4.3-S2)

### Concept: Forced tool selection for step ordering

Setting `tool_choice: {"type":"tool","name":"extract_metadata"}` forces that
specific tool to be called this turn, useful for guaranteeing a particular
extraction step runs before dependent enrichment steps.

*Example:* Forcing `extract_metadata` on the first turn guarantees document type
and author are known before a second turn (under `auto`) decides which
enrichment tools to call next.

*Source:* exam-guide.txt; mechanism at
https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
(Teaches: 4.3-S3)

### Concept: Optional/nullable fields prevent fabrication

When a source document may not contain a piece of information, designing that
schema field as optional and nullable prevents the model from having to invent a
plausible-looking value just to satisfy a `required` constraint it cannot
legitimately fill.

*Example:* A `customer.company` field marked required against a personal-purchase
receipt with no company name forces the model to fail or fabricate; marking it
optional/nullable lets it correctly return `null`.

*Source:* exam-guide.txt; nullable-field syntax at
https://platform.claude.com/docs/en/build-with-claude/structured-outputs
(Teaches: 4.3-S4)

### Concept: Format normalization rules alongside strict schemas

A strict output schema constrains the *shape* of data but not necessarily its
internal formatting consistency when source documents are formatted
inconsistently. Including explicit normalization rules in the prompt text
alongside the schema closes that gap.

*Example:* A schema requires `date` as a string, but source invoices use both
`MM/DD/YYYY` and `DD-Mon-YYYY`; a prompt rule instructing "normalize all dates to
ISO 8601" ensures schema-valid output is also internally consistent.

*Source:* exam-guide.txt (Domain 4, Task 4.3)
(Teaches: 4.3-S6)

**Quiz:** see `quiz.md`, Lesson 4.3.

---

## Module 6 summary of bullets taught

Lesson 4.1: 4.1-K1, 4.1-K2, 4.1-K3, 4.1-S1, 4.1-S2, 4.1-S3
Lesson 4.2: 4.2-K1, 4.2-K2, 4.2-K3, 4.2-K4, 4.2-S1, 4.2-S2, 4.2-S3, 4.2-S4, 4.2-S5
Lesson 4.3: 4.3-K1, 4.3-K2, 4.3-K3, 4.3-K4, 4.3-S1, 4.3-S2, 4.3-S3, 4.3-S4, 4.3-S5,
4.3-S6
