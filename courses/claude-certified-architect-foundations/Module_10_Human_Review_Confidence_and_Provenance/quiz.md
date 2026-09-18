# Module 10 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 5.5 (Human Review and Confidence Calibration)

**Q1.** An extraction pipeline reports 97% overall accuracy. Is it safe to reduce
human review across the board? What must you check first?

<details><summary>ANSWER</summary>
Not necessarily safe. The aggregate figure can mask much worse performance on a
specific document type or field segment. You must validate accuracy broken down
by document type and field before reducing review, since errors tend to cluster
in specific segments rather than spreading evenly.
</details>

**Q2.** Why does a QA process that only reviews low-confidence extractions miss
important failure modes?

<details><summary>ANSWER</summary>
It misses the case where the model develops a new failure mode that happens to
produce high, but wrong, confidence. Stratified random sampling — drawing from
each stratum (document type, confidence band, field type) including
high-confidence extractions — is needed to catch that.
</details>

**Q3.** A field-level confidence score of 0.9 means correct 96% of the time for
"invoice date" but only 82% of the time for "line-item total." What does this
tell you about using raw model confidence directly as a review threshold?

<details><summary>ANSWER</summary>
Raw model-reported confidence is not automatically a calibrated probability — the
same 0.9 score means different actual correctness rates for different fields.
The score must be calibrated against a labeled validation set per field, and
review thresholds set based on that measured relationship, not the raw number.
</details>

**Q4.** Given limited reviewer capacity, which of these two extractions should be
prioritized for human review: (a) low confidence on an unambiguous field, or (b)
low confidence on a field where two source pages state contradictory values?
Why?

<details><summary>ANSWER</summary>
(b) — routing should prioritize extractions that most need attention. Low
confidence combined with ambiguous/contradictory source documents is a stronger
signal of an untrustworthy extraction than low confidence alone, so it should be
routed ahead of (a).
</details>

---

## Quiz — Lesson 5.6 (Information Provenance and Multi-Source Synthesis)

**Q1.** Two sources report slightly different figures for the same statistic.
After an unstructured summarization pass, the report says "reports suggest
values in this range" with no way to tell which source said what. What went
wrong, and what's the structural fix?

<details><summary>ANSWER</summary>
Source attribution loss during summarization — the summarizer merged
similar-sounding claims from different sources, discarding which source said
what. The fix: require subagents to output structured claim-source mappings
(claim, source, excerpt) and require the synthesis agent to preserve and merge
those mappings, not just the claims.
</details>

**Q2.** Source A reports a market size of $4.2B and Source B reports $5.1B for
what looks like the same fact. What should the document-analysis output do,
and what should it NOT do?

<details><summary>ANSWER</summary>
It should preserve both values with source attribution, explicitly annotated as
conflicting, and let the coordinator or downstream decision-maker decide how to
reconcile them. It should NOT silently pick one value, which discards
information and can be wrong.
</details>

**Q3.** Source A (2023) reports headcount as 500; Source B (2026) reports 1,200.
Without any additional metadata, how might this be misread, and what field
prevents that misreading?

<details><summary>ANSWER</summary>
Without dates, this could be misread as two sources disagreeing about the same
current number (a contradiction). Requiring subagents to include the publication
or data-collection date of each source lets a downstream synthesis step
correctly interpret this as temporal change (growth over three years) instead.
</details>

**Q4.** How should a synthesis report structurally distinguish a finding
supported by three independent sources from a finding supported by only one
lower-confidence source?

<details><summary>ANSWER</summary>
Use explicit separate sections — e.g. "Well-established findings" versus
"Contested / single-source findings" — preserving each source's own
characterization and methodological context, rather than presenting both kinds
of claim with the same tone and certainty in undifferentiated prose.
</details>

**Q5.** A synthesis report includes quarterly revenue figures, a market
narrative, and a list of technical compliance requirements. Should all three be
rendered as prose? Why or why not?

<details><summary>ANSWER</summary>
No — each content type should be rendered in the format it's naturally
structured for: revenue figures as a table, the narrative as prose, and
compliance requirements as a structured/numbered list. Forcing everything into
one uniform format typically loses information the natural format would have
preserved and reads worse.
</details>
