# Module 6 Hands-On Exercise

## Design a reliable long-running document-processing agent

You are designing an agent that processes a backlog of 5,000 long legal
contracts over several days, extracting structured fields from each and
flagging ones a human should review, while remaining reliable and
auditable.

1. **Context strategy.** The agent works through many contracts per
   session, and each contract is long. Decide, with justification: (a)
   would you configure server-side compaction, context editing
   (`clear_tool_uses_20250919`), both, or neither, and at what trigger; (b)
   how would you place each contract's text and the extraction instructions
   in the prompt to mitigate "lost in the middle" risk.

2. **Multi-day handoff.** The backlog will take more than one day. Sketch,
   using the long-running-harness pattern from Lesson 6.3, what an
   initializer agent should set up once, and what a per-session "wake up"
   routine should read before starting work on the next contract.

3. **Provenance.** For each extracted field (e.g. "indemnification cap:
   $2,000,000"), specify how you would preserve provenance back to the
   source contract — using the Citations API, the `<document>`/`<quotes>`
   prompting pattern, or both — and justify your choice.

4. **Confidence and sampling.** Design a QA process using field-level
   confidence scores and stratified random sampling (Lesson 6.5) that would
   catch a systematic extraction error even if it only affected fields the
   model scored as high-confidence. Be specific about what you'd stratify
   by.

5. **Escalation.** Using the exam-guide escalation triggers and
   anti-patterns from Lesson 6.4, write the escalation rule for this system
   in plain language — and explicitly state two things it must NOT use as
   the sole basis for escalating (per the named anti-patterns), and what it
   should do instead for an ambiguous case (e.g. a contract that could
   belong to either of two client accounts).

6. **Error propagation.** If this system uses a coordinator that delegates
   each contract to a worker subagent, and one worker subagent's process is
   killed by a rate-limit error mid-extraction, what must the coordinator
   do, and what must it NOT assume?

### What a strong answer includes
- A context-management choice with a stated trigger and reasoning tied to
  the contract-processing workload, not a generic "use compaction."
- Document-then-query prompt ordering, explicitly linked to the "lost in
  the middle" mitigation.
- A harness design that separates one-time setup from repeatable
  per-session state reconstruction (progress file, git history equivalent).
- A provenance mechanism that ties every extracted field back to a specific
  location in a specific contract.
- A stratification scheme (e.g., by field type and by confidence band) that
  would catch a high-confidence systematic error, not just a low-confidence
  spot check.
- An escalation rule that explicitly avoids sentiment and self-reported
  confidence as sole triggers, and asks for another identifier rather than
  guessing on ambiguous cases.
- Explicit acknowledgment that the coordinator must not treat subagent
  silence as "nothing to report" and must detect/handle the missing result.
