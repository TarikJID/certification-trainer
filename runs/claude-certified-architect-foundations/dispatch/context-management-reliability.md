## Domain 5: Context Management & Reliability (15%)

Description: Covers managing conversation context to preserve critical information over long
interactions, designing escalation and ambiguity-resolution patterns, propagating errors across
multi-agent systems, managing context in large codebase exploration, designing human review
workflows with confidence calibration, and preserving information provenance in multi-source
synthesis.

Task statements: 6 · Bullets captured: 53 (24 Knowledge, 29 Skills)

### Task statements

- Task Statement 5.1: Manage conversation context to preserve critical information across long interactions
  Knowledge of:
    - 5.1-K1: Progressive summarization risks: condensing numerical values, percentages, dates, and customer-stated expectations into vague summaries
    - 5.1-K2: The "lost in the middle" effect: models reliably process information at the beginning and end of long inputs but may omit findings from middle sections
    - 5.1-K3: How tool results accumulate in context and consume tokens disproportionately to their relevance (e.g., 40+ fields per order lookup when only 5 are relevant)
    - 5.1-K4: The importance of passing complete conversation history in subsequent API requests to maintain conversational coherence
  Skills in:
    - 5.1-S1: Extracting transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block included in each prompt, outside summarized history
    - 5.1-S2: Extracting and persisting structured issue data (order IDs, amounts, statuses) into a separate context layer for multi-issue sessions
    - 5.1-S3: Trimming verbose tool outputs to only relevant fields before they accumulate in context (e.g., keeping only return-relevant fields from order lookups)
    - 5.1-S4: Placing key findings summaries at the beginning of aggregated inputs and organizing detailed results with explicit section headers to mitigate position effects
    - 5.1-S5: Requiring subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis
    - 5.1-S6: Modifying upstream agents to return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when downstream agents have limited context budgets

- Task Statement 5.2: Design effective escalation and ambiguity resolution patterns
  Knowledge of:
    - 5.2-K1: Appropriate escalation triggers: customer requests for a human, policy exceptions/gaps (not just complex cases), and inability to make meaningful progress
    - 5.2-K2: The distinction between escalating immediately when a customer explicitly demands it versus offering to resolve when the issue is straightforward
    - 5.2-K3: Why sentiment-based escalation and self-reported confidence scores are unreliable proxies for actual case complexity
    - 5.2-K4: How multiple customer matches require clarification (requesting additional identifiers) rather than heuristic selection
  Skills in:
    - 5.2-S1: Adding explicit escalation criteria with few-shot examples to the system prompt demonstrating when to escalate versus resolve autonomously
    - 5.2-S2: Honoring explicit customer requests for human agents immediately without first attempting investigation
    - 5.2-S3: Acknowledging frustration while offering resolution when the issue is within the agent's capability, escalating only if the customer reiterates their preference
    - 5.2-S4: Escalating when policy is ambiguous or silent on the customer's specific request (e.g., competitor price matching when policy only addresses own-site adjustments)
    - 5.2-S5: Instructing the agent to ask for additional identifiers when tool results return multiple matches, rather than selecting based on heuristics

- Task Statement 5.3: Implement error propagation strategies across multi-agent systems
  Knowledge of:
    - 5.3-K1: Structured error context (failure type, attempted query, partial results, alternative approaches) as enabling intelligent coordinator recovery decisions
    - 5.3-K2: The distinction between access failures (timeouts needing retry decisions) and valid empty results (successful queries with no matches)
    - 5.3-K3: Why generic error statuses ("search unavailable") hide valuable context from the coordinator
    - 5.3-K4: Why silently suppressing errors (returning empty results as success) or terminating entire workflows on single failures are both anti-patterns
  Skills in:
    - 5.3-S1: Returning structured error context including failure type, what was attempted, partial results, and potential alternatives to enable coordinator recovery
    - 5.3-S2: Distinguishing access failures from valid empty results in error reporting so the coordinator can make appropriate decisions
    - 5.3-S3: Having subagents implement local recovery for transient failures and only propagate errors they cannot resolve, including what was attempted and partial results
    - 5.3-S4: Structuring synthesis output with coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources

- Task Statement 5.4: Manage context effectively in large codebase exploration
  Knowledge of:
    - 5.4-K1: Context degradation in extended sessions: models start giving inconsistent answers and referencing "typical patterns" rather than specific classes discovered earlier
    - 5.4-K2: The role of scratchpad files for persisting key findings across context boundaries
    - 5.4-K3: Subagent delegation for isolating verbose exploration output while the main agent coordinates high-level understanding
    - 5.4-K4: Structured state persistence for crash recovery: each agent exports state to a known location, and the coordinator loads a manifest on resume
  Skills in:
    - 5.4-S1: Spawning subagents to investigate specific questions (e.g., "find all test files," "trace refund flow dependencies") while the main agent preserves high-level coordination
    - 5.4-S2: Having agents maintain scratchpad files recording key findings, referencing them for subsequent questions to counteract context degradation
    - 5.4-S3: Summarizing key findings from one exploration phase before spawning sub-agents for the next phase, injecting summaries into initial context
    - 5.4-S4: Designing crash recovery using structured agent state exports (manifests) that the coordinator loads on resume and injects into agent prompts
    - 5.4-S5: Using /compact to reduce context usage during extended exploration sessions when context fills with verbose discovery output

- Task Statement 5.5: Design human review workflows and confidence calibration
  Knowledge of:
    - 5.5-K1: The risk that aggregate accuracy metrics (e.g., 97% overall) may mask poor performance on specific document types or fields
    - 5.5-K2: Stratified random sampling for measuring error rates in high-confidence extractions and detecting novel error patterns
    - 5.5-K3: Field-level confidence scores calibrated using labeled validation sets for routing review attention
    - 5.5-K4: The importance of validating accuracy by document type and field segment before automating high-confidence extractions
  Skills in:
    - 5.5-S1: Implementing stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection
    - 5.5-S2: Analyzing accuracy by document type and field to verify consistent performance across all segments before reducing human review
    - 5.5-S3: Having models output field-level confidence scores, then calibrating review thresholds using labeled validation sets
    - 5.5-S4: Routing extractions with low model confidence or ambiguous/contradictory source documents to human review, prioritizing limited reviewer capacity

- Task Statement 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis
  Knowledge of:
    - 5.6-K1: How source attribution is lost during summarization steps when findings are compressed without preserving claim-source mappings
    - 5.6-K2: The importance of structured claim-source mappings that the synthesis agent must preserve and merge when combining findings
    - 5.6-K3: How to handle conflicting statistics from credible sources: annotating conflicts with source attribution rather than arbitrarily selecting one value
    - 5.6-K4: Temporal data: requiring publication/collection dates in structured outputs to prevent temporal differences from being misinterpreted as contradictions
  Skills in:
    - 5.6-S1: Requiring subagents to output structured claim-source mappings (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis
    - 5.6-S2: Structuring reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context
    - 5.6-S3: Completing document analysis with conflicting values included and explicitly annotated, letting the coordinator decide how to reconcile before passing to synthesis
    - 5.6-S4: Requiring subagents to include publication or data collection dates in structured outputs to enable correct temporal interpretation
    - 5.6-S5: Rendering different content types appropriately in synthesis outputs—financial data as tables, news as prose, technical findings as structured lists—rather than converting everything to a uniform format

---

