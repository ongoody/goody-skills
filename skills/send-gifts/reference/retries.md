# Retries and the idempotency key

Make one `idempotency_key` UUID per approved send. It's what stops one
send from becoming two charges.

| What happened | What to do |
|---|---|
| Timeout or unclear result | Retry with the **same** key and the same inputs. If it went through, you get the original result back. |
| Still unclear after that | Stop. Look for it with `goody_order_batches_list`, or contact support. Never switch keys. |
| "This idempotency_key was already used with different inputs" | The inputs changed since that key was used. Check whether the first send went through before approving anything new. |
| Rejected before sending (bad input, changed template) | Fix the inputs, preview again, get a new yes. |
| A failure returned by the send itself | It's saved against that key. Once you're sure nothing was sent, fix, preview, get a new yes, and use a **new** key. |

## Sends with nobody watching

For a scheduled send or a job running on a saved token, approval happens
when the automation is set up, not at each run. Show a sample gift and
the expected spend, and get a yes before turning it on.
