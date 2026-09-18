# Module 4 — Claude Code Memory and Configuration

Domain: Domain 3 — Claude Code Configuration & Workflows (20% of exam)
Covers Task Statements 3.1, 3.2, 3.3.

**Prerequisites from earlier modules:** subagent context isolation (Module 1,
Lesson 1.2) is relevant background for the `context: fork` concept below.

---

## Lesson 3.1 — CLAUDE.md Hierarchy, Scoping, and Modular Organization

**Maps to:** Task Statement 3.1: Configure CLAUDE.md files with appropriate
hierarchy, scoping, and modular organization.

### Prerequisite concept: Context window and token budget

Every Claude Code session has a finite context window; CLAUDE.md files, rule
files, tool outputs, and conversation history all consume tokens from that
budget, and performance degrades as it fills. CLAUDE.md and unconditional rules
load at every session's launch regardless of relevance, so their size is a
direct, constant cost; conditional (path-scoped) rules and skills only cost
tokens when actually loaded.

*Example:* A 1,000-line unconditional CLAUDE.md consumes roughly the same token
budget every session, even on a task that never touches most of what it
describes — which is why the documentation recommends targeting under 200 lines
per CLAUDE.md file.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: prerequisite for CLAUDE.md hierarchy, .claude/rules/, path-specific
rules, and choosing between skills and CLAUDE.md)

### Concept: CLAUDE.md configuration hierarchy (user, project, directory)

Claude Code reads CLAUDE.md from several scopes: user-level (`~/.claude/CLAUDE.md`,
applies to one person across all projects), project-level (`./CLAUDE.md` or
`./.claude/CLAUDE.md`, shared via version control), and directory-level
(subdirectory CLAUDE.md files, loaded on demand when Claude reads files there).
Content from broader scopes loads before narrower/closer scopes, so an
instruction closer to the working directory is read last.

*Example:* A monorepo has `~/.claude/CLAUDE.md` with personal preferences,
`./CLAUDE.md` with team build/test commands, and `packages/api/CLAUDE.md` with
API-specific conventions that only load when Claude works in `packages/api/`.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.1-K1, 3.1-S1)

### Concept: User-level CLAUDE.md is personal and not shared via version control

`~/.claude/CLAUDE.md` lives outside any git repository. It's loaded for that user
across every project but never committed, so it never reaches teammates
automatically. This is the single most common cause of "a new team member isn't
getting instructions I expect them to have."

*Example:* A senior engineer adds "always run `make lint` before committing" to
their own `~/.claude/CLAUDE.md`. A new hire clones the repo and never sees that
rule, because it was never in the project's `./CLAUDE.md`.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.1-K2, 3.1-S1)

### Concept: @import syntax for modular CLAUDE.md

CLAUDE.md files can pull in other files using `@path/to/file` syntax. Imported
files are expanded and loaded into context at launch; imports resolve relative to
the file containing the import (not the working directory), and can recursively
import further files up to a maximum depth of four hops. This lets a team keep
one lean root CLAUDE.md that selectively imports only relevant standards files.

*Example:* A payments package's CLAUDE.md writes
`@../../docs/pci-compliance-standards.md` to pull in PCI standards only relevant
to that package.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.1-K3, 3.1-S2)

### Concept: .claude/rules/ directory as an alternative to a monolithic CLAUDE.md

For larger projects, instructions can be organized into topic-specific markdown
files in `.claude/rules/` (e.g. `testing.md`, `api-conventions.md`,
`deployment.md`) instead of one large CLAUDE.md. All `.md` files there are
discovered recursively. Rules without `paths` frontmatter load at launch with the
same priority as `.claude/CLAUDE.md`.

*Example:* A 400-line CLAUDE.md covering testing, API design, and deployment is
split into `.claude/rules/testing.md`, `.claude/rules/api-conventions.md`, and
`.claude/rules/deployment.md`.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.1-K4, 3.1-S3)

### Concept: /memory command for inspecting loaded memory files

`/memory` lists CLAUDE.md, CLAUDE.local.md, and other memory-file locations
across user and project scopes (including entries for files that don't yet
exist), lets the user toggle auto memory, and opens the auto memory folder.
`/context` shows which memory files actually loaded into the current session —
the documented way to confirm a CLAUDE.md was picked up.

*Example:* A developer runs `/memory` to see every CLAUDE.md location Claude Code
recognizes, discovers a rule was written to the wrong package subdirectory, and
runs `/context` to confirm which files loaded for the current session.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.1-S4)

**Quiz:** see `quiz.md`, Lesson 3.1.

