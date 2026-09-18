# Evaluation — domain-mapper — round 1

- Output evaluated: `runs/claude-certified-architect-foundations/domain-map.md`
- Checklist: domain-mapper's "Done when" checklist
- Source of truth used:
  - Official certification page, fetched live: https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification
  - Archived exam guide: `runs/claude-certified-architect-foundations/sources/exam-guide.txt`
- Repeat round: no
- Verdict: REWORK

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every domain named on the official page is listed, each with a short description and its weighting where the guide states one. | PASS | All 5 domains listed with descriptions and weightings 27% / 18% / 20% / 20% / 15%, matching exam-guide.txt Section 4 blueprint (lines 75–80) exactly. |
| 2 | The official source material is saved to the run folder, and its path appears under `## Sources`. | PASS | `## Sources` (lines 3–11) cites the certification page URL, the PDF URL, and the archive path `runs/.../sources/exam-guide.txt`, which exists and contains the full guide text. Live re-fetch of the certification page confirmed it does not itself list domains/task statements and links to exactly this PDF URL, matching the mapper's account. |
| 3 | Every task statement in the guide is reproduced verbatim and complete, grouped under its domain. | PASS | All 30 task statements (1.1–1.7, 2.1–2.5, 3.1–3.6, 4.1–4.6, 5.1–5.6) are present, correctly grouped under their domains, with titles and wording matching exam-guide.txt verbatim on line-by-line comparison (e.g. Task Statement 1.2 title and both bullet lists at domain-map.md lines 75–85 match exam-guide.txt lines 164–185 word for word). |
| 4 | Every bullet beneath every task statement is reproduced verbatim too, with the captured count reported per domain and in total so it can be checked against the guide. | FAIL | Bullet *content* is verbatim and complete (spot-checked across all 5 domains against exam-guide.txt — no drops or paraphrases found). However the **reported counts are wrong**: the Summary counts table (lines 480–487) states Domain 1 = 28 Knowledge / 20 Skills, but manually tallying the actual `-K`/`-S` bullets present in Domain 1's own task statements (lines 65–140) gives 24 Knowledge / 24 Skills. Domain 4's table row states 23 Knowledge / 24 Skills, but the actual bullets present (lines 314–384) tally to 22 Knowledge / 25 Skills. Consequently the grand total row (118 Knowledge / 122 Skills) is also wrong — the bullets actually present in the file tally to 113 Knowledge / 127 Skills (domain-by-domain: D1 24/24, D2 20/23, D3 23/26, D4 22/25, D5 24/29). The overall bullet total of 240 is correct by coincidence, but the Knowledge/Skills split the mapper reports cannot be checked against the guide because it does not match either the guide or the mapper's own file. Separately, `## Notes on sourcing` (line 32) states "Total task statements: 27" while the document's own body and summary table correctly contain 30 — an internal contradiction that further undermines confidence in the self-reported figures this checklist item requires. |
| 5 | Every bullet carries an ID per the convention (positional, starting at 1 within each list, no gaps), and the convention is stated in the output. | PASS | `## Bullet ID convention` (lines 39–50) states the convention and correctly notes the `-M<n>` form is unused since every task statement has both headings. Spot-checked IDs across all domains (e.g. 1.2-K1..K4, 1.2-S1..S4, 4.3-K1..K4, 4.3-S1..S6) are positional, start at 1, and have no gaps. |
| 6 | If the guide genuinely contains no task statements, say so under `## Notes on sourcing`, naming sections checked and quoting the guide's actual structure. | N/A / PASS | Not applicable — the guide does contain task statements (Section 6, verified in exam-guide.txt lines 136–809), and the mapper correctly treated it as such rather than fabricating a "not applicable" claim. |
| 7 | If the source was prose rather than an explicit list, domains are still returned in the structured format above, not left as paraphrase. | N/A | Not applicable — the guide's Section 6 is already an explicit structured list (Task Statement headings with Knowledge/Skills sub-bullets), not prose requiring extraction. |
| 8 | If the official page couldn't be found/accessed, that failure is reported instead of a fabricated or guessed domain list. | PASS | The page was found and accessed; the mapper correctly reported that it contains no domain content itself and points to the PDF guide, which matches the live re-fetch performed for this evaluation. |

## Persisting issues

N/A — this is round 1, not a repeat round.

## Not checked

- The PDF's original binary bytes were not independently re-fetched/diffed against `exam-guide.txt` bullet-for-bullet across all 240 bullets; verification was done by structural comparison (task statement titles, all of Domain 1's bullets in full, spot checks across Domains 2–5) plus the arithmetic tally described above. No discrepancies in bullet *wording* were found in the portions checked.
- Sample questions (Section 9) and administrative sections (11–16) were not verified since the checklist does not require their reproduction.

## Summary of why this is REWORK

Checklist item 4 explicitly requires the mapper to "Report the count you captured, per domain and in total, so the figure can be checked against the guide." The reported Knowledge/Skills split is demonstrably wrong for Domain 1, Domain 4, and the grand total — verifiable by simply counting the `-K`/`-S` IDs already present in the mapper's own file. This is exactly the kind of self-reported figure the checklist asks for and it fails the check it was designed to enable, even though the underlying bullet content itself is complete and verbatim.
