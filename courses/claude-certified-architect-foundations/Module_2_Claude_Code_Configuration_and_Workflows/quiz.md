# Module 2 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 2.1 — Sessions, Settings, and Memory

1. Does a new Claude Code session automatically remember what happened in a
   previous session? What two mechanisms exist to carry knowledge across
   sessions anyway?
2. Rank these settings.json scopes from highest to lowest precedence:
   shared project, user, managed, project local, command line.
3. When both `foo/CLAUDE.md` and `foo/bar/CLAUDE.md` are loaded, are they
   merged, or does the more specific one replace the less specific one?
4. What command should you run to check what memory files actually loaded
   into the *current* session, as opposed to what exists on disk?
5. What is the purpose of a `paths` frontmatter field on a file in
   `.claude/rules/`?

### Answers
1. No — each session starts with an empty context window. CLAUDE.md files
   and "auto memory" (Claude's own persisted notes) are the two mechanisms
   that carry knowledge across sessions.
2. Managed > command line > project local > shared project > user.
3. Merged — all discovered CLAUDE.md/CLAUDE.local.md files are concatenated,
   not overridden, ordered from filesystem root down to the working
   directory.
4. `/context` (note: `/memory` lists every location Claude Code recognizes
   on disk, not just what loaded into the current session).
5. It scopes the rule to load into context only when Claude reads a file
   matching the glob pattern, instead of loading unconditionally at launch —
   reducing context noise and cost.

---

## Quiz 2.2 — Skills, Commands, and the Agentic Tool Loop

1. What syntax lets a skill or command file inject the output of a shell
   command into the prompt before Claude sees it?
2. Today, what underlying mechanism implements a "custom slash command"?
3. Name three SKILL.md frontmatter fields and what each controls.
4. What distinguishes a skill from a CLAUDE.md file in terms of when it
   loads into context?

### Answers
1. `` !`command` `` — Claude Code runs the command first and substitutes its
   output into that part of the prompt.
2. A skill (a `SKILL.md` file); custom slash commands have been merged into
   the skills system, invoked the same way (`/skill-name [arguments]`).
3. Any three of: `description` (drives auto-invocation), `allowed-tools` /
   `disallowed-tools` (scopes which tools the skill may use), `context:
   fork` (runs the skill in an isolated subagent), `agent` (which subagent
   type runs the fork), `disable-model-invocation` (human-only invocation),
   `user-invocable` (hidden from the `/` menu).
4. CLAUDE.md is always loaded every session; a skill loads only on demand —
   manually via `/skill-name`, or automatically when Claude judges the
   request matches its `description`.

---

## Quiz 2.3 — Permission Modes, Subagents, and Plan Mode

1. What is the key difference between Manual permission mode and Auto
   permission mode?
2. What does a subagent NOT receive from its parent conversation, by
   default?
3. Why does the `Explore` subagent skip loading CLAUDE.md and git status?
4. What are the four phases of Anthropic's recommended plan-mode workflow?
5. What is NOT protected by Claude Code's checkpoint/rewind system?

### Answers
1. In Manual mode, the user is asked before most file/shell/network
   actions; in Auto mode, a separate classifier model reviews most actions
   instead of the user, blocking only actions that look risky.
2. The main conversation's history — a non-fork subagent starts with a
   fresh context window and does not see prior tool results or the parent's
   full conversation.
3. To keep its own context small, since it's optimized for fast, read-only,
   verbose exploration that shouldn't be weighed down by extra context that
   isn't needed for that task.
4. Explore (read-only) → Plan (produce and review a detailed plan) →
   Implement (approve the plan, write code, verify against it) → Commit.
5. Changes made via Bash commands or other external processes — checkpoints
   only track changes made through Claude's own file-editing tools, so they
   are not a replacement for version control.

---

## Quiz 2.4 — Iterative Refinement and Effective Workflows

1. Name two of the five documented iterative-refinement techniques.
2. Why does rewriting "implement a function that validates email addresses"
   with explicit test cases make it a better prompt?
3. In the interview pattern, why is a fresh session recommended for
   implementation after the spec is written, rather than continuing the
   interview session?
4. When should two issues be given to Claude in a single message versus
   handled sequentially? What is the source of this specific guidance, and
   what caveat applies to it?
5. Why does the documentation recommend reviewing a diff from a fresh
   subagent rather than the same session that wrote the code?

### Answers
1. Any two of: giving Claude a verifiable pass/fail signal; course-correcting
   early with Esc/rewind; clearing polluted context with `/clear` after
   repeated failed corrections; using `/compact` to condense a long
   conversation; adding an adversarial review step via a fresh subagent.
2. Because it turns an open-ended notion of "correct" into an unambiguous,
   checkable specification Claude can run tests against directly, rather
   than something it can only judge as "looks done."
3. So implementation proceeds with clean context focused entirely on the
   work, using the self-contained spec as the reference instead of the full
   (and now irrelevant) interview transcript.
4. Interacting issues (where fixing one affects the other) should go in one
   detailed message; independent issues should be handled sequentially, one
   at a time. This guidance is sourced to the exam guide's Task Statement
   3.5, not to Claude Code product documentation — no vendor documentation
   page covering this exact pattern was found.
5. Because the implementing session's context already contains the
   reasoning and assumptions that produced the change, so it evaluates the
   diff already committed to it rather than from a neutral standpoint; a
   fresh subagent sees only the diff and the stated criteria.

---

## Quiz 2.5 — Headless Mode and CI/CD Integration

1. What flag runs Claude Code non-interactively, and name one context where
   this is useful?
2. What does the `--bare` flag skip, and why is it recommended for CI?
3. What two checks does Claude Code GitHub Actions enforce before running?
4. What is the difference between interactive mode and automation mode in
   Claude Code GitHub Actions?

### Answers
1. `-p` (or `--print`); useful in bash pipelines, git hooks, cron jobs, or
   CI systems where no live conversation is wanted.
2. Auto-discovery of hooks, skills, custom commands, subagents, plugins,
   MCP servers, auto memory, and CLAUDE.md — skipping this gives faster,
   more deterministic startup, which is desirable for CI and scripted/SDK
   calls.
3. That the triggering actor has write access to the repository, and that
   bot actors are rejected unless explicitly allowed.
4. Interactive mode waits for a trigger phrase (`@claude` by default) in a
   comment or issue and responds; automation mode runs on any GitHub event
   (e.g. a cron schedule) using a workflow-supplied `prompt`, without
   waiting for a mention.
