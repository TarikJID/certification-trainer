# Domain Map — Claude Certified Architect – Foundations (CCAR-F)

## Sources

- Official certification page fetched: https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification
  (this page itself contains no domain/task-statement content; it links out to the exam guide PDF)
- Official exam guide PDF fetched: https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf
  ("Claude Certified Architect – Foundations Exam Guide", Version 1.0, effective July 2026, exam code CCAR-F)
- Archived source material (verbatim text extraction of all 39 pages of the exam guide PDF, including
  the full blueprint, all task statements, all knowledge/skills bullets, sample questions summary, and
  appendix): `/home/user/certification-trainer/runs/claude-certified-architect-foundations/sources/exam-guide.txt`

## Notes on sourcing

- The Skilljar certification landing page does not itself enumerate domains or task statements; it
  links to a PDF "Exam Guide," which is the authoritative source used for this map (per Section 1 of
  the guide: "This guide is the authoritative reference for candidates preparing to sit the exam... It
  describes the exam content, lists the domains and task statements tested...").
- The PDF could not be parsed by the `Read` tool's page-rendering path (poppler/pdftoppm is not
  installed in this environment), so it was instead fetched via `WebFetch`, which returned the PDF as
  a multi-page document with full text content per page (delivered as the tool's own rendering of the
  PDF, not an AI summary). That full text is what was transcribed verbatim into
  `sources/exam-guide.txt`. The original PDF bytes could not be copied into the repo with the tools
  available (no shell/file-copy tool was available to this agent), so the archive is a verbatim text
  extraction rather than the binary PDF, as permitted by the task instructions.
- The guide's structure matches the expected shape exactly: Section 6, "Detailed Objectives by
  Domain," gives, for each of 5 domains, a set of "Task Statement N.M: <title>" items, each followed
  by a "Knowledge of:" bulleted list and a "Skills in:" bulleted list. No ambiguity was encountered
  about which exam this material describes — exam code CCAR-F and credential name "Claude Certified
  Architect – Foundations" appear consistently on every page footer/header and in Section 3 ("Exam
  Details at a Glance").
- Total task statements: 30 (Domain 1: 7, Domain 2: 5, Domain 3: 6, Domain 4: 6, Domain 5: 6).
- Total bullets captured: 240 (Domain 1: 48, Domain 2: 43, Domain 3: 49, Domain 4: 47, Domain 5: 53).
  These counts were derived by manually tallying every bullet under every "Knowledge of:" and
  "Skills in:" heading for every task statement, including bullets that fall immediately after a PDF
  page break and are visually separated from their heading in the source rendering but belong to the
  same list (verified by list continuity and lack of a new heading).

## Bullet ID convention

Every bullet under a task statement is assigned an ID of the form `<statement id>-<K|S><n>`:

- `<statement id>-K<n>` — the *n*th bullet under that task statement's "Knowledge of:" heading.
- `<statement id>-S<n>` — the *n*th bullet under that task statement's "Skills in:" heading.

Numbering is positional within each list and starts at 1 for each heading under each task statement.
For example, the second bullet under task statement 3.4's "Skills in:" heading is `3.4-S2`. There is no
renumbering across headings and no global sequence — each list restarts at 1. No task statement in
this guide has ungrouped ("Measured:") bullets; every task statement uses the "Knowledge of:" /
"Skills in:" heading pair, so the `-M<n>` form defined in the convention is unused here.

---

## Domain 1: Agentic Architecture & Orchestration (27%)

Description: Covers designing and implementing the core agentic loop, orchestrating coordinator/subagent
multi-agent systems, configuring subagent spawning and context passing, enforcing multi-step
workflows with hooks and handoffs, decomposing complex workflows into tasks, and managing session
state/resumption/forking with the Claude Agent SDK.

Task statements: 7 · Bullets captured: 48 (24 Knowledge, 24 Skills)

### Task statements

