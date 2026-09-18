# Evaluation — domain-researcher (Domain 5 — Context Management & Reliability) — round 1

- Output evaluated: `runs/claude-certified-architect-foundations/research/context-management-reliability.md`
- Checklist: domain-researcher's "Done when" checklist
- Source of truth used: every URL cited in the output (fetched and compared below), the
  domain slice `runs/claude-certified-architect-foundations/dispatch/context-management-reliability.md`,
  and the exam guide header at `runs/claude-certified-architect-foundations/sources/exam-guide.txt`
- Repeat round: no
- Verdict: PASS

## Checklist results

| # | Checklist item | Result | Evidence or issue |
|---|----------------|--------|-------------------|
| 1 | Every bullet received has at least one concept teaching it, named in `Teaches:` | PASS | Dispatch slice has exactly 53 bullets (5.1: 10, 5.2: 9, 5.3: 8, 5.4: 9, 5.5: 8, 5.6: 9 = 53). Cross-checked every bullet ID against the body's `Teaches:` fields and the file's own "Coverage index" (lines 361–420): all 53 IDs (5.1-K1…5.6-S5) appear at least once. Header states "Bullets received: 53 ... Bullets covered: 53" (line 3–4), matching. |
| 2 | Every stated count matches the file | PASS | Recounted `Concept:` blocks by task statement: 5.1=12 (10 key+2 prereq), 5.2=7 (6+1), 5.3=8 (6+2), 5.4=7 (6+1), 5.5=5 (5+0), 5.6=7 (6+1). Total = 46 concepts, 39 key, 7 prerequisite — exactly matching the header claim on line 5 ("Concepts: 46 (39 key, 7 prerequisite)"). |
| 3 | Every key concept defined, illustrated with a concrete example, attributed to a source | PASS | Spot-checked all 39 key-concept entries; each has `Definition:`, `Example:`, and `Source:` fields populated (e.g. lines 17–22, 105–110, 158–163, 218–223, 271–276, 310–315). No key concept found missing any of the three fields. |
| 4 | Each key concept's immediate prerequisites identified and defined the same way | PASS | 7 prerequisite concepts present (Attention budget, Messages API structure, System-prompt role framing, tool_result/is_error mechanics, Orchestrator-worker architecture, Subagent context isolation mechanics, Claude API citations feature), each with its own Definition/Example/Source and a `Teaches: prerequisite for <key concept>` line linking it back explicitly (e.g. lines 87–92, 147–152, 200–205). |
| 5 | Every cited source meets the quality bar (official — no forums/social/blogs) | PASS | All cited sources are: anthropic.com/engineering/* (official eng. blog, explicitly allowed by the brief), platform.claude.com/docs/*, platform.claude.com/cookbook/*, code.claude.com/docs/*, claude.com/blog/* (official product blog), and the exam guide / dispatch slice itself. No forum, social-media, or third-party blog citations found anywhere in the file. |
| 6 | No concept cites non-official source where official exists; non-official citations justified; official sources not ranked | PASS | No non-official citation appears anywhere in the file — all citations are official. Several concepts (mainly under 5.2 and 5.5, plus scattered items in 5.1/5.3/5.4/5.6) cite the exam guide/dispatch bullet wording as their primary or sole source (e.g. lines 105–110, 271–276) rather than a product-doc page; per the checklist note this is explicitly not a defect and was not treated as one. |
| 7 | Any unsourced concept carries `Status: UNSOURCED` and a `Searched:` record; none omitted for want of a source | PASS | Header claims "0 UNSOURCED" (implicit from "Concepts: 46 (39 key, 7 prerequisite)" all fully attributed); no `Status: UNSOURCED` markers found in the file, and none were expected to be found missing given full source attribution on all 46 concepts. |

### Source verification detail (fidelity of quotes/stats to cited pages)

Fetched and checked content against the following cited sources; all quotes, statistics, and technical claims attributed to them were confirmed present and accurately represented:

- `anthropic.com/engineering/effective-context-engineering-for-ai-agents` — confirmed context rot definition, n² attention/"attention budget" mechanic, compaction discarding detail, sub-agent "condensed, distilled summary... often 1,000-2,000 tokens" (verbatim range matches output line 83).
- `platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools` — confirmed `clear_tool_uses_20250919` mechanics and parameters match the output's description (lines 62–64), and the memory/compaction primitives described.
- `platform.claude.com/docs/en/build-with-claude/working-with-messages` — confirmed "The Messages API is stateless, which means that you always send the full conversational history to the API," matching 5.1-K4 (lines 41–43).
- `platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls` — confirmed `is_error`/`tool_result` mechanics, "Write instructive error messages" tip, and formatting-order rules match 5.3-K1/K3 and the prerequisite entry (lines 158–177, 200–205).
- `anthropic.com/engineering/built-multi-agent-research-system` — confirmed orchestrator-worker pattern, CitationAgent/structured-findings pattern, "letting the agent know when a tool is failing and letting it adapt works surprisingly well," the 200k-token truncation/Memory detail, "deterministic safeguards like retry logic and regular checkpoints," and the five-dimension LLM-judge rubric including "completeness" and "source quality" — all match the output's citations (lines 78, 163, 184, 191, 198, 209–212, 242–244, 343).
- `platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices` — confirmed "Use examples effectively" (relevant/diverse/`<example>` tags), "Give Claude a role," and "Long context prompting" ("Queries at the end can improve response quality by up to 30 percent...") all match the output's paraphrase at lines 69–71, 136–138, 150–152.
- `code.claude.com/docs/en/sub-agents` — confirmed "fresh, isolated context window... doesn't see your conversation history" and what a subagent receives at spawn (system prompt, task/delegation message, CLAUDE.md, git status) — matches lines 235–237, 263–265.
- `claude.com/blog/using-claude-code-session-management-and-1m-context` — confirmed "Compact asks the model to summarize the conversation so far, then replaces the history with that summary," the steerable-instructions example, and "when you start a new task, you should also start a new session" — matches lines 223, 256–258.
- `anthropic.com/engineering/effective-harnesses-for-long-running-agents` — confirmed "Read the progress notes file and git commit logs..." progress-log pattern matches lines 230, 244.
- `platform.claude.com/cookbook/misc-using-citations` — confirmed `citations: {enabled: true}`, the `citations` array with cited text/document, "will not return citations pointing to documents... not provided," and the `context` field for metadata such as publication dates — matches lines 320–322, 335–336, 355–357.
- `platform.claude.com/docs/en/about-claude/use-case-guides/customer-support-chat` — confirmed the "Escalation efficiency" metric verbatim ("This measures Claude's ability to recognize when a query needs human intervention and escalate appropriately") matches the supporting citation at line 110.
- `platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations` — confirmed "Allow Claude to say 'I don't know'" technique matches the supporting citation at line 304.
- `platform.claude.com/docs/en/agents-and-tools/tool-use/overview` — confirmed the "When required parameters are missing" accordion exists; the output correctly labels this as only an "analogous principle" rather than a direct match for the multiple-matches/clarification concept (line 131), which is an honest characterization, not a misattribution.

No quote, statistic, or specific technical claim attributed to a source was found to be fabricated, altered, or unsupported by the cited page.

## Persisting issues

Not applicable — this is round 1, not a repeat evaluation.

## Not checked

- Every one of the ~100+ individual example scenarios in the file was not independently fact-checked against a source (examples are illustrative and not required by the checklist to be sourced separately from their parent concept).
- The claim that all 53 bullets came verbatim from the exam guide's own PDF (beyond the dispatch slice, which is confirmed to match) was not independently re-verified against the original PDF at the S3 URL in the exam guide header — the dispatch slice is treated as the authoritative bullet source per the orchestrator's instructions, and it matches the researcher's `Teaches:` references exactly.
