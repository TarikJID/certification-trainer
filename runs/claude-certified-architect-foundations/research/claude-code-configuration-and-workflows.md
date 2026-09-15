Domain: Claude Code Configuration & Workflows (20% of the CCAR-F exam)

Description: Focuses on configuring CLAUDE.md file hierarchies, creating custom slash commands and skills, applying path-specific rules for conditional loading, determining when to use plan mode versus direct execution, applying iterative refinement techniques, and integrating Claude Code into CI/CD pipelines.

---

- Concept: Context window and fresh session start
  Type: prerequisite
  Definition: Claude Code's context window is the finite token budget holding the entire conversation, including messages, files read, and command output, for a single session. Each Claude Code session begins with a fresh (empty) context window — nothing carries over automatically between sessions. This constraint is the reason two mechanisms (CLAUDE.md files and auto memory) exist to carry knowledge across sessions, and is the reason instruction files must be concise: longer files consume more context and can reduce how reliably Claude follows them.
  Example: A developer runs `claude` in a project directory. The session starts with an empty context window; Claude Code then loads CLAUDE.md files and settings from the directory hierarchy into that window before the developer's first message is processed.
  Source: https://code.claude.com/docs/en/memory

- Concept: CLAUDE.md file hierarchy and memory system
  Type: key
  Definition: CLAUDE.md files are markdown files that give Claude persistent, human-written instructions (coding standards, architecture, workflows). They can live at several scopes, loaded broadest-to-narrowest: a managed policy file (e.g. `/etc/claude-code/CLAUDE.md`, org-wide, deployed by IT), user instructions (`~/.claude/CLAUDE.md`, personal, all projects), project instructions (`./CLAUDE.md` or `./.claude/CLAUDE.md`, team-shared via source control), and local instructions (`./CLAUDE.local.md`, personal and gitignored). Claude Code loads CLAUDE.md and CLAUDE.local.md from the current working directory and every directory above it at launch; files in subdirectories load on demand when Claude reads files in those subdirectories. All discovered files are concatenated (not overridden), ordered from filesystem root down to the working directory, with CLAUDE.local.md appended after CLAUDE.md at each level. Files can `@path/to/file` import other files (max depth 4), and content is delivered as a user message after the system prompt (advisory, not enforced — a hard block requires a PreToolUse hook instead). A complementary system, "auto memory," lets Claude write its own notes (user, feedback, project, reference) to `~/.claude/projects/<project>/memory/`, loaded via a size-capped `MEMORY.md` index (first 200 lines or 25KB) at every session start.
  Example: Running Claude Code inside `foo/bar/` loads `foo/CLAUDE.md` first, then `foo/bar/CLAUDE.md`, so instructions closer to the working directory are read last (higher effective priority in context). Running `/init` inspects the codebase and auto-generates a starter project CLAUDE.md with build commands and conventions.
  Source: https://code.claude.com/docs/en/memory

- Concept: Settings files and precedence
  Type: prerequisite
  Definition: Claude Code configuration (permissions, environment variables, hooks, model defaults, etc.) is stored in `settings.json` files at multiple scopes, resolved with a fixed precedence from highest to lowest: (1) Managed settings (`managed-settings.json`, MDM, or the console — organization-controlled), (2) command line (`claude --settings`), (3) project local (`.claude/settings.local.json` — personal, gitignored), (4) shared project (`.claude/settings.json` — committed, team-shared), and (5) user (`~/.claude/settings.json` — personal, all projects). A value set at a higher-precedence scope overrides the same key set at a lower one; array-type keys such as `claudeMdExcludes` merge across layers rather than overriding.
  Example: A shared project `.claude/settings.json` sets a permission default, but an individual developer overrides it locally in `.claude/settings.local.json`; that local override wins for their machine because project-local settings outrank shared-project settings.
  Source: https://code.claude.com/docs/en/settings

