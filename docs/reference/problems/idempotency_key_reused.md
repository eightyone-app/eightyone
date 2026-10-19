# Idempotency key reused (`idempotency_key_reused`)

HTTP 422.

The `Idempotency-Key` was sent again with a different body.

**What to do:** A retry must send exactly the same body; a new operation needs a new key.

[All errors](../errors.md)
