# Module 10 Hands-On Exercise

## Scenario: Launching a Multi-Source Market Research Product

Your team runs an extraction pipeline (contracts, invoices, scanned forms) with
97% aggregate accuracy, and a separate multi-source research synthesis product
that pulls statistics from several external reports.

### Part A — Human review and confidence calibration

1. Design a validation plan to check whether the 97% aggregate accuracy figure is
   hiding a segment-specific problem. Specify what you'd break the analysis down
   by and what decision it would inform.
2. Design a stratified sampling scheme for ongoing QA (not the one-time
   validation in Q1): what strata would you sample from, at what rate, and why
   would sampling only low-confidence extractions be insufficient?
3. On a labeled validation set, you discover that 0.85-confidence extractions
   for "contract termination date" are correct only 70% of the time, while
   0.85-confidence extractions for "vendor name" are correct 98% of the time.
   What should you do with the review threshold for each field?
4. Write a routing rule (as a short decision table) that determines which
   extractions go to human review, incorporating both confidence and source
   ambiguity/contradiction, and reflecting limited reviewer capacity.

### Part B — Provenance and synthesis

5. Design the structured output schema a research subagent must produce for each
   finding it reports, ensuring claim-source mapping and temporal metadata are
   preserved (include field names and types).
6. Two subagents report different figures for "2026 global AI spending" from two
   credible but different reports, dated 2025 and 2026 respectively. Write the
   synthesis agent's handling of this: what does it do with both figures, and
   how does the date field change the interpretation compared to if both reports
   were dated 2026?
7. Draft the outline (section headers only, with one sentence describing what
   goes in each) of a synthesis report that separates well-established findings
   from contested ones, and appropriately renders at least one table, one prose
   section, and one structured list.
8. Explain, in your own words, how the Claude API's citations feature could be
   used as the underlying mechanism for the claim-source mapping in Q5 — what do
   you pass in, and what comes back attached to each claim?

Write your answers as a short product design document, including your schema
definitions and report outline.
