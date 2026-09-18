## Domain 4: Prompt Engineering & Structured Output (20%)

Description: Covers writing prompts with explicit criteria to reduce false positives, applying few-shot
prompting for consistency, enforcing structured output via tool_use and JSON schemas, implementing
validation/retry/feedback loops for extraction, designing batch processing strategies, and designing
multi-instance/multi-pass review architectures.

Task statements: 6 · Bullets captured: 47 (22 Knowledge, 25 Skills)

### Task statements

- Task Statement 4.1: Design prompts with explicit criteria to improve precision and reduce false positives
  Knowledge of:
    - 4.1-K1: The importance of explicit criteria over vague instructions (e.g., "flag comments only when claimed behavior contradicts actual code behavior" vs "check that comments are accurate")
    - 4.1-K2: How general instructions like "be conservative" or "only report high-confidence findings" fail to improve precision compared to specific categorical criteria
    - 4.1-K3: The impact of false positive rates on developer trust: high false positive categories undermine confidence in accurate categories
  Skills in:
    - 4.1-S1: Writing specific review criteria that define which issues to report (bugs, security) versus skip (minor style, local patterns) rather than relying on confidence-based filtering
    - 4.1-S2: Temporarily disabling high false-positive categories to restore developer trust while improving prompts for those categories
    - 4.1-S3: Defining explicit severity criteria with concrete code examples for each severity level to achieve consistent classification

- Task Statement 4.2: Apply few-shot prompting to improve output consistency and quality
  Knowledge of:
    - 4.2-K1: Few-shot examples as the most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results
    - 4.2-K2: The role of few-shot examples in demonstrating ambiguous-case handling (e.g., tool selection for ambiguous requests, branch-level test coverage gaps)
    - 4.2-K3: How few-shot examples enable the model to generalize judgment to novel patterns rather than matching only pre-specified cases
    - 4.2-K4: The effectiveness of few-shot examples for reducing hallucination in extraction tasks (e.g., handling informal measurements, varied document structures)
  Skills in:
    - 4.2-S1: Creating 2-4 targeted few-shot examples for ambiguous scenarios that show reasoning for why one action was chosen over plausible alternatives
    - 4.2-S2: Including few-shot examples that demonstrate specific desired output format (location, issue, severity, suggested fix) to achieve consistency
    - 4.2-S3: Providing few-shot examples distinguishing acceptable code patterns from genuine issues to reduce false positives while enabling generalization
    - 4.2-S4: Using few-shot examples to demonstrate correct handling of varied document structures (inline citations vs bibliographies, methodology sections vs embedded details)
    - 4.2-S5: Adding few-shot examples showing correct extraction from documents with varied formats to address empty/null extraction of required fields

- Task Statement 4.3: Enforce structured output using tool use and JSON schemas
  Knowledge of:
    - 4.3-K1: Tool use (tool_use) with JSON schemas as the most reliable approach for guaranteed schema-compliant structured output, eliminating JSON syntax errors
    - 4.3-K2: The distinction between tool_choice: "auto" (model may return text instead of calling a tool), "any" (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)
    - 4.3-K3: That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)
    - 4.3-K4: Schema design considerations: required vs optional fields, enum fields with "other" + detail string patterns for extensible categories
  Skills in:
    - 4.3-S1: Defining extraction tools with JSON schemas as input parameters and extracting structured data from the tool_use response
    - 4.3-S2: Setting tool_choice: "any" to guarantee structured output when multiple extraction schemas exist and the document type is unknown
    - 4.3-S3: Forcing a specific tool with tool_choice: {"type": "tool", "name": "extract_metadata"} to ensure a particular extraction runs before enrichment steps
    - 4.3-S4: Designing schema fields as optional (nullable) when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields
    - 4.3-S5: Adding enum values like "unclear" for ambiguous cases and "other" + detail fields for extensible categorization
    - 4.3-S6: Including format normalization rules in prompts alongside strict output schemas to handle inconsistent source formatting

- Task Statement 4.4: Implement validation, retry, and feedback loops for extraction quality
  Knowledge of:
    - 4.4-K1: Retry-with-error-feedback: appending specific validation errors to the prompt on retry to guide the model toward correction
    - 4.4-K2: The limits of retry: retries are ineffective when the required information is simply absent from the source document (vs format or structural errors)
    - 4.4-K3: Feedback loop design: tracking which code constructs trigger findings (detected_pattern field) to enable systematic analysis of dismissal patterns
    - 4.4-K4: The difference between semantic validation errors (values don't sum, wrong field placement) and schema syntax errors (eliminated by tool use)
  Skills in:
    - 4.4-S1: Implementing follow-up requests that include the original document, the failed extraction, and specific validation errors for model self-correction
    - 4.4-S2: Identifying when retries will be ineffective (e.g., information exists only in an external document not provided) versus when they will succeed (format mismatches, structural output errors)
    - 4.4-S3: Adding detected_pattern fields to structured findings to enable analysis of false positive patterns when developers dismiss findings
    - 4.4-S4: Designing self-correction validation flows: extracting "calculated_total" alongside "stated_total" to flag discrepancies, adding "conflict_detected" booleans for inconsistent source data

- Task Statement 4.5: Design efficient batch processing strategies
  Knowledge of:
    - 4.5-K1: The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA
    - 4.5-K2: Batch processing is appropriate for non-blocking, latency-tolerant workloads (overnight reports, weekly audits, nightly test generation) and inappropriate for blocking workflows (pre-merge checks)
    - 4.5-K3: The batch API does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)
    - 4.5-K4: custom_id fields for correlating batch request/response pairs
  Skills in:
    - 4.5-S1: Matching API approach to workflow latency requirements: synchronous API for blocking pre-merge checks, batch API for overnight/weekly analysis
    - 4.5-S2: Calculating batch submission frequency based on SLA constraints (e.g., 4-hour windows to guarantee 30-hour SLA with 24-hour batch processing)
    - 4.5-S3: Handling batch failures: resubmitting only failed documents (identified by custom_id) with appropriate modifications (e.g., chunking documents that exceeded context limits)
    - 4.5-S4: Using prompt refinement on a sample set before batch-processing large volumes to maximize first-pass success rates and reduce iterative resubmission costs

- Task Statement 4.6: Design multi-instance and multi-pass review architectures
  Knowledge of:
    - 4.6-K1: Self-review limitations: a model retains reasoning context from generation, making it less likely to question its own decisions in the same session
    - 4.6-K2: Independent review instances (without prior reasoning context) are more effective at catching subtle issues than self-review instructions or extended thinking
    - 4.6-K3: Multi-pass review: splitting large reviews into per-file local analysis passes plus cross-file integration passes to avoid attention dilution and contradictory findings
  Skills in:
    - 4.6-S1: Using a second independent Claude instance to review generated code without the generator's reasoning context
    - 4.6-S2: Splitting large multi-file reviews into focused per-file passes for local issues plus separate integration passes for cross-file data flow analysis
    - 4.6-S3: Running verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing

---

