# Domain: Prompt Engineering & Structured Output

Weight: 20% of the CCAR-F exam. Covers designing prompts with explicit criteria to
reduce false positives, few-shot prompting for consistency, enforcing structured
output via tool use and JSON schemas, validation/retry loops, efficient batch
processing, and multi-instance review architectures.

---

## Key Concepts

- Concept: Defining success criteria and building evaluations
  Type: key
  Definition: Before prompt engineering a task, you should establish a clear, specific,
  measurable, achievable, and relevant (SMART) definition of what success looks like,
  and a way to empirically test outputs against it. Good criteria move from vague goals
  ("good performance") to quantifiable ones (e.g., "F1 score ≥ 0.85," "less than 0.1% of
  outputs flagged for toxicity out of 10,000 trials"). Evaluation design should be
  task-specific (mirror the real input distribution, include edge cases such as
  irrelevant/nonexistent input, overly long input, and ambiguous cases), and should favor
  automatable grading (exact match, semantic-similarity/cosine similarity, ROUGE-L,
  LLM-based Likert grading, binary classification) over small hand-graded sets, so that
  false positives/negatives can be systematically measured and reduced.
  Example: For a sentiment-classification prompt, an exact-match evaluator compares the
  model's label against 1,000 human-labeled tweets to compute accuracy deterministically,
  rather than relying on a handful of spot checks that could hide edge-case failures.
  Source: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests

- Concept: Multishot (few-shot) prompting
  Type: key
  Definition: Including a handful of well-crafted example input/output pairs directly in
  the prompt before the real task, so Claude can infer the desired format, tone, and
  structure. Anthropic recommends 3–5 diverse, relevant examples that mirror the actual
  use case and cover edge cases, wrapped in `<example>`/`<examples>` tags so Claude can
  distinguish them from instructions. This is one of the most reliable techniques for
  improving accuracy and consistency of outputs, especially for structured-output tasks.
  Example: To get consistent JSON sentiment labels, a prompt includes three example
  reviews each followed by the exact JSON object it should produce, before asking Claude
  to label a new, unseen review in the same JSON shape.
  Source: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting

- Concept: Chain-of-thought (CoT) prompting ("let Claude think")
  Type: key
  Definition: Explicitly instructing Claude to reason step-by-step before giving a final
  answer, which reduces errors on complex math, multi-step analysis, and decisions with
  many factors, and produces more organized responses. Anthropic describes three
  increasing levels of structure: a basic prompt ("Think step-by-step"), a guided prompt
  that outlines the specific steps to follow, and a structured prompt that separates
  reasoning from the answer using XML tags such as `<thinking>` and `<answer>`. Claude
  must actually output the reasoning text for thinking to occur — silent "thinking" does
  not happen.
  Example: A prompt asks Claude to place its step-by-step reasoning inside
  `<thinking>...</thinking>` and its final classification inside `<answer>...</answer>`,
  so the application can parse out only the `<answer>` block for downstream use while
  keeping the reasoning available for debugging.
  Source: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

- Concept: XML tags for structuring prompts
  Type: key
  Definition: Wrapping distinct parts of a prompt (instructions, context, examples,
  formatting rules) in XML-style tags such as `<instructions>`, `<example>`, or
  `<formatting>` so Claude can reliably tell them apart. This improves clarity (parts of
  the prompt are clearly separated), accuracy (fewer misinterpretations), flexibility
  (prompt sections can be edited independently), and parseability (asking Claude to
  respond inside tags makes post-processing/extraction easier). There is no fixed
  canonical tag vocabulary; consistency of tag names throughout the prompt and
  hierarchical nesting (`<outer><inner></inner></outer>`) for related content are
  recommended.
  Example: A document-analysis prompt puts the source document in `<document>...</document>`
  and the task instructions in `<instructions>...</instructions>`, then asks Claude to
  place its answer in `<answer>...</answer>` so the calling code can extract just that
  block from the response.
  Source: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags

- Concept: System prompts and role prompting
  Type: key
  Definition: The `system` parameter of a Messages API request sets Claude's role,
  personality, and standing context for a conversation, separately from the per-turn
  user/assistant messages. Giving Claude a role ("role prompting") is described as the
  most powerful way to use system prompts: even a single sentence assigning a role (e.g.,
  "You are a data scientist specializing in customer insight analysis for Fortune 500
  companies") measurably changes the character and quality of responses, turning Claude
  into a domain-focused assistant rather than a generalist. Task-specific instructions
  are generally kept in the user turn, with the role and standing behavior in the system
  prompt.
  Example: An enterprise support bot sets `system: "You are AcmeBot, the enterprise-grade
  AI assistant for AcmeTechCo... maintain a professional, concise tone"` so every reply
  in the conversation consistently reflects that persona and its constraints.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: Increasing output consistency
  Type: key
  Definition: A set of prompt-engineering techniques for making Claude's outputs more
  uniform across many calls, useful when guaranteed schema conformance (Structured
  Outputs) isn't in play or isn't sufficient: (1) precisely specify the desired output
  format (JSON/XML/custom templates); (2) prefill the start of Claude's response to
  bypass a preamble and force a structure (not supported on the newest model
  generations, where structured outputs or system-prompt instructions should be used
  instead); (3) constrain with concrete examples, which is more effective than abstract
  format instructions; (4) ground responses in a fixed retrieved knowledge base for
  contextually consistent answers; and (5) chain prompts, breaking a complex task into
  smaller subtasks so each gets Claude's full attention.
  Example: An IT-support bot is given a small `<kb>` knowledge base in the prompt and
  told to always check it first and respond in a fixed `<response><kb_entry>...</kb_entry>
  <answer>...</answer></response>` structure, producing consistent, source-grounded
  answers across different user questions.
  Source: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency

- Concept: Reducing hallucinations and false positives via grounding
  Type: key
  Definition: Techniques to reduce factually incorrect or unsupported output and
  therefore reduce false positives in tasks such as extraction, compliance checking, or
  classification: explicitly allowing Claude to say "I don't know" instead of guessing;
  asking Claude to extract direct, word-for-word quotes from long source documents before
  answering, so the answer is grounded in actual text; requiring citations for every
  claim and retracting claims that cannot be supported by a quote; chain-of-thought
  verification of reasoning; best-of-N verification (running the same prompt multiple
  times and comparing outputs for inconsistency); iterative refinement (feeding Claude's
  own output back in for self-verification); and restricting Claude to only the provided
  documents rather than its general knowledge. None of these eliminate hallucinations
  entirely, so critical claims still need external validation.
  Example: A compliance-review prompt asks Claude to first extract exact quotes from a
  privacy policy relevant to GDPR/CCPA, then base its compliance analysis only on those
  quotes, stating "No relevant quotes found" if none exist — reducing false-positive
  compliance findings not grounded in the source text.
  Source: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations

- Concept: Tool use (function calling)
  Type: key
  Definition: A mechanism that lets Claude call functions ("tools") that you define
  (client tools, executed in your application) or that Anthropic provides (server tools,
  executed on Anthropic's infrastructure). Claude decides whether and when to call a
  tool based on the user's request and the tool's description, then returns a
  `tool_use` content block naming the tool and a JSON object of arguments matching the
  tool's `input_schema`; the calling application executes the operation for client tools
  and returns a `tool_result` block, after which Claude continues the response. The
  default `tool_choice` of `auto` lets Claude decide per turn whether to call a tool.
  Example: A weather-lookup tool named `get_weather` with an `input_schema` requiring a
  `location` string is registered on a request; when asked "What's the weather in San
  Francisco?", Claude responds with a `tool_use` block `{"name": "get_weather", "input":
  {"location": "San Francisco, CA"}}`, and the application executes the lookup and
  returns the result in a `tool_result` block.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

- Concept: Defining tools and forcing tool use (tool_choice)
  Type: key
  Definition: Tools are defined with a `name`, a detailed plaintext `description`
  (Anthropic recommends at least 3–4 sentences explaining what the tool does, when it
  should/shouldn't be used, and what each parameter means — description quality is the
  single biggest factor in tool-use performance), and an `input_schema` (a JSON Schema
  object). Optional `input_examples` provide schema-validated example inputs for complex
  tools. The `tool_choice` parameter controls whether/which tool Claude must use: `auto`
  (Claude decides, default when tools are present), `any` (Claude must use one of the
  provided tools, but Claude picks which), `tool` (forces one specific named tool), or
  `none` (no tool use, default when no tools are given).
  Example: Setting `tool_choice: {"type": "tool", "name": "get_weather"}` forces Claude
  to call the `get_weather` tool on every turn, guaranteeing structured extraction of a
  location argument rather than a free-text reply, even for a question Claude could
  otherwise answer directly.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools

- Concept: Structured outputs (JSON schema output and strict tool use)
  Type: key
  Definition: Two complementary features that constrain Claude's responses to follow a
  specific schema, guaranteeing valid, parseable output without retries: (1) JSON
  Outputs (`output_config.format` with `type: "json_schema"`) constrain Claude's final
  text response to be valid JSON matching a supplied schema — required fields are always
  present, types are guaranteed, and no retry/re-prompting is needed because schema
  compliance is enforced through constrained decoding (a compiled grammar blocks invalid
  tokens during generation); (2) Strict tool use (`strict: true` on a tool definition)
  guarantees that a tool call's name and input parameters exactly match the tool's
  `input_schema`, which is important in multi-step agentic workflows. The two features
  can be combined so Claude calls tools with guaranteed-valid parameters and still
  returns a final structured JSON response. Supported JSON Schema features include basic
  types, `enum`, `const`, `anyOf`/`allOf`, `$ref`/`$def`, several string formats,
  `required`, and `additionalProperties: false`; recursive schemas and most numeric/string
  length constraints are not supported (SDKs strip them and validate client-side
  instead).
  Example: A lead-capture endpoint sets `output_config.format` to a JSON schema requiring
  `name`, `email`, and `demo_requested` (boolean) as required properties with
  `additionalProperties: false`; the API response is guaranteed to be valid JSON matching
  that shape, with no need to parse free text or retry on malformed output.
  Source: https://platform.claude.com/docs/en/build-with-claude/structured-outputs

- Concept: Validation and retry loops for tool calls
  Type: key
  Definition: When a client tool call fails or is malformed, the application signals this
  back to Claude in a `tool_result` block with `"is_error": true` and an instructive error
  message (e.g., "Rate limit exceeded. Retry after 60 seconds." rather than a bare
  "failed"), letting Claude decide how to recover. Two categories are distinguished: tool
  execution errors (the tool itself failed, e.g. a network error) and invalid tool calls
  (e.g. Claude omitted a required parameter) — for the latter, Claude will typically retry
  the call 2–3 times with corrections before giving up and apologizing to the user. To
  eliminate invalid tool calls at the source rather than looping through retries, strict
  tool use (`strict: true`) guarantees tool inputs always match the schema exactly.
  Example: A `search_flights` tool call missing the required `date` parameter is answered
  with `{"type": "tool_result", "tool_use_id": "...", "content": "Error: Missing required
  'date' parameter", "is_error": true}`; Claude reads the error and reissues the tool call
  with a filled-in date rather than failing the whole turn.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls

- Concept: Batch processing (Message Batches API)
  Type: key
  Definition: An asynchronous API for submitting large volumes of Messages requests
  together instead of one at a time, well suited to workloads that don't need an
  immediate response: bulk data processing, large-scale evaluations, or offline
  analyses. The system creates a Message Batch from the submitted requests, processes
  each request independently and asynchronously (most batches finish in under an hour),
  and the caller polls for status and retrieves results once processing ends. A batch is
  limited to 100,000 requests or 256 MB, whichever is reached first; results are
  available once all requests finish or after 24 hours (whichever is first), batches
  expire if not done within 24 hours, and results remain retrievable for 29 days. Using
  the Batches API gives a 50% discount versus standard API pricing on input tokens,
  output tokens, and special tokens, and almost any Messages API feature (vision, tool
  use, system prompts, multi-turn conversations, extended thinking) can be included in a
  batch request.
  Example: A company that needs to classify 50,000 support tickets overnight submits them
  as a single Message Batch instead of 50,000 sequential API calls, cutting cost in half
  and letting the batch complete unattended within the hour, then retrieves all results
  once the batch is done.
  Source: https://platform.claude.com/docs/en/build-with-claude/batch-processing

- Concept: Combining prompt caching with batch processing
  Type: key
  Definition: A cost/efficiency strategy for large-scale or repeated-prompt workloads
  (such as evaluation runs) that combines the Message Batches API with prompt caching
  rather than treating them as separate concerns. Because asynchronous batch requests
  can be processed concurrently and in any order, cache hits on a Batches API request are
  best-effort; Anthropic's recommended approach is to first send a single request
  containing the shared prompt prefix with a 1-hour cache block, and once that completes
  (writing the prefix to the 1-hour cache), submit the rest of the batch so subsequent
  requests can hit the warmed cache. The 1-hour cache is preferred over the 5-minute
  cache for batches because batch completion commonly falls in the 5-minute-to-1-hour
  range. Prompt caching can cut repeated input-token cost by up to 90%, which compounds
  with the Batches API's 50% discount for workloads with a large, consistent shared
  prefix (system prompt, policy text, reference material) across many batched requests.
  Example: An evaluation harness that re-runs the same lengthy system prompt and rubric
  against 10,000 different transcripts first "primes" the cache with a single request
  carrying that shared prefix and a 1-hour cache breakpoint, then submits the 10,000
  transcript-specific requests as a batch so most of them reuse the cached prefix instead
  of reprocessing it from scratch.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-caching

- Concept: Multi-instance review architectures
  Type: key
  Definition: Workflow patterns that use more than one LLM call/instance to improve
  reliability of a task's output, beyond a single generation call: (1) Evaluator-optimizer
  — one LLM call generates a response while a second LLM call evaluates it and gives
  feedback in a loop, effective when evaluation criteria are clear and iterative feedback
  demonstrably improves results (e.g., literary translation critique, iterative search
  refinement) — and ineffective when first-attempt quality already suffices or criteria
  are subjective/unclear; (2) Orchestrator-workers — a central LLM dynamically breaks a
  task into subtasks, delegates each to specialized worker LLM calls, and synthesizes
  their outputs, useful when subtasks can't be predicted in advance (e.g., unpredictable
  multi-file code changes); (3) Parallelization by voting — the same task is run through
  multiple independent LLM calls/prompts and the results are combined via a voting or
  consensus mechanism to balance false positives against false negatives, e.g. having
  several independent instances each flag content for a policy violation and treating any
  flag as a trigger, or having separate model calls each screen for a different concern
  (one processes the query, another screens for inappropriate content) rather than
  overloading a single call. Human oversight is still recommended as a final check across
  all of these architectures.
  Example: A code-review pipeline runs the same vulnerability-scanning prompt against a
  diff through three independent Claude calls; if any of the three flags a potential
  vulnerability, the diff is routed to a human reviewer, trading extra token cost for a
  lower false-negative rate than a single review pass.
  Source: https://www.anthropic.com/research/building-effective-agents

---

## Prerequisites (one level back from the key concepts above)

- Concept: Messages API request structure (system, user/assistant messages)
  Type: prerequisite
  Definition: The Claude Messages API request separates a top-level `system` string
  parameter (standing context/role, not part of the conversational turns) from a
  `messages` array in which every entry has a `role` of exactly `user` or `assistant` and
  a `content` field; there is no `system` role inside the messages array. Understanding
  this structure is a prerequisite for using system prompts/role prompting, tool use
  (whose `tool_use`/`tool_result` blocks live inside `assistant`/`user` message content),
  and structured outputs (whose `output_config` and schema sit alongside this same
  request).
  Example: A request sets `system="You are a customer support agent. Reply politely and
  concisely."` and `messages=[{"role": "user", "content": "What is your return policy?"}]`
  — the role/persona lives in `system`, the actual exchange lives in `messages`.
  Source: https://platform.claude.com/docs/en/api/messages/create

- Concept: Being clear and direct
  Type: prerequisite
  Definition: The foundational prompting principle that precise, explicit, unambiguous
  instructions produce better results than vague or implicit ones. Anthropic frames this
  as treating Claude "like a brilliant but new employee who lacks context on your norms
  and workflows": give contextual information, state exactly what you want (including
  what the output should and should not contain), and provide instructions as sequential
  numbered or bulleted steps. The documented "golden rule" is to show the prompt to a
  colleague with minimal context on the task and have them try to follow it — if a human
  reader is confused, Claude will likely be too. This principle underlies more advanced
  techniques such as multishot prompting, chain-of-thought prompting, and XML tag
  structuring, all of which are ways of making instructions, examples, and context
  clearer and easier for Claude to disambiguate.
  Example: A vague prompt like "Write something about our new app" is rewritten as "Write
  a 150-word marketing email for busy professionals announcing our new productivity app.
  Output only the email body — no subject line, no preamble, no explanation," giving
  Claude an explicit audience, length, and output boundary instead of leaving them
  implicit.
  Source: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices

- Concept: JSON Schema fundamentals
  Type: prerequisite
  Definition: JSON Schema is a vocabulary for annotating and validating the structure of
  JSON documents — defining property names, data types, which properties are required,
  enumerated allowed values, and nested/object structures. It is the standard that
  underlies both the `input_schema` used to define tools and the `output_config.format`
  JSON schema used for structured outputs in the Claude API.
  Example: A schema declaring `{"type": "object", "properties": {"location": {"type":
  "string"}}, "required": ["location"]}` specifies that a valid document must be an
  object with a string `location` property that must be present.
  Source: https://json-schema.org/

- Concept: Tool use (function calling)
  Type: prerequisite
  Definition: A working understanding of how Claude calls tools — receiving a `tool_use`
  block naming a tool and its input, and the application returning a `tool_result` block
  — is the immediate prerequisite for strict tool use, forcing tool use via `tool_choice`,
  and validation/retry handling of tool calls, since all of those refine or react to the
  basic tool-use round trip described in the tool use overview.
  Example: A `get_weather` tool is registered with an `input_schema` requiring a
  `location` string; when a user asks about the weather, Claude's response carries a
  `tool_use` block such as `{"name": "get_weather", "input": {"location": "San
  Francisco, CA"}}`, which the calling application must recognize and execute before the
  conversation can continue — the exact round trip that strict tool use, forced
  `tool_choice`, and error handling all build on.
  Source: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

- Concept: Basic agentic workflow building blocks (prompt chaining, routing, parallelization)
  Type: prerequisite
  Definition: Before combining multiple LLM calls into a review architecture, it helps to
  know the simpler workflow patterns they build on: the augmented LLM (an LLM enhanced
  with retrieval, tools, and memory); prompt chaining (decomposing a task into a sequence
  of LLM calls where each processes the previous call's output, often with a
  programmatic check in between); routing (classifying an input and directing it to a
  specialized follow-up task/prompt); and parallelization by sectioning (splitting a task
  into independent subtasks run concurrently for speed). Evaluator-optimizer,
  orchestrator-workers, and voting-based multi-instance review are compositions or
  variants of these simpler blocks.
  Example: A support-ticket system uses routing to classify a ticket as "billing" or
  "technical" and sends it to a specialized follow-up prompt for that category, before
  any evaluator or voting step is layered on top.
  Source: https://www.anthropic.com/research/building-effective-agents

- Concept: Success criteria and evaluations
  Type: prerequisite
  Definition: Having a measurable, SMART definition of what a "good" output looks like,
  and an automatable way to test outputs against it, is the immediate prerequisite for
  both grounding techniques that reduce false positives/hallucinations and for designing
  a multi-instance review or voting architecture, since a voting/evaluator system needs a
  concrete criterion to evaluate each instance's output against.
  Example: Before building a three-instance voting pipeline to flag policy-violating
  content, a team first defines the measurable target the voting system is meant to hit —
  e.g., "flag at least 99% of true violations while keeping false flags under 1%" — so
  that the voting threshold (how many of three instances must agree) can be tuned against
  that explicit criterion instead of chosen arbitrarily.
  Source: https://platform.claude.com/docs/en/test-and-evaluate/develop-tests

---

## Sources Cited

- https://platform.claude.com/docs/en/test-and-evaluate/develop-tests
- https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting
- https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought
- https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency
- https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools
- https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls
- https://platform.claude.com/docs/en/build-with-claude/batch-processing
- https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- https://www.anthropic.com/research/building-effective-agents
- https://platform.claude.com/docs/en/api/messages/create
- https://json-schema.org/

## Notes on the exam guide PDF

The official exam guide PDF at the provided S3 URL could not be parsed by the fetch
tool (binary/FlateDecode-encoded content); research was scoped from the domain
description provided by the orchestrator plus the official Claude API documentation
listed above. No concept below was left unsourced.
