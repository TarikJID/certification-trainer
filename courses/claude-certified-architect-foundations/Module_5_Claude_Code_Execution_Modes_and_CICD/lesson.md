# Module 5 — Claude Code Execution Modes and CI/CD Workflows

Domain: Domain 3 — Claude Code Configuration & Workflows (20% of exam)
Covers Task Statements 3.4, 3.5, 3.6.

**Prerequisites from earlier modules:** subagent context isolation (Module 1,
Lesson 1.2); CLAUDE.md as project context (Module 4, Lesson 3.1); tool permission
approval / `allowedTools` (Module 4, Lesson 3.2).

---

## Lesson 3.4 — Plan Mode vs Direct Execution

**Maps to:** Task Statement 3.4: Determine when to use plan mode vs direct
execution.

### Prerequisite concept: Subagents and isolated context — recap

Recall from Module 1, Lesson 1.2: a subagent runs in its own, separate context
window and does not see the parent conversation's history unless explicitly
passed. This is the structural basis for the Explore subagent below.

*Source:* https://code.claude.com/docs/en/sub-agents
(Teaches: prerequisite for context: fork and the Explore subagent)

### Concept: Plan mode for complex, architectural, multi-file tasks

Plan mode is a read-only exploration state in which Claude Code reads files, runs
read-only shell commands, and drafts a step-by-step plan without making any
changes, waiting for approval before writing code. It's designed for large-scale
changes, multiple valid approaches, architectural decisions, and modifications
spanning many files. Activated with `Shift+Tab`, `/plan`, or
`claude --permission-mode plan`.

*Example:* Restructuring a monolith into microservices, or migrating a library
used across 45+ files, are architectural-implication tasks where plan mode is
selected.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.4-K1, 3.4-S1)

### Concept: Direct execution for simple, well-scoped changes

Direct execution — skipping plan mode — is appropriate when the scope is clear
and the change is small: a single validation check, a one-line bug fix backed by
a clear stack trace, a straightforward conditional. Plan mode adds an extra
approval round-trip that isn't worth paying when the diff could be described in
one sentence.

*Example:* "Add a date validation conditional so `parseDate` rejects dates after
2099" is well-scoped enough for direct execution.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.4-K2, 3.4-S2)

### Concept: Plan mode enables safe exploration before committing to changes

Because plan mode is read-only until approved, it lets Claude fully investigate a
codebase — reading modules, tracing dependencies, weighing options — without risk
of a premature or wrong edit landing in the working tree. The developer reviews
and can redirect the *approach* before any code is written, preventing costly
rework.

*Example:* Before adding Google OAuth, plan mode reads the existing auth module
and produces a plan naming exactly which files will change, all before a single
line of code is touched.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.4-K3)

### Concept: Explore subagent for isolating verbose discovery

Explore is a built-in, read-only subagent (denied Write and Edit) that Claude
Code delegates to for codebase search and discovery — grepping, reading files,
tracing imports — so intermediate search results and file contents accumulate in
the subagent's own context window rather than the main conversation's. Only a
summary returns.

*Example:* During a large refactor, Explore locates every caller of a deprecated
function; the raw search output stays in Explore's isolated context, and the main
conversation only receives the list of call sites.

*Source:* https://code.claude.com/docs/en/sub-agents
(Teaches: 3.4-K4, 3.4-S3)

### Concept: Combining plan mode for investigation with direct execution for implementation

A common workflow pairs both modes: use plan mode to investigate and produce an
approved plan for a task with real uncertainty, then exit plan mode and let
Claude execute the approved plan directly, without re-planning each subsequent
step.

*Example:* Plan mode investigates migrating from one HTTP client library to
another (mapping every call site); once approved, Claude switches to direct
execution to apply the pattern across every identified file.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.4-S4)

**Quiz:** see `quiz.md`, Lesson 3.4.

---

## Lesson 3.5 — Iterative Refinement Techniques

**Maps to:** Task Statement 3.5: Apply iterative refinement techniques for
progressive improvement.

### Concept: Concrete input/output examples to clarify transformation requirements

When a prose description of a desired transformation is interpreted
inconsistently, providing 2-3 concrete input/output example pairs communicates
exactly what's expected more effectively than more prose. The same technique
works narrowly, for fixing a specific edge case once the initial description has
produced inconsistent results there.

