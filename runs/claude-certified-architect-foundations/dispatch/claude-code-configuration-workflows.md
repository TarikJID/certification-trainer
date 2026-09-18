## Domain 3: Claude Code Configuration & Workflows (20%)

Description: Covers CLAUDE.md configuration hierarchy and modular organization, custom slash
commands and Agent Skills, path-specific rules, choosing plan mode vs direct execution, iterative
refinement techniques, and integrating Claude Code into CI/CD pipelines.

Task statements: 6 · Bullets captured: 49 (23 Knowledge, 26 Skills)

### Task statements

- Task Statement 3.1: Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization
  Knowledge of:
    - 3.1-K1: The CLAUDE.md configuration hierarchy: user-level (~/.claude/CLAUDE.md), project-level (.claude/CLAUDE.md or root CLAUDE.md), and directory-level (subdirectory CLAUDE.md files)
    - 3.1-K2: That user-level settings apply only to that user—instructions in ~/.claude/CLAUDE.md are not shared with teammates via version control
    - 3.1-K3: The @import syntax for referencing external files to keep CLAUDE.md modular (e.g., importing specific standards files relevant to each package)
    - 3.1-K4: .claude/rules/ directory for organizing topic-specific rule files as an alternative to a monolithic CLAUDE.md
  Skills in:
    - 3.1-S1: Diagnosing configuration hierarchy issues (e.g., a new team member not receiving instructions because they're in user-level rather than project-level configuration)
    - 3.1-S2: Using @import to selectively include relevant standards files in each package's CLAUDE.md based on maintainer domain knowledge
    - 3.1-S3: Splitting large CLAUDE.md files into focused topic-specific files in .claude/rules/ (e.g., testing.md, api-conventions.md, deployment.md)
    - 3.1-S4: Using the /memory command to verify which memory files are loaded and diagnose inconsistent behavior across sessions

- Task Statement 3.2: Create and configure custom slash commands and skills
  Knowledge of:
    - 3.2-K1: Project-scoped commands in .claude/commands/ (shared via version control) vs user-scoped commands in ~/.claude/commands/ (personal)
    - 3.2-K2: Skills in .claude/skills/ with SKILL.md files that support frontmatter configuration including context: fork, allowed-tools, and argument-hint
    - 3.2-K3: The context: fork frontmatter option for running skills in an isolated sub-agent context, preventing skill outputs from polluting the main conversation
    - 3.2-K4: Personal skill customization: creating personal variants in ~/.claude/skills/ with different names to avoid affecting teammates
  Skills in:
    - 3.2-S1: Creating project-scoped slash commands in .claude/commands/ for team-wide availability via version control
    - 3.2-S2: Using context: fork to isolate skills that produce verbose output (e.g., codebase analysis) or exploratory context (e.g., brainstorming alternatives) from the main session
    - 3.2-S3: Configuring allowed-tools in skill frontmatter to restrict tool access during skill execution (e.g., limiting to file write operations to prevent destructive actions)
    - 3.2-S4: Using argument-hint frontmatter to prompt developers for required parameters when they invoke the skill without arguments
    - 3.2-S5: Choosing between skills (on-demand invocation for task-specific workflows) and CLAUDE.md (always-loaded universal standards)

- Task Statement 3.3: Apply path-specific rules for conditional convention loading
  Knowledge of:
    - 3.3-K1: .claude/rules/ files with YAML frontmatter paths fields containing glob patterns for conditional rule activation
    - 3.3-K2: How path-scoped rules load only when editing matching files, reducing irrelevant context and token usage
    - 3.3-K3: The advantage of glob-pattern rules over directory-level CLAUDE.md files for conventions that span multiple directories (e.g., test files spread throughout a codebase)
  Skills in:
    - 3.3-S1: Creating .claude/rules/ files with YAML frontmatter path scoping (e.g., paths: ["terraform/**/*"]) so rules load only when editing matching files
    - 3.3-S2: Using glob patterns in path-specific rules to apply conventions to files by type regardless of directory location (e.g., **/*.test.tsx for all test files)
    - 3.3-S3: Choosing path-specific rules over subdirectory CLAUDE.md files when conventions must apply to files spread across the codebase

- Task Statement 3.4: Determine when to use plan mode vs direct execution
  Knowledge of:
    - 3.4-K1: Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, architectural decisions, and multi-file modifications
    - 3.4-K2: Direct execution is appropriate for simple, well-scoped changes (e.g., adding a single validation check to one function)
    - 3.4-K3: Plan mode enables safe codebase exploration and design before committing to changes, preventing costly rework
    - 3.4-K4: The Explore subagent for isolating verbose discovery output and returning summaries to preserve main conversation context
  Skills in:
    - 3.4-S1: Selecting plan mode for tasks with architectural implications (e.g., microservice restructuring, library migrations affecting 45+ files, choosing between integration approaches with different infrastructure requirements)
    - 3.4-S2: Selecting direct execution for well-understood changes with clear scope (e.g., a single-file bug fix with a clear stack trace, adding a date validation conditional)
    - 3.4-S3: Using the Explore subagent for verbose discovery phases to prevent context window exhaustion during multi-phase tasks
    - 3.4-S4: Combining plan mode for investigation with direct execution for implementation (e.g., planning a library migration, then executing the planned approach)

- Task Statement 3.5: Apply iterative refinement techniques for progressive improvement
  Knowledge of:
    - 3.5-K1: Concrete input/output examples as the most effective way to communicate expected transformations when prose descriptions are interpreted inconsistently
    - 3.5-K2: Test-driven iteration: writing test suites first, then iterating by sharing test failures to guide progressive improvement
    - 3.5-K3: The interview pattern: having Claude ask questions to surface considerations the developer may not have anticipated before implementing
    - 3.5-K4: When to provide all issues in a single message (interacting problems) versus fixing them sequentially (independent problems)
  Skills in:
    - 3.5-S1: Providing 2-3 concrete input/output examples to clarify transformation requirements when natural language descriptions produce inconsistent results
    - 3.5-S2: Writing test suites covering expected behavior, edge cases, and performance requirements before implementation, then iterating by sharing test failures
    - 3.5-S3: Using the interview pattern to surface design considerations (e.g., cache invalidation strategies, failure modes) before implementing solutions in unfamiliar domains
    - 3.5-S4: Providing specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)
    - 3.5-S5: Addressing multiple interacting issues in a single detailed message when fixes interact, versus sequential iteration for independent issues

- Task Statement 3.6: Integrate Claude Code into CI/CD pipelines
  Knowledge of:
    - 3.6-K1: The -p (or --print) flag for running Claude Code in non-interactive mode in automated pipelines
    - 3.6-K2: --output-format json and --json-schema CLI flags for enforcing structured output in CI contexts
    - 3.6-K3: CLAUDE.md as the mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code
    - 3.6-K4: Session context isolation: why the same Claude session that generated code is less effective at reviewing its own changes compared to an independent review instance
  Skills in:
    - 3.6-S1: Running Claude Code in CI with the -p flag to prevent interactive input hangs
    - 3.6-S2: Using --output-format json with --json-schema to produce machine-parseable structured findings for automated posting as inline PR comments
    - 3.6-S3: Including prior review findings in context when re-running reviews after new commits, instructing Claude to report only new or still-unaddressed issues to avoid duplicate comments
    - 3.6-S4: Providing existing test files in context so test generation avoids suggesting duplicate scenarios already covered by the test suite
    - 3.6-S5: Documenting testing standards, valuable test criteria, and available fixtures in CLAUDE.md to improve test generation quality and reduce low-value test output

---

