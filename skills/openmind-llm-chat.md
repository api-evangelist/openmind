---
name: Run a multi-provider LLM chat completion
description: Call the OpenMind unified LLM endpoint to run an OpenAI-format chat completion against any supported provider.
api: OpenMind LLM API
base_url: https://api.openmind.com
operations:
  - POST /api/core/{provider}/chat/completions
auth: OpenMind API key (x-api-key header or Authorization Bearer)
---

# Run a multi-provider LLM chat completion

Use the OpenMind **LLM API** to send an OpenAI-compatible chat completion to any
supported provider through one interface.

## Steps

1. **Authenticate.** Include your OpenMind API key (`om1_live_...`) either as
   `x-api-key: <KEY>` or `Authorization: Bearer <KEY>`. Set
   `Content-Type: application/json`.

2. **Pick a provider and model.** Set the path segment `{provider}` to one of
   `openai`, `deepseek`, `gemini`, `xai`, `nearai`, or `openrouter`, then choose
   a `model` supported by that provider (e.g. `gpt-5`, `gemini-2.5-pro`,
   `anthropic/claude-sonnet-4.5` via `openrouter`). Model names are matched by
   prefix.

3. **Send the request.** `POST /api/core/{provider}/chat/completions` with an
   OpenAI-format body: `{"model": ..., "messages": [{"role":"user","content":...}],
   "temperature": ..., "max_tokens": ...}`.

4. **Track cost.** Usage is billed in OMCU by input/output tokens per the
   pricing table; check `GET /api/core/account/balance` to monitor credits.

## Rules

- Errors use `{"error": "<message>"}`; 429 means the per-plan requests/second
  ceiling was exceeded — back off and retry.
- No idempotency key is supported; do not assume safe automatic retries of
  non-idempotent operations.