*Example:* Instead of "validate email addresses," give test cases:
`user@example.com` → `true`, `invalid` → `false`, `user@.com` → `false`. For an
edge case: "when `customer_id` is null in the source row, insert a placeholder
row with `customer_id = 'UNKNOWN'`, not skip the row."

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.5-K1, 3.5-S1, 3.5-S4)

### Concept: Test-driven iteration

Writing a test suite covering expected behavior, edge cases, and performance
requirements before implementation gives Claude an executable pass/fail check to
run instead of leaving "looks done" as the only completion signal. Claude
implements, runs the tests, reads which fail, and iterates until the suite
passes.

*Example:* Before asking Claude to implement a rate limiter, a developer writes
tests asserting the limiter rejects the 11th request in a 10-request-per-minute
window; Claude implements against those tests and iterates until green.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.5-K2, 3.5-S2)

### Concept: The interview pattern

For larger or unfamiliar-domain features, having Claude interview the developer
— asking clarifying questions about implementation, UI/UX, edge cases, and
tradeoffs — before implementation begins surfaces considerations the developer
may not have thought of. Typically driven via the `AskUserQuestion` tool, often
ending with a written spec a fresh implementation session executes against.

*Example:* "I want to build a caching layer. Interview me in detail using the
AskUserQuestion tool — ask about cache invalidation strategy, failure modes, and
TTL policy — then write a spec to SPEC.md."

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.5-K3, 3.5-S3)

### Concept: Single-message batching vs sequential fixes for interacting vs independent issues

Whether to address multiple reported issues together in one detailed message or
fix them sequentially depends on whether the issues interact. If fixes would
touch overlapping logic or could conflict, provide all of them together so Claude
designs one coherent solution. If independent, sequential iteration keeps each
change small, reviewable, and easy to attribute if something regresses.

*Example:* Two bugs both touching the same discount-calculation function
(interacting) are reported together in one message; an unrelated footer typo and
a separate pagination off-by-one (independent) are fixed one after another.

