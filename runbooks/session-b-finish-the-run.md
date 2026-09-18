# Session B — finish the run

Researches whatever domains remain, then builds the course. Runs to completion.

**Prerequisite:** Session A has finished and you have decided the scope — either all
remaining domains, or a narrowed set. **Decide before launching**, not mid-run.

**Branch:** the same branch as the pilot and Session A.

---

## Prompt

```
Scope: finish the run. Stages 4-6 for the domains named below, then
stages 7-9. Run to completion.

Already complete on this branch, and NOT to be redone:
  - domain-map.md              (passed evaluation)
  - research for: <DOMAINS ALREADY DONE>   (passed evaluation)
Their verdict files are in runs/<cert-slug>/evaluations/ if you need to
see what was checked.

DO:
1. Spawn one domain-researcher per remaining domain, in parallel:
     - <REMAINING DOMAIN 1>
     - <REMAINING DOMAIN 2>
     - <REMAINING DOMAIN 3>
   Give each its own task statements and bullets verbatim from the
   domain map, plus the path to the archived exam guide.
2. Commit, then evaluate each with the evaluator, writing verdict files.
3. On rework, route back to that researcher only. Hold outputs that
   already passed — never re-run a green-lit researcher.
4. When every domain has passed, spawn course-builder with the domain
   map and ALL research outputs, including the ones from Session A.
5. Commit, then evaluate course-builder's output.
6. On PASS, report where the course was written.

DO NOT:
- Do not re-run domain-mapper.
- Do not re-research a domain that already passed.
- Do not skip evaluation on a stage because its output looks fine.

HARD RULE: never read anything under reference/. It holds a
human-maintained answer key used to grade runs afterwards, including
one for this certification. Reading it would make the output a copy of
the answer rather than real work, and the failure would be invisible.

BUDGET: if you reach a point where continuing would clearly exhaust the
remaining budget before the course is built, stop and say so rather
than pressing on. A complete course over fewer domains is worth more
than an abandoned run over all of them.

You are the orchestrator. Do the work through the agents.

REPORT at the end:
- Per domain: pass or rework, rounds taken.
- Bullets received vs covered, per domain and in total.
- Where the course was written, and how many modules.
- Whether every bullet in the domain map is mapped to a lesson in
  course-outline.md.
- Total cost.
- Anything that broke or that you worked around.
```

---

## When it finishes — grade it

1. **Coverage.** Does `course-outline.md` map every bullet to a lesson? Count them.
   That claim is the whole point of the coverage work; check it rather than trust it.
2. **Against ground truth.** Diff the domain map against the answer key in
   `reference/`. Statements, bullets, weights.
3. **Read the verdict files.** Every rework round left one. This is the first run
   where "was the evaluator doing real work?" is a question with an answer.
4. **Unsourced concepts.** Grep the course for `UNSOURCED`. Each one should be taught
   and visibly flagged in the lesson, never dropped and never presented as sourced.

---

## Filled in for the current test certification

> *Example only — CCAR-F. Swap for whatever you are running.*

- **Branch:** `claude/relaxed-carson-9gx0xd`
- **Cert slug:** `claude-certified-architect-foundations`
- **Already done in Session A:** Agentic Architecture & Orchestration; Context Management & Reliability
- **Remaining (full scope):** Tool Design & MCP Integration (18%); Claude Code Configuration & Workflows (20%); Prompt Engineering & Structured Output (20%)
- **Narrowed scope, if Session A came back expensive:** drop Tool Design (18%) and keep the two 20% domains — the course would then cover 87% of the exam across four domains
