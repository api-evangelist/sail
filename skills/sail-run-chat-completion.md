---
name: Run a chat completion on Sail
description: Pick a served model and run an OpenAI-compatible chat completion, choosing a completion window to trade latency for cost.
api: openapi/sail-openapi-original.json
operations: [listModels, createChatCompletion]
---

# Run a chat completion on Sail

Sail exposes an OpenAI-compatible Chat Completions endpoint at
`https://api.sailresearch.com/v1`. Existing OpenAI SDKs work as drop-in clients.

## Auth
Send `Authorization: Bearer $SAIL_API_KEY` on every request (scheme `BearerAuth`).

## Steps
1. **Discover models** — call `listModels` (`GET /models`) to get the currently
   served open-source models and which completion windows each supports.
2. **Create the completion** — call `createChatCompletion`
   (`POST /chat/completions`) with `model`, `messages`, and optional
   `max_completion_tokens`, `temperature`, `top_p`, tools, and
   `response_format` (JSON schema for structured output).
3. **Choose cost vs latency** — set `metadata.completion_window` to one of
   `asap | priority | standard | flex`. `flex`/`standard` cost less when you can
   wait; `asap` is fastest.
4. **Stream if needed** — set `stream: true` to receive `chat.completion.chunk`
   objects over Server-Sent Events.

## Rules
- **Idempotency:** set an `Idempotency-Key` header (≤255 chars) so retries return
  the same result instead of re-running inference.
- **Errors:** failures come back as an OpenAI-style envelope
  `{ error: { message, type, param, code } }`. On `429` honor `Retry-After`.
- Not supported: `n>1`, `frequency_penalty`/`presence_penalty`/`logit_bias`/`stop`/`seed`.
