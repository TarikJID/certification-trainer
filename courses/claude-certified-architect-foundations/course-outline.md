# Course Outline — Claude Certified Architect – Foundations (CCAR-F)

Source of truth for content and sequencing:
- Domain map: `runs/claude-certified-architect-foundations/domain-map.md`
- Per-domain research: `runs/claude-certified-architect-foundations/research/*.md`

This course has **10 modules and 30 lessons** — one lesson per exam task
statement (1.1 through 5.6), grouped into modules of 2–5 lessons each. Modules
are sequenced so that every concept's prerequisites are taught at an earlier
module/lesson than the concept itself (see "Prerequisite ordering" below).

---

## Module → Domain → Lesson map

| Module | Domain (weight) | Task statements covered | Lessons |
|---|---|---|---|
| Module 1 — The Agentic Loop and Multi-Agent Orchestration | Domain 1: Agentic Architecture & Orchestration (27%) | 1.1, 1.2, 1.3 | Lesson 1.1, Lesson 1.2, Lesson 1.3 |
| Module 2 — Workflow Enforcement, Decomposition, and Sessions | Domain 1 (27%) | 1.4, 1.5, 1.6, 1.7 | Lesson 1.4, Lesson 1.5, Lesson 1.6, Lesson 1.7 |
| Module 3 — Tool Design & MCP Integration | Domain 2: Tool Design & MCP Integration (18%) | 2.1, 2.2, 2.3, 2.4, 2.5 | Lesson 2.1 – Lesson 2.5 |
| Module 4 — Claude Code Memory and Configuration | Domain 3: Claude Code Configuration & Workflows (20%) | 3.1, 3.2, 3.3 | Lesson 3.1, Lesson 3.2, Lesson 3.3 |
| Module 5 — Claude Code Execution Modes and CI/CD Workflows | Domain 3 (20%) | 3.4, 3.5, 3.6 | Lesson 3.4, Lesson 3.5, Lesson 3.6 |
| Module 6 — Precision Prompting and Structured Output Enforcement | Domain 4: Prompt Engineering & Structured Output (20%) | 4.1, 4.2, 4.3 | Lesson 4.1, Lesson 4.2, Lesson 4.3 |
| Module 7 — Extraction Quality, Batching, and Review Architectures | Domain 4 (20%) | 4.4, 4.5, 4.6 | Lesson 4.4, Lesson 4.5, Lesson 4.6 |
| Module 8 — Context Management and Escalation Design | Domain 5: Context Management & Reliability (15%) | 5.1, 5.2 | Lesson 5.1, Lesson 5.2 |
| Module 9 — Error Propagation and Codebase Context | Domain 5 (15%) | 5.3, 5.4 | Lesson 5.3, Lesson 5.4 |
| Module 10 — Human Review, Confidence Calibration, and Provenance | Domain 5 (15%) | 5.5, 5.6 | Lesson 5.5, Lesson 5.6 |

Each module folder contains `lesson.md` (concept explanations + examples, in
teaching order, one quiz reference per lesson), `quiz.md` (all of that module's
lesson quizzes, answers marked), and `exercises.md` (one hands-on exercise for
the whole module).

---

## Prerequisite ordering rationale

Domain 1 is taught first because the agentic loop, the Messages API
request/response cycle, and the coordinator-subagent architecture it introduces
are load-bearing prerequisites for every later domain:

- **Module 1** (Task 1.1–1.3) establishes: the Messages API cycle and `stop_reason`
  loop; hub-and-spoke coordinator/subagent architecture and context isolation;
  subagent invocation mechanics. These recur as prerequisites in Module 3 (tool
  use round trip, coordinator-subagent architecture), Module 6/7 (tool_use blocks
  for structured output, extended thinking contrast), Module 9 (orchestrator-
  worker architecture, subagent isolation), and Module 10 (structured data
  formats).
- **Module 2** (Task 1.4–1.7) establishes permission evaluation order and hooks
  (used again for MCP error handling in Module 3) and session management.
- **Module 3** (Domain 2) builds directly on Module 1's tool_use mechanics and
  coordinator-subagent architecture to teach tool interface design and MCP
  integration; its MCP `isError` and error-taxonomy material is a direct
  prerequisite for Module 9's multi-agent error propagation.
- **Module 4–5** (Domain 3) is self-contained Claude Code product configuration,
  drawing on subagent isolation (Module 1) for `context: fork` and the Explore
  subagent, and introducing JSON Schema basics (Module 5, Lesson 3.6) ahead of
  Module 6's deeper JSON Schema application.
