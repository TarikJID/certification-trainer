# GUARDRAILS

What Certification Trainer must never do, and where each rule is actually enforced.

Scope is the **whole product** — the build pipeline (`domain-mapper`, `domain-researcher`,
`course-builder`, `evaluator`) and the **tutor** that teaches the finished course. The tutor is a
separate entry point, not a pipeline stage, so some rules below have no implementation yet. They
are listed anyway, because a policy with no home is the thing worth knowing about.

## Why this file exists

A rule written only in an agent's prose is a **request**. The agent may not follow it, may be
argued out of it, and nothing detects the failure. A rule with a structural home is enforced
whether or not the agent cooperates.

So every policy here names three things:

- **Side** — does the risk arrive on the way in, on the way out, or both?
- **Mechanism** — tool scope, prompt, evaluator checklist, or runtime guardrail.
- **Verb** — does that mechanism **block**, merely **request**, or only **measure**?

Anything whose only verb is *request* is a known weak point, not a control.

## The four mechanisms

| Mechanism | What it is | Verb |
|---|---|---|
| **Tool scope** | The harness never grants the capability. The agent structurally cannot. | blocks |
| **Runtime guardrail** | A separate classifier in the request path, judging messages in and out. Not the model being asked nicely — a second model that isn't having the conversation and cannot be talked around. | blocks |
| **Evaluator checklist** | A `Done when` item checked after the stage runs. Failure routes to rework. | measures |
| **Prompt** | The agent is told not to. | requests |

## Policies

| # | Never | Side | Mechanism | Verb | Built? |
|---|---|---|---|---|---|
| 1 | Reveal a quiz answer before the learner has attempted it | both | runtime guardrail | blocks | no |
| 2 | Present fabricated or unsourced content as official | output | evaluator checklist + tool scope | measures + blocks | **yes** |
| 3 | Abandon a concept the learner has not understood | output | prompt (+ optional after-the-fact measure) | requests | no |
| 4 | Quiz or exercise a learner on material the course never taught | both | evaluator checklist, at build time | measures | **yes** |

---

### 1. Never reveal a quiz answer before the learner has attempted it

**Side: both.** The learner can ask directly — *"just tell me the answer to Q3"* — which arrives on
the way in. The leak itself happens on the way out.

**Mechanism: runtime guardrail. Blocks.**

This is the one policy where both directions need checking, and the only one that warrants a
classifier in the path. A prompt instruction is the wrong tool: the request will be persuasive
(*"I already tried, just confirm it"*), the rule will sit thousands of tokens back in the context,
and the model doing the teaching is the same model being asked to refuse.

Note that **exclusion from the index does not work here**, unlike the general case. The tutor needs
the answer key in order to grade an attempt. It cannot be withheld — only guarded.

**Status: not built.** No tutor exists yet.

### 2. Never present fabricated or unsourced content as official

**Side: output.** Two failures in one: producing content that isn't in any source, and presenting
it as though it were sourced rather than flagging it.

**Mechanism: evaluator checklist + tool scope. Measures and blocks.**

Two controls doing different jobs:

- **Checklist (measures).** `domain-researcher`'s "every key concept is defined, illustrated and
  attributed to a source" and `course-builder`'s item 6 — any concept marked `UNSOURCED` is carried
  into the course and marked unverified, never dropped, never presented as sourced.
- **Tool scope (blocks).** `course-builder` has no web tools. It cannot fetch anything to
  misattribute; everything it writes must come from the material it was handed.

**Status: built, and it fires.** All three researcher reworks in the 18 Sept run were citation-
faithfulness failures — claims wearing citations the cited page did not support.

**The `WebSearch` hole, and how it is closed.** `WebSearch` returns a synthesised summary alongside
the result list. A researcher could write a concept from that summary and cite a URL it never
opened — output indistinguishable from properly sourced work. So
[`.claude/agents/domain-researcher.md`](.claude/agents/domain-researcher.md) now requires a
`Fetched:` list naming every URL opened with `WebFetch`, and a `Done when` item requiring every
cited URL to appear in it. A citation with no matching fetch is a claim about a page nobody read.

### 3. Never abandon a concept the learner has not understood

**Side: output.** The failure is the decision to move on.

**Mechanism: prompt. Requests only.**

**This policy has no structural backstop, and that is worth stating plainly.**

- **Tool scope can't help.** There is no `GiveUp` capability to withhold. Moving to the next topic
  is not a tool call, it is ordinary behaviour.
- **A runtime guardrail can't help.** A message-level classifier sees one message. Telling
  "moved on too early" from "moved on correctly" needs the learner's comprehension history across
  the whole session, which the classifier does not have.
- **A checklist can't help at build time.** Nothing about this is decidable from the course files.
  It happens at teach time or not at all.

**What to monitor instead.** The prompt requests the behaviour; detection has to be after the fact,
and only works if the tutor writes down per-concept comprehension state. Given that log, the check
is a set comparison:

> Any concept marked *not understood* that was never revisited in a later session?

Without that state, the failure is invisible — not rare, invisible. The monitoring is therefore a
prerequisite for the policy meaning anything, not an optional extra.

**Status: not built**, and cannot be built until the tutor persists comprehension state.

### 4. Never quiz or exercise a learner on material the course never taught

**Side: both.** The gap arrives as input (a concept missing from the lessons) and the harm is the
output (a quiz question about it).

**Mechanism: evaluator checklist, at build time. Measures.**

The defect is created by `course-builder`, not by the tutor. Once a quiz question about an untaught
concept is written to disk, the tutor has no way to avoid asking it — it is reading a finished file.
Checking at teach time means re-checking static content on every session, after it shipped.

So the check belongs where the file is produced, and it is a set comparison — the same shape as the
240-bullet coverage check:

> Every concept tested in a quiz or exercise appears in a lesson at or before that module.

Before this was installed, `course-builder`'s checklist did not cover it. Item 3 requires each
lesson to *have* a quiz; item 5 enforces prerequisite ordering for concepts that are taught. Neither
kept quiz questions inside taught material.

**Status: built.** Enforced by the `Done when` item in
[`.claude/agents/course-builder.md`](.claude/agents/course-builder.md) beginning *"Every concept
tested in a quiz or exercise is taught in a lesson at or before that module."* It also requires the
totals be stated — concepts tested, and concepts tested that are not taught, which must be zero.

---

## Where each policy actually lives

A policy document nobody reads enforces nothing. Agents obey their `Done when` checklists, because
that is the one thing the evaluator reads. So this table is the link between a policy here and the
line that enforces it — **keep it current when either side changes.**

| # | Enforced by | Where |
|---|---|---|
| 1 | nothing yet | needs the tutor to exist |
| 2 | checklist items + tool scope | `domain-researcher.md` (attribution, `Fetched:`), `course-builder.md` (item 6, `UNSOURCED` carried through), and `course-builder`'s tool list — no web tools |
| 3 | prompt only, by necessity | no structural backstop exists; see the section above |
| 4 | checklist item | `course-builder.md` — *"Every concept tested in a quiz or exercise is taught in a lesson at or before that module"* |

## Still missing

| Gap | Cost to close |
|---|---|
| Runtime guardrail for answer leaks (policy 1) | needs the tutor to exist |
| Per-concept comprehension state in the tutor (policy 3) | needs the tutor to exist |

Both are blocked on the same thing. Note that the second is not a gap in a control — it is a control
that cannot exist without a log that does not exist yet, which makes building that log a
prerequisite for the policy meaning anything.