- Task Statement 1.1: Design and implement agentic loops for autonomous task execution
  Knowledge of:
    - 1.1-K1: The agentic loop lifecycle: sending requests to Claude, inspecting stop_reason ("tool_use" vs "end_turn"), executing requested tools, and returning results for the next iteration
    - 1.1-K2: How tool results are appended to conversation history so the model can reason about the next action
    - 1.1-K3: The distinction between model-driven decision-making (Claude reasons about which tool to call next based on context) and pre-configured decision trees or tool sequences
  Skills in:
    - 1.1-S1: Implementing agentic loop control flow that continues when stop_reason is "tool_use" and terminates when stop_reason is "end_turn"
    - 1.1-S2: Adding tool results to conversation context between iterations so the model can incorporate new information into its reasoning
    - 1.1-S3: Avoiding anti-patterns such as parsing natural language signals to determine loop termination, setting arbitrary iteration caps as the primary stopping mechanism, or checking for assistant text content as a completion indicator

- Task Statement 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns
  Knowledge of:
    - 1.2-K1: Hub-and-spoke architecture where a coordinator agent manages all inter-subagent communication, error handling, and information routing
    - 1.2-K2: How subagents operate with isolated context—they do not inherit the coordinator's conversation history automatically
    - 1.2-K3: The role of the coordinator in task decomposition, delegation, result aggregation, and deciding which subagents to invoke based on query complexity
    - 1.2-K4: Risks of overly narrow task decomposition by the coordinator, leading to incomplete coverage of broad research topics
  Skills in:
    - 1.2-S1: Designing coordinator agents that analyze query requirements and dynamically select which subagents to invoke rather than always routing through the full pipeline
    - 1.2-S2: Partitioning research scope across subagents to minimize duplication (e.g., assigning distinct subtopics or source types to each agent)
    - 1.2-S3: Implementing iterative refinement loops where the coordinator evaluates synthesis output for gaps, re-delegates to search and analysis subagents with targeted queries, and re-invokes synthesis until coverage is sufficient
    - 1.2-S4: Routing all subagent communication through the coordinator for observability, consistent error handling, and controlled information flow

- Task Statement 1.3: Configure subagent invocation, context passing, and spawning
  Knowledge of:
    - 1.3-K1: The Task tool as the mechanism for spawning subagents, and the requirement that allowedTools must include "Task" for a coordinator to invoke subagents
    - 1.3-K2: That subagent context must be explicitly provided in the prompt—subagents do not automatically inherit parent context or share memory between invocations
    - 1.3-K3: The AgentDefinition configuration including descriptions, system prompts, and tool restrictions for each subagent type
    - 1.3-K4: Fork-based session management for exploring divergent approaches from a shared analysis baseline
  Skills in:
    - 1.3-S1: Including complete findings from prior agents directly in the subagent's prompt (e.g., passing web search results and document analysis outputs to the synthesis subagent)
    - 1.3-S2: Using structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution
    - 1.3-S3: Spawning parallel subagents by emitting multiple Task tool calls in a single coordinator response rather than across separate turns
    - 1.3-S4: Designing coordinator prompts that specify research goals and quality criteria rather than step-by-step procedural instructions, to enable subagent adaptability

