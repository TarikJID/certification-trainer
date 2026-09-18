# Module 5 Hands-On Exercise

## Scenario: Building a PR Review and Test-Generation CI Pipeline

Your team wants two GitHub Actions jobs: (1) an automated code-review job that
posts inline PR comments, and (2) a nightly job that generates additional test
coverage for recently changed files. Separately, a developer is about to start a
large refactor: replacing a deprecated internal ORM across roughly 60 files.

### Part A — Choosing execution mode

1. Should the ORM replacement task start in plan mode or direct execution?
   Justify your answer, and describe what the plan-mode output should contain
   before any code is touched.
2. During the investigation phase, the agent needs to find every file that
   imports the deprecated ORM. Should the main agent do this search itself, or
   delegate it? Name the mechanism and explain the context-budget reason for your
   choice.

### Part B — Iterative refinement

3. The team's initial instruction to Claude for the ORM replacement — "update all
   usages to the new ORM's API" — produces inconsistent results on ambiguous call
   sites. Apply the concrete input/output examples technique to fix this: write
   2-3 example pairs showing an old-API call site and its correct new-API
   replacement.
4. Before starting the refactor, the developer isn't sure how the new ORM should
   handle a few edge cases (connection pooling, transaction retries). Describe
   how they'd use the interview pattern to surface these before implementation.
5. Partway through, three independent style nits and one bug that interacts with
   the transaction-retry logic are found. How should these four issues be
   reported to Claude? Justify by which are interacting and which are
   independent.

### Part C — CI/CD pipeline design

6. Write the `claude` CLI invocation for the code-review job: it must run
   non-interactively, output machine-parseable JSON conforming to a schema with a
   `findings` array (each finding having `file`, `line`, `severity`, `comment`),
   and use a fresh session with no shared state from any code-generation job.
7. This review job re-runs on every new commit to the same PR. Design how you'd
   prevent it from re-posting comments for already-fixed issues.
8. Design the nightly test-generation job: what context must it receive to avoid
   generating duplicate test cases, and where should the project's testing
   standards be documented so the CI-invoked session actually uses them?

Write your answers as a short pipeline design document, including the actual CLI
flags/commands where relevant.