- Concept: Path-specific rules for conditional loading (`.claude/rules/`)
  Type: key
  Definition: For larger projects, instructions can be organized into modular files under `.claude/rules/` instead of one large CLAUDE.md, with each `.md` file covering one topic (discovered recursively, including subdirectories). Rules with no `paths` frontmatter load unconditionally at launch, at the same priority as `.claude/CLAUDE.md`. Rules can instead carry YAML frontmatter with a `paths` field (glob patterns, supporting brace expansion such as `src/**/*.{ts,tsx}`) that scopes them to specific files; these conditional rules load into context only when Claude reads a file matching the pattern, reducing noise and context cost versus loading everything up front. User-level rules in `~/.claude/rules/` apply to every project on the machine and load before project rules. Rules directories also support symlinks to share a common rule set across projects, and `claudeMdExcludes` (a settings key) lets a project exclude specific ancestor CLAUDE.md/rules files that aren't relevant, e.g. in a monorepo.
  Example: A rule file `.claude/rules/api-design.md` with frontmatter `paths: ["src/api/**/*.ts"]` and body "All API endpoints must include input validation" only enters Claude's context when Claude opens a file under `src/api/`, not for unrelated frontend work.
  Source: https://code.claude.com/docs/en/memory

- Concept: Agentic tool loop (Bash, Read, Edit, and dynamic context injection)
  Type: prerequisite
  Definition: Claude Code operates as an agentic loop that can read files, run shell commands, and edit files using a fixed set of built-in tools, deciding for itself which tools to call to accomplish a task rather than following a rigid script. Custom extensions (skills, slash commands, subagents) are built on top of this loop: their instructions are handed to Claude as a prompt, and Claude orchestrates the underlying tools to carry them out. Skill/command files can also inject dynamic context by running a shell command before Claude ever sees the file, using the `` !`command` `` syntax, so the output of that command becomes part of the loaded instructions.
  Example: A skill file contains `!`git diff HEAD`` under a "Current changes" heading; before Claude reads the skill, Claude Code runs `git diff HEAD` and substitutes its output into that section of the prompt.
  Source: https://code.claude.com/docs/en/skills

- Concept: Custom slash commands
  Type: key
  Definition: Slash commands are commands beginning with `/` that control a Claude Code session; recognized only when they appear at the start of a message, with any trailing text passed as arguments. Claude Code ships many built-in commands (e.g. `/compact`, `/clear`, `/context`, `/plan`, `/model`, `/resume`). Custom slash commands were historically defined as standalone markdown files (in `.claude/commands/`), but as of current Claude Code documentation, custom slash commands have been merged into the skills system: a skill's SKILL.md file is invoked the same way (`/skill-name [arguments]`), so creating a "custom slash command" today means authoring a skill.
  Example: Typing `/fix-issue 123` invokes a skill/command named `fix-issue`, passing `123` as its argument, which the skill file receives via `$ARGUMENTS` or a named argument.
  Source: https://code.claude.com/docs/en/skills

- Concept: Agent Skills (SKILL.md)
  Type: key
  Definition: A skill extends Claude's capabilities by bundling instructions, reference material, and supporting files in a directory containing a `SKILL.md` file with YAML frontmatter plus markdown instructions. Unlike CLAUDE.md (always loaded every session), a skill loads only on demand — either invoked manually by typing `/skill-name`, or automatically when Claude judges the user's request matches the skill's `description` field. Key frontmatter fields include `name`, `description` (drives auto-invocation), `disable-model-invocation` (only a human can invoke it), `user-invocable` (hidden from the `/` menu, only Claude can invoke it), `allowed-tools`/`disallowed-tools` (scope which tools the skill may use), `context: fork` (run in an isolated subagent), `arguments`/`argument-hint` (named parameters), `paths` (glob patterns limiting auto-activation), and `model`/`effort` (override session defaults). Skills can live personally (`~/.claude/skills/`), per-project (`.claude/skills/`, committed to share with a team), in an enterprise managed location, or bundled in a plugin (invoked as `/plugin-name:skill-name`).
  Example: A project-level skill at `.claude/skills/api-conventions/SKILL.md` with `description: REST API design conventions for our services` auto-loads into context whenever Claude works on API code that matches its description, without consuming context on unrelated turns.
  Source: https://code.claude.com/docs/en/skills

