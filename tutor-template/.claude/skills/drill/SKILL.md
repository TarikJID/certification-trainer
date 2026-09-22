---
name: drill
description: Fast cold-recall checks on vocabulary and exact names the learner has not yet closed. Use at the start of a session, or when the learner asks to be drilled, tested on terms, or wants to check what stuck. Seconds per item, not a quiz.
---

# Drill — cold recall checks

## When to use
Opening a session when any concept has `recall: shaky`. Or the learner asks to check
what stuck.

This is **not** the quiz. It tests one thing: can they produce the exact term, path or
metric name with nothing in front of them.

## Steps

1. Read `progress/learner-progress.md`. Take concepts with `recall: shaky`.
   **Skip anything `closed`** — trust the record.
2. Pick 2–4. Fewer is fine. This should take under two minutes.
3. Ask for the name only. No context, no hints, nothing in the question that contains
   the answer.
   - *"What's the metric for 'of everything relevant that exists, how much did I
     retrieve'?"*
   - *"Exact path — where does a subagent definition live?"*
4. Wait. Do not fill the silence with the answer.
5. Record each attempt immediately, with condition `cold`:
   ```
   - `2026-09-22` · recall · cold · correct
   ```
6. Two clean cold recalls on **separate days** closes `recall`. Say so when it happens
   — "that's it closed, I won't ask again."
7. Wrong or blank: give it plainly, one line, no lecture. Log it. Move on.

## Don't
- Don't drill a closed item. That is the failure this whole design exists to prevent.
- Don't drill more than four items. It is a warm-up, not an exam.
- Don't accept "something like X" for an exact name — the point is precision. Be kind
  about it, but log it as wrong.
- Don't turn it into teaching. If they miss two in a row on the same concept, stop
  drilling and go re-teach it.