*Source:* exam-guide.txt (Task 3.5, non-official but exam guide's own wording is
an acceptable official source per this course's sourcing standard)
(Teaches: 3.5-K4, 3.5-S5)

**Quiz:** see `quiz.md`, Lesson 3.5.

---

## Lesson 3.6 — Integrating Claude Code into CI/CD Pipelines

**Maps to:** Task Statement 3.6: Integrate Claude Code into CI/CD pipelines.

### Prerequisite concept: JSON Schema specification

JSON Schema is a standards-body specification (json-schema.org) for describing
the shape of JSON data — required and optional properties, types, nested objects
and arrays. Claude Code's `--json-schema` flag accepts a JSON Schema document and
validates/constrains the model's structured output against it.

*Example:* `{"type":"object","properties":{"functions":{"type":"array","items":
{"type":"string"}}},"required":["functions"]}` describes an object with a
required `functions` array of strings.

*Source:* https://json-schema.org/; corroborated by
https://code.claude.com/docs/en/headless
(Teaches: prerequisite for --output-format json / --json-schema; this course
returns to JSON Schema design in depth in Module 6, Lesson 4.3)

### Prerequisite concept: Non-interactive scripting and CI/CD pipeline steps

A CI/CD pipeline runs discrete, unattended steps on a runner with no interactive
terminal and no human to respond to prompts; each step must run to completion
without input and communicate success/failure via exit codes or output files.
Treating Claude Code as one such step is the premise behind flags like `-p`,
`--output-format json`, and `--allowedTools`.

*Example:* A GitHub Actions job runs `claude -p "..." --output-format json` as
one workflow step, consuming its JSON stdout the same way a later step consumes a
linter's output.

*Source:* https://code.claude.com/docs/en/github-actions
(Teaches: prerequisite for -p / --print flag and CLAUDE.md for CI project
context)

### Concept: -p / --print flag for non-interactive mode

Adding `-p` (or `--print`) to a `claude` command runs it non-interactively:
Claude Code executes the given prompt, prints the result, and exits, rather than
opening the interactive terminal UI. This is what makes it usable as a pipeline
step. Claude Code exits with code 0 on success, non-zero on failure, so a script
can branch on exit status.

*Example:* `claude -p "Find and fix the bug in auth.py" --allowedTools
"Read,Edit,Bash"` runs as a single non-interactive CI step.

*Source:* https://code.claude.com/docs/en/headless
(Teaches: 3.6-K1, 3.6-S1)

### Concept: --output-format json and --json-schema for structured CI output

`--output-format json` returns the response as a single structured JSON object
instead of plain text, making it parseable by downstream scripts. Adding
`--json-schema` further constrains the response to conform to a given schema —
Claude Code validates it at startup and returns the payload in a
`structured_output` field.

*Example:* `claude -p "Review this diff for bugs" --output-format json
--json-schema '{...findings array...}'` produces a `structured_output.findings`
array a CI script can loop over to post inline PR comments.

*Source:* https://code.claude.com/docs/en/headless
(Teaches: 3.6-K2, 3.6-S2)

### Concept: CLAUDE.md as the mechanism for CI-invoked project context

A CI-invoked `claude -p` run still loads the same CLAUDE.md files an interactive
session would, so CLAUDE.md is the mechanism for giving automated invocations
project context: testing standards, fixture conventions, review criteria.
Documenting these directly in CLAUDE.md measurably improves CI-generated output
quality and reduces low-value output.

*Example:* A project's CLAUDE.md states "Tests must cover the happy path and at
least one failure mode" and "Fixtures live in `tests/fixtures/`, reuse them
rather than inlining test data" — a CI-invoked test-generation step follows both.

*Source:* https://code.claude.com/docs/en/github-actions
(Teaches: 3.6-K3, 3.6-S5)

### Concept: Session context isolation for code review

A Claude session that just generated code carries the reasoning that produced it
— the same assumptions and blind spots that led to any mistakes. An independent
review instance, with no memory of that reasoning, evaluates the diff purely on
its merits and is measurably more likely to catch subtle issues. CI code-review
pipelines should invoke a fresh session for review rather than reusing the
generating session.

*Example:* A pipeline runs `claude -p "implement the feature"` in one job, then a
separate `claude -p "review this diff"` job with no shared session state —
catching an edge case the implementing session rationalized away.

*Source:* https://code.claude.com/docs/en/best-practices
(Teaches: 3.6-K4)

### Concept: Including prior review findings to avoid duplicate PR comments

When a review pipeline re-runs after new commits, including the prior review's
findings in the new invocation's context — and instructing Claude to report only
new or still-unaddressed issues — prevents re-posting the same comment on every
commit.

*Example:* After an initial review flags a missing null check, a developer pushes
a fix; the re-run review step is given the earlier findings and told "only report
issues not already listed below, and note if a listed issue is now resolved."

*Source:* https://code.claude.com/docs/en/github-actions
(Teaches: 3.6-S3)

### Concept: Providing existing test files to avoid duplicate test generation

When Claude Code generates new tests in CI, providing existing test files as
context lets it recognize scenarios already covered and avoid suggesting
duplicate cases. Without that context, an automated test-generation step will
regenerate overlapping, low-value tests each run.

*Example:* A nightly test-generation job passes the contents of
`tests/auth.test.ts` alongside the source file being covered, so Claude generates
only tests for scenarios not already asserted.

*Source:* exam-guide.txt (Task 3.6, exam guide's own wording; corroborating but
not identical guidance on CLAUDE.md testing-standards documentation at
https://code.claude.com/docs/en/github-actions)
(Teaches: 3.6-S4)

**Quiz:** see `quiz.md`, Lesson 3.6.

---

## Module 5 summary of bullets taught

Lesson 3.4: 3.4-K1, 3.4-K2, 3.4-K3, 3.4-K4, 3.4-S1, 3.4-S2, 3.4-S3, 3.4-S4
Lesson 3.5: 3.5-K1, 3.5-K2, 3.5-K3, 3.5-K4, 3.5-S1, 3.5-S2, 3.5-S3, 3.5-S4, 3.5-S5
Lesson 3.6: 3.6-K1, 3.6-K2, 3.6-K3, 3.6-K4, 3.6-S1, 3.6-S2, 3.6-S3, 3.6-S4, 3.6-S5
