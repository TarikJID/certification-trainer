# Evaluation — domain-researcher (Context Management & Reliability) — round 1

- Output evaluated: runs/claude-certified-architect-foundations/research-context-management-reliability.md
- Checklist: domain-researcher's "Done when" checklist
- Source of truth used: the sources cited inside the output itself, verified by fetching:
  - https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  - https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
  - https://csrc.nist.gov/glossary/term/stratified_sampling
  - https://code.claude.com/docs/en/context-window
  - https://www.anthropic.com/engineering/multi-agent-research-system
  - https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
  Plus the bullet slice at /tmp/claude-0/-home-user-certification-trainer/5f9acdef-c166-585e-8b38-cc60c4e8c9c9/scratchpad/domain-5-context-management.md and the archived exam guide at runs/claude-certified-architect-foundations/source-exam-guide.txt (lines 675-813), used to confirm bullet coverage and the tier-1 baseline.
- Repeat round: no
- Verdict: REWORK

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every bullet you were given has at least one concept that teaches it; state bullets received/covered | PASS | Walked all 53 bullets (5.1: K1-K4,S1-S6; 5.2: K1-K4,S1-S5; 5.3: K1-K4,S1-S4; 5.4: K1-K4,S1-S5; 5.5: K1-K4,S1-S4; 5.6: K1-K4,S1-S5) against every `Teaches:` field in the file. Every bullet reference is named by at least one concept (note: the concept "Context rot," filed under the Task 5.1 heading, actually teaches 5.4-K1 — an odd placement but the coverage claim is still satisfied since the Teaches tag is correct). Claimed 53/53, confirmed 53/53. |
| 2 | Every key concept defined, illustrated with a concrete example, attributed to a source | PASS | Checked all 33 key-concept entries (Task 5.1-5.6 sections); each has Definition, Example, and Source fields populated with substantive content (e.g. "Context rot" lines 17-22, "Structured claim-source mapping preservation" lines 243-248). Spot-fetched several cited sources (Anthropic context-engineering blog, prompting-best-practices docs, NIST glossary, multi-agent-research-system blog, context-engineering-tools cookbook) and confirmed the definitions/examples in the output (e.g. n² attention/context rot, the exact "30 percent" long-context quote, the NIST stratified-sampling definition, the CitationAgent description, the 335K→169K token benchmark) accurately reflect what the sources actually say. |
| 3 | Each key concept's immediate prerequisites are identified and defined the same way | PASS | 11 prerequisite concepts (lines 282-357), each with Type: prerequisite, Teaches (mapped back to the key concepts/bullets that need them), Definition, Example, and Source. All fields populated; spot-checked "Context window as a finite, shared resource" (line 282) against code.claude.com/docs/en/context-window — the cited 4,200/680/280-token breakdown in the example matches the live page exactly. |
| 4 | Every cited source meets the quality bar (no forums/social/blogs) | PASS | All unique sources are the archived exam guide, official Anthropic/Claude documentation (platform.claude.com/docs, code.claude.com/docs, platform.claude.com/cookbook), official Anthropic engineering blog posts (anthropic.com/engineering), and a NIST CSRC glossary entry. None are forums, social media, personal blogs, Q&A sites, or aggregators. The Anthropic engineering blog posts are first-party vendor content (published on Anthropic's own domain, about their own product's internals) rather than a "personal blog," so treating them as acceptable ("tier 2, official vendor documentation" as the output itself labels them) is a reasonable application of the stated bar. |
| 5 | Every source recorded with its precedence tier; no higher tier used where a higher one was available | FAIL | The output systematically mislabels official Claude product documentation as **tier 1** instead of **tier 2**. Per the researcher's own stated precedence (1 = exam guide only, valid alone; 2 = official product/vendor documentation), every citation to platform.claude.com/docs, code.claude.com/docs, and platform.claude.com/cookbook should be tier 2 — but the output labels essentially all of them "tier 1," identically to the exam guide. Confirmed instances (not exhaustive): lines 29, 43, 50, 64, 119, 132, 173, 180, 187, 201, 228, 248 (corroborating citations) and lines 287, 294, 301, 308, 315, 329, 336, 343, 350 (sole citations for 9 of the 11 prerequisite concepts). By contrast, the same document correctly distinguishes anthropic.com/engineering blog posts as tier 2 (e.g. lines 22, 36, 91, 146, 153, 194, 235, 269) and the NIST glossary as tier 3 (line 221) — showing the researcher does understand tiering, but applied it inconsistently for the largest single bucket of citations (official docs sites). This is not a formatting nitpick: it is ~20+ citations recorded under the wrong precedence tier, which is exactly what this checklist item requires to be correct. |
| 6 | Any UNSOURCED concept carries Status: UNSOURCED and a Searched: record | PASS | No `Status: UNSOURCED` or `Searched:` entries appear anywhere in the file; the header claims 0 UNSOURCED concepts. Consistent with checking every one of the 44 concept entries present, all of which have a populated `Source:` field — so there is nothing that should have been marked UNSOURCED and wasn't. |

## Additional finding (not a checklist item, but requested verification)

The file's header (lines 5-6) claims "Key concepts: 35" and "Prerequisite concepts: 11" (46 total). A manual count of every `- Concept:` entry gives **33 key concepts + 11 prerequisite concepts = 44 total**, matching the orchestrator's grep count of 44. All 44 entries are well-formed (Type/Teaches/Definition/Example/Source all present) — this is not a case of malformed or missing concepts, and bullet coverage (item 1) is independently confirmed complete regardless of the count. The discrepancy is simply an inaccurate self-reported summary statistic (35 key claimed vs. 33 actual), not a content gap. No checklist item requires the summary counts themselves to be accurate, so this is not scored as a FAIL against a specific item, but it is a factual error worth the researcher correcting alongside the tier-labeling fix.

## Persisting issues

Not applicable — this is round 1.

## Not checked

- Did not individually re-verify every one of the ~30 total citations against their live pages (spot-checked 6 representative sources covering the largest citation clusters: the two most-cited Anthropic blog posts, the most-cited official docs page, the NIST glossary, the multi-agent-research-system blog, and the context-engineering-tools cookbook). No accuracy problems were found in any source checked; the tier-labeling defect found in the spot-checked sources appears to hold uniformly across the same URLs wherever they recur, based on direct inspection of every line containing "tier 1" in the file.
