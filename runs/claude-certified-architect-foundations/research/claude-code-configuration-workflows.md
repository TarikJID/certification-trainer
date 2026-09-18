# Domain 3 Research — Claude Code Configuration & Workflows

Bullets received: 49 (23 Knowledge, 26 Skills)
Bullets covered: 49 (23 Knowledge, 26 Skills)
Concepts in this file: 37 (31 key, 6 prerequisite)

Coverage map (bullet ID → concept #) is embedded in each concept's `Teaches:` field below;
every one of the 49 bullet IDs listed in the dispatch slice appears in at least one
`Teaches:` field.

---

## Task Statement 3.1 — CLAUDE.md hierarchy, scoping, modular organization

- Concept: CLAUDE.md configuration hierarchy (user, project, directory)
  Type: key
  Teaches: 3.1-K1, 3.1-S1
  Definition: Claude Code reads persistent-instruction files named CLAUDE.md from several scopes: user-level (`~/.claude/CLAUDE.md`, applies to one person across all their projects), project-level (`./CLAUDE.md` or `./.claude/CLAUDE.md`, shared with the team via version control), and directory-level (CLAUDE.md files in subdirectories, loaded on demand when Claude reads files in that subdirectory). CLAUDE.md and CLAUDE.local.md files in the directory hierarchy above the working directory load at launch; files in subdirectories under the working directory load only when Claude reads files there. Content from broader scopes is loaded before content from narrower/closer scopes, so an instruction closer to the working directory is read last (and therefore has more recency weight). Diagnosing hierarchy problems (why a teammate isn't seeing an instruction) means checking which of these scopes actually holds the file.
  Example: A monorepo has `~/.claude/CLAUDE.md` with a developer's personal formatting preferences, `./CLAUDE.md` at the repo root with the team's build/test commands, and `packages/api/CLAUDE.md` with API-package-specific conventions that only load when Claude works inside `packages/api/`.
  Source: https://code.claude.com/docs/en/memory ("Choose where to put CLAUDE.md files" and "How CLAUDE.md files load" sections)

- Concept: User-level CLAUDE.md is personal and not shared via version control
  Type: key
  Teaches: 3.1-K2, 3.1-S1
  Definition: `~/.claude/CLAUDE.md` lives outside any git repository, in the user's home directory. It is loaded for that user across every project on their machine, but it is never committed to a repository and therefore never reaches teammates automatically. This is the single most common cause of "a new team member isn't getting instructions I expect them to have": the instruction was written to the individual's user-level file instead of the project-level file that ships through source control.
  Example: A senior engineer adds "always run `make lint` before committing" to their own `~/.claude/CLAUDE.md`. A new hire clones the repository, runs Claude Code, and never sees that rule — because it was never in the project's `./CLAUDE.md`, only in the senior engineer's personal file.
  Source: https://code.claude.com/docs/en/memory (table row "User instructions... Shared with: Just you (all projects)" vs "Project instructions... Shared with: Team members via source control")

- Concept: @import syntax for modular CLAUDE.md
  Type: key
  Teaches: 3.1-K3, 3.1-S2
  Definition: CLAUDE.md files can pull in other files using `@path/to/file` syntax. Imported files are expanded and loaded into context at launch alongside the file that references them; imports can be relative or absolute, resolve relative to the file containing the import (not the working directory), and can recursively import further files up to a maximum depth of four hops. This lets a team keep one lean root CLAUDE.md that selectively imports only the standards files relevant to a given package or maintainer, rather than growing one monolithic file. Wrapping a path in backticks (`` `@README` ``) prevents it from being treated as an import.
  Example: A package-level CLAUDE.md for the payments module writes `@../../docs/pci-compliance-standards.md` to pull in PCI standards only relevant to that package, while the frontend package's CLAUDE.md imports `@../../docs/accessibility-standards.md` instead.
  Source: https://code.claude.com/docs/en/memory ("Import additional files" section)

- Concept: .claude/rules/ directory as an alternative to a monolithic CLAUDE.md
  Type: key
  Teaches: 3.1-K4, 3.1-S3
  Definition: For larger projects, instructions can be organized into multiple topic-specific markdown files placed in `.claude/rules/` (e.g., `testing.md`, `api-conventions.md`, `deployment.md`) instead of one large CLAUDE.md. All `.md` files in `.claude/rules/` are discovered recursively, so rules can also be grouped into subdirectories. Rules without `paths` frontmatter load at launch with the same priority as `.claude/CLAUDE.md`. Splitting a large CLAUDE.md into `.claude/rules/` files keeps instructions modular and easier for a team to maintain, and — when combined with path scoping — reduces the context loaded in any one session.
  Example: A 400-line CLAUDE.md covering testing conventions, API design rules, and deployment steps is split into `.claude/rules/testing.md`, `.claude/rules/api-conventions.md`, and `.claude/rules/deployment.md`, each focused on one topic.
  Source: https://code.claude.com/docs/en/memory ("Organize rules with .claude/rules/" section)

- Concept: /memory command for inspecting loaded memory files
  Type: key
  Teaches: 3.1-S4
  Definition: The `/memory` command lists CLAUDE.md, CLAUDE.local.md, and other memory-file locations across user and project scopes (including entries for files that don't yet exist), lets the user toggle auto memory on or off, and opens the auto memory folder. Selecting a file opens it in an editor. `/context` (a related but distinct command) shows which memory files actually loaded into the current session — the documented way to confirm a CLAUDE.md was picked up, and the first diagnostic step when a session behaves inconsistently with expectations (e.g., a rule seems to apply in one session but not another).
  Example: A developer notices Claude isn't following a rule they wrote. They run `/memory` to see every CLAUDE.md location Claude Code recognizes, discover the file they edited was `./.claude/CLAUDE.md` in the wrong package subdirectory, and run `/context` to confirm which memory files loaded for the current session.
  Source: https://code.claude.com/docs/en/memory ("View and edit with /memory" and "Troubleshoot memory issues" sections)

---

## Task Statement 3.2 — Custom slash commands and skills

- Concept: Project-scoped vs user-scoped commands and skills
  Type: key
  Teaches: 3.2-K1, 3.2-S1
  Definition: Custom commands/skills can be stored at project scope (`.claude/commands/` or `.claude/skills/<name>/SKILL.md` at the repository root) or user scope (`~/.claude/commands/` or `~/.claude/skills/<name>/SKILL.md` in the home directory). Project-scoped commands load only in sessions within that repository, are committed to version control, and are therefore shared with the whole team. User-scoped (personal) commands load in all of that user's projects on the machine but are not shared through version control because they live outside any repository.
  Example: A team commits `.claude/commands/deploy.md` (or the newer `.claude/skills/deploy/SKILL.md`) so every teammate who clones the repo gets a `/deploy` command with the team's deployment checklist, while an individual engineer keeps a personal `~/.claude/commands/standup.md` for drafting their own daily standup notes that nobody else needs.
  Source: https://code.claude.com/docs/en/slash-commands ("Project-Scoped vs Personal Commands" — location, scope, and sharing tables)

- Concept: SKILL.md frontmatter configuration
  Type: key
  Teaches: 3.2-K2
  Definition: A skill lives in `.claude/skills/<skill-name>/SKILL.md`, a markdown file with YAML frontmatter. Beyond the base Agent Skills fields (`name`, `description`), Claude Code supports extended frontmatter fields including `context: fork` (run the skill in an isolated subagent), `allowed-tools` (pre-approve specific tools for the turn that invokes the skill), and `argument-hint` (display hint shown in the slash-command autocomplete menu). The `description` field is what Claude uses to decide when to apply the skill automatically; if omitted, Claude falls back to the first non-empty line of the skill body.
  Example:
  ```
  ---
  name: pr-summary
  description: Summarize changes in a pull request
  context: fork
  argument-hint: "[PR-number]"
  allowed-tools: Bash(gh *)
  ---
  ```
  Source: https://code.claude.com/docs/en/skills (frontmatter reference: `context`, `allowed-tools`, `argument-hint`, `name`, `description` fields)

- Concept: context: fork frontmatter (isolated subagent execution)
  Type: key
  Teaches: 3.2-K3, 3.2-S2
  Definition: Adding `context: fork` to a skill's frontmatter runs that skill in a forked subagent context instead of the main conversation. Claude Code starts a new subagent, gives it the skill content as its prompt, and the subagent does not see the main conversation history — its instructions have to stand on their own. Only the subagent's summarized result returns to the main session, so verbose output (large codebase analyses, log dumps) or exploratory/divergent reasoning (brainstorming alternatives) never pollutes or consumes tokens in the primary conversation.
  Example: A `/analyze-codebase` skill with `context: fork` runs a deep structural analysis across hundreds of files in an isolated subagent; the main session only receives the final summary, not the intermediate file reads and greps.
  Source: https://code.claude.com/docs/en/skills ("context: fork" behavior section)

- Concept: allowed-tools frontmatter (pre-approving tool access for a skill)
  Type: key
  Teaches: 3.2-S3
  Definition: The `allowed-tools` frontmatter field pre-approves a specific set of tools for the turn that invokes the skill, so Claude can use exactly those tools without a permission prompt; the grant clears once the invoking turn ends. It does not itself add a blocking restriction on other tools — other tools still follow normal permission settings — but by scoping the pre-approved set (e.g., only file-write operations) a skill author steers execution toward the intended, non-destructive operations and avoids over-broad blanket approval.
  Example: A skill intended only to update documentation sets `allowed-tools: Write Edit` so Claude can write and edit files during that skill's turn without a permission prompt, while destructive operations like `Bash(rm *)` still require normal approval.
  Source: https://code.claude.com/docs/en/skills ("allowed-tools" field: "grants permission for the listed tools during the turn that invokes the skill... Grant clears when you send your next message")

- Concept: argument-hint frontmatter
  Type: key
  Teaches: 3.2-S4
  Definition: The `argument-hint` frontmatter field is a display-only hint shown in the `/` autocomplete menu, indicating what arguments a skill/command expects when a developer invokes it. It does not enforce or parse arguments itself — it simply prompts the developer with the expected shape of the input (e.g., `[issue-number]`) so they know what to supply.
  Example: A skill with `argument-hint: "[filename] [format]"` shows that hint text under `/convert` in the autocomplete list, prompting a developer who types `/convert` with no arguments to supply a filename and a format.
  Source: https://code.claude.com/docs/en/skills ("argument-hint" field)

- Concept: Personal skill customization in ~/.claude/skills/
  Type: key
  Teaches: 3.2-K4
  Definition: A developer can create a personal variant of a team skill in `~/.claude/skills/<different-name>/SKILL.md`, giving it a different name from the project-scoped version so it does not collide with or override the team's shared skill. Because personal skills live outside the repository, this customization is invisible to and does not affect teammates — it is a way to tailor a workflow to one person's preferences without touching shared, version-controlled configuration.
  Example: The team's shared `.claude/skills/deploy/SKILL.md` runs a standard deployment checklist; one engineer who also needs to update a personal staging environment creates `~/.claude/skills/deploy-personal/SKILL.md` with extra steps, invoked as a separate `/deploy-personal` command that only they have.
  Source: https://code.claude.com/docs/en/skills ("Personal skills" scope description — available across the user's own projects, not shared)

- Concept: Choosing between skills and CLAUDE.md
  Type: key
  Teaches: 3.2-S5
  Definition: CLAUDE.md content loads into every session's context automatically ("always loaded"), so it should hold only universal, broadly-applicable standards; it costs context tokens on every conversation whether or not it's relevant to the current task. Skills load only on demand — when invoked directly (`/skill-name`) or when Claude determines they're relevant — so they are the right home for task-specific workflows, detailed procedures, or large reference material that would bloat CLAUDE.md if kept there permanently. The rule of thumb: if a CLAUDE.md section has grown into a multi-step procedure rather than a fact Claude needs every session, move it to a skill.
  Example: "Use 2-space indentation" belongs in CLAUDE.md (applies to every session). A 40-step "migrate a service from v1 to v2 API" procedure, needed only occasionally, belongs in a skill invoked with `/migrate-service` so it doesn't consume context on unrelated tasks.
  Source: https://code.claude.com/docs/en/memory ("When to add to CLAUDE.md" — "If an entry is a multi-step procedure or only matters for one part of the codebase, move it to a skill")

---

## Task Statement 3.3 — Path-specific rules for conditional convention loading

- Concept: Path-specific rules with YAML frontmatter `paths` field
  Type: key
  Teaches: 3.3-K1, 3.3-S1
  Definition: Files in `.claude/rules/` can be scoped to specific files using YAML frontmatter with a `paths` field containing one or more glob patterns (e.g., `paths: ["terraform/**/*"]`). A rule with a `paths` field is conditional: it only applies when Claude is working with files matching one of those patterns. Rules without a `paths` field are unconditional and load at launch for every session, applying to all files.
  Example:
  ```
  ---
  paths:
    - "src/api/**/*.ts"
  ---
  # API Development Rules
  - All API endpoints must include input validation
  ```
  This rule only enters context when Claude reads or edits a TypeScript file under `src/api/`.
  Source: https://code.claude.com/docs/en/memory ("Path-specific rules" section — YAML frontmatter `paths` field and example)

- Concept: Path-scoped conditional loading reduces context and token usage
  Type: key
  Teaches: 3.3-K2
  Definition: Because a path-scoped rule loads only when Claude reads a file matching its `paths` pattern (rather than at every session launch), it keeps irrelevant conventions out of the context window for tasks that never touch those files. This is a direct token-budget benefit over an unconditional CLAUDE.md or rule file, which is loaded regardless of relevance. Path-scoped rules trigger when Claude reads a file matching the pattern, not on every tool use.
  Example: A `.claude/rules/terraform.md` file with `paths: ["terraform/**/*"]` never enters context during a session that only touches the frontend codebase, saving the tokens that a monolithic always-loaded CLAUDE.md section on Terraform conventions would have consumed.
  Source: https://code.claude.com/docs/en/memory ("Rules let you scope instructions to specific file types or subdirectories" / "Path-scoped rules trigger when Claude reads files matching the pattern, not on every tool use")

- Concept: Glob patterns for conventions spanning multiple directories
  Type: key
  Teaches: 3.3-K3, 3.3-S2
  Definition: A `paths` glob pattern matches files by type or name regardless of where they live in the directory tree (e.g., `**/*.test.tsx` matches every test file anywhere in the repo). This is the key advantage over directory-level CLAUDE.md files, which are scoped to a physical directory: a convention that applies to a file *type* spread across many unrelated directories (like "all test files" or "all Terraform files") cannot be expressed as a single directory-level CLAUDE.md, but is naturally expressed as one glob-scoped rule.
  Example: `paths: ["**/*.test.tsx"]` in `.claude/rules/testing.md` applies testing conventions to every `.test.tsx` file across `src/`, `lib/`, and `tools/` directories alike, without needing a CLAUDE.md placed in each of those directories.
  Source: https://code.claude.com/docs/en/memory ("Use glob patterns in the paths field to match files by extension, directory, or any combination" and glob pattern table)

- Concept: Choosing path-specific rules over subdirectory CLAUDE.md files
  Type: key
  Teaches: 3.3-S3
  Definition: When conventions must apply to files that are spread throughout a codebase rather than confined to one directory subtree, a glob-scoped rule in `.claude/rules/` is the appropriate mechanism instead of placing a CLAUDE.md in every relevant subdirectory. A directory-level CLAUDE.md is naturally suited to conventions specific to that one directory's contents (e.g., "this package's API layer"), while a path-specific rule is suited to conventions defined by file type or pattern that cut across the directory structure.
  Example: For "all `*.test.tsx` files must use the shared test-utils wrapper," a single `.claude/rules/testing.md` with `paths: ["**/*.test.tsx"]` is preferable to placing a nearly identical CLAUDE.md in every directory that happens to contain test files.
  Source: https://code.claude.com/docs/en/memory (contrast between directory-level CLAUDE.md loading and glob-scoped rules in "Organize rules with .claude/rules/" and "Path-specific rules")

---

## Task Statement 3.4 — Plan mode vs direct execution

- Concept: Plan mode for complex, architectural, multi-file tasks
  Type: key
  Teaches: 3.4-K1, 3.4-S1
  Definition: Plan mode is a read-only exploration state in which Claude Code reads files, runs read-only shell commands, and drafts a step-by-step plan without making any changes, waiting for approval before writing code. It is designed for tasks involving large-scale changes, multiple valid implementation approaches, architectural decisions, and modifications spanning many files — situations where jumping straight to editing risks solving the wrong problem or committing to a poor approach before the tradeoffs are understood. Plan mode is activated with `Shift+Tab` (cycling permission modes), by prefixing a prompt with `/plan`, or by starting the session with `claude --permission-mode plan`.
  Example: Restructuring a monolith into microservices, or migrating a library used across 45+ files, are architectural-implication tasks where plan mode is selected so Claude proposes and gets approval on an approach before touching any file.
  Source: https://code.claude.com/docs/en/best-practices ("Explore first, then plan, then code" section)

- Concept: Direct execution for simple, well-scoped changes
  Type: key
  Teaches: 3.4-K2, 3.4-S2
  Definition: Direct execution — skipping plan mode and letting Claude implement immediately — is appropriate when the scope is clear and the change is small: a single validation check added to one function, a one-line bug fix backed by a clear stack trace, or a straightforward conditional added to existing logic. Plan mode adds overhead (an extra approval round-trip) that isn't worth paying when the diff could be described in one sentence.
  Example: "Add a date validation conditional so `parseDate` rejects dates after 2099" is well-scoped enough that a developer tells Claude to implement it directly rather than first drafting a plan.
  Source: https://code.claude.com/docs/en/best-practices ("For tasks where the scope is clear and the fix is small... ask Claude to do it directly. If you could describe the diff in one sentence, skip the plan.")

- Concept: Plan mode enables safe exploration before committing to changes
  Type: key
  Teaches: 3.4-K3
  Definition: Because plan mode is read-only until the plan is approved, it lets Claude fully investigate a codebase — reading relevant modules, tracing dependencies, weighing implementation options — without any risk of a premature or wrong edit landing in the working tree. This front-loads the discovery and decision-making work, so the developer reviews and can redirect the *approach* before any code is written, which prevents costly rework compared to discovering a design problem partway through an in-progress edit.
  Example: Before adding Google OAuth, plan mode is used to read the existing auth module and environment-variable handling, and to produce a plan naming exactly which files will change and what the session flow will be — all before a single line of code is touched, so a flawed approach can be caught and corrected for free.
  Source: https://code.claude.com/docs/en/best-practices ("Letting Claude jump straight to coding can produce code that solves the wrong problem. Use plan mode to separate exploration from execution.")

- Concept: Explore subagent for isolating verbose discovery
  Type: key
  Teaches: 3.4-K4, 3.4-S3
  Definition: Explore is a built-in, read-only subagent (it is denied Write and Edit) that Claude Code delegates to for codebase search and discovery work — grepping, reading files, tracing imports — so that the volume of intermediate search results and file contents accumulates in the subagent's own context window rather than the main conversation's. Only a summary returns to the main session. This is essential in multi-phase tasks where an unbounded discovery phase would otherwise consume enough of the context window to degrade performance on the later implementation phase.
  Example: During a large refactor, Claude uses the Explore subagent to locate every caller of a deprecated function across the codebase; the (potentially hundreds of file reads') worth of raw search output stays in Explore's isolated context, and the main conversation only receives the list of call sites.
  Source: https://code.claude.com/docs/en/sub-agents (Explore built-in subagent: read-only, used for file discovery/code search, returns summaries to preserve main context)

- Concept: Combining plan mode for investigation with direct execution for implementation
  Type: key
  Teaches: 3.4-S4
  Definition: A common workflow pairs the two modes sequentially: use plan mode to investigate and produce an approved plan for a task with real uncertainty (e.g., a library migration), then exit plan mode and let Claude execute the approved plan directly, without re-planning each subsequent step. This captures the safety benefit of planning for the uncertain, high-stakes part of the work while avoiding the overhead of plan mode for the now-well-defined implementation steps that follow.
  Example: Plan mode is used to investigate how to migrate from one HTTP client library to another (mapping every call site and deciding the replacement pattern); once the plan is approved, Claude switches to direct execution to apply that established pattern across all the files identified in the plan.
  Source: https://code.claude.com/docs/en/best-practices ("Explore first, then plan, then code" — Implement step: "Switch out of plan mode by approving the plan... then let Claude code, verifying against its plan")

---

## Task Statement 3.5 — Iterative refinement techniques

- Concept: Concrete input/output examples to clarify transformation requirements
  Type: key
  Teaches: 3.5-K1, 3.5-S1, 3.5-S4
  Definition: When a prose description of a desired transformation is interpreted inconsistently, providing 2-3 concrete input/output example pairs is a more effective way to communicate exactly what's expected than adding more prose. Examples remove ambiguity that natural-language descriptions leave open, and this same technique — providing specific example input and expected output — is equally effective narrowly, for fixing a specific edge case (e.g., how null values should be handled in a migration script) once the initial description has produced inconsistent results there.
  Example: Instead of "validate email addresses," give test cases: `user@example.com` → `true`, `invalid` → `false`, `user@.com` → `false`. For an edge case fix: "when `customer_id` is null in the source row, the migration should insert a placeholder row with `customer_id = 'UNKNOWN'`, not skip the row — see input/output pair X."
  Source: https://code.claude.com/docs/en/best-practices ("Give Claude a way to verify its work" table, "Provide verification criteria" row: prose vs. example-based prompt)

- Concept: Test-driven iteration
  Type: key
  Teaches: 3.5-K2, 3.5-S2
  Definition: Writing a test suite that covers expected behavior, edge cases, and performance requirements before implementation gives Claude an executable, pass/fail check to run instead of leaving "looks done" as the only signal that work is complete. Once that check exists, Claude implements, runs the tests, reads which ones fail, and iterates — repeating that cycle until the suite passes — rather than relying on a human to manually re-verify correctness after every change. Sharing the specific test failures on each pass is what drives the progressive improvement, in the same way that any other executable check (a build, a script comparing output to a fixture) closes Claude's verification loop.
  Example: Before asking Claude to implement a rate limiter, a developer writes tests asserting the limiter rejects the 11th request in a 10-request-per-minute window and allows the 1st after the window resets; Claude implements against those tests, reads which assertions fail, and iterates until the suite is green.
  Source: https://code.claude.com/docs/en/best-practices ("Give Claude a way to verify its work" — "Give Claude something that produces a pass or fail, and the loop closes on its own. Claude does the work, runs the check, reads the result, and iterates until the check passes"; "Provide verification criteria" table row uses example test cases run after implementing). The "write tests first, then iterate by sharing failures" framing is additionally corroborated by the exam guide's own wording at bullet 3.5-K2 ("Test-driven iteration: writing test suites first, then iterating by sharing test failures to guide progressive improvement"); no currently-live code.claude.com page documents a specific "write tests, confirm they fail, commit the failing tests as a checkpoint" procedure, so that narrower claim has been removed rather than attributed to a page that does not contain it.

- Concept: The interview pattern
  Type: key
  Teaches: 3.5-K3, 3.5-S3
  Definition: For larger or unfamiliar-domain features, having Claude interview the developer — asking clarifying questions about technical implementation, UI/UX, edge cases, and tradeoffs — before any implementation begins surfaces considerations the developer may not have thought of on their own. The developer starts with a minimal description and directs Claude to keep asking questions (typically via the `AskUserQuestion` tool) until the space is well covered, often ending with a written spec that a fresh implementation session can then execute against.
  Example: "I want to build a caching layer. Interview me in detail using the AskUserQuestion tool — ask about cache invalidation strategy, failure modes, and TTL policy — then write a spec to SPEC.md" surfaces cache-invalidation and failure-mode decisions the developer hadn't explicitly considered before implementation starts.
  Source: https://code.claude.com/docs/en/best-practices ("Let Claude interview you" section)

- Concept: Single-message batching vs sequential fixes for interacting vs independent issues
  Type: key
  Teaches: 3.5-K4, 3.5-S5
  Definition: When multiple issues are reported at once, whether to address them together in one detailed message or fix them one at a time in sequence depends on whether the issues interact. If fixes for different issues would touch overlapping logic or could conflict (interacting problems), providing all of them together in a single, detailed message lets Claude design one coherent solution that accounts for all constraints simultaneously. If the issues are independent (fixing one has no bearing on another), sequential iteration — fix one, verify, move to the next — keeps each change small, reviewable, and easy to attribute if something regresses.
  Example: Two bugs both touching the same discount-calculation function (interacting) are reported together in one message so Claude designs a single fix that satisfies both; an unrelated typo in a footer and a separate off-by-one error in pagination (independent) are fixed one after another so each change stays isolated and easy to verify.
  Source: /home/user/certification-trainer/runs/claude-certified-architect-foundations/sources/exam-guide.txt (Task Statement 3.5, bullets 3.5-K4 and 3.5-S5, verbatim) — non-official: no Claude Code product-documentation page states this single-message-vs-sequential decision rule; the exam guide's own task-statement wording is used directly as the official source per the "exam guide itself" category.

---

## Task Statement 3.6 — Integrating Claude Code into CI/CD pipelines

- Concept: -p / --print flag for non-interactive mode
  Type: key
  Teaches: 3.6-K1, 3.6-S1
  Definition: Adding `-p` (or `--print`) to a `claude` command runs it non-interactively: Claude Code executes the given prompt, prints the result, and exits, rather than opening the interactive terminal UI. This is what makes Claude Code usable as a step in an automated pipeline — an interactive session would hang waiting for input that a CI runner can never supply. Claude Code exits with code 0 on success and a non-zero code on failure, so a calling script or pipeline can branch on the exit status.
  Example: `claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"` runs as a single non-interactive CI step that completes and returns control (and an exit code) to the pipeline, instead of waiting for a human to respond to prompts.
  Source: https://code.claude.com/docs/en/headless ("Basic usage" — "Add the -p (or --print) flag to any claude command to run it non-interactively")

- Concept: --output-format json and --json-schema for structured CI output
  Type: key
  Teaches: 3.6-K2, 3.6-S2
  Definition: `--output-format json` returns the response as a single structured JSON object (with a `result` field, session ID, and metadata) instead of plain text, making it parseable by downstream scripts. Adding `--json-schema` with a JSON Schema definition further constrains the response to conform to that schema — Claude Code validates the schema itself at startup and returns the structured payload in a `structured_output` field. Together these let a CI pipeline reliably extract machine-readable findings (e.g., a list of review comments with file, line, and severity) for automated actions like posting inline PR comments, rather than parsing free-form text.
  Example: `claude -p "Review this diff for bugs" --output-format json --json-schema '{"type":"object","properties":{"findings":{"type":"array","items":{"type":"object","properties":{"file":{"type":"string"},"line":{"type":"integer"},"issue":{"type":"string"}}}}}}'` produces a `structured_output.findings` array a CI script can loop over to post one inline PR comment per finding.
  Source: https://code.claude.com/docs/en/headless ("Get structured output" section)

- Concept: CLAUDE.md as the mechanism for CI-invoked project context
  Type: key
  Teaches: 3.6-K3, 3.6-S5
  Definition: A CI-invoked `claude -p` run (without `--bare`) still loads the same CLAUDE.md files an interactive session would, so CLAUDE.md is the mechanism for giving automated invocations project context: testing standards, fixture conventions, and review criteria. Documenting testing standards, what counts as a valuable test, and which fixtures already exist directly in CLAUDE.md measurably improves the quality of CI-generated output (code reviews, generated tests) and reduces low-value output, because the invocation has the same standing context a human reviewer or test author would have.
  Example: A project's CLAUDE.md states "Tests must cover the happy path and at least one failure mode; do not write tests that only assert a function was called" and "Fixtures live in `tests/fixtures/`, reuse them rather than inlining test data." A CI-invoked Claude Code test-generation step reads this and avoids generating shallow assertion-only tests or duplicate fixtures.
  Source: https://code.claude.com/docs/en/github-actions ("Define project standards in CLAUDE.md" — "Create a CLAUDE.md file in your repository root to define code style guidelines, review criteria, project-specific rules, and preferred patterns. Claude follows these guidelines when creating PRs and responding to requests.")

- Concept: Session context isolation for code review
  Type: key
  Teaches: 3.6-K4
  Definition: A Claude session that just generated a piece of code carries the reasoning that produced it in its own context — the same assumptions and blind spots that led to any mistakes in the first place. An independent review instance, with no memory of that reasoning, evaluates the diff purely on its own merits and the stated review criteria, and is measurably more likely to catch subtle issues the generating session would have rationalized or overlooked. This is why CI code-review pipelines should invoke a fresh Claude Code session for review rather than reusing the same session (or asking the generating session to "review its own work").
  Example: A pipeline runs `claude -p "implement the feature"` in one job, then a separate `claude -p "review this diff"` job with no shared session state reviews the resulting diff — catching an edge case the implementing session didn't flag because it never questioned its own initial design choice.
  Source: https://code.claude.com/docs/en/best-practices ("A fresh context improves code review since Claude won't be biased toward code it just wrote" — Writer/Reviewer pattern and "Add an adversarial review step" sections)

- Concept: Including prior review findings to avoid duplicate PR comments
  Type: key
  Teaches: 3.6-S3
  Definition: When a review pipeline re-runs after new commits are pushed to a pull request, including the prior review's findings in the new invocation's context — and instructing Claude to report only new or still-unaddressed issues — prevents the pipeline from re-posting the same comment on every commit. Claude Code's own GitHub Actions review workflow implements a version of this by skipping pull requests that already have a comment from Claude; a custom pipeline achieves the same effect by feeding the prior findings back in as context on subsequent runs.
  Example: After an initial review flags a missing null check, a developer pushes a fix. The re-run review step is given the earlier findings list and instructed "only report issues not already listed below, and note if a listed issue is now resolved," so the fixed null-check finding isn't posted again as a fresh comment.
  Source: https://code.claude.com/docs/en/github-actions ("Claude skips... pull requests that already have a comment from Claude" — Run a skill section)

- Concept: Providing existing test files to avoid duplicate test generation
  Type: key
  Teaches: 3.6-S4
  Definition: When Claude Code is used in CI to generate new tests, providing the existing test files as context lets it recognize scenarios that are already covered and avoid suggesting duplicate test cases. Without that context, an automated test-generation step has no way to know what's already tested and will regenerate overlapping, low-value tests each run.
  Example: A nightly test-generation job passes the contents of `tests/auth.test.ts` into the prompt context alongside the source file being covered, so Claude generates only tests for scenarios the existing suite doesn't already assert, instead of re-proposing the same "returns 401 for missing token" test that already exists.
  Source: /home/user/certification-trainer/runs/claude-certified-architect-foundations/sources/exam-guide.txt (Task Statement 3.6, bullet 3.6-S4, verbatim) — non-official: no Claude Code product-documentation page describes this exact "existing test files as context to avoid duplicate scenarios" scenario as explicitly as the exam guide bullet itself; the exam guide's own task-statement wording is used directly as the official source per the "exam guide itself" category. (Corroborating, but not identical, general guidance on documenting testing standards in CLAUDE.md appears at https://code.claude.com/docs/en/github-actions.)

---

## Prerequisites

- Concept: Context window and token budget
  Type: prerequisite
  Teaches: prerequisite for CLAUDE.md configuration hierarchy, .claude/rules/ directory, path-specific rules (3.1/3.3 concepts), and Choosing between skills and CLAUDE.md (3.2-S5)
  Definition: Every Claude Code session has a finite context window; CLAUDE.md files, rule files, tool outputs, and conversation history all consume tokens from that budget, and Claude's performance degrades as it fills. CLAUDE.md and unconditional rules are loaded at every session's launch regardless of relevance, so their size is a direct, constant cost; conditional (path-scoped) rules and skills only cost tokens when actually loaded. Understanding this tradeoff is the reason hierarchy, modularization, and path-scoping exist as configuration tools in the first place.
  Example: A 1,000-line unconditional CLAUDE.md consumes roughly the same token budget every single session, even on a task that never touches the parts of the codebase most of those lines describe — which is why the documentation recommends targeting under 200 lines per CLAUDE.md file and moving conditional material into path-scoped rules or skills.
  Source: https://code.claude.com/docs/en/memory ("Write effective instructions" — "Size: target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence.")

- Concept: YAML frontmatter in markdown configuration files
  Type: prerequisite
  Teaches: prerequisite for SKILL.md frontmatter configuration, context: fork, allowed-tools, argument-hint (3.2 concepts) and Path-specific rules paths field (3.3 concepts)
  Definition: Claude Code's skill files (SKILL.md) and rule files (`.claude/rules/*.md`) both use a YAML frontmatter block — a `---`-delimited section at the top of the markdown file — to declare structured configuration (fields like `name`, `description`, `context`, `allowed-tools`, `paths`) that is parsed separately from the markdown body that forms the actual instruction content shown to Claude.
  Example:
  ```
  ---
  paths:
    - "src/api/**/*.ts"
  ---
  # API Development Rules
  ...
  ```
  The block between the `---` markers is parsed as YAML configuration; everything after it is the rule's markdown content.
  Source: https://code.claude.com/docs/en/memory ("Path-specific rules" — YAML frontmatter example) and https://code.claude.com/docs/en/skills (SKILL.md frontmatter fields)

- Concept: Subagents and isolated context
  Type: prerequisite
  Teaches: prerequisite for context: fork (3.2-K3/S2) and Explore subagent (3.4-K4/S3)
  Definition: A subagent is a specialized assistant that runs in its own, separate context window, with its own system prompt, tool access, and permissions — it does not see the parent conversation's history, files already read, or skills already invoked, unless that context is explicitly passed to it. Because a subagent's work happens in a context window that is discarded (or reduced to a summary) when it finishes, delegating a verbose task to a subagent is how Claude Code keeps large volumes of intermediate output out of the main conversation's token budget.
  Example: A subagent asked to "run the test suite and report only the failing tests" processes potentially thousands of lines of test-runner output inside its own isolated context, and returns only the short list of failures to the main conversation.
  Source: https://code.claude.com/docs/en/sub-agents ("Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions" / "do not see your conversation history, the skills you've already invoked, or the files Claude has already read")

- Concept: Tool permission approval and allowedTools
  Type: prerequisite
  Teaches: prerequisite for allowed-tools skill frontmatter (3.2-S3) and -p / CI non-interactive execution (3.6-K1/S1)
  Definition: Claude Code normally prompts for approval before using tools that could modify the system (file writes, Bash commands, MCP tools). The `--allowedTools` CLI flag (and the analogous `allowed-tools` frontmatter field on skills) lets specific tools be pre-approved so Claude can use them without an interactive prompt — which is what makes both scripted skill execution and any non-interactive (`-p`) CI run practical, since there is no human present to answer a permission prompt.
  Example: `claude -p "Run the test suite and fix any failures" --allowedTools "Bash,Read,Edit"` pre-approves exactly the tools that task needs, so the CI run completes without stalling on a permission prompt nobody in the pipeline can answer.
  Source: https://code.claude.com/docs/en/headless ("Auto-approve tools" — "Use --allowedTools to let Claude use certain tools without prompting")

- Concept: JSON Schema specification
  Type: prerequisite
  Teaches: prerequisite for --output-format json / --json-schema (3.6-K2/S2)
  Definition: JSON Schema is a standards-body specification (json-schema.org) for describing the shape of JSON data — required and optional properties, types, nested objects and arrays. Claude Code's `--json-schema` flag accepts a JSON Schema document and validates/constrains the model's structured output against it, so understanding what a JSON Schema document expresses (types, `required` arrays, nested `properties`) is a precondition for writing one that correctly shapes CI review or extraction output.
  Example: `{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}` describes an object with a required `functions` array of strings — the schema vocabulary `--json-schema` expects.
  Source: https://json-schema.org/ (JSON Schema specification — standards body primary source), corroborated by https://code.claude.com/docs/en/headless ("Get structured output" — "use --output-format json with --json-schema and a JSON Schema definition")

- Concept: Non-interactive scripting and CI/CD pipeline steps
  Type: prerequisite
  Teaches: prerequisite for -p / --print flag (3.6-K1/S1) and CLAUDE.md for CI project context (3.6-K3/S5)
  Definition: A CI/CD pipeline runs a sequence of discrete, unattended steps (build, test, lint, deploy) on a runner with no interactive terminal and no human available to respond to prompts; each step must run to completion (or a defined failure) without input and communicate success/failure via exit codes or files it produces. Treating Claude Code as one such step — rather than as an interactive assistant — is the premise behind flags like `-p`, `--output-format json`, and `--allowedTools`: they exist specifically to make a Claude Code invocation behave like any other unattended pipeline step.
  Example: A GitHub Actions job runs `claude -p "..." --output-format json` as one step in a workflow, consuming its JSON stdout the same way a later step would consume the output of a linter or test runner, with no human watching the run.
  Source: https://code.claude.com/docs/en/github-actions (workflow YAML examples showing Claude Code invoked as an unattended job step in `on: pull_request` / `on: schedule` triggered workflows)

---

## Summary of bullet coverage

Task 3.1 (8 bullets: K1-K4, S1-S4) — covered by concepts 1-5.
Task 3.2 (9 bullets: K1-K4, S1-S5) — covered by concepts 6-12.
Task 3.3 (6 bullets: K1-K3, S1-S3) — covered by concepts 13-16.
Task 3.4 (8 bullets: K1-K4, S1-S4) — covered by concepts 17-21.
Task 3.5 (9 bullets: K1-K4, S1-S5) — covered by concepts 22-25.
Task 3.6 (9 bullets: K1-K4, S1-S5) — covered by concepts 26-31.
Total: 49 bullets, all covered. 31 key concepts + 6 prerequisite concepts = 37 concepts total.
