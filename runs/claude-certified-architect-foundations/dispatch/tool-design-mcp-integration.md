## Domain 2: Tool Design & MCP Integration (18%)

Description: Covers designing effective tool interfaces and descriptions, implementing structured MCP
error responses, distributing tools across agents and configuring tool_choice, integrating MCP servers
into Claude Code and agent workflows, and selecting/applying Claude Code's built-in tools.

Task statements: 5 · Bullets captured: 43 (20 Knowledge, 23 Skills)

### Task statements

- Task Statement 2.1: Design effective tool interfaces with clear descriptions and boundaries
  Knowledge of:
    - 2.1-K1: Tool descriptions as the primary mechanism LLMs use for tool selection; minimal descriptions lead to unreliable selection among similar tools
    - 2.1-K2: The importance of including input formats, example queries, edge cases, and boundary explanations in tool descriptions
    - 2.1-K3: How ambiguous or overlapping tool descriptions cause misrouting (e.g., analyze_content vs analyze_document with near-identical descriptions)
    - 2.1-K4: The impact of system prompt wording on tool selection: keyword-sensitive instructions can create unintended tool associations
  Skills in:
    - 2.1-S1: Writing tool descriptions that clearly differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives
    - 2.1-S2: Renaming tools and updating descriptions to eliminate functional overlap (e.g., renaming analyze_content to extract_web_results with a web-specific description)
    - 2.1-S3: Splitting generic tools into purpose-specific tools with defined input/output contracts (e.g., splitting a generic analyze_document into extract_data_points, summarize_content, and verify_claim_against_source)
    - 2.1-S4: Reviewing system prompts for keyword-sensitive instructions that might override well-written tool descriptions

- Task Statement 2.2: Implement structured error responses for MCP tools
  Knowledge of:
    - 2.2-K1: The MCP isError flag pattern for communicating tool failures back to the agent
    - 2.2-K2: The distinction between transient errors (timeouts, service unavailability), validation errors (invalid input), business errors (policy violations), and permission errors
    - 2.2-K3: Why uniform error responses (generic "Operation failed") prevent the agent from making appropriate recovery decisions
    - 2.2-K4: The difference between retryable and non-retryable errors, and how returning structured metadata prevents wasted retry attempts
  Skills in:
    - 2.2-S1: Returning structured error metadata including errorCategory (transient/validation/permission), isRetryable boolean, and human-readable descriptions
    - 2.2-S2: Including retriable: false flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately
    - 2.2-S3: Implementing local error recovery within subagents for transient failures, propagating to the coordinator only errors that cannot be resolved locally along with partial results and what was attempted
    - 2.2-S4: Distinguishing between access failures (needing retry decisions) and valid empty results (representing successful queries with no matches)

- Task Statement 2.3: Distribute tools appropriately across agents and configure tool choice
  Knowledge of:
    - 2.3-K1: The principle that giving an agent access to too many tools (e.g., 18 instead of 4-5) degrades tool selection reliability by increasing decision complexity
    - 2.3-K2: Why agents with tools outside their specialization tend to misuse them (e.g., a synthesis agent attempting web searches)
    - 2.3-K3: Scoped tool access: giving agents only the tools needed for their role, with limited cross-role tools for specific high-frequency needs
    - 2.3-K4: tool_choice configuration options: "auto", "any", and forced tool selection ({"type": "tool", "name": "..."})
  Skills in:
    - 2.3-S1: Restricting each subagent's tool set to those relevant to its role, preventing cross-specialization misuse
    - 2.3-S2: Replacing generic tools with constrained alternatives (e.g., replacing fetch_url with load_document that validates document URLs)
    - 2.3-S3: Providing scoped cross-role tools for high-frequency needs (e.g., a verify_fact tool for the synthesis agent) while routing complex cases through the coordinator
    - 2.3-S4: Using tool_choice forced selection to ensure a specific tool is called first (e.g., forcing extract_metadata before enrichment tools), then processing subsequent steps in follow-up turns
    - 2.3-S5: Setting tool_choice: "any" to guarantee the model calls a tool rather than returning conversational text

- Task Statement 2.4: Integrate MCP servers into Claude Code and agent workflows
  Knowledge of:
    - 2.4-K1: MCP server scoping: project-level (.mcp.json) for shared team tooling vs user-level (~/.claude.json) for personal/experimental servers
    - 2.4-K2: Environment variable expansion in .mcp.json (e.g., ${GITHUB_TOKEN}) for credential management without committing secrets
    - 2.4-K3: That tools from all configured MCP servers are discovered at connection time and available simultaneously to the agent
    - 2.4-K4: MCP resources as a mechanism for exposing content catalogs (e.g., issue summaries, documentation hierarchies, database schemas) to reduce exploratory tool calls
  Skills in:
    - 2.4-S1: Configuring shared MCP servers in project-scoped .mcp.json with environment variable expansion for authentication tokens
    - 2.4-S2: Configuring personal/experimental MCP servers in user-scoped ~/.claude.json
    - 2.4-S3: Enhancing MCP tool descriptions to explain capabilities and outputs in detail, preventing the agent from preferring built-in tools (like Grep) over more capable MCP tools
    - 2.4-S4: Choosing existing community MCP servers over custom implementations for standard integrations (e.g., Jira), reserving custom servers for team-specific workflows
    - 2.4-S5: Exposing content catalogs as MCP resources to give agents visibility into available data without requiring exploratory tool calls

- Task Statement 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively
  Knowledge of:
    - 2.5-K1: Grep for content search (searching file contents for patterns like function names, error messages, or import statements)
    - 2.5-K2: Glob for file path pattern matching (finding files by name or extension patterns)
    - 2.5-K3: Read/Write for full file operations; Edit for targeted modifications using unique text matching
    - 2.5-K4: When Edit fails due to non-unique text matches, using Read + Write as a fallback for reliable file modifications
  Skills in:
    - 2.5-S1: Selecting Grep for searching code content across a codebase (e.g., finding all callers of a function, locating error messages)
    - 2.5-S2: Selecting Glob for finding files matching naming patterns (e.g., **/*.test.tsx)
    - 2.5-S3: Using Read to load full file contents followed by Write when Edit cannot find unique anchor text
    - 2.5-S4: Building codebase understanding incrementally: starting with Grep to find entry points, then using Read to follow imports and trace flows, rather than reading all files upfront
    - 2.5-S5: Tracing function usage across wrapper modules by first identifying all exported names, then searching for each name across the codebase

---

