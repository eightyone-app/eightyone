# Errors

Generated from the problem catalog (`src/capabilities/problem-catalog.ts`): don't edit by hand.

Errors are [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) problem details (`application/problem+json`). Branch on `code`: it's stable. `detail` is written for a person and may change. `type` links to the code's page.

```json
{ "type": "https://docs.eightyone.app/api/problems/missing_permission", "title": "forbidden", "status": 403,
  "detail": "Needs inventory:adjust", "code": "missing_permission" }
```

| Code | Status | Meaning |
| --- | --- | --- |
| [`bad_request`](problems/bad_request.md) | 400 | The request was understood, but EightyOne refused it: a business rule says no. `detail` says which rule. |
| [`invalid_input`](problems/invalid_input.md) | 400 | The request doesn't match the capability's input schema. |
| [`unauthenticated`](problems/unauthenticated.md) | 401 | No valid credential: the API key or token is missing, expired or revoked. |
| [`forbidden`](problems/forbidden.md) | 403 | The caller is known but may not do this in this company. |
| [`missing_permission`](problems/missing_permission.md) | 403 | The user (or the key, narrowed to fewer permissions) lacks a permission this capability needs. `detail` names it. |
| [`not_found`](problems/not_found.md) | 404 | Nothing with that id in this company, or nothing the caller may see. |
| [`unknown_capability`](problems/unknown_capability.md) | 404 | `capabilities_call` named a capability that doesn't exist, isn't available over MCP, or the caller can't run. |
| [`conflict`](problems/conflict.md) | 409 | The request clashes with the current state (for example, something already exists). |
| [`idempotency_in_progress`](problems/idempotency_in_progress.md) | 409 | A request with the same `Idempotency-Key` is still running. |
| [`idempotency_conflict`](problems/idempotency_conflict.md) | 409 | The `Idempotency-Key` was used for a different capability or caller. |
| [`unprocessable`](problems/unprocessable.md) | 422 | The request is well-formed but can't be done as asked. |
| [`idempotency_key_reused`](problems/idempotency_key_reused.md) | 422 | The `Idempotency-Key` was sent again with a different body. |
| [`idempotency_unsupported`](problems/idempotency_unsupported.md) | 422 | This capability doesn't take an `Idempotency-Key`: reads are safe to repeat, and a few writes can't be replayed. |
| [`dry_run_unsupported`](problems/dry_run_unsupported.md) | 422 | `Prefer: dry-run` (or `dryRun: true`) was asked of a capability that can't be previewed. |
| [`rate_limited`](problems/rate_limited.md) | 429 | Too many requests from this caller. |
| [`internal_error`](problems/internal_error.md) | 500 | Something failed on EightyOne's side. It's logged with the request's `X-EightyOne-Transaction-ID`. |
