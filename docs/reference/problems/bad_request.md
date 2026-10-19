# Bad request (`bad_request`)

HTTP 400.

The request was understood, but EightyOne refused it: a business rule says no. `detail` says which rule.

**What to do:** Read `detail`: it names the rule (a blocked task, a missing reason, stock that isn't there). Change the request, don't retry it as is.

[All errors](../errors.md)
