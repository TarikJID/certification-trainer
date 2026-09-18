# Evaluation — domain-researcher (Agentic Architecture & Orchestration, CCAR-F Domain 1) — round 1

- Output evaluated: runs/claude-certified-architect-foundations/research-agentic-architecture-orchestration.md
- Checklist: domain-researcher's "Done when" checklist
- Source of truth used: the sources the output itself cites — fetched and compared against the output's definitions/examples. Fetched:
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works
  - https://code.claude.com/docs/en/agent-sdk/subagents
  - https://code.claude.com/docs/en/agent-sdk/sessions
  - https://www.anthropic.com/engineering/building-effective-agents
  - https://www.anthropic.com/engineering/multi-agent-research-system
  - https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons
  - https://code.claude.com/docs/en/agent-sdk/hooks
  - https://code.claude.com/docs/en/sessions
  Also read runs/claude-certified-architect-foundations/source-exam-guide.txt in full for Domain 1's task statements and the sample questions cited as tier-1 sources, and the exact 48-bullet slice at
  /tmp/claude-0/-home-user-certification-trainer/5f9acdef-c166-585e-8b38-cc60c4e8c9c9/scratchpad/domain-1-agentic-architecture.md.
- Repeat round: no
- Verdict: PASS

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every bullet given has at least one concept that teaches it, named in that concept's `Teaches:` field; state bullets received/covered | PASS | Manually walked all 48 bullets (1.1 K1-K3/S1-S3; 1.2 K1-K4/S1-S4; 1.3 K1-K4/S1-S4; 1.4 K1-K3/S1-S3; 1.5 K1-K3/S1-S3; 1.6 K1-K3/S1-S3; 1.7 K1-K4/S1-S4) against every `Teaches:` field in the key-concept sections. All 48 are named in at least one key concept's `Teaches:` field (several, e.g. 1.5-K3/S3 and 1.7-K2/S2, are covered via an explicit cross-reference note pointing to the concept whose own `Teaches:` field lists them). Doc states "Bullets received: 48 ... Bullets covered: 48/48" — confirmed accurate. |
| 2 | Every key concept is defined, illustrated with a concrete example, and attributed to a source | PASS | Counted 30 `Type: key` entries (5+6+6+4+2+4+3 across 1.1–1.7), matching the doc's claimed "Key concepts: 30". Every entry has non-empty Definition, Example, and Source fields; spot-checked ~12 across all seven task statements — all have concrete, specific examples (not generic placeholders). |
| 3 | Each key concept's immediate prerequisites are identified and defined the same way | PASS | 5 prerequisite concepts are given at the top, each in the same Concept/Type/Teaches/Definition/Example/Source format, and each is linked back to the key concepts it underlies via its `Teaches: supports <bullet-refs>` field (e.g. the tool-use contract prerequisite supports all of 1.1's bullets; the augmented-LLM prerequisite supports 1.6's pattern concepts). Matches the doc's claimed "Prerequisite concepts: 5". |
| 4 | Every cited source meets the quality bar (no forums/social/blogs) | PASS | All cited sources are either the archived official exam guide (tier 1) or official Anthropic/Claude documentation and engineering-blog posts on platform.claude.com, code.claude.com, or anthropic.com/engineering (tier 2). No forums, social media, Q&A sites, or personal blogs found anywhere in the document. |
| 5 | Every source recorded with its precedence tier; no higher tier available where a lower one was used | PASS | Every `Source:` line carries an explicit `(tier N, ...)` annotation. Fetched 8 of the distinct tier-2 URLs cited and confirmed each supports the concept it's attached to (see below). Tier-1 exam guide is used as primary source wherever the claim is a scenario-specific application detail from the guide's Knowledge/Skills bullets or sample questions (e.g. the 12%-skip-rate example matches Sample Question 1 verbatim in content; the "creative industries" decomposition example matches Sample Question 7 verbatim); tier-2 official docs are used where the concept is a general product mechanism documented in the product docs. No case found where a lower tier was used despite a higher tier being available for the same claim. |
| 6 | Any unsourced concept carries `Status: UNSOURCED` and a `Searched:` record | PASS (vacuous) | Doc claims 0 UNSOURCED concepts; scanned the full document and found no concept lacking a Source field, and no `UNSOURCED` status anywhere — consistent with the claim. |

### Source-accuracy spot checks (supporting item 5)

- `how-tool-use-works`: confirmed the exact 5-step agentic loop, `stop_reason` values, and the tool-use contract description match the "agentic loop" and "tool-result accumulation" concepts almost verbatim.
- `agent-sdk/subagents`: confirmed "What subagents inherit" table matches the context-isolation concept's claims (no parent conversation history/tool results; only the Agent tool's prompt string passed); confirmed `AgentDefinition` field table matches (`description`, `prompt` required; `tools`, `model` optional); confirmed "Parallelization" benefit language matches the parallel-spawning concept.
- `agent-sdk/sessions`: confirmed "Fork to explore alternatives" section text ("The fork gets its own session ID; the original's ID and history stay unchanged...") matches the fork-based session management concept's quoted text exactly.
- `building-effective-agents`: confirmed "workflows" vs "agents" quoted definitions, augmented-LLM quoted definition, prompt-chaining and orchestrator-workers pattern quotes, and evaluator-optimizer quote all match the source verbatim.
- `multi-agent-research-system`: confirmed "3-5 subagents in parallel", the fact-finding/comparison/complex-research scaling rules, the CitationAgent description, and the vague-task-description failure mode all match what the doc attributes to this source.
- `handling-stop-reasons`: confirmed `stop_reason` value list and the documented anti-patterns (don't parse text, don't rely on hardcoded caps as primary control, don't add text after tool_result, don't treat an empty end_turn response as done) substantively support the "anti-patterns" concept, though the source's exact anti-pattern framing differs slightly in wording from the doc's third anti-pattern ("checking for assistant text content as a completion indicator") — the underlying principle (stop_reason is authoritative, not text presence) is still directly supported by the page.
- `agent-sdk/hooks`: confirmed `PreToolUse` output fields (`permissionDecision`: allow/deny/ask/defer, `permissionDecisionReason`, `updatedInput`) and `PostToolUse` fields (`additionalContext`, `updatedToolOutput`), and confirmed "deny takes priority over allow" ("If any hook returns `deny`, the operation is blocked regardless of other hooks") — all match the doc's PreToolUse/PostToolUse concepts exactly.
- `code.claude.com/docs/en/sessions`: confirmed named-session resumption (`claude -n <name>`, `/rename <name>`, `claude --resume <name>` "Resumes the named session directly") matches the named-session-resumption concept exactly.

## Persisting issues

N/A — first round.

## Not checked

- The exam guide file exceeds this tool's single-read window; I read pages 1–1089 of 1408, which covers the entirety of Domain 1's task statements and Domain 1's cited sample questions (Sample Questions 1 and 7, both fully verified). Domains 2–5 content later in the file was not read, but nothing in this output cites material beyond Domain 1's section.
