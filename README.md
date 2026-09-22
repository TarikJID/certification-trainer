<div align="center">

# 🎓 Certification Trainer

**Point it at a certification. Get back a course you can actually check.**

![Agents](https://img.shields.io/badge/4%20Agents-%2B%20orchestrator-2EA043?style=for-the-badge)
&nbsp;
![Verified](https://img.shields.io/badge/Coverage-240%2F240%20verified-6E40C9?style=for-the-badge)
&nbsp;
![Cost](https://img.shields.io/badge/Full%20run-%2428.09-FF6F61?style=for-the-badge)
&nbsp;
[![Built with Claude](https://img.shields.io/badge/Built%20with-Claude%20Code-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://claude.com/claude-code)

[**How it works**](#how-it-works) · [**The agents**](#the-agents) · [**Running it**](#running-it) · [**What it produced**](#what-it-produced) · [**Guardrails**](#guardrails)

</div>

---

A multi-agent pipeline that takes a certification exam, researches every domain it covers, and
assembles a sequenced course — lessons, quizzes and exercises — that a tutor can teach from.

It is built to work on **any certification**, not one. Nothing certification-specific lives in the
agent definitions. The pipeline is the product; a course is what falls out of running it.

The design goal that shapes everything else: **every claim the pipeline makes about its own output
should be checkable.** Not "the research passed evaluation" but *here are the 240 exam bullets, here
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

An **evaluator runs between every stage**, not once at the end. That is deliberate: in a chained
system a bad early output is consumed by everything downstream, so a single check at the end tells
you the course is wrong without telling you *which stage* made it wrong — and wastes all the work
built on the bad foundation.

Each evaluation writes a **verdict file** recording every checklist item, with an evidence column
and a separate `Not checked` section. Passes are recorded too: a file listing only failures is no
evidence the rest was examined.

## The agents

| Agent | Job | Tools | Notable |
|-------|-----|-------|---------|
| **`domain-mapper`** | Fetch the official exam page, produce structured domains with task statements verbatim and stable bullet IDs | `Read` `WebSearch` `WebFetch` `Write` | The IDs it assigns are cited by every downstream stage — that is what makes coverage a set comparison |
| **`domain-researcher`** | Research one domain: concepts, prerequisites, definitions, examples, sources. Runs as N parallel instances | `Read` `WebSearch` `WebFetch` `Write` | Prerequisite tracing is capped at one level. Must record a `Fetched:` list, and every cited URL must appear in it |
| **`course-builder`** | Assemble the research into a sequenced course on disk | `Read` `Write` | **No web tools, deliberately.** "Use only the material you were handed" is enforced by tool scope, not prompt text |
| **`evaluator`** | Check a stage's output against *that stage's own* `Done when` checklist | `Read` `WebSearch` `WebFetch` `Write` | Never modifies the work it judges. Verifies against whatever source the checklist designates — so it is reusable across all three stages |

The orchestrator lives in [`CLAUDE.md`](CLAUDE.md): user-agreement phase, dispatch, per-stage retry
cap of 2 with escalation, partial re-run on parallel failure, and file-path passing between stages
so the corpus never flows through the orchestrator's own context.

## Running it

**This repo is the pipeline, and it needs its own session.** Open it in
[Claude Code](https://claude.com/claude-code) from its own directory so `CLAUDE.md` loads as the
orchestrator. Do not run it from inside another project — two `CLAUDE.md` identities in one context
means the orchestrator ends up doing the work it is explicitly forbidden to do.

The pipeline needs **general web access**. Three of the four agents fetch pages; only
`course-builder` can run without it.

Then simply name a certification:

> *"Build a course for the AWS Certified Solutions Architect – Associate exam."*

The orchestrator will agree the exact certification with you first — same exam, same level, same
version — before any research starts.

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

Everything is committed between stages, so `git log -p runs/` reconstructs the whole run
afterwards — what changed, when, and in what order.

## What it produced

Two runs against the same certification (Claude Certified Architect — Foundations):

| | Run 1 · 15 Sept | Run 2 · 18 Sept |
|---|---|---|
| Cost | $53.68 | **$28.09** |
| Concepts | 115 | **186** |
| Course | 6 modules, ~3,880 lines | **10 modules, 5,512 lines** |
| Coverage | unverifiable | **240/240 bullet IDs, exact set match** |
| Verdict files | 0 | **11** |
| What reworks were about | source-tier labels | **citation faithfulness** |

The last row is the one worth reading twice. All three researcher reworks in run 2 were citation
faithfulness failures — a prerequisite mis-attributed, a concept whose cited source did not support
it, two attributions "not faithful to the source they cite". Real, official, on-topic pages carrying
claims that were not on them, caught mechanically.

**Coverage is not quality.** Every bullet has a lesson and the citations survived scrutiny. Whether
the course *teaches well* still needs a human reading a lesson.

## Guardrails

[`GUARDRAILS.md`](GUARDRAILS.md) lists what the system must never do, and — more usefully — *where
each rule is actually enforced*. Every policy names a mechanism and an honest verb:

| Mechanism | Verb |
|---|---|
| Tool scope — the harness never grants the capability | **blocks** |
| Runtime guardrail — a classifier in the request path | **blocks** |
| Evaluator checklist — a `Done when` item, failure routes to rework | **measures** |
| Prompt — the agent is told not to | **requests** |

A rule whose only verb is *request* is a known weak point, not a control. One of the four policies
has no structural backstop available at all, and the file says so rather than implying otherwise.

## Repo layout

```
.claude/agents/     the four agent definitions — one file, one role
CLAUDE.md           the orchestrator
GUARDRAILS.md       policies and their enforcement points
reference/          test fixtures for grading runs — agents never read from here
runbooks/           procedures for specific run shapes
runs/               produced by a run (see Output layout above)
courses/            produced by a run — the deliverable
```

`reference/` holds an independently extracted exam guide used to *grade* a run's coverage.
`domain-mapper` is explicitly told never to read from it, and the file carries its own extraction
fingerprints so a mapper that copies it can be caught.

---

<div align="center">

Built as the case study for the **Agent Engineering Bootcamp** —
[TarikJID/multi-agent-course](https://github.com/TarikJID/multi-agent-course)

</div>
