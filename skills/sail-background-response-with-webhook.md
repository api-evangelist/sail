---
name: Run a background Response and receive a completion webhook
description: Submit a long-horizon Responses-API task in background mode and get notified via a completion webhook, with polling as a fallback.
api: openapi/sail-openapi-original.json
operations: [createResponse, getResponse]
---

# Run a background Response and receive a completion webhook

Use the OpenAI-compatible Responses API for long-running / background inference.

## Auth
`Authorization: Bearer $SAIL_API_KEY`.

## Steps
1. **Create the response** — call `createResponse` (`POST /responses`) with
   `model` and `input`. Set `background=true` to run asynchronously; the call
   returns `202` with a response `id`.
2. **Register a webhook (optional)** — include `metadata.completion_webhook`
   (your HTTPS URL) and `metadata.webhook_token`. On completion Sail POSTs the
   full response JSON to your URL with `Authorization: Bearer <webhook_token>`.
3. **Poll as fallback** — call `getResponse` (`GET /responses/{response_id}`)
   until `status` is terminal. This returns the same object the webhook delivers.

## Rules
- **Idempotency:** set `Idempotency-Key` (≤255 chars) on `createResponse`.
- **Webhook delivery:** up to 3 retries, 30s timeout, duplicates possible — log
  response IDs and dedupe; return `2xx` promptly to stop retries.
- **Errors/timeouts:** `408`/`504` mean still-waiting — prefer background mode
  and a longer completion window; retry with backoff.
- Not supported: `previous_response_id` chaining, server-side tools, delete/cancel.
