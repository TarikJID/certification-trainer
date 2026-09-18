# Module 4 Hands-On Exercise

## Scenario: Configuring Claude Code for a Growing Monorepo

Your company's monorepo has a `packages/api/` (Python), `packages/web/`
(TypeScript/React), and `infra/terraform/` directory. Onboarding feedback says
new hires "don't get the same Claude Code behavior senior engineers do," and the
root `CLAUDE.md` has grown to 600 lines covering build commands, API design
rules, React component conventions, Terraform conventions, testing standards, and
a 35-step "add a new microservice" procedure.

### Part A — Diagnosing and restructuring the hierarchy

1. Propose a full CLAUDE.md hierarchy for this repo: what goes in
   `~/.claude/CLAUDE.md` (if anything), what goes in the root `./CLAUDE.md`, and
   what goes in directory-level CLAUDE.md files under `packages/api/` and
   `packages/web/`. Justify each placement.
2. Diagnose the onboarding complaint: what is the most likely configuration
   mistake, and how would you confirm it using the tools from Lesson 3.1?
3. Restructure the 600-line root CLAUDE.md: decide what stays in CLAUDE.md, what
   moves to `.claude/rules/` files, and what moves to a skill. Justify each move
   against the context-window-budget principle.

### Part B — Path-specific rules

4. Testing conventions apply to every `*.test.ts` and `*.test.tsx` file
   regardless of which package it's in. Write the `.claude/rules/testing.md`
   frontmatter that scopes this rule correctly, and explain why a directory-level
   CLAUDE.md wouldn't work as well here.
5. Terraform conventions should only ever load when Claude is working inside
   `infra/terraform/`. Write that rule's frontmatter.

### Part C — Skills and commands

6. Turn the 35-step "add a new microservice" procedure into a skill. Write its
   SKILL.md frontmatter, including a sensible `argument-hint`, and decide whether
   it should use `context: fork`. Justify the `context: fork` decision.
7. One senior engineer wants a personal variant of this skill with two extra
   internal-only steps, without affecting anyone else. Show where they'd put it
   and how they'd avoid colliding with the team version.
8. Should the 35-step procedure live in CLAUDE.md or as a skill? Justify using
   the decision rule from Lesson 3.2.

Write your answers as a short configuration design document, including the
directory tree you propose.
