# Module 5 Hands-On Exercise

## Build a reliable content-compliance reviewer

You must design a prompt-and-architecture solution for flagging
policy-violating user-generated posts, minimizing both false positives and
false negatives.

1. **Success criteria.** Write a SMART success criterion for this system
   (e.g., in terms of a measurable false-positive and false-negative rate),
   and describe an automatable evaluation method you'd use to measure it,
   per Lesson 5.1.

2. **Prompt design.** Draft the key sections of the review prompt using: (a)
   at least 3 multishot examples (input post + correct JSON verdict), (b) a
   chain-of-thought instruction separating `<thinking>` from `<answer>`, and
   (c) XML tags to separate the post being reviewed from the review
   instructions.

3. **Explicit criteria.** Following Lesson 5.2's "explicit categorical
   review criteria" pattern, write a short "flag" list and "do not flag"
   list for your reviewer (not a vague "flag anything concerning").

4. **Structured output.** Define a JSON schema for the reviewer's verdict
   (e.g., `violates_policy: boolean`, `category: enum`, `confidence:
   number`, `evidence_quote: string`) and specify whether you'd enforce it
   via JSON Outputs, strict tool use, or both.

5. **Scale and reliability.** The system must review 200,000 posts nightly.
   Describe how you'd use the Batches API and prompt caching together
   (per Lesson 5.4), and then describe a multi-instance voting architecture
   (per Lesson 5.5) for the subset of posts near the decision boundary,
   including what would trigger escalation to a human.

### What a strong answer includes
- A measurable, testable success criterion — not a vague goal.
- Multishot examples in `<example>` tags with a JSON verdict shape that
  matches the schema defined in step 4.
- A "do not flag" list that mirrors the discipline of Anthropic's Code
  Review defaults (style/subjective concerns excluded), and a "flag" list
  that is concrete and checkable.
- A schema-enforcement choice with a stated reason.
- Correct batch-plus-caching sequencing (prime the cache first, then submit
  the batch), and a voting rule (e.g. "any flag among N independent calls
  routes to a human") tied back to the false-positive/false-negative
  tradeoff from step 1.
