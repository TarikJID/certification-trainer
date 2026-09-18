# Module 6 Hands-On Exercise

## Scenario: Building an Automated Invoice-Extraction Reviewer

You're building a system that (a) extracts structured data from invoices and (b)
runs an automated code-review prompt over a separate codebase, both of which are
currently producing too many false positives / inconsistent output.

### Part A — Precision prompting

1. The code-review prompt currently says "check that comments are accurate" and
   "be conservative about what you flag." Rewrite it using explicit, checkable
   criteria, following the guidance in Lesson 4.1. Include at least one category
   the prompt should explicitly skip.
2. The review categories are "bugs," "security," and "style." Style has a 45%
   false-positive rate reported by developers. What should you do immediately,
   and what should happen before you re-enable it?
3. Write concrete severity criteria (critical/high/medium/low) for the "bugs"
   category, each paired with a short illustrative code snippet.

### Part B — Few-shot design

4. Write 2 few-shot examples for the code-review prompt that demonstrate an
   ambiguous case: a short-circuit pattern that's idiomatic (acceptable) versus
   one that's a genuine bug. Include the reasoning in each example, not just the
   verdict.
5. Write 2 few-shot examples for the invoice extractor that demonstrate correct
   handling of an informal measurement and a citation embedded in an unusual
   document structure.

### Part C — Structured output enforcement

6. Design the JSON Schema `input_schema` for an `extract_invoice` tool covering
   `vendor` (string, required), `invoice_date` (string, required, normalized to
   ISO 8601), `line_items` (array of {description, amount}), `total` (number,
   required), and `customer_company` (string, optional/nullable). Justify your
   required/optional choices.
7. You don't know ahead of time whether an incoming document is an invoice,
   receipt, or purchase order, and you have three separate extraction tools.
   Specify the `tool_choice` configuration you'd use, and explain what would go
   wrong with the default `tool_choice: "auto"` here.
8. Add a `document_type` enum field with an "other" + detail pattern to handle
   document types you didn't anticipate, and explain why this design choice
   prevents both fabrication and forced miscategorization.
9. Your extractor sometimes returns line items that don't sum to the stated
   total. Is this a syntax error or a semantic error? Does forcing tool use fix
   it? What would?

Write your answers as a short design document including the actual JSON Schema
and few-shot example text.
