# Idempotency not supported (`idempotency_unsupported`)

HTTP 422.

This capability doesn't take an `Idempotency-Key`: reads are safe to repeat, and a few writes can't be replayed.

**What to do:** Send it without the header.

[All errors](../errors.md)