- Concept: Permission modes overview (Manual, Auto, acceptEdits, dontAsk)
  Type: prerequisite
  Definition: A permission mode sets which actions Claude can take in a session without asking the user first. In Manual mode (`default`), Claude Code stops and asks before most actions that edit files, run shell commands, or reach the network. In Auto mode (the built-in starting mode on Pro/Max/Team plans for interactive sessions), a separate classifier model reviews most actions instead of the user, blocking only actions that look risky (e.g. scope escalation, unknown infrastructure). `acceptEdits` auto-approves file writes and common filesystem commands. `dontAsk` denies every call that would otherwise prompt, useful for locked-down CI runs. Modes can be switched at any time (e.g. with Shift+Tab in the CLI) and are the base permission layer that plan mode is built on top of.
  Example: Running `claude --permission-mode acceptEdits` lets Claude apply lint fixes and write files without prompting for each edit, while still asking before running an un-approved shell command outside the pre-approved set.
  Source: https://code.claude.com/docs/en/permission-modes

- Concept: Plan mode versus direct execution
  Type: key
  Definition: Plan mode is a permission mode in which Claude Code's write and execute tools are disabled: Claude can read files, search the codebase, inspect git history, and run read-only commands to understand a project, then produces a step-by-step implementation plan for review — but cannot modify any files or state until the plan is approved. It is activated interactively by pressing Shift+Tab until the status bar shows plan mode, by prefixing a prompt with `/plan`, or by starting a session with `claude --permission-mode plan`. Anthropic's recommended workflow is a four-phase cycle: Explore (read-only, in plan mode) → Plan (ask Claude to produce a detailed plan, editable in an external editor with Ctrl+G) → Implement (exit plan mode by approving the plan, then let Claude write code and verify against the plan) → Commit. Plan mode trades speed for safety and should be reserved for higher-risk or higher-uncertainty changes — direct execution (skipping plan mode) is preferred when the scope is clear and the change is small, since planning adds overhead that isn't justified for a change you could describe in one sentence.
  Example: Before implementing a Google OAuth login flow that touches multiple files, a developer enters plan mode, asks Claude to read the existing session/auth code, then requests "Create a plan," reviews and edits the plan, and only then exits plan mode to let Claude implement it — versus fixing a single typo, which is done with a direct prompt and no plan.
  Source: https://code.claude.com/docs/en/best-practices

- Concept: Checkpointing and rewind
  Type: prerequisite
  Definition: Claude Code automatically snapshots files before each change; every prompt that starts a turn creates a checkpoint that can later be restored. Pressing Escape twice, or running `/rewind`, opens a rewind menu that can restore the conversation only, the code only, or both, back to any earlier checkpoint, or summarize the conversation from a selected message forward or backward. Checkpoints only track changes made through Claude's own file-editing tools — not changes made via Bash commands or external processes — so they are not a replacement for version control (git).
  Example: Claude attempts a risky refactor that breaks the build; the developer runs `/rewind`, restores the code state to the checkpoint before the refactor, and tries a different approach instead of manually undoing the changes.
  Source: https://code.claude.com/docs/en/best-practices