- Task Statement 1.4: Implement multi-step workflows with enforcement and handoff patterns
  Knowledge of:
    - 1.4-K1: The difference between programmatic enforcement (hooks, prerequisite gates) and prompt-based guidance for workflow ordering
    - 1.4-K2: When deterministic compliance is required (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate
    - 1.4-K3: Structured handoff protocols for mid-process escalation that include customer details, root cause analysis, and recommended actions
  Skills in:
    - 1.4-S1: Implementing programmatic prerequisites that block downstream tool calls until prerequisite steps have completed (e.g., blocking process_refund until get_customer has returned a verified customer ID)
    - 1.4-S2: Decomposing multi-concern customer requests into distinct items, then investigating each in parallel using shared context before synthesizing a unified resolution
    - 1.4-S3: Compiling structured handoff summaries (customer ID, root cause, refund amount, recommended action) when escalating to human agents who lack access to the conversation transcript

- Task Statement 1.5: Apply Agent SDK hooks for tool call interception and data normalization
  Knowledge of:
    - 1.5-K1: Hook patterns (e.g., PostToolUse) that intercept tool results for transformation before the model processes them
    - 1.5-K2: Hook patterns that intercept outgoing tool calls to enforce compliance rules (e.g., blocking refunds above a threshold)
    - 1.5-K3: The distinction between using hooks for deterministic guarantees versus relying on prompt instructions for probabilistic compliance
  Skills in:
    - 1.5-S1: Implementing PostToolUse hooks to normalize heterogeneous data formats (Unix timestamps, ISO 8601, numeric status codes) from different MCP tools before the agent processes them
    - 1.5-S2: Implementing tool call interception hooks that block policy-violating actions (e.g., refunds exceeding $500) and redirect to alternative workflows (e.g., human escalation)
    - 1.5-S3: Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance

- Task Statement 1.6: Design task decomposition strategies for complex workflows
  Knowledge of:
    - 1.6-K1: When to use fixed sequential pipelines (prompt chaining) versus dynamic adaptive decomposition based on intermediate findings
    - 1.6-K2: Prompt chaining patterns that break reviews into sequential steps (e.g., analyze each file individually, then run a cross-file integration pass)
    - 1.6-K3: The value of adaptive investigation plans that generate subtasks based on what is discovered at each step
  Skills in:
    - 1.6-S1: Selecting task decomposition patterns appropriate to the workflow: prompt chaining for predictable multi-aspect reviews, dynamic decomposition for open-ended investigation tasks
    - 1.6-S2: Splitting large code reviews into per-file local analysis passes plus a separate cross-file integration pass to avoid attention dilution
    - 1.6-S3: Decomposing open-ended tasks (e.g., "add comprehensive tests to a legacy codebase") by first mapping structure, identifying high-impact areas, then creating a prioritized plan that adapts as dependencies are discovered

- Task Statement 1.7: Manage session state, resumption, and forking
  Knowledge of:
    - 1.7-K1: Named session resumption using --resume <session-name> to continue a specific prior conversation
    - 1.7-K2: fork_session for creating independent branches from a shared analysis baseline to explore divergent approaches
    - 1.7-K3: The importance of informing the agent about changes to previously analyzed files when resuming sessions after code modifications
    - 1.7-K4: Why starting a new session with a structured summary is more reliable than resuming with stale tool results
  Skills in:
    - 1.7-S1: Using --resume with session names to continue named investigation sessions across work sessions
    - 1.7-S2: Using fork_session to create parallel exploration branches (e.g., comparing two testing strategies or refactoring approaches from a shared codebase analysis)
    - 1.7-S3: Choosing between session resumption (when prior context is mostly valid) and starting fresh with injected summaries (when prior tool results are stale)
    - 1.7-S4: Informing a resumed session about specific file changes for targeted re-analysis rather than requiring full re-exploration

---

## Domain 2: Tool Design & MCP Integration (18%)

Description: Covers designing effective tool interfaces and descriptions, implementing structured MCP
error responses, distributing tools across agents and configuring tool_choice, integrating MCP servers
into Claude Code and agent workflows, and selecting/applying Claude Code's built-in tools.

Task statements: 5 · Bullets captured: 43 (20 Knowledge, 23 Skills)

### Task statements

- Task Statement 2.1: Design effective tool interfaces with clear descriptions and boundaries
  Knowledge of:
    - 2.1-K1: Tool descriptions as the primary mechanism LLMs use for tool selection; minimal descriptions lead to unreliable selection among similar tools
    - 2.1-K2: The importance of including input formats, example queries, edge cases, and boundary explanations in tool descriptions
    - 2.1-K3: How ambiguous or overlapping tool descriptions cause misrouting (e.g., analyze_content vs analyze_document with near-identical descriptions)
    - 2.1-K4: The impact of system prompt wording on tool selection: keyword-sensitive instructions can create unintended tool associations
  Skills in:
    - 2.1-S1: Writing tool descriptions that clearly differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives
    - 2.1-S2: Renaming tools and updating descriptions to eliminate functional overlap (e.g., renaming analyze_content to extract_web_results with a web-specific description)
    - 2.1-S3: Splitting generic tools into purpose-specific tools with defined input/output contracts (e.g., splitting a generic analyze_document into extract_data_points, summarize_content, and verify_claim_against_source)
    - 2.1-S4: Reviewing system prompts for keyword-sensitive instructions that might override well-written tool descriptions

- Task Statement 2.2: Implement structured error responses for MCP tools
  Knowledge of:
    - 2.2-K1: The MCP isError flag pattern for communicating tool failures back to the agent
    - 2.2-K2: The distinction between transient errors (timeouts, service unavailability), validation errors (invalid input), business errors (policy violations), and permission errors
    - 2.2-K3: Why uniform error responses (generic "Operation failed") prevent the agent from making appropriate recovery decisions
    - 2.2-K4: The difference between retryable and non-retryable errors, and how returning structured metadata prevents wasted retry attempts
  Skills in:
    - 2.2-S1: Returning structured error metadata including errorCategory (transient/validation/permission), isRetryable boolean, and human-readable descriptions
    - 2.2-S2: Including retriable: false flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately
    - 2.2-S3: Implementing local error recovery within subagents for transient failures, propagating to the coordinator only errors that cannot be resolved locally along with partial results and what was attempted
    - 2.2-S4: Distinguishing between access failures (needing retry decisions) and valid empty results (representing successful queries with no matches)

- Task Statement 2.3: Distribute tools appropriately across agents and configure tool choice
  Knowledge of:
    - 2.3-K1: The principle that giving an agent access to too many tools (e.g., 18 instead of 4-5) degrades tool selection reliability by increasing decision complexity
    - 2.3-K2: Why agents with tools outside their specialization tend to misuse them (e.g., a synthesis agent attempting web searches)
    - 2.3-K3: Scoped tool access: giving agents only the tools needed for their role, with limited cross-role tools for specific high-frequency needs
    - 2.3-K4: tool_choice configuration options: "auto", "any", and forced tool selection ({"type": "tool", "name": "..."})
  Skills in:
    - 2.3-S1: Restricting each subagent's tool set to those relevant to its role, preventing cross-specialization misuse
    - 2.3-S2: Replacing generic tools with constrained alternatives (e.g., replacing fetch_url with load_document that validates document URLs)
    - 2.3-S3: Providing scoped cross-role tools for high-frequency needs (e.g., a verify_fact tool for the synthesis agent) while routing complex cases through the coordinator
    - 2.3-S4: Using tool_choice forced selection to ensure a specific tool is called first (e.g., forcing extract_metadata before enrichment tools), then processing subsequent steps in follow-up turns
    - 2.3-S5: Setting tool_choice: "any" to guarantee the model calls a tool rather than returning conversational text

- Task Statement 2.4: Integrate MCP servers into Claude Code and agent workflows
  Knowledge of:
    - 2.4-K1: MCP server scoping: project-level (.mcp.json) for shared team tooling vs user-level (~/.claude.json) for personal/experimental servers
    - 2.4-K2: Environment variable expansion in .mcp.json (e.g., ${GITHUB_TOKEN}) for credential management without committing secrets
    - 2.4-K3: That tools from all configured MCP servers are discovered at connection time and available simultaneously to the agent
    - 2.4-K4: MCP resources as a mechanism for exposing content catalogs (e.g., issue summaries, documentation hierarchies, database schemas) to reduce exploratory tool calls
  Skills in:
    - 2.4-S1: Configuring shared MCP servers in project-scoped .mcp.json with environment variable expansion for authentication tokens
    - 2.4-S2: Configuring personal/experimental MCP servers in user-scoped ~/.claude.json
    - 2.4-S3: Enhancing MCP tool descriptions to explain capabilities and outputs in detail, preventing the agent from preferring built-in tools (like Grep) over more capable MCP tools
    - 2.4-S4: Choosing existing community MCP servers over custom implementations for standard integrations (e.g., Jira), reserving custom servers for team-specific workflows
    - 2.4-S5: Exposing content catalogs as MCP resources to give agents visibility into available data without requiring exploratory tool calls

- Task Statement 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively
  Knowledge of:
    - 2.5-K1: Grep for content search (searching file contents for patterns like function names, error messages, or import statements)
    - 2.5-K2: Glob for file path pattern matching (finding files by name or extension patterns)
    - 2.5-K3: Read/Write for full file operations; Edit for targeted modifications using unique text matching
    - 2.5-K4: When Edit fails due to non-unique text matches, using Read + Write as a fallback for reliable file modifications
  Skills in:
    - 2.5-S1: Selecting Grep for searching code content across a codebase (e.g., finding all callers of a function, locating error messages)
    - 2.5-S2: Selecting Glob for finding files matching naming patterns (e.g., **/*.test.tsx)
    - 2.5-S3: Using Read to load full file contents followed by Write when Edit cannot find unique anchor text
    - 2.5-S4: Building codebase understanding incrementally: starting with Grep to find entry points, then using Read to follow imports and trace flows, rather than reading all files upfront
    - 2.5-S5: Tracing function usage across wrapper modules by first identifying all exported names, then searching for each name across the codebase

---

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

## Domain 4: Prompt Engineering & Structured Output (20%)

Description: Covers writing prompts with explicit criteria to reduce false positives, applying few-shot
prompting for consistency, enforcing structured output via tool_use and JSON schemas, implementing
validation/retry/feedback loops for extraction, designing batch processing strategies, and designing
multi-instance/multi-pass review architectures.

Task statements: 6 · Bullets captured: 47 (22 Knowledge, 25 Skills)

### Task statements

- Task Statement 4.1: Design prompts with explicit criteria to improve precision and reduce false positives
  Knowledge of:
    - 4.1-K1: The importance of explicit criteria over vague instructions (e.g., "flag comments only when claimed behavior contradicts actual code behavior" vs "check that comments are accurate")
    - 4.1-K2: How general instructions like "be conservative" or "only report high-confidence findings" fail to improve precision compared to specific categorical criteria
    - 4.1-K3: The impact of false positive rates on developer trust: high false positive categories undermine confidence in accurate categories
  Skills in:
    - 4.1-S1: Writing specific review criteria that define which issues to report (bugs, security) versus skip (minor style, local patterns) rather than relying on confidence-based filtering
    - 4.1-S2: Temporarily disabling high false-positive categories to restore developer trust while improving prompts for those categories
    - 4.1-S3: Defining explicit severity criteria with concrete code examples for each severity level to achieve consistent classification

- Task Statement 4.2: Apply few-shot prompting to improve output consistency and quality
  Knowledge of:
    - 4.2-K1: Few-shot examples as the most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results
    - 4.2-K2: The role of few-shot examples in demonstrating ambiguous-case handling (e.g., tool selection for ambiguous requests, branch-level test coverage gaps)
    - 4.2-K3: How few-shot examples enable the model to generalize judgment to novel patterns rather than matching only pre-specified cases
    - 4.2-K4: The effectiveness of few-shot examples for reducing hallucination in extraction tasks (e.g., handling informal measurements, varied document structures)
  Skills in:
    - 4.2-S1: Creating 2-4 targeted few-shot examples for ambiguous scenarios that show reasoning for why one action was chosen over plausible alternatives
    - 4.2-S2: Including few-shot examples that demonstrate specific desired output format (location, issue, severity, suggested fix) to achieve consistency
    - 4.2-S3: Providing few-shot examples distinguishing acceptable code patterns from genuine issues to reduce false positives while enabling generalization
    - 4.2-S4: Using few-shot examples to demonstrate correct handling of varied document structures (inline citations vs bibliographies, methodology sections vs embedded details)
    - 4.2-S5: Adding few-shot examples showing correct extraction from documents with varied formats to address empty/null extraction of required fields

- Task Statement 4.3: Enforce structured output using tool use and JSON schemas
  Knowledge of:
    - 4.3-K1: Tool use (tool_use) with JSON schemas as the most reliable approach for guaranteed schema-compliant structured output, eliminating JSON syntax errors
    - 4.3-K2: The distinction between tool_choice: "auto" (model may return text instead of calling a tool), "any" (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)
    - 4.3-K3: That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)
    - 4.3-K4: Schema design considerations: required vs optional fields, enum fields with "other" + detail string patterns for extensible categories
  Skills in:
    - 4.3-S1: Defining extraction tools with JSON schemas as input parameters and extracting structured data from the tool_use response
    - 4.3-S2: Setting tool_choice: "any" to guarantee structured output when multiple extraction schemas exist and the document type is unknown
    - 4.3-S3: Forcing a specific tool with tool_choice: {"type": "tool", "name": "extract_metadata"} to ensure a particular extraction runs before enrichment steps
    - 4.3-S4: Designing schema fields as optional (nullable) when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields
    - 4.3-S5: Adding enum values like "unclear" for ambiguous cases and "other" + detail fields for extensible categorization
    - 4.3-S6: Including format normalization rules in prompts alongside strict output schemas to handle inconsistent source formatting

- Task Statement 4.4: Implement validation, retry, and feedback loops for extraction quality
  Knowledge of:
    - 4.4-K1: Retry-with-error-feedback: appending specific validation errors to the prompt on retry to guide the model toward correction
    - 4.4-K2: The limits of retry: retries are ineffective when the required information is simply absent from the source document (vs format or structural errors)
    - 4.4-K3: Feedback loop design: tracking which code constructs trigger findings (detected_pattern field) to enable systematic analysis of dismissal patterns
    - 4.4-K4: The difference between semantic validation errors (values don't sum, wrong field placement) and schema syntax errors (eliminated by tool use)
  Skills in:
    - 4.4-S1: Implementing follow-up requests that include the original document, the failed extraction, and specific validation errors for model self-correction
    - 4.4-S2: Identifying when retries will be ineffective (e.g., information exists only in an external document not provided) versus when they will succeed (format mismatches, structural output errors)
    - 4.4-S3: Adding detected_pattern fields to structured findings to enable analysis of false positive patterns when developers dismiss findings
    - 4.4-S4: Designing self-correction validation flows: extracting "calculated_total" alongside "stated_total" to flag discrepancies, adding "conflict_detected" booleans for inconsistent source data

- Task Statement 4.5: Design efficient batch processing strategies
  Knowledge of:
    - 4.5-K1: The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA
    - 4.5-K2: Batch processing is appropriate for non-blocking, latency-tolerant workloads (overnight reports, weekly audits, nightly test generation) and inappropriate for blocking workflows (pre-merge checks)
    - 4.5-K3: The batch API does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)
    - 4.5-K4: custom_id fields for correlating batch request/response pairs
  Skills in:
    - 4.5-S1: Matching API approach to workflow latency requirements: synchronous API for blocking pre-merge checks, batch API for overnight/weekly analysis
    - 4.5-S2: Calculating batch submission frequency based on SLA constraints (e.g., 4-hour windows to guarantee 30-hour SLA with 24-hour batch processing)
    - 4.5-S3: Handling batch failures: resubmitting only failed documents (identified by custom_id) with appropriate modifications (e.g., chunking documents that exceeded context limits)
    - 4.5-S4: Using prompt refinement on a sample set before batch-processing large volumes to maximize first-pass success rates and reduce iterative resubmission costs

- Task Statement 4.6: Design multi-instance and multi-pass review architectures
  Knowledge of:
    - 4.6-K1: Self-review limitations: a model retains reasoning context from generation, making it less likely to question its own decisions in the same session
    - 4.6-K2: Independent review instances (without prior reasoning context) are more effective at catching subtle issues than self-review instructions or extended thinking
    - 4.6-K3: Multi-pass review: splitting large reviews into per-file local analysis passes plus cross-file integration passes to avoid attention dilution and contradictory findings
  Skills in:
    - 4.6-S1: Using a second independent Claude instance to review generated code without the generator's reasoning context
    - 4.6-S2: Splitting large multi-file reviews into focused per-file passes for local issues plus separate integration passes for cross-file data flow analysis
    - 4.6-S3: Running verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing

---

## Domain 5: Context Management & Reliability (15%)

Description: Covers managing conversation context to preserve critical information over long
interactions, designing escalation and ambiguity-resolution patterns, propagating errors across
multi-agent systems, managing context in large codebase exploration, designing human review
workflows with confidence calibration, and preserving information provenance in multi-source
synthesis.

Task statements: 6 · Bullets captured: 53 (24 Knowledge, 29 Skills)

### Task statements

- Task Statement 5.1: Manage conversation context to preserve critical information across long interactions
  Knowledge of:
    - 5.1-K1: Progressive summarization risks: condensing numerical values, percentages, dates, and customer-stated expectations into vague summaries
    - 5.1-K2: The "lost in the middle" effect: models reliably process information at the beginning and end of long inputs but may omit findings from middle sections
    - 5.1-K3: How tool results accumulate in context and consume tokens disproportionately to their relevance (e.g., 40+ fields per order lookup when only 5 are relevant)
    - 5.1-K4: The importance of passing complete conversation history in subsequent API requests to maintain conversational coherence
  Skills in:
    - 5.1-S1: Extracting transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block included in each prompt, outside summarized history
    - 5.1-S2: Extracting and persisting structured issue data (order IDs, amounts, statuses) into a separate context layer for multi-issue sessions
    - 5.1-S3: Trimming verbose tool outputs to only relevant fields before they accumulate in context (e.g., keeping only return-relevant fields from order lookups)
    - 5.1-S4: Placing key findings summaries at the beginning of aggregated inputs and organizing detailed results with explicit section headers to mitigate position effects
    - 5.1-S5: Requiring subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis
    - 5.1-S6: Modifying upstream agents to return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when downstream agents have limited context budgets

- Task Statement 5.2: Design effective escalation and ambiguity resolution patterns
  Knowledge of:
    - 5.2-K1: Appropriate escalation triggers: customer requests for a human, policy exceptions/gaps (not just complex cases), and inability to make meaningful progress
    - 5.2-K2: The distinction between escalating immediately when a customer explicitly demands it versus offering to resolve when the issue is straightforward
    - 5.2-K3: Why sentiment-based escalation and self-reported confidence scores are unreliable proxies for actual case complexity
    - 5.2-K4: How multiple customer matches require clarification (requesting additional identifiers) rather than heuristic selection
  Skills in:
    - 5.2-S1: Adding explicit escalation criteria with few-shot examples to the system prompt demonstrating when to escalate versus resolve autonomously
    - 5.2-S2: Honoring explicit customer requests for human agents immediately without first attempting investigation
    - 5.2-S3: Acknowledging frustration while offering resolution when the issue is within the agent's capability, escalating only if the customer reiterates their preference
    - 5.2-S4: Escalating when policy is ambiguous or silent on the customer's specific request (e.g., competitor price matching when policy only addresses own-site adjustments)
    - 5.2-S5: Instructing the agent to ask for additional identifiers when tool results return multiple matches, rather than selecting based on heuristics

- Task Statement 5.3: Implement error propagation strategies across multi-agent systems
  Knowledge of:
    - 5.3-K1: Structured error context (failure type, attempted query, partial results, alternative approaches) as enabling intelligent coordinator recovery decisions
    - 5.3-K2: The distinction between access failures (timeouts needing retry decisions) and valid empty results (successful queries with no matches)
    - 5.3-K3: Why generic error statuses ("search unavailable") hide valuable context from the coordinator
    - 5.3-K4: Why silently suppressing errors (returning empty results as success) or terminating entire workflows on single failures are both anti-patterns
  Skills in:
    - 5.3-S1: Returning structured error context including failure type, what was attempted, partial results, and potential alternatives to enable coordinator recovery
    - 5.3-S2: Distinguishing access failures from valid empty results in error reporting so the coordinator can make appropriate decisions
    - 5.3-S3: Having subagents implement local recovery for transient failures and only propagate errors they cannot resolve, including what was attempted and partial results
    - 5.3-S4: Structuring synthesis output with coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources

- Task Statement 5.4: Manage context effectively in large codebase exploration
  Knowledge of:
    - 5.4-K1: Context degradation in extended sessions: models start giving inconsistent answers and referencing "typical patterns" rather than specific classes discovered earlier
    - 5.4-K2: The role of scratchpad files for persisting key findings across context boundaries
    - 5.4-K3: Subagent delegation for isolating verbose exploration output while the main agent coordinates high-level understanding
    - 5.4-K4: Structured state persistence for crash recovery: each agent exports state to a known location, and the coordinator loads a manifest on resume
  Skills in:
    - 5.4-S1: Spawning subagents to investigate specific questions (e.g., "find all test files," "trace refund flow dependencies") while the main agent preserves high-level coordination
    - 5.4-S2: Having agents maintain scratchpad files recording key findings, referencing them for subsequent questions to counteract context degradation
    - 5.4-S3: Summarizing key findings from one exploration phase before spawning sub-agents for the next phase, injecting summaries into initial context
    - 5.4-S4: Designing crash recovery using structured agent state exports (manifests) that the coordinator loads on resume and injects into agent prompts
    - 5.4-S5: Using /compact to reduce context usage during extended exploration sessions when context fills with verbose discovery output

- Task Statement 5.5: Design human review workflows and confidence calibration
  Knowledge of:
    - 5.5-K1: The risk that aggregate accuracy metrics (e.g., 97% overall) may mask poor performance on specific document types or fields
    - 5.5-K2: Stratified random sampling for measuring error rates in high-confidence extractions and detecting novel error patterns
    - 5.5-K3: Field-level confidence scores calibrated using labeled validation sets for routing review attention
    - 5.5-K4: The importance of validating accuracy by document type and field segment before automating high-confidence extractions
  Skills in:
    - 5.5-S1: Implementing stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection
    - 5.5-S2: Analyzing accuracy by document type and field to verify consistent performance across all segments before reducing human review
    - 5.5-S3: Having models output field-level confidence scores, then calibrating review thresholds using labeled validation sets
    - 5.5-S4: Routing extractions with low model confidence or ambiguous/contradictory source documents to human review, prioritizing limited reviewer capacity

- Task Statement 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis
  Knowledge of:
    - 5.6-K1: How source attribution is lost during summarization steps when findings are compressed without preserving claim-source mappings
    - 5.6-K2: The importance of structured claim-source mappings that the synthesis agent must preserve and merge when combining findings
    - 5.6-K3: How to handle conflicting statistics from credible sources: annotating conflicts with source attribution rather than arbitrarily selecting one value
    - 5.6-K4: Temporal data: requiring publication/collection dates in structured outputs to prevent temporal differences from being misinterpreted as contradictions
  Skills in:
    - 5.6-S1: Requiring subagents to output structured claim-source mappings (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis
    - 5.6-S2: Structuring reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context
    - 5.6-S3: Completing document analysis with conflicting values included and explicitly annotated, letting the coordinator decide how to reconcile before passing to synthesis
    - 5.6-S4: Requiring subagents to include publication or data collection dates in structured outputs to enable correct temporal interpretation
    - 5.6-S5: Rendering different content types appropriately in synthesis outputs—financial data as tables, news as prose, technical findings as structured lists—rather than converting everything to a uniform format

---

## Summary counts

| Domain | Weight | Task statements | Knowledge bullets | Skills bullets | Total bullets |
|---|---|---|---|---|---|
| 1. Agentic Architecture & Orchestration | 27% | 7 | 24 | 24 | 48 |
| 2. Tool Design & MCP Integration | 18% | 5 | 20 | 23 | 43 |
| 3. Claude Code Configuration & Workflows | 20% | 6 | 23 | 26 | 49 |
| 4. Prompt Engineering & Structured Output | 20% | 6 | 22 | 25 | 47 |
| 5. Context Management & Reliability | 15% | 6 | 24 | 29 | 53 |
| **Total** | **100%** | **30** | **113** | **127** | **240** |

Note: the table's "task statements" column sums to 7+5+6+6+6 = 30, matching the 30 task statements
(1.1–1.7, 2.1–2.5, 3.1–3.6, 4.1–4.6, 5.1–5.6) enumerated above. The Knowledge/Skills bullet counts in
this table were recomputed by counting the `-K<n>` and `-S<n>` IDs actually present under each task
statement above (not carried over from an earlier, miscalculated draft): Domain 1 = 24 Knowledge / 24
Skills, Domain 2 = 20/23, Domain 3 = 23/26, Domain 4 = 22/25, Domain 5 = 24/29, for a grand total of
113 Knowledge bullets, 127 Skills bullets, and 240 bullets overall — consistent with the per-domain
"Bullets captured" line stated under each domain heading above.
