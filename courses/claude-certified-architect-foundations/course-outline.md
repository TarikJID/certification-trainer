# Course Outline — Claude Certified Architect – Foundations (CCAR-F)

Source materials:
- Domain map: `runs/claude-certified-architect-foundations/domain-map.md`
- Per-domain research (all passed evaluation):
  - `research/agentic-architecture-and-orchestration.md` (27% of exam, 14 concepts)
  - `research/claude-code-configuration-and-workflows.md` (20%, 19 concepts)
  - `research/prompt-engineering-and-structured-output.md` (20%, 21 concepts)
  - `research/tool-design-and-mcp-integration.md` (18%, 31 concepts)
  - `research/context-management-and-reliability.md` (15%, 30 concepts)

Total concepts in the research corpus: 115. All 115 are placed in this
course (either taught in full where first needed, or explicitly
cross-referenced to their single teaching point when the same concept
appeared in more than one domain file). None were excluded.

**Sequencing note:** exam weighting (shown per module below) reflects how
much of the exam tests each domain — it does not reflect teaching order.
Module order was chosen to satisfy prerequisite relationships that cross
domain-file boundaries (tool use, the context window, MCP architecture, and
the general subagent notion are each used as prerequisites by more than one
domain's research file). Learners preparing under time pressure should
weight study time toward Module 4 (27%) and Modules 2/5 (20% each), while
still completing Modules 1, 3, and 6 first/in sequence, since later modules
depend on concepts taught there.

---

## Module 1: Foundations — Prompts, Tool Use, and the Context Window
**No dedicated exam weight** — this module consolidates prerequisite
concepts drawn from all five domains (each concept it teaches was
originally filed as a "prerequisite" in one or more domain research files:
Agentic Architecture & Orchestration, Prompt Engineering & Structured
Output, Tool Design & MCP Integration, Context Management & Reliability).
Read first; every later module depends on it.

- Lesson 1.1: The Messages API and Clear Prompting
- Lesson 1.2: Tool Use and the Agentic Loop
- Lesson 1.3: The Context Window, Tokens, and Extended Thinking

Domain traceability: Messages API structure, being clear/direct (Prompt
Engineering research); System Prompt (Agentic Architecture research) merged
with System prompts/role prompting (Prompt Engineering research); Tool Use
(cited independently in all four of Agentic Architecture, Prompt
Engineering, Tool Design, and Context Management research); JSON Schema
fundamentals (Prompt Engineering) merged with JSON Schema for tool inputs
(Tool Design); Tool definition structure (Tool Design); Agentic loop,
Tokens/token counting, Extended thinking, Beta headers, Messages API turn
structure, Needle-in-a-haystack methodology's companions (Context
Management); The Context Window (Agentic Architecture).

---

## Module 2: Claude Code Configuration & Workflows
**Exam weight: 20%.** Maps to the Claude Code Configuration & Workflows
domain, plus one concept ("Claude Code subagents," general notion) imported
early from the Tool Design & MCP domain because it is needed here before
Module 3 formally teaches Tool Design.

- Lesson 2.1: Sessions, Settings, and Memory
- Lesson 2.2: Skills, Commands, and the Agentic Tool Loop
- Lesson 2.3: Permission Modes, Subagents, and Plan Mode
- Lesson 2.4: Iterative Refinement and Effective Workflows
- Lesson 2.5: Headless Mode and CI/CD Integration

Domain traceability: all 19 concepts from the Claude Code Configuration &
Workflows research file, plus "Claude Code subagents" (prerequisite,
originally filed in the Tool Design & MCP Integration research).

**Sourcing flag:** "Single message for interacting issues versus sequential
iteration for independent issues" (Lesson 2.4) is sourced to the official
Exam Guide (Task Statement 3.5) only — no corroborating Claude Code product
documentation was found by the researcher despite a thorough search. This
is properly sourced (the exam guide is official certification material),
not unsourced, but the lesson flags it explicitly so the distinction from
vendor-documented material is visible to the learner.

---

