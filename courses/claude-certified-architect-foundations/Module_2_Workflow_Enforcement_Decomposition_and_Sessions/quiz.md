# Module 2 Quizzes

Answers are marked clearly. The tutor must never reveal an answer before the
learner has attempted the question.

---

## Quiz — Lesson 1.4 (Workflows, Enforcement, Handoff)

**Q1.** Why do prompt instructions alone have a non-zero failure rate for
enforcing order-of-operations, and what is the deterministic alternative?

<details><summary>ANSWER</summary>
The model can occasionally skip, misorder, or forget a stated rule under pressure
from other context — prompt-based guidance relies on the model reliably following
it every time. The deterministic alternative is programmatic enforcement: hooks
or prerequisite gates evaluated in code before a tool executes.
</details>

**Q2.** How does a `PreToolUse`-based prerequisite gate guarantee that
`process_refund` can never run before `get_customer` has verified the customer —
even in a permissive permission mode?

<details><summary>ANSWER</summary>
Because hooks are evaluated before deny rules, ask rules, permission mode, and
allow rules in the SDK's permission evaluation order, a hook's `deny` decision
cannot be overridden by any looser setting downstream — including
`bypassPermissions` mode.
</details>

**Q3.** A customer message says "my order arrived damaged AND I was charged
twice." What decomposition approach should the agent use, and how should the
final response be structured?

<details><summary>ANSWER</summary>
Decompose into the constituent concerns (damage claim, double-charge claim),
investigate each using the same shared context (e.g., the already-verified
customer ID), then synthesize the individual findings into one unified
resolution — not two disconnected partial answers.
</details>

**Q4.** What four elements should a structured handoff summary to a human agent
include, and why does it matter that a human receiving the handoff typically
lacks conversation transcript access?

<details><summary>ANSWER</summary>
Customer identifying details, root-cause analysis, relevant amounts (e.g. a
proposed refund), and a recommended action — organized as discrete fields. It
matters because an unstructured note forces the human to reconstruct the
situation from scratch; a structured handoff lets them act immediately.
</details>

---

## Quiz — Lesson 1.5 (Hooks: Interception and Normalization)

**Q1.** What is the difference between a `PostToolUse` hook and a `PreToolUse`
hook in terms of when they run and what they're typically used for?

<details><summary>ANSWER</summary>
`PostToolUse` fires after a tool returns its result and can transform/normalize
it before Claude sees it (e.g., unifying timestamp formats). `PreToolUse` fires
before a tool call executes and can deny/redirect it to enforce compliance rules
(e.g., blocking refunds over a threshold).
</details>

**Q2.** Two MCP tools — `orders` and `billing` — represent order status
differently: one as an integer code, one as a string enum. What mechanism fixes
this before Claude reasons over both, and why does this discrepancy exist in the
first place?

<details><summary>ANSWER</summary>
A `PostToolUse` hook normalizes both to one consistent representation before the
combined data reaches Claude. The discrepancy exists because the MCP spec lets
each tool declare its own `outputSchema` with no requirement that different tools
represent the same real-world data the same way.
</details>

**Q3.** A `PreToolUse` hook denies a `process_refund` call for exceeding $500. What
should the hook's denial *also* do, beyond simply blocking the call, and why?

<details><summary>ANSWER</summary>
It should redirect — the denial reason can instruct Claude toward an alternative
path, such as calling `escalate_to_human`, rather than merely failing silently,
so the agent has a productive next step instead of a dead end.
</details>

**Q4.** What does the MCP `isError` flag let an agent do that a protocol-level
JSON-RPC error would not?

<details><summary>ANSWER</summary>
`isError: true` reports a failure inside the normal tool-result content channel,
letting the calling model see the failure as ordinary result content and
potentially self-correct or reason about it, rather than the call simply erroring
out at the transport level.
</details>

---

## Quiz — Lesson 1.6 (Task Decomposition Strategies)

**Q1.** You're designing a workflow to review 20 pull requests every week, each
checked against the same 5 fixed criteria. Should you use prompt chaining or
dynamic adaptive decomposition? Why?

<details><summary>ANSWER</summary>
Prompt chaining — the subtasks are known and predictable in advance (5 fixed
criteria, same sequence every time), which is exactly what fixed sequential
decomposition suits.
</details>

**Q2.** Why does reviewing a 40-file pull request as one single LLM call risk
worse quality than splitting it into per-file passes plus a cross-file pass?

<details><summary>ANSWER</summary>
Attention dilution: a single call asked to review many files at high depth
produces inconsistent depth across files and misses issues, because effective
attention per file drops as total input grows. Splitting into per-file local
passes (full attention per file) plus a separate cross-file integration pass
(looking specifically for cross-file issues) avoids this.
</details>

**Q3.** For "add comprehensive tests to a legacy codebase" — an open-ended task —
describe the staged, adaptive decomposition strategy from this lesson.

<details><summary>ANSWER</summary>
First map the codebase's structure (directories, modules, dependencies); then
identify high-impact areas (most-used modules, historically buggy code,
untested critical paths) from that map; then create a prioritized plan — and let
that plan continue to adapt as dependencies and complications are discovered
during execution, rather than treating the initial plan as fixed.
</details>

**Q4.** What is the key difference between prompt chaining and dynamic adaptive
decomposition?

<details><summary>ANSWER</summary>
In prompt chaining, the sequence of subtasks is fixed and known in advance. In
dynamic adaptive decomposition, a central orchestrator determines subtasks at run
time based on what earlier steps discover — the subtasks themselves aren't known
upfront.
</details>

---

## Quiz — Lesson 1.7 (Session State, Resumption, Forking)

**Q1.** A developer wants to return to a specific named investigation among
several sessions run in the same directory. Which command do they use, and how
does it differ from simply "continuing"?

<details><summary>ANSWER</summary>
`--resume <session-name>` (e.g., `claude --resume auth-refactor-investigation`).
"Continue" always picks up the most-recently-used session with no name/ID
required; `--resume` returns to one specific session among several.
</details>

**Q2.** A developer has fully analyzed a codebase's authentication module in one
session and now wants to explore two different refactoring approaches (JWT vs
OAuth2) without either exploration contaminating the other or redoing the shared
analysis. What feature should they use?

<details><summary>ANSWER</summary>
`fork_session` — it creates an independent session copying the existing session's
full history from a shared baseline, leaving the original unchanged, so both the
fork and the original can be extended separately.
</details>

**Q3.** A session previously analyzed `payment.py`, which has since been modified
by a teammate. What should the developer do when resuming the session, and why?

<details><summary>ANSWER</summary>
Inform the resumed session about the specific file change (ideally how it
changed), enabling targeted re-analysis of just that file — because the session
persists conversation history, not the filesystem, so its understanding of
`payment.py` is stale until told otherwise.
</details>

**Q4.** When is starting a fresh session with an injected structured summary more
reliable than resuming a prior session?

<details><summary>ANSWER</summary>
When the prior session's accumulated tool results are likely stale, or the
transcript is too large/noisy to be worth carrying forward — resuming risks the
model treating stale tool results in the long history as still current. Resume
instead when prior context is mostly still valid.
</details>
