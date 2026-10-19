# Getting started

## Accounts and companies

- **One company is one tenant:** a factory or a managed-services business, with its own people, machines, stock and settings.
- **Companies are created by EightyOne staff only.** To get one for a client, ask EightyOne.
- A person can belong to several companies, with different permissions in each.

## Credentials

| | Use it for | How |
| --- | --- | --- |
| **API key** | Integrations and headless agents | A signed-in user creates it under **Platform → API → Keys**. It acts as that user, in one company, and can be narrowed to fewer permissions than the user has. Send it as `Authorization: Bearer <key>`. |
| **OAuth sign-in** | Interactive agents (Claude, Cursor) | The MCP client opens an EightyOne sign-in page; the user approves. |
| **Service account** | Integrations that shouldn't act as a named person | A non-human user in a permission group (**Platform → API → Service accounts**), with its own keys. |

EightyOne stores no copy of your keys: they're issued and checked by EightyOne's identity provider. A key shows once, when it's created. A revoked key stops working at once.

## Your first call

```bash
curl -H "Authorization: Bearer $EIGHTYONE_KEY" https://api.eightyone.app/api/v1/me
```

`me` returns the company the key acts in, the user, and their permissions. Every other call is scoped to that company: you never pass a company id.

```bash
curl -H "Authorization: Bearer $EIGHTYONE_KEY" "https://api.eightyone.app/api/v1/tasks?limit=20"
```

## Conventions

- **Ids** are external ids: short strings such as `rpbVkyAl`. Products are addressed by SKU. Database numbers never appear in the API, except where a resource has no other id (reason codes, downtime events, purchase order lines). Each capability's reference says which.
- **Paged lists** take `limit` and `offset` and return `{ data, pagination: { limit, offset, total, hasMore } }`.
- **Errors** are [problem details](../reference/errors.md). Branch on `code`.
- **Retries:** send an `Idempotency-Key` header on writes. A retry with the same key and body returns the first answer, for 24 hours.
- **Dry runs:** `Prefer: dry-run` (REST) or `dryRun: true` (MCP) previews a write. It answers what would change and what it would set off, and changes nothing. Every write an agent can call can be previewed.
- **Rate limits** answer `429` with `rate_limited`. Back off and retry.
- **Tracing:** send `X-EightyOne-Transaction-ID` to tie your logs to ours. Each response carries one either way.

## Logging and audit

Every write is recorded under the person the key or agent acts as, in the company's **Audit Logs**. EightyOne also keeps a call log of every API and MCP call: who, which capability, the result and the duration. For writes, it also keeps the input (secrets redacted) for 90 days.