- Concept: Iterative refinement techniques
  Type: key
  Definition: Because Claude stops once work "looks done," effective use of Claude Code depends on closing a verification loop and course-correcting quickly rather than only reviewing after the fact. Documented techniques include: (1) giving Claude a way to verify its own work — a test suite, build, linter, or screenshot comparison that returns a pass/fail signal Claude can read and iterate against, gated at increasing levels of rigor via a single prompt, a `/goal` condition, a deterministic `Stop` hook, or a second-opinion verification subagent; (2) course-correcting early — using `Esc` to interrupt Claude mid-action without losing context, `Esc+Esc`/`/rewind` to restore an earlier checkpoint, or telling Claude to "undo that"; (3) clearing polluted context with `/clear` after two failed correction attempts on the same issue, rather than continuing to accumulate failed approaches; (4) using `/compact` (optionally with instructions) to condense a long conversation while preserving key decisions; and (5) adding an adversarial review step — having a fresh subagent review a diff against stated criteria in an isolated context, so the reviewer isn't biased by the reasoning that produced the change.
  Example: A developer asks Claude to "write a validateEmail function; example test cases: user@example.com is true, invalid is false, user@.com is false; run the tests after implementing," giving Claude a concrete verification signal, then — after Claude's second unsuccessful attempt to fix a lint failure — runs `/clear` and restarts with a more specific prompt incorporating what was learned, instead of continuing to correct in place.
  Source: https://code.claude.com/docs/en/best-practices

- Concept: Headless mode / Agent SDK CLI (`claude -p`)
  Type: prerequisite
  Definition: Adding the `-p` (or `--print`) flag to any `claude` command runs it non-interactively: Claude Code takes a prompt, runs it to completion without a live conversation, prints the result, and exits, making it a scriptable tool usable in bash pipelines, git hooks, cron jobs, and CI systems. Output can be requested as plain `text` (default), structured `json` (includes `result`, `session_id`, cost metadata, and can be validated against a `--json-schema`), or `stream-json` (newline-delimited events for real-time streaming). Non-interactive runs can auto-approve tools with `--allowedTools`, set a baseline via `--permission-mode`, and suppress prompts entirely for unattended jobs with `--permission-prompts none`. The `--bare` flag skips auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP servers, auto memory, and CLAUDE.md for faster, more deterministic startup — recommended for CI and scripted/SDK calls.
  Example: `cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt` pipes a build log into Claude non-interactively and writes Claude's explanation to a file, with no interactive session required — a pattern directly usable inside a CI job step.
  Source: https://code.claude.com/docs/en/headless

- Concept: CI/CD integration via Claude Code GitHub Actions
  Type: key
  Definition: Claude Code GitHub Actions (`anthropics/claude-code-action`) is a GitHub Action that runs the full Claude Code runtime inside a GitHub Actions workflow, built on top of the Agent SDK. It supports two modes: interactive mode, where Claude waits for a trigger phrase (`@claude` by default) in an issue/PR comment or newly opened issue and then responds, posting progress as a comment; and automation mode, where the workflow supplies a `prompt` input and Claude runs on any GitHub event (e.g. a `schedule` cron trigger) without waiting for a mention, with results by default written to the workflow run log. Setup requires installing the Claude GitHub App (granting Contents, Issues, and Pull Requests read/write) and storing an `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN` as a repository secret; Claude Code's `/install-github-app` command automates this. The action enforces two checks before running: the triggering actor must have write access to the repository, and bot actors are rejected unless explicitly allowed. `claude_args` passes standard Claude Code CLI flags (e.g. `--max-turns`, `--model`, `--allowedTools`) into the workflow run, and a project's CLAUDE.md is read the same as in any other Claude Code session, so it should define code style and review criteria the action should follow.
  Example: A workflow triggers on `pull_request` events and runs `anthropics/claude-code-action@v1` with `prompt: "/code-review:code-review --comment ..."`, so every opened or updated pull request automatically gets an inline code review comment from Claude, without a human needing to type `@claude`.
  Source: https://code.claude.com/docs/en/github-actions

---

Sources cited:
- https://code.claude.com/docs/en/memory
- https://code.claude.com/docs/en/settings
- https://code.claude.com/docs/en/skills
- https://code.claude.com/docs/en/permission-modes
- https://code.claude.com/docs/en/best-practices
- https://code.claude.com/docs/en/headless
- https://code.claude.com/docs/en/github-actions
