# Dry run not supported (`dry_run_unsupported`)

HTTP 422.

`Prefer: dry-run` (or `dryRun: true`) was asked of a capability that can't be previewed.

**What to do:** Leave it out. Agent-callable writes can always be previewed; this applies to a few REST-only ones.

[All errors](../errors.md)