## Module 3: Tool Design & MCP Integration
**Exam weight: 18%.** Maps to the Tool Design & MCP Integration domain.

- Lesson 3.1: Writing Effective Tool Descriptions
- Lesson 3.2: Structuring the Tool Set
- Lesson 3.3: Error Handling for Tools and MCP
- Lesson 3.4: MCP Architecture and Integration
- Lesson 3.5: Distributing Tools and Selecting Built-in Tools

Domain traceability: all 31 concepts from the Tool Design & MCP Integration
research file (3 of them — Tool use, JSON Schema for tool inputs, Tool
definition structure — were taught earlier in Module 1 at their true
earliest point of need and are referenced here rather than re-taught; 1 —
Claude Code subagents — was taught in Module 2 for the same reason).

**Sourcing flags (exam-guide-only, no vendor corroboration found):**
- "Keyword-sensitive system prompt wording can create unintended tool
  associations" (Lesson 3.1, Task 2.1) — distinct from, and stronger than,
  the vendor-documented claim about system prompt wording steering whether
  a tool is called at all, which is taught alongside it for contrast.
- "Read + Write as a fallback when Edit fails due to non-unique text
  matches" (Lesson 3.5, Task 2.5) — distinct from, and not the same
  mechanism as, Claude Code's own vendor-documented Edit recovery path
  (more context, or `replace_all: true`), which is taught alongside it for
  contrast.

Both are marked in the lesson text with explicit sourcing notes; neither is
presented as vendor-confirmed.

---

## Module 4: Agentic Architecture & Orchestration
**Exam weight: 27% — the largest domain.** Maps to the Agentic Architecture
& Orchestration domain. Taught fourth (not first) because most of its
content presupposes tool use and the context window (Module 1), the general
subagent notion and permission modes (Module 2), and MCP (Module 3).

- Lesson 4.1: The Agent Loop and the query() Function
- Lesson 4.2: Coordinator–Subagent Orchestration
- Lesson 4.3: Task Decomposition and Dynamic Workflows
- Lesson 4.4: Enforcement, Handoff, and Hooks
- Lesson 4.5: Session State Management

Domain traceability: all 14 concepts from the Agentic Architecture &
Orchestration research file (3 of them — Tool Use, System Prompt, The
Context Window — were taught in Module 1 and referenced here; 1 — Model
Context Protocol/MCP — was taught in full in Module 3, Lesson 3.4, and is
explicitly cross-referenced at the end of this module).

---

## Module 5: Prompt Engineering & Structured Output
**Exam weight: 20%.** Maps to the Prompt Engineering & Structured Output
domain. Builds on Module 1 (Messages API, tool use, JSON Schema) and Module
3 (tool_choice, tool_result error handling), which is why it is taught
after Tool Design rather than earlier.

- Lesson 5.1: Defining Success and Core Prompting Techniques
- Lesson 5.2: Increasing Consistency and Reducing False Positives
- Lesson 5.3: Structured Outputs and Tool-Call Reliability
- Lesson 5.4: Batch Processing and Prompt Caching for Scale
- Lesson 5.5: Multi-Instance Review Architectures

Domain traceability: all 21 concepts from the Prompt Engineering &
Structured Output research file. Several of this file's key/prerequisite
entries (Tool use, Defining tools/tool_choice, Validation and retry loops)
are the same concepts already taught with the same source URLs in Module 1
or Module 3, and are referenced here rather than re-taught in full — this
is noted explicitly in Lesson 5.3.

**Flagged as unverified (not omitted, per course-builder policy):** within
Lesson 5.2 ("Designing explicit, categorical review criteria to reduce
false positives"), a specific comparative sub-claim — that generic
instructions like "be conservative" or "only report high-confidence
findings" do *not* improve precision relative to specific categorical
criteria — is called out in a boxed note as **unverified**. The researcher
searched Anthropic's official documentation and could not find this exact
comparative claim stated there; it was found verbatim only on a
disqualified third-party aggregator site. The lesson explicitly instructs
the learner not to treat this specific sub-claim as established, while
noting that the broader, underlying principle it's attached to (specific,
checkable criteria beat vague confidence instructions) IS independently
documented and should be treated as established.

