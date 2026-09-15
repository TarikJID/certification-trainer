# Module 1 Quizzes

Answers are provided for the tutor's use in checking learner attempts. Do not
reveal an answer until the learner has made a genuine attempt at the
question.

## Quiz 1.1 — The Messages API and Clear Prompting

1. Where does the "role" or persona of a conversation live in a Messages API
   request — in the `system` parameter, or as a message with `role: "system"`
   in the `messages` array?
2. True or false: within a single session, each new turn's input includes
   only the current message, not prior turns.
3. According to Anthropic's "golden rule" for clear prompting, how should you
   test whether a prompt is clear enough?
4. Why is role prompting (e.g., "You are a data scientist...") described as
   one of the most powerful uses of the system prompt?

### Answers
1. In the top-level `system` string parameter. There is no `system` role
   inside the `messages` array — every message there is `user` or
   `assistant`.
2. False. Each turn's input is all previous history plus the new message —
   context accumulates and does not reset automatically within a session.
3. Show the prompt to a colleague with minimal context on the task and have
   them try to follow it; if a human reader is confused, Claude will likely
   be too.
4. Because even a single sentence assigning a role measurably changes the
   character and quality of Claude's responses, turning it into a
   domain-focused assistant rather than a generalist — a small amount of
   effort with an outsized effect on output quality.

---

## Quiz 1.2 — Tool Use and the Agentic Loop

1. What are the three content blocks/fields involved in a single tool-use
   round trip, and in what order do they occur?
2. What does the default `tool_choice` value of `auto` mean?
3. What JSON Schema keyword marks a property as mandatory in a tool's
   `input_schema`?
4. In the general "agentic loop" pattern, what are the two things the model
   does on each iteration before deciding whether to continue or stop?

### Answers
1. (1) Claude emits a `tool_use` block naming the tool and its arguments; (2)
   the calling application executes the tool and returns a `tool_result`
   block; (3) Claude incorporates the result and continues (possibly with
   another tool call, or a final text response).
2. Claude decides for itself, per turn, whether to call a tool at all — it is
   not forced to use any particular tool, or to use a tool rather than reply
   directly.
3. `required` — an array of property names that must be present.
4. It selects and invokes a tool (or tools), then incorporates the result(s)
   into its next decision about whether to continue looping or stop.

---

## Quiz 1.3 — The Context Window, Tokens, and Extended Thinking

1. What four kinds of content accumulate in the context window over a
   session?
2. What is a token, in terms of what it measures?
3. True or false: thinking-block tokens are free and do not count as output
   tokens.
4. Why do beta features like compaction and context editing require an
   `anthropic-beta` header?

### Answers
1. The system prompt, tool definitions, conversation history (messages), and
   the model's own tool inputs/outputs across every turn.
2. The base unit the API uses to measure the size of a request or response —
   every part of a request and the model's output is measured in tokens, and
   it is the unit context-window limits and cost are expressed in.
3. False. Thinking tokens are billed as output tokens.
4. Because they are opt-in, versioned capabilities rather than default
   behavior — a request that omits the required beta header will not have
   the feature applied, silently.
