# Module 2 Hands-On Exercise

## Design a project's Claude Code configuration

You are setting up Claude Code for a mid-size monorepo with a frontend
(`web/`), a backend (`api/`), and a shared CI pipeline.

1. **Memory layout.** Decide what belongs in the committed `./CLAUDE.md`
   versus a personal `./CLAUDE.local.md`. Give two concrete examples of
   content for each, and justify the split using the fresh-session-start
   rationale from Lesson 2.1.

2. **Path-specific rules.** Write two `.claude/rules/` files (just the
   frontmatter + one instruction each) — one that should load
   unconditionally, and one that should load only when Claude touches files
   under `web/src/components/**/*.tsx`.

3. **A skill.** Design a skill named `db-migration` that should only be
   invoked explicitly by a human (never auto-triggered by Claude), and
   should run in an isolated subagent context. Specify the relevant
   frontmatter fields and explain why each setting is appropriate for a
   database migration task.

4. **Permission mode choice.** For each of the following, state which
   permission mode (Manual, Auto, acceptEdits, dontAsk, or plan mode) is
   most appropriate, and why:
   a. A developer doing exploratory research before a risky refactor.
   b. An unattended nightly CI job that must never modify files outside a
      designated output directory.
   c. A developer applying routine lint fixes across many files.

5. **CI wiring.** Sketch (in prose) a GitHub Actions workflow that runs
   Claude Code in automation mode on a `schedule` trigger to produce a daily
   dependency-update report, and separately explain what credential and App
   installation steps are required before this workflow can run.

### What a strong answer includes
- Correct attribution of which content is safe to gitignore (`.local.md`)
  versus which should be team-shared, tied to the concept that a session
  starts with a fresh context window.
- A `paths` frontmatter example using a valid glob, and a rule with no
  `paths` field loading unconditionally.
- A skill using `disable-model-invocation: true` (or equivalent) plus
  `context: fork`, with reasoning about why a migration tool should not be
  silently auto-triggered.
- Plan mode or Manual mode for (a), `dontAsk` for (b), `acceptEdits` for (c),
  each justified against Lesson 2.3's definitions.
- Mention of `-p`/automation mode, the `schedule` trigger, and the
  `ANTHROPIC_API_KEY`/`CLAUDE_CODE_OAUTH_TOKEN` secret plus GitHub App
  installation from Lesson 2.5.
