# Evaluation and guardrails — what was implemented

A record of the controls added to this pipeline: what each one does, where it lives, and
whether it is enforced or merely requested.

Ordered by what they protect. Nothing here is aspirational — anything not yet built is
in [Not enforced](#not-enforced) at the end.

---

## Evaluation between stages

**An evaluator runs after every stage, not once at the end.**
`CLAUDE.md`, phase 2 — after `domain-mapper`, after each `domain-researcher` instance,
after `course-builder`. Returns PASS or REWORK.

**Each evaluation writes a verdict file.**
`.claude/agents/evaluator.md`. One file per stage, per round, at
`runs/<cert>/evaluations/<agent>-<unit>-round-<n>.md`. Each lists every checklist item
with an evidence column, including items that passed, plus a separate `Not checked`
section naming what was not verified.

**The evaluator never modifies the work it judges.**
`.claude/agents/evaluator.md`. It returns a verdict and a file path. Web access is for
verifying what is in the output, never for researching what is missing.

**The evaluator is checklist-driven, not source-driven.**
It verifies against whatever source *that stage's* checklist designates, rather than
naming one stage's source of truth in its own prompt. One evaluator serves all three
stages.

**Rework is routed to the instance that failed.**
`CLAUDE.md`, phase 2 step 8. On a parallel researcher fan-out, outputs that already
passed are held rather than regenerated.

**Rework is capped at 2 rounds per stage.**
`CLAUDE.md`. A third failure stops the run and reports to the user with the paths to
every verdict file for that stage. The orchestrator counts; the evaluator has no memory
across rounds.

**Work is committed before every evaluation.**
`CLAUDE.md`. `git log -p runs/` reconstructs a whole run afterwards — what changed,
when, in what order.

## Coverage

**The exam guide is archived as fetched.**
`runs/<cert>/sources/`. `domain-mapper` reproduces task statements verbatim rather than
summarising them.

**Every exam bullet gets a stable ID at the mapping stage.**
`.claude/agents/domain-mapper.md`. The IDs are emitted in the mapper's own output — not
assigned by the orchestrator — so they survive a lost session.

**Those IDs are cited through to the course outline.**
`.claude/agents/course-builder.md` requires a bullet-ID → lesson table. Coverage is
therefore a set comparison in both directions: nothing in the guide missing from the
course, nothing in the course invented.

**Dispatch slices are committed to the repo.**
`CLAUDE.md`, phase 2 step 4. Verdict files cite the inputs they judged; a slice left in
a scratchpad vanishes with the container and leaves the audit trail pointing at nothing.

**Researcher self-reported counts are checked against the file.**
`CLAUDE.md`, phase 2 step 6. A mismatch is flagged to the evaluator rather than
corrected by the orchestrator.

## Sourcing

**`course-builder` has no web tools.**
`.claude/agents/course-builder.md`, tool list. "Use only the material you were handed"
is enforced by tool scope, not prompt text. It cannot fetch anything to misattribute.

**Source quality was promoted from prose into the checklist.**
`.claude/agents/domain-researcher.md`. The rule previously lived only in a "You don't"
section, where the evaluator never reads it.

**Source tiers were collapsed to official / non-official, unranked.**
A four-tier precedence had made "prefer the highest tier" mean preferring a syllabus
line over the documentation that defines the thing, and created a way to be wrong about
a correct citation. A label is now required only where it changes a decision — on
non-official sources.

**Unsourceable concepts are flagged, never dropped.**
`.claude/agents/domain-researcher.md`. `Status: UNSOURCED` plus a `Searched:` record of
pages opened and queries run. Missing sources never block the pipeline. More than two
per domain is itself a rework trigger.

**Every cited URL must appear in a `Fetched:` list.**
`.claude/agents/domain-researcher.md`, output spec and checklist. `WebSearch` returns a
synthesised summary alongside the result list, so a concept can be written from that
summary and cited to a page that was never opened. A citation with no matching fetch is
now a checklist failure.

## Course output

**Quiz questions and answers are in separate files.**
`.claude/agents/course-builder.md`. `quiz.md` holds questions, `quiz-answers.md` holds
answers, and no answer text appears in `quiz.md`. Previously answers sat inline behind
`<details>` blocks, which hide nothing from a model reading the file.

**Quiz content is checked against taught content.**
`.claude/agents/course-builder.md` checklist. Every concept tested in a quiz or exercise
must be taught in a lesson at or before that module, with totals stated.

**Unsourced concepts are carried into the course, marked unverified.**
`.claude/agents/course-builder.md` checklist item 6. Never silently dropped, never
presented as sourced.

**Prerequisite ordering is enforced.**
`.claude/agents/course-builder.md` checklist. No concept is taught before its
prerequisites.

## Tutor

Shipped in `tutor-template/`, copied into a course at the end of a run.

**Quiz attempts are logged before answers are opened.**
`tutor-template/CLAUDE.md`. Combined with the `quiz.md` / `quiz-answers.md` split, the
answers are not in context while the question is asked. This is a rule the tutor
follows, not a wall — see [Not enforced](#not-enforced).

**Recall and application are tracked separately per concept.**
`tutor-template/progress/learner-progress.md`. Closed independently, with the condition
of each attempt recorded (`cold` / `cued` / `just-taught`). Only a cold attempt can
close recall.

**Concepts that did not land stay on a list.**
`tutor-template/CLAUDE.md`. Marked `shaky` and carried into later sessions.

## Removed

**A live integrity check was built and then cut.**
It ran `git status` after each evaluation to catch the evaluator editing work it judged.
Removed because committing between stages already records the same thing — `git log -p
runs/` detects it afterwards at no runtime cost — and because it put the most complex
procedure in the file into the orchestrator, the one component whose rules nothing
enforces. It also risked halting an expensive run on a false positive.

---

## Not enforced

Stated here rather than implied to exist elsewhere.

| | Status |
|---|---|
| **Quiz answer leaks** | The tutor logs an attempt before opening answers, which makes a leak detectable afterwards. Nothing reads that log, so today this **requests**, and does not measure |
| **Abandoning a concept that has not landed** | Prompt only. No capability to withhold, no check that can catch it in the moment, nothing decidable from the files. The per-concept record makes the failure visible after the fact; that is all |
| **Hallucination rate as a rate** | Individual faithfulness failures are caught per stage. Nothing aggregates them across a run, so there is no trend |
| **Observation Utilization** | The `Fetched:` list makes it computable. Nothing computes it |
| **Trajectory efficiency** | Not measurable — no baseline for the minimum number of searches a domain needs |
| **Teaching quality** | Not measured. Coverage and citation accuracy are verified; whether a lesson teaches well needs a human reading it |
| **Genericity** | Every control above was designed against a single certification. A run against a different exam, with no fixture in `reference/`, is what would test it |
