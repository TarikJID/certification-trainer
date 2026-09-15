# Module 2: Claude Code Configuration & Workflows

Domain weight on the exam: 20%. This module maps to the Claude Code
Configuration & Workflows domain, plus one prerequisite concept ("Claude Code
subagents") imported early from the Tool Design & MCP domain because Lesson
2.3 needs it before Module 3 does.

Prerequisites from Module 1 used throughout: the context window, the
agentic-loop pattern, and tool use.

---

## Lesson 2.1: Sessions, Settings, and Memory

### Concept: Context window and fresh session start
Claude Code's context window is the finite token budget holding the entire
conversation for a single session — messages, files read, and command
output (an application of Module 1's general Context Window concept to the
Claude Code CLI specifically). Each Claude Code session begins with a fresh,
empty context window: nothing carries over automatically between sessions.
This constraint is exactly why CLAUDE.md files and auto memory (below) exist
— to carry knowledge across sessions deliberately — and why instruction
files must stay concise: longer files consume more context and can reduce
how reliably Claude follows them.

**Example:** A developer runs `claude` in a project directory. The session
starts with an empty context window; Claude Code then loads CLAUDE.md files
and settings from the directory hierarchy into that window before the
developer's first message is processed.

**Source:** Claude Code Configuration & Workflows research — Context window
and fresh session start (prerequisite), https://code.claude.com/docs/en/memory

### Concept: Settings files and precedence
Claude Code configuration (permissions, environment variables, hooks, model
defaults) lives in `settings.json` files at multiple scopes, resolved with a
fixed precedence from highest to lowest: (1) Managed settings
(`managed-settings.json`, organization-controlled), (2) command line
(`claude --settings`), (3) project local (`.claude/settings.local.json` —
personal, gitignored), (4) shared project (`.claude/settings.json` —
committed, team-shared), and (5) user (`~/.claude/settings.json` — personal,
all projects). A value at a higher-precedence scope overrides the same key
at a lower one; array-type keys such as `claudeMdExcludes` merge across
layers instead of overriding. This scope/precedence system is the same
mechanism Module 3 builds on for permission rules on specific tools.

**Example:** A shared project `.claude/settings.json` sets a permission
default, but an individual developer overrides it locally in
`.claude/settings.local.json`; the local override wins on their machine
because project-local settings outrank shared-project settings.

**Source:** Claude Code Configuration & Workflows research — Settings files
and precedence (prerequisite), https://code.claude.com/docs/en/settings

### Concept: CLAUDE.md file hierarchy and memory system
CLAUDE.md files are markdown files giving Claude persistent, human-written
instructions (coding standards, architecture, workflows). They exist at
several scopes, loaded broadest-to-narrowest: a managed policy file
(org-wide), user instructions (`~/.claude/CLAUDE.md`), project instructions
(`./CLAUDE.md` or `./.claude/CLAUDE.md`, team-shared via source control), and
local instructions (`./CLAUDE.local.md`, personal and gitignored). Claude
Code loads CLAUDE.md and CLAUDE.local.md from the working directory and
every directory above it at launch; subdirectory files load on demand. All
discovered files are concatenated (not overridden), root-to-working-directory
order, with CLAUDE.local.md appended after CLAUDE.md at each level. Files
can `@path/to/file` import other files (max depth 4), and content is
delivered as a user message after the system prompt — advisory, not
enforced (a hard block needs a `PreToolUse` hook, covered in Module 4). A
complementary system, "auto memory," lets Claude write its own notes to
`~/.claude/projects/<project>/memory/`, loaded via a size-capped
`MEMORY.md` index (first 200 lines or 25KB) at every session start.

**Example:** Running Claude Code inside `foo/bar/` loads `foo/CLAUDE.md`
first, then `foo/bar/CLAUDE.md` — instructions closer to the working
directory are read last (higher effective priority in context). Running
`/init` auto-generates a starter project CLAUDE.md with build commands and
conventions.

**Source:** Claude Code Configuration & Workflows research — CLAUDE.md file
hierarchy and memory system (key), https://code.claude.com/docs/en/memory

### Concept: The /memory command
`/memory` is a built-in command that lists every CLAUDE.md, CLAUDE.local.md,
and other memory-file location across user and project scopes — including
entries for files that don't exist yet. It also toggles auto memory on/off
and opens the auto memory folder. Selecting a listed file opens it in your
editor (creating it first if needed). `/memory` is the tool for diagnosing
why loaded instructions appear inconsistent across sessions or machines,
because it surfaces every location Claude Code recognizes, not just what
happened to load into the current session — to check what actually loaded
into the *current* session, use `/context` instead.

**Example:** A developer notices Claude follows a convention on one machine
but not another. Running `/memory` on each machine reveals the convention
lives only in a `CLAUDE.local.md` present on the first machine and absent
(gitignored, per-machine) on the second.

**Source:** Claude Code Configuration & Workflows research — The /memory
command (key), https://code.claude.com/docs/en/memory

### Concept: Path-specific rules for conditional loading (`.claude/rules/`)
For larger projects, instructions can be organized into modular files under
`.claude/rules/` instead of one large CLAUDE.md, each `.md` file covering
one topic (discovered recursively). Rules with no `paths` frontmatter load
unconditionally at launch, at the same priority as `.claude/CLAUDE.md`.
Rules with YAML frontmatter carrying a `paths` field (glob patterns,
supporting brace expansion like `src/**/*.{ts,tsx}`) load into context only
when Claude reads a file matching that pattern — reducing context noise and
cost versus loading everything up front. User-level rules in
`~/.claude/rules/` apply to every project on the machine and load before
project rules. Rules directories support symlinks to share a rule set across
projects, and `claudeMdExcludes` (a settings key) lets a project exclude
specific ancestor CLAUDE.md/rules files, e.g. in a monorepo.

**Example:** A rule file `.claude/rules/api-design.md` with frontmatter
`paths: ["src/api/**/*.ts"]` and body "All API endpoints must include input
validation" only enters context when Claude opens a file under `src/api/`,
not for unrelated frontend work.

**Source:** Claude Code Configuration & Workflows research — Path-specific
rules for conditional loading (key), https://code.claude.com/docs/en/memory

---

## Lesson 2.2: Skills, Commands, and the Agentic Tool Loop

### Concept: Agentic tool loop (Bash, Read, Edit, and dynamic context injection)
Claude Code operates as an agentic loop (the general pattern from Module 1)
that can read files, run shell commands, and edit files using a fixed set of
built-in tools, deciding for itself which tools to call rather than
following a rigid script. Custom extensions (skills, slash commands,
subagents) are built on top of this loop: their instructions are handed to
Claude as a prompt, and Claude orchestrates the underlying tools to carry
them out. Skill/command files can also inject dynamic context by running a
shell command before Claude ever sees the file, using `` !`command` ``
syntax, so the output becomes part of the loaded instructions.

**Example:** A skill file contains `` !`git diff HEAD` `` under a "Current
changes" heading; before Claude reads the skill, Claude Code runs `git diff
HEAD` and substitutes its output into that section of the prompt.

**Source:** Claude Code Configuration & Workflows research — Agentic tool
loop (prerequisite), https://code.claude.com/docs/en/skills

### Concept: Custom slash commands
Slash commands begin with `/` and control a Claude Code session — recognized
only at the start of a message, with any trailing text passed as arguments.
Claude Code ships many built-ins (`/compact`, `/clear`, `/context`, `/plan`,
`/model`, `/resume`). Custom slash commands were historically standalone
markdown files in `.claude/commands/`, but as of current documentation
they've been merged into the skills system: a skill's `SKILL.md` is invoked
the same way (`/skill-name [arguments]`), so authoring a "custom slash
command" today means authoring a skill.

**Example:** Typing `/fix-issue 123` invokes a skill/command named
`fix-issue`, passing `123` as its argument via `$ARGUMENTS` or a named
argument.

**Source:** Claude Code Configuration & Workflows research — Custom slash
commands (key), https://code.claude.com/docs/en/skills

### Concept: Agent Skills (SKILL.md)
A skill extends Claude's capabilities by bundling instructions, reference
material, and supporting files in a directory containing a `SKILL.md` file
with YAML frontmatter plus markdown instructions. Unlike CLAUDE.md (always
loaded every session), a skill loads only on demand — manually via
`/skill-name`, or automatically when Claude judges the request matches the
skill's `description` field. Key frontmatter fields: `name`, `description`
(drives auto-invocation), `disable-model-invocation` (human-only),
`user-invocable` (hidden from the `/` menu, Claude-only), `allowed-tools` /
`disallowed-tools`, `context: fork` (run in an isolated subagent), `agent`
(which subagent type runs the fork, e.g. `Explore` — see Lesson 2.3),
`arguments`/`argument-hint`, `paths` (glob patterns limiting
auto-activation), and `model`/`effort`. Skills can live personally
(`~/.claude/skills/`), per-project (`.claude/skills/`), in an enterprise
managed location, or bundled in a plugin.

**Example:** A project-level skill at `.claude/skills/api-conventions/SKILL.md`
with `description: REST API design conventions for our services` auto-loads
whenever Claude works on matching API code, without consuming context on
unrelated turns.

**Source:** Claude Code Configuration & Workflows research — Agent Skills
(key), https://code.claude.com/docs/en/skills

---

## Lesson 2.3: Permission Modes, Subagents, and Plan Mode

### Concept: Permission modes overview (Manual, Auto, acceptEdits, dontAsk)
A permission mode sets which actions Claude can take without asking first.
In Manual mode (`default`), Claude Code stops and asks before most
file/shell/network actions. In Auto mode (the built-in starting mode on
paid plans for interactive sessions), a separate classifier model reviews
most actions instead of the user, blocking only actions that look risky.
`acceptEdits` auto-approves file writes and common filesystem commands.
`dontAsk` denies every call that would otherwise prompt — useful for
locked-down CI runs. Modes can be switched at any time (e.g. Shift+Tab in
the CLI) and are the base layer that plan mode (below) and, later, the Agent
SDK's full permission-evaluation order (Module 4) build on.

**Example:** Running `claude --permission-mode acceptEdits` lets Claude
apply lint fixes and write files without prompting for each edit, while
still asking before an un-approved shell command outside the pre-approved
set.

**Source:** Claude Code Configuration & Workflows research — Permission
modes overview (prerequisite), https://code.claude.com/docs/en/permission-modes

### Concept: Claude Code subagents (general notion)
Subagents are specialized AI assistants, each defined with its own system
prompt, model, and permissions, that run in an isolated context window
separate from the main conversation. Claude automatically delegates matching
tasks to a subagent based on its description, and (for non-fork subagents)
the subagent does not see the main conversation's history — only its final
summary returns to the caller. Claude Code ships built-in subagents
(`Explore`: read-only codebase search — see next concept; `Plan`: read-only
research for plan mode; `general-purpose`: full tool access for complex
multi-step work) in addition to user-defined ones. This general notion is
deepened substantially in Module 4 (the full Coordinator–Subagent
orchestration pattern) and referenced again in Module 3 (distributing tools
across subagents) and Module 6 (context isolation for codebase exploration).

**Example:** Instead of running a large log-searching task in the main
conversation (flooding it with irrelevant output), Claude Code can delegate
it to a subagent that reads the logs in its own context and returns only a
short summary of findings.

**Source:** Tool Design & MCP Integration research — Claude Code subagents
(prerequisite; taught here, at its earliest point of need, ahead of that
module), https://code.claude.com/docs/en/sub-agents

### Concept: The Explore subagent
`Explore` is a built-in subagent type selectable via a skill's `agent`
frontmatter field combined with `context: fork`. It is optimized for
read-only codebase exploration and research: it runs in an isolated
subagent context with a narrower, read-focused tool set (e.g. Glob, Grep,
file reading), skips loading CLAUDE.md and git status to keep its own
context small, and performs verbose, multi-step discovery entirely within
that isolated context. Only a concise final summary returns to the main
conversation, so detailed exploration output never consumes the main
session's context window — this is the documented mechanism for isolating
verbose discovery work, whether invoked via a skill or by telling Claude
interactively to "use a subagent to investigate X."

**Example:** A skill defined with `context: fork` and `agent: Explore` is
invoked as `/deep-research authentication flow`; the Explore subagent uses
Glob and Grep to locate every file touching authentication, reads and
analyzes them, and returns only a summary with specific file references.

**Source:** Claude Code Configuration & Workflows research — The Explore
subagent (key), https://code.claude.com/docs/en/skills

### Concept: Plan mode versus direct execution
Plan mode is a permission mode in which Claude Code's write and execute
tools are disabled: Claude can read files, search the codebase, inspect git
history, and run read-only commands, then produces a step-by-step
implementation plan for review — but cannot modify anything until the plan
is approved. Activated interactively (Shift+Tab, `/plan`, or `claude
--permission-mode plan`). Anthropic's recommended workflow is a four-phase
cycle: Explore (read-only, in plan mode) → Plan (ask for a detailed plan,
editable externally) → Implement (approve the plan, let Claude write code
and verify against it) → Commit. Plan mode trades speed for safety and
should be reserved for higher-risk or higher-uncertainty changes — direct
execution is preferred when scope is clear and the change is small, since
planning overhead isn't justified for a one-sentence change.

**Example:** Before implementing a Google OAuth login flow touching multiple
files, a developer enters plan mode, has Claude read the existing
session/auth code, requests "Create a plan," reviews and edits it, then
exits plan mode to implement — versus fixing a single typo, done with a
direct prompt and no plan.

**Source:** Claude Code Configuration & Workflows research — Plan mode
versus direct execution (key), https://code.claude.com/docs/en/best-practices

### Concept: Checkpointing and rewind
Claude Code automatically snapshots files before each change; every prompt
that starts a turn creates a checkpoint that can later be restored. Pressing
Escape twice, or running `/rewind`, opens a rewind menu that can restore the
conversation only, the code only, or both, back to any earlier checkpoint,
or summarize the conversation from a selected message forward or backward.
Checkpoints only track changes made through Claude's own file-editing
tools — not changes via Bash commands or external processes — so they are
not a replacement for version control.

**Example:** Claude attempts a risky refactor that breaks the build; the
developer runs `/rewind`, restores the code state to before the refactor,
and tries a different approach instead of manually undoing the changes.

**Source:** Claude Code Configuration & Workflows research — Checkpointing
and rewind (prerequisite), https://code.claude.com/docs/en/best-practices

---

## Lesson 2.4: Iterative Refinement and Effective Workflows

### Concept: Iterative refinement techniques
Because Claude stops once work "looks done," effective use depends on
closing a verification loop and course-correcting quickly rather than only
reviewing after the fact. Documented techniques: (1) give Claude a way to
verify its own work — a test suite, build, linter, or screenshot comparison
that returns pass/fail, gated at increasing rigor via a prompt, a `/goal`
condition, a deterministic `Stop` hook, or a second-opinion verification
subagent; (2) course-correct early — `Esc` to interrupt mid-action without
losing context, `Esc+Esc`/`/rewind` to restore a checkpoint, or "undo that";
(3) clear polluted context with `/clear` after two failed correction
attempts on the same issue rather than accumulating failed approaches; (4)
use `/compact` (optionally with instructions) to condense a long
conversation while preserving key decisions; (5) add an adversarial review
step — a fresh subagent reviewing a diff against stated criteria in an
isolated context, unbiased by the reasoning that produced the change.

**Example:** A developer asks Claude to "write a validateEmail function;
example test cases: user@example.com is true, invalid is false,
user@.com is false; run the tests after implementing" — a concrete
verification signal — then, after Claude's second unsuccessful fix attempt
on a lint failure, runs `/clear` and restarts with a more specific prompt
rather than continuing to correct in place.

**Source:** Claude Code Configuration & Workflows research — Iterative
refinement techniques (key), https://code.claude.com/docs/en/best-practices

### Concept: Concrete input/output examples as verification criteria
One specific way to give Claude a checkable signal for its own work: replace
a prose description of desired behavior with concrete input/output
examples. "Implement a function that validates email addresses" leaves
"correct" open to interpretation; restating it with explicit example cases
turns it into an unambiguous, checkable specification Claude can run against
directly.

**Example:** Before: "implement a function that validates email addresses."
After: "write a validateEmail function. example test cases: user@example.com
is true, invalid is false, user@.com is false. run the tests after
implementing."

**Source:** Claude Code Configuration & Workflows research — Concrete
input/output examples as verification criteria (key), https://code.claude.com/docs/en/best-practices

### Concept: The interview pattern
For larger features, rather than describing the feature and letting Claude
implement immediately, the developer starts with a minimal prompt asking
Claude to interview them first, using the `AskUserQuestion` tool, before any
implementation begins. Claude asks about technical implementation, UI/UX,
edge cases, concerns, and tradeoffs, continuing until everything relevant is
covered, then writes a complete spec (e.g. `SPEC.md`). The recommended
follow-up is to start a fresh session to execute that spec, so
implementation proceeds with clean context. A self-contained spec — naming
files/interfaces, stating what's out of scope, ending with an end-to-end
verification step — pays off more than time spent watching implementation.

**Example:** "I want to build a real-time notification system. Interview me
in detail using the AskUserQuestion tool... Keep interviewing until we've
covered everything, then write a complete spec to SPEC.md." Claude asks
about delivery mechanism, read/unread state, retry behavior, and rate
limits before producing SPEC.md; only then does a fresh session begin
implementation from that spec.

**Source:** Claude Code Configuration & Workflows research — The interview
pattern (key), https://code.claude.com/docs/en/best-practices

### Concept: Single message for interacting issues versus sequential iteration for independent issues
> **Sourcing note:** this concept is sourced to the official CCAR-F Exam
> Guide (Task Statement 3.5) rather than to Claude Code product
> documentation. A thorough search of code.claude.com/docs and
> anthropic.com/engineering did not turn up a page covering this specific
> pattern. Treat it as an exam-guide fact rather than a documented product
> behavior.

When a developer has multiple issues for Claude to address, the deciding
factor for how to present them is whether the issues interact — i.e.,
whether fixing one touches the same code or affects behavior the other
depends on. If they interact, provide all of them together in one detailed
message. If they are independent, address them one at a time (sequential
iteration) rather than bundling unrelated work into a single request.

**Example:** A caching layer and the validation function it wraps both need
changes touching the same code path — these are interacting issues, so both
are described together in one message. An unrelated README typo does not
interact with that fix, so it is handled as its own, later, sequential
request.

**Source:** Claude Certified Architect – Foundations Exam Guide, Task
Statement 3.5 (exam-guide-only; not found in vendor documentation).

### Concept: Session context isolation in code review
The same Claude Code session that generated a piece of code is less
effective at reviewing it than an independent review instance, because the
implementing session's context contains the reasoning, assumptions, and
decisions that produced the change — it evaluates the diff while already
committed to it, not from a neutral standpoint. The documented fix: run the
review in a fresh subagent that receives only the diff and the review
criteria, not the implementing conversation's history. This is why CI/CD
review integrations (Lesson 2.5) are structured as a separate subagent
invocation over the diff.

**Example:** After Claude implements a rate limiter, instead of asking the
same session "is this correct?", the developer runs "Use a subagent to
review the rate limiter diff against PLAN.md... Report gaps, not style
preferences" — the reviewing subagent sees only the diff and PLAN.md, not
the implementation session's reasoning.

**Source:** Claude Code Configuration & Workflows research — Session context
isolation in code review (key), https://code.claude.com/docs/en/best-practices

---

## Lesson 2.5: Headless Mode and CI/CD Integration

### Concept: Headless mode / Agent SDK CLI (`claude -p`)
Adding `-p` (or `--print`) to any `claude` command runs it non-interactively:
Claude Code takes a prompt, runs it to completion without a live
conversation, prints the result, and exits — scriptable in bash pipelines,
git hooks, cron jobs, and CI systems. Output can be `text` (default), `json`
(includes `result`, `session_id`, cost metadata, and can be validated
against a `--json-schema`), or `stream-json` (newline-delimited streaming
events). Non-interactive runs can auto-approve tools with `--allowedTools`,
set a baseline via `--permission-mode`, and suppress prompts entirely for
unattended jobs with `--permission-prompts none`. The `--bare` flag skips
auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP
servers, auto memory, and CLAUDE.md for faster, deterministic startup —
recommended for CI and scripted/SDK calls.

**Example:** `cat build-error.txt | claude -p 'concisely explain the root
cause of this build error' > output.txt` pipes a build log into Claude
non-interactively and writes the explanation to a file — directly usable
inside a CI job step.

**Source:** Claude Code Configuration & Workflows research — Headless mode /
Agent SDK CLI (prerequisite), https://code.claude.com/docs/en/headless

### Concept: CI/CD integration via Claude Code GitHub Actions
Claude Code GitHub Actions (`anthropics/claude-code-action`) runs the full
Claude Code runtime inside a GitHub Actions workflow, built on the Agent
SDK. Two modes: interactive mode, where Claude waits for a trigger phrase
(`@claude` by default) in an issue/PR comment or newly opened issue and
posts progress as a comment; and automation mode, where the workflow
supplies a `prompt` input and Claude runs on any GitHub event (e.g. a
`schedule` cron trigger) without waiting for a mention. Setup requires
installing the Claude GitHub App (Contents, Issues, Pull Requests read/write)
and storing an `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN` as a
repository secret; `/install-github-app` automates this. The action enforces
two checks: the triggering actor must have write access, and bot actors are
rejected unless explicitly allowed. `claude_args` passes standard CLI flags
(`--max-turns`, `--model`, `--allowedTools`) into the workflow run, and a
project's CLAUDE.md is read the same as any other session.

**Example:** A workflow triggers on `pull_request` events and runs
`anthropics/claude-code-action@v1` with `prompt: "/code-review:code-review
--comment ..."`, so every opened or updated pull request automatically gets
an inline code review comment from Claude, without a human typing `@claude`.

**Source:** Claude Code Configuration & Workflows research — CI/CD
integration via Claude Code GitHub Actions (key), https://code.claude.com/docs/en/github-actions
