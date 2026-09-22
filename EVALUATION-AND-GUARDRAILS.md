# Evaluation and guardrails

**Who this is for:** anyone deciding whether to trust this pipeline, or borrowing ideas
from it. It assumes you have read [`README.md`](README.md) and nothing else.

A multi-agent system that researches and writes a course can produce something fluent,
complete-looking and wrong. This file is about the two things standing between that and
what actually ships:

- **Evaluation** — how the pipeline measures its own output, and which of those
  measurements are worth anything.
- **Guardrails** — what it must never do, and whether each rule can actually stop it.

[`GUARDRAILS.md`](GUARDRAILS.md) is the register: the policies, where each is enforced,
and its status. **This file is the reasoning behind it** — including the parts that do
not work, which are more informative than the parts that do.

**If you read one thing here**, make it the distinction running through all of it:

> A rule is only as strong as the place it lives. Tool scope and a runtime classifier
> **block**. A checklist item **measures**. A line in a prompt **requests** — and
> anything whose only verb is *request* is a known weak point, not a control.

What follows is mostly that test, applied honestly, including where it comes out
badly.

---

## Part 1 — Evaluation

### Where the pipeline sits

Evaluation metrics stack in five layers, and a weakness low down invalidates every
measurement above it:

| Layer | Question | In this pipeline |
|---|---|---|
| 5 · Agent | Can we trust the system end to end? | **Yes, and it is ours** |
| 4 · Generation | Can we trust the generated output? | **Yes, and it is ours** |
| 3 · Retrieval | Can we trust the material it found? | **Yes, and it is ours** |
| 2 · Reasoning | Can we trust the reasoning? | Present, but bought |
| 1 · Quality | Can we trust the answer at all? | Present, but bought |

All five exist. Only three are **actionable**, and the test that separates them is:
*if this layer scored badly, what would I change?*

Layer 1 scoring badly cannot be fixed from inside this repo. No prompt, checklist or
tool scope moves that number — the only lever is using a different model. Layers 1 and
2 are bought from the model provider, not built here. Measuring them would be measuring
somebody else's work.

### Layer 3 is a selection step, not a tool

Worth stating precisely, because the obvious reading is wrong.

Retrieval is not about having web access. **It is about there being a selection step
that can select wrongly.**

- `domain-researcher` searches, gets many candidates, picks some. It can pick badly.
  **It has a layer 3.**
- `course-builder` is handed N files by the dispatch and uses all N. No query, no
  candidate pool, no ranking. **It has no layer 3** — there is nothing for Precision@K
  to range over, because there is no K.

That distinction is structural, not about tooling. Give `course-builder` the job of
searching across fifty research files for relevant passages and it gains a layer 3
without ever touching the internet.

### The retrieval metrics, and which of them mean anything here

The researcher does live web search rather than retrieval from a fixed corpus, which
changes what each standard metric is worth.

| Metric | Applies? | Why |
|---|---|---|
| **Precision@K** | Yes, with a caveat | Of the pages it opened, how many were useful. Needs a definition of *relevant* — and if you define it as "got cited", precision and Observation Utilization collapse into the same number |
| **Recall@K** | **No** | Of all useful material that exists, how much did it find? The web is not enumerable, so the denominator does not exist |
| **MRR** | Computable, meaningless | See below |

**Recall is measurable exactly when the universe of relevant items is enumerable.** The
web is not. The exam guide is — 240 bullets, bounded, countable. Which is why the
240/240 coverage check works and is genuinely a recall measurement, while recall over
retrieved web pages is not available at any price.

**MRR measures the cost of a cutoff.** It matters when something mechanically truncates
the result list — a RAG pipeline taking the top 5 chunks, or a human who scans three
results and stops. The good page at rank 7 never reaches the model, and `1/7` predicts
exactly that damage.

The researcher has no cutoff. It receives nine results — titles and URLs only — reads
all of them, and picks by recognising an authoritative domain in the URL. That signal
is entirely independent of position. **No cutoff, nothing to measure.**

It would bite again if the researcher only ever looked at the first few results. That
is an empirical question about the agent's behaviour, and today nothing logs enough to
answer it.

### Outcome and trajectory, for the researcher stage

Two metrics, chosen because they are the cheapest useful pair:

**Outcome: hallucination rate.** Is each claim supported by the source it cites?

Not the same as correctness, and the difference is what evidence each one needs:

| | Asks | Needs |
|---|---|---|
| Correctness | Is the claim true about the world? | An authority you trust more than the researcher. Usually unavailable |
| Hallucination rate | Is the claim supported by the source it cites? | Only the cited source, which is already archived |

A claim can be perfectly true and still a fabricated attribution. This is not a weaker
proxy for correctness — it is a different, cheaper, and actually measurable thing.

**Trajectory: Observation Utilization.** Of the pages it fetched, how many did it cite?

Chosen over Trajectory Efficiency deliberately. Efficiency is *steps taken versus the
minimum needed*, and nobody can say what the minimum number of web searches
needed to research one domain is. A metric with no threshold to fail against is not yet a judgement.
Observation Utilization needs no baseline — it is a set comparison between what was
fetched and what was used.

### What is measured today

