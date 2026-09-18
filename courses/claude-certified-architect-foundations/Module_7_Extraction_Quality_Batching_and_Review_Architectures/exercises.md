# Module 7 Hands-On Exercise

## Scenario: Scaling Up the Invoice Extraction Pipeline

The invoice extractor from Module 6 now needs to process 80,000 invoices weekly,
with retry logic for failures and an independent verification pass before results
reach production.

### Part A — Retry and feedback design

1. Design the retry flow for a failed extraction: what exactly goes into the
   retry request, and what validation-error message format would you produce for
   a "line items don't sum to total" failure?
2. A batch of failures includes: (a) extractions where `due_date` is genuinely
   absent from 200 source invoices, and (b) extractions where the model put
   `due_date` in the wrong nested object across 150 other invoices. Which group
   should you retry, and which should you handle differently? Why?
3. Add a `detected_pattern` field to your extraction's finding schema and
   describe how you'd use it three months from now to decide which extraction
   rule to improve.

### Part B — Batch processing

4. Justify using the Message Batches API for this weekly 80,000-invoice run
   instead of the synchronous API, and identify one workflow in this same company
   (name it) that should NOT use the batch API.
5. The business needs extraction results within 36 hours of document arrival.
   Calculate the maximum interval between batch submissions that guarantees this
   SLA, showing your work.
6. 300 of the 80,000 requests come back `errored` because the source PDFs
   exceeded the context window. Describe exactly how you identify and resubmit
   only those 300.
7. Before running the full 80,000-document batch for the first time, what should
   you do, and why does skipping this step risk higher total cost?

### Part C — Review architecture

8. Design a verification architecture for extracted invoices: should the same
   Claude call that did the extraction also verify it, or should this be a
   separate call? Justify using the self-review limitations concept.
9. The verification pass reviews all 80,000 extractions in a single pass. What
   risk does this introduce, and how would you restructure it using multi-pass
   review principles? (Note: this is extraction verification, not multi-file code
   review — adapt the underlying principle rather than assuming file structure.)
10. Add a confidence score to each verification finding and describe your routing
    rule: what confidence range gets auto-approved, and what gets routed to a
    human.

Write your answers as a short pipeline design document.
