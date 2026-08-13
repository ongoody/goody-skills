---
name: gifting-budget-guardrails
description: Keep automated Goody gifting inside its intended spend — sizing batches, checking totals before committing, choosing payment methods, and auditing what has already been sent. Use when setting up gifting automation, when a send is large or recurring, or when asked how much has been spent on gifts.
---

# Budget guardrails for automated gifting

Automated gifting fails in one direction: it spends more than intended,
quietly, at a scale no one reviews. These are the checks that catch it
before `goody_order_batches_create` charges.

## The one-way door

`goody_order_batches_create` charges the payment method and generates
gift links. It cannot be undone from these tools. `goody_orders_cancel`
covers individual orders under Goody's own cancellation rules — it is not
an undo for a committed batch.

Everything below exists to make sure the number the user approves is the
number that gets charged.

## Before committing

### 1. Price it, and do the arithmetic out loud

`goody_order_batches_preview` returns `cart_price` and `total_price`, and
gives you a `preview_url` the user can open. It charges nothing.

State the total **and** its shape:

> 47 recipients × $25 Gift of Choice = $1,175 plus fees — total $1,283.50.

A single number invites a reflexive yes. A number with its multiplier
makes an accidental 470-person send visible.

### 2. Sanity-check the recipient count against the source

Recipient lists assembled from a query are where blowups start. Before
pricing, restate what you filtered on and how many rows came back. If the
count is far off what the user implied — they said "a few candidates" and
you have 200 — stop and ask. Do not price it and hope they notice.

### 3. Pick the payment method deliberately

`goody_payment_methods_list`, then pass `payment_method_id` explicitly and
name it in the confirmation. Teams commonly separate a recruiting card
from a people-ops card, and a mis-attributed $2,000 batch is a real
accounting cleanup even though the gift itself was fine.

### 4. Confirm against the current gift

`confirmed: true` requires a yes to the exact gift and total shown. If
anything changed after their yes — the product, the count, the amount,
the send method — show it again and re-ask. A stale approval is not an
approval.

## Sizing recurring commitments

For autogift rules, the exposure is `expected_spend_per_send` × how often
it fires over a year, and it is approved once at activation and never
again. Compute the annual figure explicitly and say it:

> $75 per send × 34 people with birthdays on file = about $2,550/year.

Do this before `goody_autogift_rules_activate`, not after.

## Auditing what has been sent

- `goody_order_batches_list` / `goody_order_batches_get` — batch history
  and detail.
- `goody_orders_list` — individual orders.
- `goody_autogift_rules_list` — every rule and its status. Worth a
  periodic pass: paused rules are invisible until someone reactivates
  them, and duplicate active rules on one event double-send.

When asked "how much have we spent on gifts," pull from these rather than
estimating, and say what window you covered.

## The daily cap

Goody enforces a **rolling 24-hour spend cap on AI-initiated gifting**.
It defaults to **$5,000** and is set per user at
`https://www.ongoody.com/plus/account/ai-tools` → *Daily AI gifting
limit*. It cannot be read or changed through these tools.

Two things worth telling a user:

- It applies **only** to gifts sent through connected AI tools. Gifts they
  send themselves in the Goody web app never count against it.
- Leaving the field blank means the $5,000 default, not "no limit."

When someone is nervous about handing an agent a payment method — which is
the correct instinct — point them here first. Most people do not know it
exists, and it is the only control that still holds when a query returns
ten times the rows anyone expected.

If a send fails against the cap, say so plainly and stop. Do not split the
batch across the window boundary to get under it.

## Scopes as a guardrail

`mcp.gifts` is the scope that permits spending —
`goody_order_batches_create` and `goody_autogift_rules_activate` both
require it. `mcp.read` and `mcp.write` cannot spend money.

For an unattended automation that only needs to sync contacts or assemble
previews for a human to approve, mint a token with `mcp.read` + `mcp.write`
and leave `mcp.gifts` off. A token that cannot spend is a stronger
guarantee than a prompt that says not to.

## What not to do

- Do not split a batch to slip under a cap. If a send is too large, say so.
- Do not retry a failed create without re-reading the error. A balance or
  validation failure retried blindly is how one intended send becomes two
  charges.
- Do not infer a budget from an earlier conversation. Ask.
- Do not treat silence as approval, on any timescale.
