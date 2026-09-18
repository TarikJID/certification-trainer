# Module 8 Hands-On Exercise

## Scenario: Hardening a Long-Running Customer Support Session

You're improving a customer-support agent that handles long, multi-issue
conversations and must decide when to escalate to a human.

### Part A — Context preservation

1. A customer conversation spans 45 turns and includes: an order ID, a promised
   refund amount, a promised delivery date, and a long back-and-forth about
   troubleshooting a login issue (unrelated to the refund). Design a context
   structure that protects the transactional facts (order ID, refund amount,
   delivery date) from being lost during summarization, and explain which
   concept from Lesson 5.1 you're applying and why plain reliance on the running
   summary would fail.
2. The same session calls `mcp__orders__lookup`, which returns 38 fields. Only 6
   are relevant to this case. Show what you'd keep and design the trimming rule
   you'd apply before the result enters context.
3. This session later needs to synthesize findings from three separate tool
   calls into one response to the customer. Describe how you'd structure that
   synthesis input to counter the "lost in the middle" effect.

### Part B — Escalation design

4. Write the escalation trigger taxonomy as a decision checklist an engineer
   could hand to another engineer implementing this agent, and give one example
   scenario that should NOT trigger escalation despite being complex.
5. A frustrated customer says "This service is terrible, nothing works" but
   hasn't explicitly asked for a human, and the actual issue (a stuck password
   reset) is well within the agent's ability to resolve. Write the agent's
   correct response, and explain why escalating immediately here would be wrong
   per Lesson 5.2.
6. Write two few-shot `<example>` blocks for the system prompt: one showing an
   explicit-request escalation, and one showing a policy-silence escalation
   (invent a plausible policy-gap scenario).
7. A lookup by email returns two matching customer accounts. Write the exact
   clarifying question the agent should ask, and explain what heuristic-based
   approach you're deliberately avoiding.

Write your answers as a short design document, including your case-facts JSON
structure and the two few-shot examples in full.
