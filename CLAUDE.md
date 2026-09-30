# CLAUDE.md — Certification Trainer, development

This repo is the definition of a multi-agent pipeline that turns a certification exam
into a course. In a plain session here you are a **developer of that pipeline**: you
change how it behaves, you do not run it.

The orchestrator is the skill at `.claude/skills/build-certification/SKILL.md`. It loads only when
the user types `/build-certification`. Once it has been invoked, that skill governs the rest of the
session and this file's development role no longer applies: the pipeline definition is
frozen until the session ends.

## What the pipeline is made of

| File | What it defines |
|---|---|
| `.claude/skills/build-certification/SKILL.md` | The orchestrator: phases, dispatch, retry caps, commit points, path conventions |
| `.claude/agents/*.md` | The four agents: `domain-mapper`, `domain-researcher`, `evaluator`, `course-builder` |
| `tutor-template/` | The tutor copied into every finished course |
| `EVALUATION-AND-GUARDRAILS.md` | Every control in the pipeline, where it lives, and whether it is enforced |
| `runbooks/` | Prompts a human pastes to launch a staged run |
| `reference/` | Answer keys for grading a run afterwards |

`README.md` explains the design to someone who has just arrived. Read it before a
change that touches more than one file.

## Rules for changing it

- **Nothing certification-specific in the pipeline.** The exam is an input. Agent
  definitions, the skill and `tutor-template/` must work for any certification.
- **A rule the evaluator must check goes in the agent's `Done when` checklist.** The
  evaluator reads the checklist, not the prose. A rule that lives only in a "You don't"
  section is requested, not checked.
- **Prefer tool scope to prompt text.** If an agent must not do something, removing the
  tool that does it beats telling it not to. `course-builder` has no web tools for this
  reason.
- **Keep `EVALUATION-AND-GUARDRAILS.md` true.** When you add, change or remove a
  control, update its entry in the same change, including whether it is enforced or
  prompt only. A removed control moves to **Removed** with the reason.
- **Cross-references cite files and step numbers.** The guardrails doc and the runbooks
  point at things like "phase 2 step 8". After renaming a file or renumbering steps,
  grep for the old reference and fix every mention.
- **`main` holds the pipeline, not its output.** `runs/` and `courses/` from a test run
  stay on that run's branch. They are evidence, not product.

## How to work

- One improvement per branch, merged into `main` by PR. Say in the PR what problem the
  change fixes, with the verdict file or run output that showed it where there is one.
- A prompt change is only shown to work by running the pipeline. For a change to one
  stage, the smallest useful test is a staged run up to that stage (see `runbooks/`).
  A full run is expensive: agree scope and budget with the user before starting one.
- Start a run in a **fresh session** with `/build-certification`, not in the session where you made
  the change. A run session should hold the orchestrator's rules only, not your
  reasoning about why they changed.
- You may read `reference/` to grade a run. Never start a run in a session that has
  read it: the orchestrator and its agents must not have the answer key in context.