---

## Module 6: Context Management & Reliability
**Exam weight: 15%.** Maps to the Context Management & Reliability domain.
Taught last: it is the capstone module, deepening Module 1's context-window
basics and drawing on subagent/permission-mode material from Modules 2 and
4 and the grounding/caching material from Module 5.

- Lesson 6.1: Context Window Depth — Context Rot and Long-Context Recall
- Lesson 6.2: Managing Context at Scale — Compaction, Editing, and Caching
- Lesson 6.3: Long-Running Agents and Large Codebase Exploration
- Lesson 6.4: Error Propagation and Escalation to Humans
- Lesson 6.5: Ambiguity, Confidence, and Provenance

Domain traceability: all 30 concepts from the Context Management &
Reliability research file (several of its prerequisite entries — Tokens,
Messages API turn structure, Tool use loop, Extended thinking, Beta
headers, Agentic loop, Prompt engineering fundamentals, Permission/approval
model, Subagent/orchestrator-worker basics — were taught in Modules 1, 2,
4, or 5 and are referenced here rather than re-taught).

**Sourcing flags (exam-guide-only, no vendor corroboration found for the
specific named claim):**
- "Lost in the middle" positional recall degradation (Lesson 6.1, Task 5.1)
  — the exact phrase is exam-guide-only; the closest vendor corroboration
  (a related but differently-framed needle-in-a-haystack measurement and a
  document-placement recommendation) is cited alongside it and clearly
  distinguished as not a verbatim match.
- "Escalation triggers and anti-patterns for human handoff" (Lesson 6.4,
  Task 5.2) — business-process criteria and named anti-patterns
  (sentiment-based escalation, self-reported confidence scores) with no
  vendor documentation found.
- "Stratified random sampling and field-level confidence scores for
  validating extractions" (Lesson 6.5, Task 5.5) — QA methodology with no
  vendor documentation found.
- "Structured note-taking (scratchpad files / agentic memory)" (Lesson 6.3,
  Task 5.4) — this one IS corroborated by vendor documentation, but under a
  different name ("structured note-taking"/"agentic memory" rather than
  "scratchpad files"); both names and both sources are given so the
  learner can recognize either framing on the exam.

**Caveated, not flagged as unverified:** "Confidence expression and
calibration in human-review workflows" (Lesson 6.5) is fully sourced to
vendor documentation, but the lesson carries forward the researcher's
explicit caveat that Anthropic does not define a single named "confidence
calibration" framework — the concept is a composition of two separately
documented mechanisms (prompted uncertainty expression; denial-count-based
escalation), not one unified first-party feature. This is a transparency
note about scope, not a doubt about the underlying facts.

---

## Concepts deliberately excluded
None. All 115 concepts from the five domain research files are covered in
this course, either taught in full at their earliest point of need or
explicitly cross-referenced to that teaching point where the same concept
was independently documented in more than one domain file (see the "Domain
traceability" notes above for each module).

## Summary of sourcing transparency carried into the course
- **Vendor-documented concepts** (the large majority): sourced directly to
  Anthropic product/engineering documentation (platform.claude.com,
  code.claude.com, docs.claude.com, anthropic.com/engineering,
  modelcontextprotocol.io) or, where applicable, json-schema.org.
- **Exam-guide-only concepts** (properly sourced, not unsourced, but
  distinguished from vendor documentation in every lesson where they
  appear): Module 2 Lesson 2.4 (single message vs. sequential iteration);
  Module 3 Lessons 3.1 and 3.5 (keyword-sensitive tool associations; Read+Write
  Edit fallback); Module 6 Lessons 6.1, 6.3, 6.4, 6.5 ("lost in the
  middle"; scratchpad files/structured note-taking; escalation
  triggers/anti-patterns; stratified sampling/field-level confidence).
- **Explicitly flagged as unverified** (not presented as established fact):
  the "be conservative"/"only report high-confidence findings" comparative
  precision claim in Module 5 Lesson 5.2.
