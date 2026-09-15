# Module 1 Hands-On Exercise

## Design a minimal tool-using request

You are building a small assistant that looks up a customer's order status.

1. Write out (on paper/in a text file — no execution required) a Messages API
   request skeleton that includes:
   - A `system` parameter giving the assistant a role appropriate to a
     customer-support context.
   - A single `messages` entry with a plausible user question about an order.
2. Define one tool, `get_order_status`, with a `name`, a `description` that
   follows the "clear and direct" principle (state what it does, when to use
   it, and what its parameter means), and an `input_schema` requiring an
   `order_id` string.
3. Trace the full round trip in prose: what does Claude return first, what
   does your application do with it, and what does it send back before
   Claude produces its final answer?
4. Identify, in your own trace, exactly where the "context window" is
   growing at each step (system prompt, first user message, tool_use block,
   tool_result block, final assistant text) — list them in accumulation
   order.

### What a strong answer includes
- A `system` string that assigns a role, not just a topic.
- A tool description of at least a couple of sentences, not a one-line
  label — covering what the tool does, when it should be used, and what the
  parameter means.
- A JSON Schema `input_schema` with a `required` array.
- A correctly ordered trace: user message → `tool_use` block → application
  executes → `tool_result` block → final `assistant` text — matching the
  round trip described in Lesson 1.2.
- Correct identification that every one of those pieces adds to the same,
  accumulating context window (Lesson 1.3), none of them replace what came
  before.
