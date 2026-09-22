<div align="center">

# 🎓 Certification Trainer

**Point it at a certification. Get back a course you can actually check.**

![Agents](https://img.shields.io/badge/4%20Agents-%2B%20orchestrator-2EA043?style=for-the-badge)
&nbsp;
![Scope](https://img.shields.io/badge/Works%20on-any%20certification-6E40C9?style=for-the-badge)
&nbsp;
![Evaluated](https://img.shields.io/badge/Evaluated-every%20stage-FF6F61?style=for-the-badge)
&nbsp;
[![Built with Claude](https://img.shields.io/badge/Built%20with-Claude%20Code-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://claude.com/claude-code)

[**How it works**](#how-it-works) · [**The agents**](#the-agents) · [**Running it**](#running-it) · [**Repo layout**](#repo-layout) · [**Status**](#status)

</div>

---

A multi-agent pipeline that takes a certification exam, researches every domain it covers, and
assembles a sequenced course — lessons, quizzes and exercises — that a tutor can teach from.

It is built to work on **any certification**, not one. Nothing certification-specific lives in the
agent definitions; the exam is an input, not a configuration. The pipeline is the product, and a
course is what falls out of running it.

The design goal that shapes everything else: **every claim the pipeline makes about its own output
should be checkable.** Not *"the research passed evaluation"* but *here are the exam's bullets, here
is the lesson teaching each one, and here is the set comparison showing none missing and none
invented.*

## How it works

```
   ┌──────────────┐
   │ domain-mapper│  official exam page ──▶ structured domains + task statements + bullet IDs
   └──────┬───────┘
          │  evaluator  ──▶ PASS / REWORK
          ▼
   ┌──────────────────────────────────┐
   │ domain-researcher  ×N (parallel) │  one instance per domain
   └──────┬───────────────────────────┘
          │  evaluator  ──▶ PASS / REWORK   (routed to the failing instance only)
          ▼
   ┌──────────────┐
   │ course-builder│  modules ▸ lessons ▸ quizzes ▸ exercises, written to disk
   └──────┬───────┘
          │  evaluator  ──▶ PASS / REWORK
          ▼
      a course
```

An **evaluator runs between every stage**, not once at the end. In a chained system a bad early
output is consumed by everything downstream, so a single check at the end tells you the course is
wrong without telling you *which stage* made it wrong — and throws away all the work built on the
bad foundation.

Each evaluation writes a **verdict file** recording every checklist item, with an evidence column
and a separate `Not checked` section. Passes are recorded too: a file listing only failures is no
evidence that the rest was examined.

## The agents

| Agent | Job | Tools | Notable |
|-------|-----|-------|---------|
| **`domain-mapper`** | Fetch the official exam page; produce structured domains with task statements verbatim and stable bullet IDs | `Read` `WebSearch` `WebFetch` `Write` | The IDs it assigns are cited by every downstream stage — that is what turns coverage into a set comparison |
| **`domain-researcher`** | Research one domain: concepts, prerequisites, definitions, examples, sources. Runs as N parallel instances | `Read` `WebSearch` `WebFetch` `Write` | Prerequisite tracing capped at one level. Records a `Fetched:` list, and every cited URL must appear in it |
| **`course-builder`** | Assemble the research into a sequenced course on disk | `Read` `Write` | **No web tools, deliberately.** "Use only the material you were handed" is enforced by tool scope, not prompt text |
| **`evaluator`** | Check a stage's output against *that stage's own* `Done when` checklist | `Read` `WebSearch` `WebFetch` `Write` | Never modifies the work it judges. Verifies against whatever source the checklist designates — which is what makes it reusable across all three stages |

The orchestrator lives in [`CLAUDE.md`](CLAUDE.md): the user-agreement phase, dispatch, a per-stage
retry cap with escalation, partial re-run on parallel failure, and file-path passing between stages
so the corpus never flows through the orchestrator's own context.

## Running it

**The pipeline needs its own session.** Open this repo in
[Claude Code](https://claude.com/claude-code) from its own directory, so `CLAUDE.md` loads as the
orchestrator. Running it from inside another project puts two `CLAUDE.md` identities in one context,
and the orchestrator ends up doing the work it is explicitly forbidden to do.

**It needs general web access.** Three of the four agents fetch pages. Only `course-builder` can run
without it — which is a consequence of its tool scope, not a coincidence.

Then name a certification:

> *"Build a course for the AWS Certified Solutions Architect – Associate exam."*

The orchestrator agrees the exact certification with you first — same exam, same level, same
version — before any research starts. A link helps; a name alone will do, and it will ask until
there is no ambiguity left.

### Output layout

```
runs/<certification-name>/
  sources/         archived exam guide, fetched once
  dispatch/        per-domain slices, committed so verdict files cite inputs that survive
  research/        one file per domain
  evaluations/     one verdict file per stage per round
courses/<certification-name>/
  Module_1_<Name>/
    lesson.md  quiz.md  exercises.md
  ...
  course-outline.md    includes the bullet-ID → lesson table
```

Work is committed between stages, so `git log -p runs/` reconstructs the whole run afterwards —
what changed, when, and in what order.

## Repo layout

The repo is the pipeline definition. Everything else is produced by running it.

```
.claude/agents/
  domain-mapper.md          the four agent definitions — one file, one role
  domain-researcher.md
  course-builder.md
  evaluator.md
CLAUDE.md                   the orchestrator: phases, dispatch, retry caps, context rules
GUARDRAILS.md               what the system must never do, and where each rule is enforced
reference/                  fixtures for grading a run — agents never read from here
runbooks/                   prompts for a human to paste when launching a run
```

`runs/` and `courses/` appear only once you run it — see [Output layout](#output-layout)
above — and neither is committed here. A finished course is its own repo, so this one
stays the tool rather than accumulating outputs.

**`reference/`** holds exam material extracted independently of the pipeline, used to
*grade* a run's coverage after the fact. `domain-mapper` is explicitly told never to
read from it, and a fixture carries its own extraction fingerprints so a mapper that
copied it could be caught.

**`runbooks/`** are for a human to paste, not for any agent to load — launch prompts
and the reasoning behind how a long run is split into sessions.

## Status

The pipeline has been run end-to-end against a real certification and produced a complete,
coverage-verified course. What it does **not** yet verify is teaching quality: every bullet having a
lesson, and every citation holding up, is not the same as the course teaching well. That still needs
a human reading a lesson.

---

<div align="center">

Built as the working case study for the
[Agent Engineering Bootcamp](https://github.com/TarikJID/multi-agent-course).

</div>
