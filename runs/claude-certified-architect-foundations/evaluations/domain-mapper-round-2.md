# Evaluation — domain-mapper — round 2

- Output evaluated: `runs/claude-certified-architect-foundations/domain-map.md`
- Checklist: domain-mapper's "Done when" checklist
- Source of truth used:
  - Official certification page: https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification (not re-fetched this round; round 1 already confirmed it carries no domain content and links to the archived PDF — no change to this claim in round 2's diff)
  - Archived exam guide: `runs/claude-certified-architect-foundations/sources/exam-guide.txt` (read in full: lines 1–200, 280–450, 450–670, 673–810)
- Repeat round: yes (round 1 verdict: `runs/claude-certified-architect-foundations/evaluations/domain-mapper-round-1.md`)
- Verdict: PASS

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every domain named on the official page is listed, each with a short description and its weighting where the guide states one. | PASS | All 5 domains present with descriptions and weightings 27% / 18% / 20% / 20% / 15%, matching exam-guide.txt Section 4 blueprint (lines 75–79) exactly. Unchanged from round 1 (previously PASS). |
| 2 | The official source material is saved to the run folder, and its path appears under `## Sources`. | PASS | `## Sources` (lines 3–11) unchanged from round 1: cites the certification page URL, the PDF URL, and the archive path, which exists. Previously PASS, not touched by the round-2 revision (mapper reported only count figures changed). |
| 3 | Every task statement in the guide is reproduced verbatim and complete, grouped under its domain. | PASS | All 30 task statements present and grouped correctly. Re-verified by reading exam-guide.txt lines 1–200 (Domain 1), 280–450 (Domains 2–3), 450–670 (Domains 3–4), 673–810 (Domain 5) side-by-side against domain-map.md — wording is verbatim throughout, including titles. No regression from round 1's PASS. |
| 4 | Every bullet beneath every task statement is reproduced verbatim too, with the captured count reported per domain and in total so it can be checked against the guide. | PASS | Bullet content is verbatim (confirmed this round by full read of exam-guide.txt Domains 1, 2, 3, 4, 5 against domain-map.md — no drops, additions, or paraphrases). **The count figures flagged wrong in round 1 are now correct**: manually tallying `-K`/`-S` IDs actually present in the file and independently tallying bullets in exam-guide.txt itself give identical results for every domain: D1 24K/24S/48, D2 20K/23S/43, D3 23K/26S/49, D4 22K/25S/47 (the previously-wrong domain, now fixed from the reported 23K/24S to the correct 22K/25S), D5 24K/29S/53. Grand total 113 Knowledge / 127 Skills / 240 bullets (Summary counts table, lines 480–487) matches both the file's own bullets and the guide. |
| 5 | Every bullet carries an ID per the convention (positional, starting at 1 within each list, no gaps), and the convention is stated in the output. | PASS | `## Bullet ID convention` (lines 39–50) unchanged from round 1, still correctly stated. IDs spot-checked across all domains are positional, start at 1, no gaps. Unchanged from round 1's PASS. |
| 6 | If the guide genuinely contains no task statements, say so under `## Notes on sourcing`... | N/A / PASS | Not applicable — guide does contain task statements. Unchanged from round 1. |
| 7 | If the source was prose rather than an explicit list, domains are still returned in structured format. | N/A | Not applicable — guide's Section 6 is already an explicit structured list. Unchanged from round 1. |
| 8 | If the official page couldn't be found/accessed, that failure is reported instead of a fabricated/guessed list. | PASS | Page was found and accessed; correctly reported as containing no domain content. Unchanged from round 1. |

## Persisting issues

- Round 1 flagged two problems under item 4: (a) the Domain 1 and Domain 4 rows of the Summary counts table, and the grand total, did not match the bullets actually present in the file; (b) `## Notes on sourcing` stated "Total task statements: 27" while the body and table correctly total 30 — an internal contradiction.
  - (a) is now resolved: Domain 1 (24K/24S), Domain 4 (22K/25S, was wrongly 23K/24S), and the grand total (113K/127S/240) all now match both the file's own bullet IDs and an independent tally against exam-guide.txt.
  - (b) is now resolved: line 32 of `## Notes on sourcing` now reads "Total task statements: 30 (Domain 1: 7, Domain 2: 5, Domain 3: 6, Domain 4: 6, Domain 5: 6)," which is correct and consistent with the rest of the document.
- No other issue was raised in round 1 (all other checklist items already passed then and remain unchanged and passing now).

## Not checked

- The certification page itself was not re-fetched live this round, since the mapper reported no change to sourcing prose and round 1 already verified it via a live fetch. The `## Sources` and `## Notes on sourcing` text is byte-identical to what round 1 evaluated (aside from the corrected total-task-statements figure), so re-fetching would not add information.
- The PDF's original binary bytes were not independently re-diffed bullet-for-bullet against every one of the 240 bullets; verification was done by reading the full text of exam-guide.txt covering all 5 domains and comparing wording and per-list bullet counts against domain-map.md, plus arithmetic cross-checks of the Knowledge/Skills tallies. No discrepancies were found in the portions checked (which this round covered end-to-end for all 5 domains, more thoroughly than round 1's spot-check approach).