---

## Lesson 3.2 — Custom Slash Commands and Skills

**Maps to:** Task Statement 3.2: Create and configure custom slash commands and
skills.

### Prerequisite concept: YAML frontmatter in markdown configuration files

Skill files (SKILL.md) and rule files (`.claude/rules/*.md`) both use a YAML
frontmatter block — a `---`-delimited section at the top of the file — to
declare structured configuration, parsed separately from the markdown body that
forms the actual instruction content shown to Claude.

*Example:*
```
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
```

*Source:* https://code.claude.com/docs/en/memory; https://code.claude.com/docs/en/skills
(Teaches: prerequisite for SKILL.md frontmatter, context: fork, allowed-tools,
argument-hint, and path-specific rules paths field)

### Prerequisite concept: Tool permission approval and allowedTools

Claude Code normally prompts for approval before using tools that could modify
the system. The `--allowedTools` CLI flag (and the analogous `allowed-tools`
frontmatter field on skills) lets specific tools be pre-approved so Claude can
use them without an interactive prompt.

*Example:* `claude -p "Run the test suite and fix any failures" --allowedTools
"Bash,Read,Edit"` pre-approves exactly the tools that task needs.

*Source:* https://code.claude.com/docs/en/headless
(Teaches: prerequisite for allowed-tools skill frontmatter — full coverage
continues in Module 5, Lesson 3.6)

### Concept: Project-scoped vs user-scoped commands and skills

Custom commands/skills can be stored at project scope (`.claude/commands/` or
`.claude/skills/<name>/SKILL.md` at the repo root) or user scope
(`~/.claude/commands/` or `~/.claude/skills/<name>/SKILL.md`). Project-scoped
commands are committed to version control and shared with the whole team.
User-scoped commands load in all of that user's projects but aren't shared.

*Example:* A team commits `.claude/commands/deploy.md` so every teammate gets a
`/deploy` command, while an individual keeps a personal
`~/.claude/commands/standup.md`.

*Source:* https://code.claude.com/docs/en/slash-commands
(Teaches: 3.2-K1, 3.2-S1)

### Concept: SKILL.md frontmatter configuration

A skill lives in `.claude/skills/<skill-name>/SKILL.md`, with YAML frontmatter
supporting fields including `context: fork`, `allowed-tools`, and
`argument-hint`, beyond the base `name`/`description` fields. `description` is
what Claude uses to decide when to apply the skill automatically; if omitted,
Claude falls back to the first non-empty line of the body.

*Example:*
```
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
argument-hint: "[PR-number]"
allowed-tools: Bash(gh *)
---
```

*Source:* https://code.claude.com/docs/en/skills
(Teaches: 3.2-K2)

### Concept: context: fork frontmatter (isolated subagent execution)

`context: fork` runs a skill in a forked subagent context instead of the main
conversation. Claude Code starts a new subagent, gives it the skill content as
its prompt, and only the subagent's summarized result returns to the main
session — so verbose output or exploratory reasoning never pollutes the primary
conversation. This is the same context-isolation mechanism you saw with
subagents in Module 1, Lesson 1.2, applied to skills.

*Example:* A `/analyze-codebase` skill with `context: fork` runs a deep
structural analysis across hundreds of files in an isolated subagent; the main
session only receives the final summary.

*Source:* https://code.claude.com/docs/en/skills
(Teaches: 3.2-K3, 3.2-S2)

### Concept: allowed-tools frontmatter (pre-approving tool access for a skill)

`allowed-tools` pre-approves a specific set of tools for the turn that invokes
the skill, so Claude can use exactly those tools without a permission prompt; the
grant clears once the invoking turn ends. It does not itself block other tools —
they still follow normal permission settings.

*Example:* A documentation-update skill sets `allowed-tools: Write Edit` so Claude
can write/edit files during that skill's turn without a prompt, while destructive
operations like `Bash(rm *)` still require normal approval.

*Source:* https://code.claude.com/docs/en/skills
(Teaches: 3.2-S3)

### Concept: argument-hint frontmatter

`argument-hint` is a display-only hint shown in the `/` autocomplete menu,
indicating what arguments a skill/command expects. It does not enforce or parse
arguments — it prompts the developer with the expected shape of input.

*Example:* A skill with `argument-hint: "[filename] [format]"` shows that hint
under `/convert` in the autocomplete list.

*Source:* https://code.claude.com/docs/en/skills
(Teaches: 3.2-S4)

### Concept: Personal skill customization in ~/.claude/skills/

