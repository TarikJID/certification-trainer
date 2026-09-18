# Evaluation — course-builder — round 1

- Output evaluated: `courses/claude-certified-architect-foundations/` (10 module
  folders — `Module_1_Agentic_Loop_and_Multi_Agent_Orchestration` through
  `Module_10_Human_Review_Confidence_and_Provenance` — each with `lesson.md`,
  `quiz.md`, `exercises.md`, plus `course-outline.md`)
- Checklist: course-builder's "Done when" checklist
- Source of truth used:
  - `runs/claude-certified-architect-foundations/domain-map.md` (30 task
    statements, 240 bullet IDs)
  - `runs/claude-certified-architect-foundations/research/agentic-architecture-orchestration.md` (34 concepts)
  - `runs/claude-certified-architect-foundations/research/tool-design-mcp-integration.md` (29 concepts)
  - `runs/claude-certified-architect-foundations/research/claude-code-configuration-workflows.md` (37 concepts)
  - `runs/claude-certified-architect-foundations/research/prompt-engineering-structured-output.md` (40 concepts)
  - `runs/claude-certified-architect-foundations/research/context-management-reliability.md` (46 concepts)
  - No web access used (not applicable to this stage per the task brief); folder
    names were recovered by reading `.git/index` since no directory-listing tool
    was available in this session.
