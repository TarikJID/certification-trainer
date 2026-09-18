# Module 4 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 3.1 (CLAUDE.md Hierarchy)

**Q1.** A new team member says "Claude never applies our lint standards." Where
should you check first, and why is this the most common cause of that complaint?

<details><summary>ANSWER</summary>
Check whether the instruction was written to `~/.claude/CLAUDE.md` (user-level)
instead of the project-level `./CLAUDE.md`. User-level settings apply only to the
person who wrote them and are never shared via version control, so a new team
member cloning the repo simply never receives them.
</details>

**Q2.** What does `@path/to/file` do inside a CLAUDE.md file, and relative to
what does the path resolve?

<details><summary>ANSWER</summary>
It imports another file's content into context at launch. It resolves relative
to the file containing the import (not the working directory), and can
recursively import further files up to a depth of four hops.
</details>

**Q3.** A 400-line CLAUDE.md covers testing, API conventions, and deployment.
What's the recommended restructuring, and what's the token-budget benefit?

<details><summary>ANSWER</summary>
Split it into topic-specific files in `.claude/rules/` (e.g. `testing.md`,
`api-conventions.md`, `deployment.md`). Rules without `paths` frontmatter still
load at launch like a CLAUDE.md, but the split makes the instructions modular and
easier to maintain (full token-savings benefit comes from adding path scoping —
covered in Lesson 3.3).
</details>

**Q4.** Which command lists every CLAUDE.md/memory-file location Claude Code
recognizes, and which command confirms what actually loaded in the current
session?

<details><summary>ANSWER</summary>
`/memory` lists locations (including files that don't yet exist) and lets you
toggle auto memory. `/context` shows which memory files actually loaded into the
current session.
</details>

---

## Quiz — Lesson 3.2 (Slash Commands and Skills)

**Q1.** What's the difference between a command/skill in `.claude/commands/` (or
`.claude/skills/`) versus `~/.claude/commands/` (or `~/.claude/skills/`)?

<details><summary>ANSWER</summary>
`.claude/commands/` and `.claude/skills/` are project-scoped: committed to
version control and shared with the whole team. `~/.claude/commands/` and
`~/.claude/skills/` are user-scoped: available across that user's own projects
but not shared with teammates.
</details>

**Q2.** What does `context: fork` do when added to a skill's frontmatter, and why
would you use it for a codebase-analysis skill?

<details><summary>ANSWER</summary>
It runs the skill in a forked subagent context instead of the main conversation;
only the subagent's summarized result returns to the main session. For a
codebase-analysis skill, this keeps the verbose intermediate output (file reads,
greps) out of the main conversation's token budget.
</details>

**Q3.** What does `allowed-tools` in skill frontmatter actually restrict, and
what does it NOT do?

<details><summary>ANSWER</summary>
It pre-approves a specific set of tools for the turn that invokes the skill, so
Claude can use exactly those without a permission prompt (the grant clears at the
end of that turn). It does NOT block other tools from being used — those still
follow normal permission settings.
</details>

**Q4.** An engineer wants a personal variant of the team's shared `/deploy`
skill without affecting teammates. What should they do?

<details><summary>ANSWER</summary>
Create a personal skill in `~/.claude/skills/<different-name>/SKILL.md` (a
different name than the team's shared skill, so it doesn't collide with or
override it). It lives outside the repository and is invisible to teammates.
</details>

**Q5.** A CLAUDE.md section has grown into a 40-step "migrate a service" procedure
needed only occasionally. Should it stay in CLAUDE.md? Why or why not?

<details><summary>ANSWER</summary>
No — it should move to a skill. CLAUDE.md loads into every session automatically
and costs tokens whether or not it's relevant to the current task; a multi-step
procedure needed only occasionally belongs in a skill, which loads only on
demand.
</details>

---

## Quiz — Lesson 3.3 (Path-Specific Rules)

**Q1.** What makes a rule file in `.claude/rules/` "conditional" rather than
always-loaded?

<details><summary>ANSWER</summary>
Having a `paths` field in its YAML frontmatter containing one or more glob
patterns. A rule with `paths` only applies (loads) when Claude works with files
matching one of those patterns; a rule without `paths` is unconditional and loads
at launch for every session.
</details>

**Q2.** Why does path-scoped loading save tokens compared to an unconditional
CLAUDE.md covering the same content?

<details><summary>ANSWER</summary>
A path-scoped rule only enters context when Claude reads a file matching its
pattern, so irrelevant conventions stay out of the context window on tasks that
never touch those files — unlike an unconditional CLAUDE.md, which is loaded
regardless of relevance on every session.
</details>

**Q3.** All `*.test.tsx` files across a codebase (in `src/`, `lib/`, and `tools/`)
must follow the same testing convention. Should you use directory-level
CLAUDE.md files or a path-specific rule? Why?

<details><summary>ANSWER</summary>
A path-specific rule with `paths: ["**/*.test.tsx"]`. The convention is defined
by file type, not by directory — a glob pattern applies it everywhere the file
type appears, whereas directory-level CLAUDE.md files would each need a nearly
identical copy placed in every directory that happens to contain test files.
</details>
