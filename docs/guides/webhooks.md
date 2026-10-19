# Webhooks

**What it is:** EightyOne tells your systems when something happens, such as a task changing status, a work order released, a machine stopping or a lot created. It does this by sending an HTTPS `POST` to a URL you choose. Each message is signed, so your system can check it really came from EightyOne.

**Where:** in the dashboard, **Platform → API → Webhooks**, or the API's `webhooks.*` capabilities. You need **Manage webhooks** to create or change one.

## Setting one up

1. **New webhook:** give it a name, your HTTPS endpoint, and the events it should receive. Private and internal addresses are refused.
2. **Send test:** sends a signed `webhook.test` message to that endpoint only, so you can check your receiver.
3. **Recent deliveries:** each webhook lists what it sent, with the result. Filter by event type or status to find a failure.

A delivery your endpoint doesn't answer with a 2xx is retried after 1 minute, 5 minutes, 30 minutes, 2 hours and 24 hours. Then it's marked failed, and the webhook counts the failure.

## What a delivery looks like

Messages follow the [Standard Webhooks](https://www.standardwebhooks.com/) format:

```
POST /your/endpoint
webhook-id: msg_2b9f…
webhook-timestamp: 1760860800
webhook-signature: v1a,<base64 signature>
Content-Type: application/json

{ "type": "task.status_changed", "timestamp": "2026-10-19T08:00:00.000Z", "data": { … } }
```

- **`type`** is the event; **`data`** is its content.
- Ids in `data` are the same ones the API uses: task and machine external ids, SKUs, work order ids. They are never internal database numbers.
- **`webhook-id`** stays the same on every retry of a message. Use it to ignore a message you've already handled.
- Messages can arrive out of order. Use `timestamp`, or fetch the current state from the API, when order matters.

The [event reference](../reference/events.md) lists every event and its fields; the [OpenAPI document](../reference/openapi.json) has them too, in its `webhooks` section.

## Checking a signature

EightyOne signs every message with its own key (Ed25519). There is **no secret** to copy or store: you verify with EightyOne's **public key**.
- The public key is shown on each webhook.
- Your system can fetch it from `https://api.eightyone.app/api/v1/webhook-signing-keys`. That URL needs no login.

To check a message:
1. Join `webhook-id`, `webhook-timestamp` and the raw request body with dots: `<id>.<timestamp>.<body>`.
2. The `webhook-signature` header holds one or more `v1a,<signature>` entries. Accept the message if any of them verifies against any published key.
3. Refuse a timestamp older than a few minutes, so an old message can't be replayed.

In Node.js:

```js
import { createPublicKey, verify } from "node:crypto";

// Fetch once and cache. During a key change two keys are listed, and both sign.
const response = await fetch("https://api.eightyone.app/api/v1/webhook-signing-keys");
const keys = (await response.json()).keys.map((k) =>
    createPublicKey({
        // An Ed25519 public key in the SPKI format Node expects: a fixed prefix plus the 32 key bytes.
        key: Buffer.concat([Buffer.from("302a300506032b6570032100", "hex"), Buffer.from(k.publicKey.slice(5), "base64")]),
        format: "der",
        type: "spki",
    }),
);

export function isFromEightyOne(headers, rawBody) {
    const id = headers["webhook-id"];
    const timestamp = Number(headers["webhook-timestamp"]);
    if (Math.abs(Date.now() / 1000 - timestamp) > 300) return false; // older than 5 minutes
    const signed = Buffer.from(`${id}.${timestamp}.${rawBody}`);
    return headers["webhook-signature"].split(" ").some((entry) => {
        const [version, signature] = entry.split(",");
        return version === "v1a" && keys.some((key) => verify(null, signed, key, Buffer.from(signature, "base64")));
    });
}
```

Verify the **raw** body, exactly as received. Parsing it and serializing it again changes the bytes, and the signature won't match.

## What changed (October 2026)

Webhooks moved to Standard Webhooks. If you built a receiver before, update it:

- **Signature:** `X-EightyOne-Signature` (HMAC with a per-webhook secret) is replaced by `webhook-signature`, verified with EightyOne's public key. Existing webhook secrets were removed, and none are issued any more.
- **Body:** `{ event, eventId, createdAt, data }` is now `{ type, timestamp, data }`. `eventId` is now the `webhook-id` header.
- **Content:** events carry the API's ids. For example, `task.created` has `task.id` (the external id) and `task.type: { id, name }`, instead of database ids such as `typeId`.
- **Reliability:** a message is recorded with the change that caused it. A change that's undone sends nothing, and a message is never lost when EightyOne restarts.
