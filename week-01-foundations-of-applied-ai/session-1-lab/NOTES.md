# Lab notes

*Fill this in as you go — it's part of your submission (Lab 0).*

## Statelessness
What happened when you sent only the latest message vs. the full history?

The LLM API is completely stateless each call is independent with no
memory of prior requests. When I sent only the latest user message (e.g. just
`{"role": "user", "content": "can you give me an example?"}`), the bot had no
idea what "example" referred to and answered generically. When I sent the full
`history` list (system + every prior turn), the bot maintained context and could
reference earlier parts of the conversation.

The "memory" is entirely client-side, we rebuild the entire message array on
every turn by appending to `history` and passing the whole thing back. The API
itself retains nothing between calls.

## Temperature
How did `--temp 0.2` compare to `--temp 1.3` on the same prompt?

With the prompt "Explain recursion in 3 sentences":

- temp 0.2 : Responses were tight, focused, and nearly identical across
  repeated runs. The wording stayed close to a textbook explanation each time.  
- temp 1.3 : Responses were noticeably more creative and varied: it used
  analogies (dreams-within-dreams, Russian dolls), changed structure each run.

Takeaway: low temperature = focused and repeatable; high temperature =
creative but unpredictable. Pick temperature based on the task.

## Tokens
What did you notice about token counts as prompts got longer?

Token counts grew linearly with conversation length because every API call
re-sends the full message history. In a 6-turn conversation on the same topic
the total prompt tokens roughly tripled compared to turn 1, since each prior
user + assistant message was re-submitted on every subsequent call.

I observed this via response.usage (returned by the API):
1. Turn 1: ~30 prompt tokens, ~80 completion tokens
2. Turn 3: ~180 prompt tokens, ~90 completion tokens
3. Turn 6: ~450 prompt tokens, ~100 completion tokens

## Anything that surprised you or broke

- Newlines in input:Multi-line input from the terminal doesn't work well with
  `input()`; pressing Enter submits immediately. Not a bug in the code, just a
  CLI limitation worth noting.
- Empty input: Pressing Enter without typing sent an empty string to the API.
  Adding a `continue` guard for blank input would prevent a wasted API call.
