---
name: Execute a Minicor desktop-automation workflow
description: >-
  Trigger a Laminar/Minicor desktop-automation workflow by id with API-key auth,
  optionally binding a configuration store and receiving an async completion
  callback, then handle the structured JSON result and error cases.
api: openapi/minicor-openapi.json
operations:
  - executeExternalFlow
  - getConfigurationKeysExternal
  - bulkUpdatePropertiesExternal
---

# Execute a Minicor workflow

Minicor (Laminar Run) drives legacy desktop software with a single API call. Use
this skill to run a workflow reliably.

## Prerequisites
- A workflow id (`workflowId`) copied from the workflow editor's TRIGGER URL panel.
- A Laminar API key from workspace settings.

## Auth
Pass the key either as the `api_key` query parameter (documented primary method)
or the `X-API-KEY` header. Never commit keys to version control; rotate periodically.

## Steps
1. (Optional) Prepare configuration: if the workflow needs stored variables/credentials,
   ensure a configuration store exists and set its values with **`bulkUpdatePropertiesExternal`**
   (`PUT /configurations/{externalId}/properties/external`). Inspect current keys with
   **`getConfigurationKeysExternal`** (`GET /configurations/{externalId}/keys/external`).
2. Execute the workflow with **`executeExternalFlow`**
   (`POST /workflow/execute/external/{workflowId}`). Send your input as the request body.
   Useful query parameters:
   - `configuration_id` — bind a configuration store to the run.
   - `response_type` — response format (default `json`).
   - `start_from_step` / `end_at_step` — run a partial range of steps.
   - `callback_url` — receive an asynchronous completion callback for long-running
     desktop workflows instead of blocking.
3. Read the result: a `200` returns the structured JSON output of the workflow.

## Errors
- `400 Execution failed` — the workflow could not complete; inspect the run trace.
- `401 Unauthorized` — missing/invalid API key; check the `api_key`/`X-API-KEY` value.
- `404` (configuration ops) — the configuration store or property key does not exist.

## Notes
- No idempotency-key contract is documented; avoid blind retries on non-idempotent
  desktop actions — prefer the `callback_url` async pattern and check run traces.
- See `conventions/minicor-conventions.yml` and `errors/minicor-problem-types.yml`.

## Example
```bash
curl -X POST 'https://api.laminar.run/workflow/execute/external/{workflowId}?api_key=<your_api_key>' \
  -H 'Content-Type: application/json' \
  -d '{ "data": "your workflow input" }'
```