| | Status |
|---|---|
| Coverage (recall over exam bullets) | **Built.** Set comparison, both directions |
| Citation faithfulness | **Built.** Checklist item; the evaluator fetches cited pages and searches them |
| Observation Utilization | **Possible now.** The `Fetched:` list was added; nothing computes the ratio yet |
| Hallucination rate as a *rate* | Not aggregated. Individual failures are caught per stage; nobody counts them across a run |
| Trajectory efficiency | Not measurable — no baseline exists |
| Teaching quality | **Not measured, and not measurable here.** Needs a human reading a lesson |

### Trajectory logging

Every evaluation writes a verdict file: one per stage, per round, listing every
checklist item with an evidence column — **including the ones that passed** — plus a
separate `Not checked` section.

Recording passes is not padding. A file listing only failures is no evidence the rest
was ever examined. And a run whose reasoning exists only in a session's context window
produces a course nobody can audit afterwards: outcomes with no trajectory.

---

## Part 2 — Guardrails

### The four mechanisms

Writing a policy down is not enforcing it. Every rule lands in one of four places, and
the place determines the verb:

| Mechanism | What it is | Verb |
|---|---|---|
| **Tool scope** | The harness never grants the capability | **blocks** |
| **Runtime guardrail** | A separate classifier in the request path | **blocks** |
| **Evaluator checklist** | A `Done when` item; failure routes to rework | **measures** |
| **Prompt** | The agent is told not to | **requests** |

**Anything whose only verb is *request* is a known weak point, not a control.** The
register says so explicitly rather than letting the wording imply otherwise.

### What is enforced, and how

**By tool scope — `course-builder` has no web tools.** "Use only the material you were
handed" is structurally true rather than politely requested. It cannot fetch anything to
misattribute. Note the side effect: it is the only stage that can run without network
access, which is a consequence of the design, not a coincidence.

**By checklist — citation faithfulness.** Concepts must be attributed to a source, and
the evaluator verifies by fetching the page and searching it. This is the control with
the most evidence behind it. On the first fully-instrumented run, every researcher
rework was triggered by this check and nothing else.

**By checklist — the `Fetched:` list.** `WebSearch` returns a synthesised summary of
pages alongside the result list. A researcher could write a concept from that summary
and cite a URL it never opened, producing output indistinguishable from properly sourced
work. So the research file must name every URL opened with `WebFetch`, and every cited
URL must appear in that list. A citation with no matching fetch is a claim about a page
nobody read.

**By checklist — quiz coverage.** Every concept tested in a quiz or exercise must be
taught in a lesson at or before that module. The defect is created at build time: once a
question about an untaught concept is on disk, whoever teaches from the file cannot
avoid asking it.

**By file layout — quiz answers.** Questions in `quiz.md`, answers in
`quiz-answers.md`. A tutor opening the questions does not thereby hold every answer.
This reduces exposure; it does not block anything, and the register says so.

**By prompt only — never abandon a concept that has not landed.** There is no
capability to withhold, no classifier with enough history to judge it, and nothing
decidable from the files. This policy has **no structural backstop**, which is stated
plainly rather than dressed up.

---

## Part 3 — What building this turned up

Three findings, each of which changed the design. They are here rather than in a private
retrospective because each is a way this kind of system fails *quietly*, and none was
caught by re-reading the specs.

### A rule in prose is a rule the evaluator never sees

The researcher's source-quality rule originally lived only in its "You don't" section.
An agent ignoring it would have passed evaluation cleanly, because the evaluator reads
the `Done when` checklist and nothing else.

**Every rule you actually want enforced has to be promoted into the checklist.** Both
guardrails added since were added in two places for this reason: the spec, so the agent
knows, and the checklist, so the evaluator can catch it.

### A policy file can misreport its own status

`GUARDRAILS.md` was written before the agent definitions were edited. It went on saying
a policy was `not built` after it had shipped, and listing a closed gap as open.

A register that is wrong about itself is worse than none — it is a control that reports
success while doing nothing. It now carries a **"Where each policy actually lives"**
table mapping every policy to the file enforcing it, precisely so the two can be checked
against each other.

### Testing found what review did not

The tutor's answer-gating design assumed `quiz.md` held questions and `quiz-answers.md`
held answers. `course-builder` emitted neither — answers sat inline in `quiz.md` inside
`<details>` blocks, under a note reading *"the tutor must never reveal an answer before
the learner attempts."*

Two failures in one. That note is a **prompt instruction sitting in a data file**,
already failed by the time anything reads it. And `<details>` collapses in a browser,
not in a model's context — a tutor opening `quiz.md` to ask the first question holds
every answer for that module.

The guardrail had been resting on a file split that never existed. A structural check
found it in seconds. No amount of re-reading the specs would have.

---

## Part 4 — Still open

| Gap | What it would take |
|---|---|
| **Nothing reads the attempt log.** The tutor records quiz attempts before opening answers, which makes a leak *detectable* — but only becomes a **measure** once something looks. Today the honest verb is still *requests* | Something that reads the log looking for answers revealed without a recorded attempt |
| **Hallucination rate is not aggregated.** Individual failures are caught; nobody counts them across a run, so there is no trend | Tally faithfulness findings per run from the verdict files |
| **Observation Utilization is possible but not computed.** The `Fetched:` list exists now | Compare fetched URLs against cited URLs |
| **Teaching quality is unverified** — and no automated check here will settle it | A human reading a lesson |
| **The second-certification test.** Everything above was learned on one exam. Whether the pipeline is genuinely generic is unproven | A run against a different certification with no answer key in `reference/` |

The last one is the real test. Every control described here was designed while watching
a single certification go through the pipeline, which is exactly the condition under
which you build something that works once.
