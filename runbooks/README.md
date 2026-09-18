# Runbooks

Prompts for launching a run, and the reasoning behind how a run is split up.

These are **for a human to paste**, not for any agent to read. Nothing in the
pipeline loads this directory.

## Why runs are staged

A run is only worth its cost if it finishes. A pipeline that dies two-thirds of the
way through a parallel fan-out leaves you with research and no course — the worst
outcome, because the budget is spent and there is nothing to study from.

So a run is broken into sessions with an explicit stop, and each stop is a decision
point:

| Session | Scope | Question it answers |
|---------|-------|---------------------|
| **Pilot** | `domain-mapper` + its evaluation | Can the mapper find, extract and archive this certification's guide? |
| **A** | 2 researchers + evaluations | What does the expensive stage actually cost per domain? |
| **B** | Remaining researchers, then `course-builder` | — (committed by this point) |

After A you have a real price per domain. Multiply out, compare against the budget,
and if it does not fit, **narrow the scope rather than truncating the run** — three
domains researched and taught properly beats five domains abandoned at four.

## Rules that belong in every prompt

Copy these into any run prompt. They are the two fences that stop a staged run from
quietly becoming something else:

1. **A hard scope stop.** Name what to run, then name what NOT to run, explicitly —
   including the case where everything passes. "Even if it passes, stop."
2. **Never read `reference/`.** It holds human-maintained answer keys used to grade
   runs afterwards, possibly for the very certification being run. An agent reading
   from it produces a copy of the answer instead of real work, and the failure leaves
   no trace.

A third, when continuing a staged run:

3. **Name the work that is already done**, and say not to redo it. A fresh session
   has no memory of the previous stage and will happily re-run the mapper.

## Which branch

The pilot creates a branch. **Every later session continues on that same branch** —
that is where the verified domain map lives. Never start a later stage from `main`,
or you will pay for the mapper again and lose the artifact you already graded.

Run output stays on its branch. It is evidence, not product; `main` holds the
pipeline, not the results of running it.