- **Module 6–7** (Domain 4) builds on tool_use/JSON Schema (Modules 1, 3, 5) to
  teach structured output enforcement, and on error taxonomies (Module 3) for
  retry/validation design.
- **Module 8–10** (Domain 5) is taught last because it draws together
  prerequisites from every earlier module: the Messages API statelessness
  (Module 1) for context management, few-shot prompting (Module 6) for
  escalation criteria, error taxonomies and orchestrator-worker architecture
  (Modules 1, 3) for error propagation, subagent isolation (Modules 1, 5) for
  codebase exploration, and structured claim-source formats (Module 1) for
  provenance preservation.

No concept in this course is taught before a module/lesson in which its
prerequisite (as identified in the per-domain research) has already been
covered. Where a concept recurs across domains (e.g., MCP `isError`,
coordinator-subagent architecture, few-shot prompting, structured error
taxonomies), later lessons explicitly recap it with a pointer back to where it
was first taught, rather than re-deriving it or silently assuming it.

---

## Coverage check 1 — Task-statement bullets (domain map → lesson)

**Bullets in the domain map: 240. Bullets covered in the course: 240 (240/240).**

IDs are exactly as assigned in `domain-map.md` (`<statement>-K<n>` /
`<statement>-S<n>`).

| Task Statement | Bullet IDs taught | Module | Lesson |
|---|---|---|---|
| 1.1 | 1.1-K1, 1.1-K2, 1.1-K3, 1.1-S1, 1.1-S2, 1.1-S3 | Module 1 | Lesson 1.1 |
| 1.2 | 1.2-K1, 1.2-K2, 1.2-K3, 1.2-K4, 1.2-S1, 1.2-S2, 1.2-S3, 1.2-S4 | Module 1 | Lesson 1.2 |
| 1.3 | 1.3-K1, 1.3-K2, 1.3-K3, 1.3-K4, 1.3-S1, 1.3-S2, 1.3-S3, 1.3-S4 | Module 1 | Lesson 1.3 |
| 1.4 | 1.4-K1, 1.4-K2, 1.4-K3, 1.4-S1, 1.4-S2, 1.4-S3 | Module 2 | Lesson 1.4 |
| 1.5 | 1.5-K1, 1.5-K2, 1.5-K3, 1.5-S1, 1.5-S2, 1.5-S3 | Module 2 | Lesson 1.5 |
| 1.6 | 1.6-K1, 1.6-K2, 1.6-K3, 1.6-S1, 1.6-S2, 1.6-S3 | Module 2 | Lesson 1.6 |
| 1.7 | 1.7-K1, 1.7-K2, 1.7-K3, 1.7-K4, 1.7-S1, 1.7-S2, 1.7-S3, 1.7-S4 | Module 2 | Lesson 1.7 |
| 2.1 | 2.1-K1, 2.1-K2, 2.1-K3, 2.1-K4, 2.1-S1, 2.1-S2, 2.1-S3, 2.1-S4 | Module 3 | Lesson 2.1 |
| 2.2 | 2.2-K1, 2.2-K2, 2.2-K3, 2.2-K4, 2.2-S1, 2.2-S2, 2.2-S3, 2.2-S4 | Module 3 | Lesson 2.2 |
| 2.3 | 2.3-K1, 2.3-K2, 2.3-K3, 2.3-K4, 2.3-S1, 2.3-S2, 2.3-S3, 2.3-S4, 2.3-S5 | Module 3 | Lesson 2.3 |
| 2.4 | 2.4-K1, 2.4-K2, 2.4-K3, 2.4-K4, 2.4-S1, 2.4-S2, 2.4-S3, 2.4-S4, 2.4-S5 | Module 3 | Lesson 2.4 |
| 2.5 | 2.5-K1, 2.5-K2, 2.5-K3, 2.5-K4, 2.5-S1, 2.5-S2, 2.5-S3, 2.5-S4, 2.5-S5 | Module 3 | Lesson 2.5 |
| 3.1 | 3.1-K1, 3.1-K2, 3.1-K3, 3.1-K4, 3.1-S1, 3.1-S2, 3.1-S3, 3.1-S4 | Module 4 | Lesson 3.1 |
| 3.2 | 3.2-K1, 3.2-K2, 3.2-K3, 3.2-K4, 3.2-S1, 3.2-S2, 3.2-S3, 3.2-S4, 3.2-S5 | Module 4 | Lesson 3.2 |
| 3.3 | 3.3-K1, 3.3-K2, 3.3-K3, 3.3-S1, 3.3-S2, 3.3-S3 | Module 4 | Lesson 3.3 |
| 3.4 | 3.4-K1, 3.4-K2, 3.4-K3, 3.4-K4, 3.4-S1, 3.4-S2, 3.4-S3, 3.4-S4 | Module 5 | Lesson 3.4 |
| 3.5 | 3.5-K1, 3.5-K2, 3.5-K3, 3.5-K4, 3.5-S1, 3.5-S2, 3.5-S3, 3.5-S4, 3.5-S5 | Module 5 | Lesson 3.5 |
| 3.6 | 3.6-K1, 3.6-K2, 3.6-K3, 3.6-K4, 3.6-S1, 3.6-S2, 3.6-S3, 3.6-S4, 3.6-S5 | Module 5 | Lesson 3.6 |
| 4.1 | 4.1-K1, 4.1-K2, 4.1-K3, 4.1-S1, 4.1-S2, 4.1-S3 | Module 6 | Lesson 4.1 |
| 4.2 | 4.2-K1, 4.2-K2, 4.2-K3, 4.2-K4, 4.2-S1, 4.2-S2, 4.2-S3, 4.2-S4, 4.2-S5 | Module 6 | Lesson 4.2 |
| 4.3 | 4.3-K1, 4.3-K2, 4.3-K3, 4.3-K4, 4.3-S1, 4.3-S2, 4.3-S3, 4.3-S4, 4.3-S5, 4.3-S6 | Module 6 | Lesson 4.3 |
| 4.4 | 4.4-K1, 4.4-K2, 4.4-K3, 4.4-K4, 4.4-S1, 4.4-S2, 4.4-S3, 4.4-S4 | Module 7 | Lesson 4.4 |
| 4.5 | 4.5-K1, 4.5-K2, 4.5-K3, 4.5-K4, 4.5-S1, 4.5-S2, 4.5-S3, 4.5-S4 | Module 7 | Lesson 4.5 |
| 4.6 | 4.6-K1, 4.6-K2, 4.6-K3, 4.6-S1, 4.6-S2, 4.6-S3 | Module 7 | Lesson 4.6 |
| 5.1 | 5.1-K1, 5.1-K2, 5.1-K3, 5.1-K4, 5.1-S1, 5.1-S2, 5.1-S3, 5.1-S4, 5.1-S5, 5.1-S6 | Module 8 | Lesson 5.1 |
| 5.2 | 5.2-K1, 5.2-K2, 5.2-K3, 5.2-K4, 5.2-S1, 5.2-S2, 5.2-S3, 5.2-S4, 5.2-S5 | Module 8 | Lesson 5.2 |
| 5.3 | 5.3-K1, 5.3-K2, 5.3-K3, 5.3-K4, 5.3-S1, 5.3-S2, 5.3-S3, 5.3-S4 | Module 9 | Lesson 5.3 |
| 5.4 | 5.4-K1, 5.4-K2, 5.4-K3, 5.4-K4, 5.4-S1, 5.4-S2, 5.4-S3, 5.4-S4, 5.4-S5 | Module 9 | Lesson 5.4 |
| 5.5 | 5.5-K1, 5.5-K2, 5.5-K3, 5.5-K4, 5.5-S1, 5.5-S2, 5.5-S3, 5.5-S4 | Module 10 | Lesson 5.5 |
| 5.6 | 5.6-K1, 5.6-K2, 5.6-K3, 5.6-K4, 5.6-S1, 5.6-S2, 5.6-S3, 5.6-S4, 5.6-S5 | Module 10 | Lesson 5.6 |

