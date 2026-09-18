# Session A — price the fan-out

Runs **two** `domain-researcher` instances and their evaluations, then stops.

**Purpose:** the researcher fan-out is the most expensive stage in the pipeline. This
session buys a real cost-per-domain figure before committing to all of them.

**Which two domains:** pick the one with the **highest exam weight** and the one with
the **most bullets**. They bracket the range, so the average is a fair basis for
extrapolation. Often they are different domains; if they are the same one, take the
second-heaviest as the other.

**Branch:** continue on the branch the pilot created. The domain map lives there.

---

## Prompt

```
Scope: stages 4-5 only, TWO DOMAINS ONLY, then STOP.

The domain map is already complete and has PASSED evaluation on this
branch:
  runs/<cert-slug>/domain-map.md
Do NOT re-run domain-mapper. Do not re-fetch or re-extract the exam
guide. That stage is done and graded.

DO:
1. Spawn one domain-researcher for each of these two domains ONLY:
     - <DOMAIN A NAME>
     - <DOMAIN B NAME>
   Give each its own task statements and their bullets, verbatim from
   the domain map, plus the path to the archived exam guide.
2. Commit their output.
3. Evaluate each output with the evaluator, writing a verdict file per
   the path convention in CLAUDE.md.
4. STOP.

DO NOT:
- Do not research any other domain.
- Do not run course-builder.
- Do not stop early either: if a researcher is sent back for rework,
  work the rework rounds as normal, up to the retry cap.
- Even if both domains pass on round 1, STOP. The remaining domains are
  deliberately out of scope and will be run in a separate session.

HARD RULE: never read anything under reference/. It holds a
human-maintained answer key used to grade runs afterwards, including
one for this certification. Reading it would make the output a copy of
the answer rather than real research, and the failure would be
invisible.

You are the orchestrator. Do the work through the agents — do not
research or evaluate yourself, even if it looks faster.

REPORT when you stop:
- Per domain: pass or rework, and how many rounds.
- Paths written: research files, verdict files.
- Per domain: bullets received vs bullets covered.
- Concepts produced per domain.
- The session's total cost so far.
- Anything that broke, was ambiguous, or that you worked around.
```

---

## After it stops — the decision

```
cost per domain  = session cost / 2
projected total  = cost per domain × (number of domains) × 1.25
```

The 1.25 covers `course-builder` and its evaluation. Compare against remaining budget:

| Projection | Do |
|------------|-----|
| Comfortably under | Run all remaining domains (Session B) |
| Marginal | Narrow to the heaviest domains that still cover most of the exam |
| Way over | Stop. Keep these two. Rethink before spending more |

Also read one verdict file properly before continuing. A cheap run that produced
shallow research is not a bargain.

---

## Filled in for the current test certification

> *Example only — CCAR-F, the certification being used to test the pipeline. Swap
> these values for whatever you are running.*

- **Branch:** `claude/relaxed-carson-9gx0xd`
- **Cert slug:** `claude-certified-architect-foundations`
- **Domain A** (highest weight, 27%): `Agentic Architecture & Orchestration` — 7 statements, 48 bullets
- **Domain B** (most bullets, 53): `Context Management & Reliability` — 6 statements
- **Domains total:** 5
- **Projection multiplier:** `cost / 2 × 5 × 1.25`