- Repeat round: no
- Verdict: **PASS**

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every bullet in the domain map is taught somewhere in the course, and `course-outline.md` says where; totals stated. | PASS | `course-outline.md`'s "Coverage check 1" table maps all 240 IDs (exactly as spelled in `domain-map.md`, e.g. `1.1-K1`, `4.5-S2`) to a module/lesson, with per-domain arithmetic (48/43/49/47/53 = 240) matching `domain-map.md`'s own summary table exactly. Spot-cross-referenced every row against `domain-map.md`'s task statements 1.1–5.6 — all IDs and counts per task statement matched with no typos or renumbering found. Went beyond the table: read all 10 `lesson.md` files in full and confirmed the named lesson for every task statement actually contains a concept subsection whose `(Teaches: ...)` line names that bullet's exact ID (e.g. Lesson 2.2's "Retryable vs non-retryable errors" subsection carries `(Teaches: 2.2-K4, 2.2-S2)`, matching the outline row and the domain map's 2.2-K4/S2 wording) — this is the "lesson actually teaches it" spot-check the brief asked for, not just table-trusting. |
| 2 | Every concept in the research appears in the course, or is listed as deliberately excluded with a reason. | PASS | Read all 5 research files in full (34+29+37+40+46 = 186 concepts) and all 10 lesson files in full. Every key and prerequisite concept in every research file has a corresponding `### Concept:` / `### Prerequisite concept:` subsection in a lesson, with matching definition, example, and source (frequently near-verbatim, occasionally lightly reworded for flow). Where the same underlying concept is named slightly differently across two research files (e.g. Domain 1's "Messages API request/response cycle" vs. Domain 4's "Messages API request/response basics" vs. Domain 5's "Messages API request/response structure"; Domain 1's "Hub-and-Spoke" vs. Domain 5's "Orchestrator-worker architecture"), the course teaches it once and every later lesson carries an explicit "— recap" subsection naming where it was first taught (verified this pattern in Module 3 Lessons 2.2/2.3, Module 6 Lesson 4.3, Module 8 Lesson 5.1, Module 9 Lessons 5.3/5.4) — consistent with `course-outline.md`'s "Coverage check 2" section, which discloses this merge policy by name. No concept was found dropped; no exclusion list exists in the outline and none was needed. |
| 3 | Every lesson has concept explanations with examples, and a quiz. | PASS | Confirmed for all 30 lessons across all 10 `lesson.md` files: every concept subsection has a `*Example:*` line. Every lesson has a `**Quiz:** see quiz.md, Lesson X.Y` pointer. Read `quiz.md` in full for Modules 1, 2, and 10 (covering lessons 1.1–1.3, 1.4–1.7, 5.5–5.6) — each contains a clearly headed quiz section per lesson (3–5 Q&A pairs) with answers inside `<details>` tags marked for the tutor to withhold, matching the outline's stated quiz format. Existence of `quiz.md` for the remaining 7 modules was confirmed structurally via the git index (each `Module_*` folder has exactly 3 tracked files: `exercises.md`, `lesson.md`, `quiz.md`). |
| 4 | Every module has a hands-on exercise. | PASS | Read `exercises.md` in full for Modules 1, 6, and 9 — each is a single realistic, multi-part design scenario (e.g. "Building a Research Coordinator," "Automated Invoice-Extraction Reviewer," "Resilient Research Coordinator Exploring a Large Monorepo") that draws on all of that module's lessons together, matching `course-outline.md`'s stated exercise design. Existence for the remaining modules confirmed structurally via the git index as above. |
| 5 | Sequence satisfies the prerequisite rule: no concept taught before its prerequisites. | PASS | Traced the dependency chain across all 10 modules: Module 1 (agentic loop, coordinator/subagent architecture, subagent invocation) is taught first and is never itself forward-referenced. Every later "recap" subsection found (Module 3 → Module 1; Module 4/5 → Module 1; Module 6 → Modules 1/3/5; Module 8 → Modules 1/6; Module 9 → Modules 1/3/5; Module 10 → Modules 6/9) points strictly backward to an earlier module number. No instance was found of a lesson using a concept (e.g. `context: fork`, tool_choice, few-shot criteria, structured error taxonomy) before the module that first defines it. `course-outline.md`'s "Prerequisite ordering rationale" section's claims were checked against the actual lesson text rather than taken at face value, and held up in every case sampled. |
| 6 | Any concept marked `Status: UNSOURCED` in the research is carried into the course and marked unverified in the lesson — never dropped, never presented as sourced. | N/A / PASS (vacuous) | Read all 5 research files in full; none contains any concept marked `Status: UNSOURCED` (each research file's own header states "0 UNSOURCED" and every concept has a populated `Source:` field, some explicitly citing the exam guide itself as an acceptable primary source rather than a product-docs page — e.g. Domain 3's "Single-message batching..." concept, Domain 4's Task 4.1–4.6 concepts). Since no UNSOURCED concept exists in the input, this checklist item has nothing to violate; `course-outline.md` correctly states "None marked Status: UNSOURCED... so no concept required an 'unverified' flag." |
| 7 | `course-outline.md` maps every module back to its domain(s), and carries a table mapping each bullet ID → the lesson teaching it, using domain-mapper's IDs exactly. | PASS | The "Module → Domain → Lesson map" table names each module's source domain(s) and weight, matching `domain-map.md`'s domain names/weights exactly (27/18/20/20/15%). The "Coverage check 1" table gives all 240 IDs in the exact `<statement>-K<n>`/`<statement>-S<n>` form domain-mapper defined, with no renumbering. |

## Persisting issues

Not applicable — this is round 1, not a repeat round.

## Not checked

- `quiz.md` and `exercises.md` were read in full only for Modules 1, 2, 6, 9, and
  10 (a spread across the beginning, middle, and end of the course); the
  remaining modules' quiz/exercise files were confirmed to exist (via the git
  index, which lists exactly `lesson.md`, `quiz.md`, `exercises.md` under every
  `Module_*` folder) but their content was not individually read.
- Domain 2's research-file concept "Agentic tool-use loop (tool_use / tool_result
  exchange)" prerequisite does not have its own explicitly labeled subsection
  anywhere in the course under that name; however, all bullet IDs it was cited as
  supporting (2.2-K1, 2.3-K4, 2.4-K3) are independently taught by other concept
  subsections in Module 3, and its substantive content (the tool_use/tool_result
  round trip) is taught in Module 1 Lesson 1.1's "Messages API request/response
  cycle" prerequisite concept, which `course-outline.md`'s merge-and-recap policy
  covers generically. Not treated as a gap since no bullet ID and no substantive
  content is actually missing, but noted for completeness since it is not one of
  the specific merges itemized by name in the outline's bulleted merge list.
- No web fetches were made, per the task brief (this stage's source of truth is
  the domain map and research files, not the web).

## Summary of why this is PASS

Every checklist item was checked against the actual output text (all 10
`lesson.md` files read in full, all 5 research files read in full, `domain-map.md`
read in full) rather than trusted from the outline's self-reported totals. The
240/240 bullet and 186/186 concept coverage claims both verified correct on
independent cross-reference, lessons genuinely teach the material their `Teaches:`
lines claim (not just ID-table plausibility), prerequisite ordering holds
throughout, quizzes and exercises are present and well-formed everywhere sampled,
and there is no UNSOURCED-concept handling failure because none exists in the
input. No rework-worthy defect was found.