**Per-domain bullet count check** (against `domain-map.md`'s summary table):
Domain 1: 6+8+8+6+6+6+8 = 48/48. Domain 2: 8+8+9+9+9 = 43/43. Domain 3:
8+9+6+8+9+9 = 49/49. Domain 4: 6+9+10+8+8+6 = 47/47. Domain 5:
10+9+8+9+8+9 = 53/53. **Total: 48+43+49+47+53 = 240/240.**

---

## Coverage check 2 — Research concepts (research → lesson)

**Concepts across the five research files: 186 (per-file self-reported counts:
Domain 1 = 34, Domain 2 = 29, Domain 3 = 37, Domain 4 = 40, Domain 5 = 46; sum =
186). Concepts appearing in the course: 186/186. None marked `Status: UNSOURCED`
in any research file, so no concept required an "unverified" flag in the
lessons — every concept below is presented as sourced material, each carrying
its research file's citation forward into the lesson text.**

Every "key" concept from each research file has its own named subsection in the
corresponding lesson (`### Concept: ...`), with the definition, example, and
source carried over from the research, and a `(Teaches: ...)` line naming the
exact bullet IDs it covers.

Every "prerequisite" concept from each research file also has its own named
subsection (`### Prerequisite concept: ...`), placed in the lesson of the first
task statement that needs it, so a prerequisite is always taught before (never
after) the concept that depends on it. Where two research files independently
name essentially the same prerequisite (e.g., Domain 1's "Messages API
request/response cycle" and Domain 2's "Agentic tool-use loop", or Domain 1's
"Hub-and-spoke architecture" and Domain 5's "Orchestrator-worker architecture"),
the course teaches it once, at its earliest point of need, and every later
lesson that would otherwise re-teach it instead carries an explicit "— recap"
subsection that cross-references back to where it was first taught (naming the
module/lesson) and restates its citation, rather than repeating the full
explanation or silently assuming the reader remembers it. This applies to:

- Messages API request/response cycle & `tool_use` blocks: taught in Module 1,
  Lesson 1.1; recapped in Module 3 Lesson 2.2 (as "Agentic tool-use loop" /
  MCP client-server), Module 6 Lesson 4.3, and Module 8 Lesson 5.1 (as "Full
  conversation history and the stateless Messages API" / "Messages API
  request/response structure").
- Hub-and-spoke / orchestrator-worker coordinator-subagent architecture: taught
  in Module 1, Lesson 1.2; recapped in Module 3 Lesson 2.3 and Module 9 Lesson
  5.3.
- Subagent context isolation: taught in Module 1, Lesson 1.2; recapped in
  Module 4 Lesson 3.2 (`context: fork`), Module 5 Lesson 3.4 (Explore subagent),
  and Module 9 Lesson 5.4.
- MCP `isError` / structured tool error reporting: taught in Module 2, Lesson
  1.5; extended into the full error-category taxonomy in Module 3, Lesson 2.2;
  recapped in Module 7 Lesson 4.4 ("Structured/typed error results") and Module
  9 Lesson 5.3 ("tool_result / is_error mechanics").
- Few-shot / multishot prompting: taught in Module 6, Lesson 4.2; recapped in
  Module 8 Lesson 5.2 (few-shot escalation criteria).
- Permission evaluation order and hooks: taught in Module 2, Lesson 1.4;
  extended in Lesson 1.5.
- JSON Schema fundamentals: introduced in Module 5, Lesson 3.6 (CI/CD
  `--json-schema`); extended in Module 6, Lesson 4.3 (tool-use schema design).
- Session forking (`fork_session`): introduced in Module 1, Lesson 1.3 (as
  prerequisite for `AgentDefinition`/context-passing, bullet 1.3-K4); given full
  skill coverage in Module 2, Lesson 1.7 (bullets 1.7-K2, 1.7-S2).

No concept was omitted and none required exclusion — every concept in every
research file maps to at least one lesson subsection above. There is therefore
no "deliberately excluded" list for this course.

---

## Quizzes and exercises

Every lesson ends with a quiz in that module's `quiz.md` (answers included,
clearly marked for the tutor to withhold until the learner has attempted the
question). Every module ends with one hands-on exercise in `exercises.md` that
draws on all of that module's lessons together, structured as a realistic
design scenario (e.g., hardening a customer-support agent, redesigning a
toolset, scaling a batch pipeline) rather than isolated recall questions.

## Sourcing note

Every concept explanation in this course carries forward the source citation(s)
given in its originating research file — either an official Anthropic/Claude
documentation URL, or (where the research file notes no matching product-docs
page exists) the exam guide's own verbatim task-statement wording at
`runs/claude-certified-architect-foundations/sources/exam-guide.txt`, which the
research files treat as an acceptable primary source in its own right. No fact
in this course was introduced by the course-builder beyond what appears in the
domain map or the per-domain research; where the research cross-references a
worked example number (e.g., the 4.5-S2 batch-submission-frequency
calculation), the course reproduces that same worked example rather than
inventing a new one.
