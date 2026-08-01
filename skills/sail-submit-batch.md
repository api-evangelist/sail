---
name: Submit and poll a batch of requests
description: Submit up to 100,000 Responses-API requests as one asynchronous batch, then poll status and fetch per-request results by custom_id.
api: openapi/sail-openapi-original.json
operations: [createBatch, getBatch, getBatchRequestResult, listBatches]
---

# Submit and poll a batch of requests

The Batch API processes large volumes of Responses-API requests asynchronously
(up to 100,000 per call) at lower cost.

## Auth
`Authorization: Bearer $SAIL_API_KEY`.

## Steps
1. **Create the batch** — call `createBatch` (`POST /batches`) with the request
   items; give each a caller-chosen `custom_id` for correlation.
2. **Poll status** — call `getBatch` (`GET /batches/{batch_id}`) until the batch
   completes.
3. **Fetch a result** — call `getBatchRequestResult`
   (`GET /batches/{batch_id}/{custom_id}`) to retrieve one request's output.
4. **List batches** — call `listBatches` (`GET /batches`), paginating with
   `after_id` / `before_id` / `limit` (cursor pagination).

## Rules
- **Idempotency:** set `Idempotency-Key` (≤255 chars) on `createBatch`.
- **Errors:** OpenAI-style `{ error: { message, type, param, code } }`; on `429`
  honor `Retry-After` and back off.
- Pagination is cursor-based — carry `after_id`/`before_id` forward, do not assume offsets.
