# Evaluation — domain-mapper — round 1

- Output evaluated: runs/claude-certified-architect-foundations/domain-map.md
- Checklist: domain-mapper's "Done when" (verbatim, as supplied)
- Source of truth used:
  - Archived exam guide extraction: runs/claude-certified-architect-foundations/source-exam-guide.txt (read in full, 1407 lines, all 18 sections)
  - Live official page fetched to confirm archive fidelity: https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification
- Repeat round: no
- Verdict: PASS

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every domain named on the official page is listed, each with a short description and its weighting where the guide states one. | PASS | The live Skilljar page (fetched directly) does not itself enumerate domains — it only names four "core technology areas" and links to the Exam Guide PDF, confirming the mapper's Notes on sourcing claim. All 5 domains from the PDF's Section 4 blueprint table (source lines 76-82) are listed in domain-map.md with 1-2 sentence descriptions and weightings 27% / 18% / 20% / 20% / 15%, matching the source table exactly (domain-map.md lines 18, 92, 155, 227, 297). |
| 2 | The official source material is saved to the run folder, and its path appears under `## Sources`. | PASS | `## Sources` (domain-map.md lines 1-5) lists the Skilljar page URL, the PDF URL, and the archive path `runs/claude-certified-architect-foundations/source-exam-guide.txt`, which exists and was used for this evaluation. |
| 3 | Every task statement in the guide is reproduced verbatim and complete, grouped under its domain. | PASS | Source Section 6 (lines 145-814) contains exactly 30 numbered task statements (7+5+6+6+6 across Domains 1-5). All 30 appear in domain-map.md, correctly grouped by domain. Spot-checked full text of Task Statement 1.1 and Task Statement 5.6 (title, all Knowledge-of and Skills-in bullets) character-for-character against the archive — identical. No statement is missing, summarized, or paraphrased. |
| 4 | Every bullet beneath every task statement is reproduced verbatim too; the count is reported per domain and in total. | PASS | Recomputed bullet counts directly from the archived source, task by task: Domain 1 = 6+8+8+6+6+6+8 = 48; Domain 2 = 8+8+9+9+9 = 43; Domain 3 = 8+9+6+8+9+9 = 49; Domain 4 = 6+9+10+8+8+6 = 47; Domain 5 = 10+9+8+9+8+9 = 53. Total = 240. These match domain-map.md's claimed counts exactly, both the per-task breakdowns given inline ("Bullet count for this domain: 48 (Task 1.1: 6, 1.2: 8, ...)") and the `## Coverage summary` totals (30 task statements, 240 bullets). Spot-checked bullets are verbatim (see item 3). |
| 5 | If the guide genuinely contains no task statements, say so under `## Notes on sourcing`, naming sections checked and quoting the guide's actual structure. | PASS (not applicable, correctly documented) | The guide does contain explicit task statements (Section 6, confirmed above), so the negative case doesn't apply. `## Notes on sourcing` correctly identifies Section 6 as "an explicit, formally structured enumeration" and states "no prose-extraction fallback was needed" — an accurate, source-grounded claim rather than an unsupported assertion. |
| 6 | If the source was prose rather than an explicit list, domains are still returned in structured format, not left as paraphrase. | PASS (not applicable) | Not triggered — source was already structured (see item 5). Notes on sourcing explicitly states this and the output format used is fully structured regardless. |
| 7 | If the official page couldn't be found/accessed, report that failure instead of a fabricated/guessed domain list. | PASS (not applicable) | The page was accessible — confirmed directly by fetching it during this evaluation. No fabrication needed or present. |

## Persisting issues

None — this is round 1, no prior rounds exist.

## Not checked

- Did not independently re-verify every single one of the 240 bullets word-for-word against the archive (impractical at this volume); instead verified exact totals/per-task breakdowns arithmetically against the source and spot-checked two full task statements (1.1 and 5.6, the first and last-but-one) character-for-character with no discrepancies found. Given the per-domain and per-task bullet counts match the source exactly in every one of the 30 cases, and the spot-checked text is exact, there is strong evidence against silent truncation or paraphrase elsewhere.
- Did not verify that the linked PDF URL in `## Sources` is byte-identical to the PDF currently linked from the live page (the PDF itself was not re-downloaded); relied on the archived text's internal consistency (title, exam code, section count) and the live page's description matching the archive's Section 1 text almost verbatim as corroborating evidence of fidelity.