A developer can create a personal variant of a team skill in
`~/.claude/skills/<different-name>/SKILL.md`, giving it a different name so it
doesn't collide with the team's shared skill. Personal skills live outside the
repository and are invisible to teammates.

*Example:* The team's `.claude/skills/deploy/SKILL.md` runs a standard
deployment checklist; one engineer creates
`~/.claude/skills/deploy-personal/SKILL.md` with extra steps for their own
staging environment.

*Source:* https://code.claude.com/docs/en/skills
(Teaches: 3.2-K4)

### Concept: Choosing between skills and CLAUDE.md

CLAUDE.md content loads into every session's context automatically, so it should
hold only universal, broadly-applicable standards; it costs tokens on every
conversation regardless of relevance. Skills load only on demand, so they are
the right home for task-specific workflows or large reference material. Rule of
thumb: if a CLAUDE.md section has grown into a multi-step procedure rather than a
fact Claude needs every session, move it to a skill.

*Example:* "Use 2-space indentation" belongs in CLAUDE.md. A 40-step "migrate a
service from v1 to v2 API" procedure, needed only occasionally, belongs in a
skill invoked with `/migrate-service`.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.2-S5)

**Quiz:** see `quiz.md`, Lesson 3.2.

---

## Lesson 3.3 — Path-Specific Rules for Conditional Convention Loading

**Maps to:** Task Statement 3.3: Apply path-specific rules for conditional
convention loading.

### Concept: Path-specific rules with YAML frontmatter `paths` field

Files in `.claude/rules/` can be scoped to specific files using YAML frontmatter
with a `paths` field containing one or more glob patterns (e.g. `paths:
["terraform/**/*"]`). A rule with a `paths` field is conditional: it only applies
when Claude is working with matching files. Rules without a `paths` field are
unconditional and load at launch for every session.

*Example:*
```
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- All API endpoints must include input validation
```
This rule only enters context when Claude reads or edits a TypeScript file under
`src/api/`.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.3-K1, 3.3-S1)

### Concept: Path-scoped conditional loading reduces context and token usage

Because a path-scoped rule loads only when Claude reads a matching file (rather
than at every session launch), it keeps irrelevant conventions out of the context
window for tasks that never touch those files — a direct token-budget benefit
over an unconditional CLAUDE.md or rule file. Path-scoped rules trigger when
Claude reads a matching file, not on every tool use.

*Example:* A `.claude/rules/terraform.md` file with `paths: ["terraform/**/*"]`
never enters context during a session that only touches the frontend codebase.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.3-K2)

### Concept: Glob patterns for conventions spanning multiple directories

A `paths` glob pattern matches files by type or name regardless of where they
live in the directory tree (e.g. `**/*.test.tsx` matches every test file
anywhere). This is the key advantage over directory-level CLAUDE.md files, which
are scoped to a physical directory: a convention that applies to a file *type*
spread across many directories cannot be expressed as a single directory-level
CLAUDE.md.

*Example:* `paths: ["**/*.test.tsx"]` in `.claude/rules/testing.md` applies
testing conventions to every `.test.tsx` file across `src/`, `lib/`, and
`tools/` alike.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.3-K3, 3.3-S2)

### Concept: Choosing path-specific rules over subdirectory CLAUDE.md files

When conventions must apply to files spread throughout a codebase rather than
confined to one directory subtree, a glob-scoped rule in `.claude/rules/` is the
appropriate mechanism instead of placing a CLAUDE.md in every relevant
subdirectory. A directory-level CLAUDE.md suits conventions specific to that one
directory's contents; a path-specific rule suits conventions defined by file type
or pattern that cut across the directory structure.

*Example:* For "all `*.test.tsx` files must use the shared test-utils wrapper," a
single `.claude/rules/testing.md` with `paths: ["**/*.test.tsx"]` is preferable
to placing a nearly identical CLAUDE.md in every directory that happens to
contain test files.

*Source:* https://code.claude.com/docs/en/memory
(Teaches: 3.3-S3)

**Quiz:** see `quiz.md`, Lesson 3.3.

---

## Module 4 summary of bullets taught

Lesson 3.1: 3.1-K1, 3.1-K2, 3.1-K3, 3.1-K4, 3.1-S1, 3.1-S2, 3.1-S3, 3.1-S4
Lesson 3.2: 3.2-K1, 3.2-K2, 3.2-K3, 3.2-K4, 3.2-S1, 3.2-S2, 3.2-S3, 3.2-S4, 3.2-S5
Lesson 3.3: 3.3-K1, 3.3-K2, 3.3-K3, 3.3-S1, 3.3-S2, 3.3-S3
