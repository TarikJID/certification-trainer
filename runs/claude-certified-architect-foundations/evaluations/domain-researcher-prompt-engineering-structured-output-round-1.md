# Evaluation — domain-researcher (Domain 4: Prompt Engineering & Structured Output) — round 1

- Output evaluated: `runs/claude-certified-architect-foundations/research/prompt-engineering-structured-output.md`
- Checklist: domain-researcher's "Done when" checklist
- Source of truth used: sources cited in the output itself —
  `runs/claude-certified-architect-foundations/sources/exam-guide.txt` (Domain 4 section, lines 538-668, read directly),
  and the following fetched live pages: https://platform.claude.com/docs/en/build-with-claude/batch-processing,
  https://platform.claude.com/docs/en/build-with-claude/structured-outputs,
  https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview,
  https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools,
  https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices,
  https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations,
  https://platform.claude.com/docs/en/build-with-claude/extended-thinking,
  https://claude.com/blog/code-review.
  Also cross-checked the dispatch slice `runs/claude-certified-architect-foundations/dispatch/prompt-engineering-structured-output.md`
  for the 47 bullet IDs.
- Repeat round: no
- Verdict: PASS

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every bullet has at least one concept that teaches it, named in the concept's `Teaches:` field; state bullets received/covered | PASS | Header states "Bullets received: 47 ... Bullets covered: 47/47." Verified by hand: every one of the 47 IDs in the dispatch slice (4.1-K1..K3, 4.1-S1..S3, 4.2-K1..K4, 4.2-S1..S5, 4.3-K1..K4, 4.3-S1..S6, 4.4-K1..K4, 4.4-S1..S4, 4.5-K1..K4, 4.5-S1..S4, 4.6-K1..K3, 4.6-S1..S3) appears in the `Teaches:` field of at least one **key** concept (not merely a prerequisite's "supports" list). E.g. 4.3-S6 → "Format normalization rules alongside strict schemas"; 4.6-S3 → "Confidence self-reporting for calibrated review routing." |
| 2 | Every count stated matches the file | PASS | Header claims 40 concepts (32 key, 8 prerequisite). Manual count: 8 prerequisite concepts (JSON Schema fundamentals; Tool use round trip; Messages API basics; Be clear and direct; Hallucination in LLM outputs; Multishot/few-shot prompting; Extended thinking; Structured/typed error results) = 8. Key concepts counted per task: 4.1=4, 4.2=5, 4.3=8, 4.4=5, 4.5=6, 4.6=4 → 32. Total 40. Matches header exactly. |
| 3 | Every key concept defined, illustrated with a concrete example, attributed to a source | PASS | All 32 key concepts have `Definition:`, `Example:`, and `Source:` fields present (spot-checked all task sections 4.1–4.6). |
| 4 | Immediate prerequisites identified and defined the same way | PASS | All 8 prerequisite concepts have `Definition:`, `Example:`, `Source:` fields, and each names which key concepts it supports. |
| 5 | Every cited source meets the quality bar (official; no forums/social/blogs except vendor engineering write-ups) | PASS | All citations are either the exam guide itself or `platform.claude.com/docs/...` pages, or `claude.com/blog/code-review` (Anthropic's own product blog, explicitly counted as official per the brief). No forum/social/unofficial sources used anywhere in the file. |
| 6 | No non-official source used where avoidable; official sources not ranked | PASS | No non-official sources appear at all, so the "why nothing official covers it" justification requirement never triggers. Not raising any finding about which official source (exam guide vs. product docs) was chosen for a given concept, per the brief's explicit instruction. |
| 7 | UNSOURCED concepts carry `Status: UNSOURCED` and a `Searched:` record | PASS | Header claims 0 UNSOURCED. Scanned the full file: no `Status: UNSOURCED` marker appears anywhere, consistent with the claim. |

### Spot-verification of source fidelity (beyond presence/absence)

Fetched and compared several of the cited pages against what the output attributes to them:
- Batch processing page: confirms 50% discount on both input/output pricing, most batches finish under an hour, 24-hour expiration for unprocessed requests, `custom_id` per-request correlation, results returned in `succeeded`/`errored`/`expired`/`canceled` types, results not guaranteed in submission order — all matches what the output's "Message Batches API fundamentals" and "custom_id request/response correlation" concepts claim.
- `define-tools` page: confirms the four `tool_choice` values (`auto`, `any`, `tool`, `none`) exactly as described in "tool_choice options (auto / any / forced tool)."
- `claude-prompting-best-practices` page: confirms "Be clear and direct" framing (new-employee analogy) and the multishot/few-shot guidance (relevant/diverse/structured examples, `<example>`/`<examples>` tags, 3–5 examples) exactly as paraphrased in the two matching prerequisite concepts.
- `reduce-hallucinations` page: confirms all four listed mitigations (allow "I don't know," ground in direct quotes, verify with citations, restrict to provided material) as described in the "Hallucination in LLM outputs" prerequisite.
- `tool-use/overview` page: confirms the `tool_use`/`tool_result` round trip exactly as described in the matching prerequisite.
- `claude.com/blog/code-review`: confirms the "less than 1% of findings are marked incorrect" statistic cited under "False positive rate and developer trust" (output paraphrases as "fewer than 1%," accurate). The same blog is also cited, more loosely, as supporting "independent-agent verification in production" for the 4.6 concept; the fetched page describes parallel agents that "look for bugs in parallel, verify bugs to filter out false positives" without explicitly confirming those verifying agents lack the generating agent's reasoning context. This is a looser paraphrase than ideal, but the core claim for that concept is grounded primarily in the exam guide (lines 657-658, 663-664), and the blog is cited only as secondary "production" color, not as the concept's sole source, so it does not amount to citing an official source for content the source doesn't contain.
- Cross-checked ~10 of the exam-guide line citations (e.g. Task 4.1 lines 543-545/552-553, Task 4.3 lines 592-593/604-605, Task 4.5 lines 646-647, Task 4.6 lines 655-668) against the archived exam guide text: all line ranges cited correspond to the actual bullet text they are used to support.

Note: several "Example:" illustrations in the file use invented concrete numbers not literally present in the exam guide (e.g. a $210/$250 invoice discrepancy, a 40-file PR, 0.9/0.4 confidence values, a 6-hour submission interval, 10,000 documents with 40 errored). The checklist's "illustrated with a concrete example" item does not require examples to be verbatim-sourced — only the concept's definition/attribution needs a source — and the file's own source note correctly scopes its "exam guide's own illustrative content" claim to specific terminology (e.g. `detected_pattern`, `calculated_total`, the code-review criteria wording) that is genuinely present in the guide's bullets, not to the invented numeric scenarios built around that terminology. This is not a checklist violation.

## Persisting issues

None — this is round 1, not a repeat.

## Not checked

- `https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools` was fetched in full and cross-checked against multiple concepts (tool_choice options, tool use with JSON schemas); the nullable/enum-specific claims in "Schema design" and "Optional/nullable fields" were checked against a summarized fetch of `structured-outputs` rather than the full raw page text, since the full page was not read verbatim. The summarized fetch did confirm the general nullable-field and enum mechanism exists, which is sufficient to support the checklist's "attributed to a source" requirement, but exact JSON Schema syntax details (e.g. whether Anthropic's structured-outputs feature literally supports `["string","null"]` unions vs. only `anyOf`) were not verified character-for-character.
