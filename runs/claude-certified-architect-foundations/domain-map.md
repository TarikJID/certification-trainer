# Domain Map — Claude Certified Architect – Foundations

## Certification
Claude Certified Architect – Foundations

## Sources fetched
- https://anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification (official certification landing page)
- https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F6nizmqk8tpzpfjvt6qmmav7rh%2Fpublic%2F1783542750%2FClaude+Certified+Architect+%E2%80%93+Foundations+Exam+Guide.pdf (official Exam Guide PDF, linked from the landing page; fetched via a text-rendering proxy after direct extraction of the compressed PDF stream failed)

## Notes on sourcing
The landing page itself describes the exam only at a high level, as covering four core technology areas (Claude Code, Claude Agent SDK, Claude API, Model Context Protocol) without weightings. The linked Exam Guide PDF contains the explicit, weighted domain enumeration used below. Direct text extraction of the PDF's compressed content streams failed on first attempts; the content below was obtained by re-fetching the identical PDF URL through a text-rendering proxy, which successfully rendered a domain/weighting table and per-domain summaries. This is presented as the official Exam Guide content for Claude Certified Architect – Foundations specifically (not Associate – Foundations, Developer – Foundations, or Architect – Professional).

## Domains

- Domain: Agentic Architecture & Orchestration (27%)
  Description: Covers building autonomous agent loops, orchestrating multi-agent systems with coordinator-subagent patterns, configuring subagent invocation and context passing, implementing multi-step workflows with enforcement and handoff patterns, using Agent SDK hooks for tool interception, designing task decomposition strategies, and managing session state.

- Domain: Claude Code Configuration & Workflows (20%)
  Description: Focuses on configuring CLAUDE.md file hierarchies, creating custom slash commands and skills, applying path-specific rules for conditional loading, determining when to use plan mode versus direct execution, applying iterative refinement techniques, and integrating Claude Code into CI/CD pipelines.

- Domain: Prompt Engineering & Structured Output (20%)
  Description: Emphasizes designing prompts with explicit criteria to reduce false positives, applying few-shot prompting for consistency, enforcing structured output using tool use and JSON schemas, implementing validation and retry loops, designing efficient batch processing strategies, and creating multi-instance review architectures.

- Domain: Tool Design & MCP Integration (18%)
  Description: Addresses designing effective tool interfaces with clear descriptions, implementing structured error responses for MCP tools, distributing tools across agents appropriately, integrating MCP servers into workflows, and selecting built-in tools effectively.

- Domain: Context Management & Reliability (15%)
  Description: Encompasses managing conversation context across long interactions, designing escalation and ambiguity resolution patterns, implementing error propagation strategies across multi-agent systems, managing context in large codebase exploration, designing human review workflows with confidence calibration, and preserving information provenance in multi-source synthesis.
