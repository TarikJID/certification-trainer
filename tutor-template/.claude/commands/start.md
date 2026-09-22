---
description: Onboard a new learner and begin the first module
---

Onboard the learner.

1. Read `course-outline.md` — what certification this is, and what the modules cover.
2. Read `progress/learner-progress.md`. If a profile already exists, this is not a new
   learner: greet them and run `/progress` instead.
3. Introduce the course in a few lines — **in plain language.**

   Say what the exam covers, how many modules there are, and why they are in that
   order. Domain names and their weights are worth giving: they tell a learner where
   the marks are.

   **Do not use the exam guide's internal vocabulary.** Terms like *task statement*,
   bullet IDs, and section numbers such as "1.1 through 5.6" mean nothing to someone
   who has not read the guide — and on day one, nobody has. They are how the pipeline
   tracks coverage, not how a person thinks about what they are learning. Say "30
   lessons, one per exam objective" rather than "one lesson per task statement
   (1.1–5.6)".
4. Ask their name, and how they like to learn:
   - **Socratic** — guiding questions
   - **Lecture + checkpoints** — explain, then check
   - **Hands-on** — drive them through doing it
5. Ask whether anything about studying usually trips them up. Record it verbatim under
   Notes — it is more useful than a paraphrase.
6. Write the profile to the progress file, then start Module 1 with `teach-module`.
