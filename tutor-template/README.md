# Tutor template

Copied into `courses/<certification-name>/` at the end of a run, turning the generated
course into a folder a learner can open in Claude Code and be taught from.

```
CLAUDE.md                        the tutor's behaviour
progress/learner-progress.md     learner state, empty
.claude/skills/                  teach-module · quiz-me · drill · explain-eli5 · build-along
.claude/commands/                /start · /progress
```

**Nothing here is generated, and nothing here is certification-specific.** It is a
straight copy, byte-identical every run. The module list is not duplicated into the
tutor — it reads `course-outline.md`, which the course already carries, so there is
nothing to go stale.

## The two design decisions worth knowing

**Two axes, closed independently.** Every concept is tracked on `recall` (can name it
cold) and `application` (can use it correctly). They fail separately. A learner whose
reasoning is strong and whose vocabulary is weak will apply a concept perfectly and
still not be able to name it — collapse the axes into one status and the strong one
hides the weak one.

**The progress file records evidence, not intentions.** A note saying *"check whether
they remember X"* can never be satisfied, so it survives every session and X gets asked
again forever. A record of a clean cold answer closes the item, and a closed item is
never re-asked. This is the single most important property of the file, and it came
from watching the failure happen.

## Guardrail status

Two of the four policies in [`../GUARDRAILS.md`](../GUARDRAILS.md) are tutor policies,
and this template is where they land. Be precise about what they achieve:

| Policy | Mechanism here | Verb |
|---|---|---|
| Never reveal a quiz answer before an attempt | Attempt written to the log before `quiz-answers.md` is opened; answers held in a separate file so they are not in context during the question | **requests**, and leaves evidence |
| Never abandon a concept that has not landed | Per-concept `shaky` status and a **Still open** list | **requests** |

Neither is a wall, and the template says so in its own words rather than implying
otherwise. The attempt log makes a leak *visible afterwards* — but only becomes a
**measure** once something actually reads it looking for one. Nothing does yet.
