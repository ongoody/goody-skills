---
name: send-gifts
description: Send business gifts through Goody — to one person or a few hundred. Covers picking a gift and greeting card, writing the message, previewing, and confirming before anything is charged. Use whenever someone wants to send a gift, thank-you, swag, or gift card to customers, employees, candidates, partners, or event attendees.
---

# Sending gifts with Goody

The core send flow. Every other Goody skill ends up here.

## Before anything else

1. `goody_me_get` — confirm the connected account, and capture
   `server_time` for any date math. Do not use your own clock.
2. `goody_workspaces_list` — pick the workspace and pass `workspace_id`
   explicitly on every later call. Never let it default.

If there is more than one workspace, ask which one before spending.

Sending requires the `mcp.gifts` scope. Without it you can still search,
preview, and price — only the commit fails. If it does, the fix is a
reconnection or a token with that scope, not a retry.

## Flow

### 1. Gather the recipients

You need a name and an email for each person.

- **One-off send** — pass them inline as `recipients: [{ name, email }]`.
  Nothing is stored in Goody.
- **People you'll gift again** — create contacts first so they're reusable
  and can drive autogift rules. See the `contact-sync` skill.

Never invent an email address. If one is missing, say who you skipped.

If the list came from a query, restate what you filtered on and how many
people came back before you price anything. A count that doesn't match
what the user described is the single most common way a send goes wrong.

### 2. Pick the gift

`goody_products_search` — `price_min` / `price_max` are in **cents**.

A **Gift of Choice** (`goody_gift_of_choice_list`) is the safest default
when you don't know the person's taste: you set an amount with
`variable_price` and they choose what they want. That covers most business
gifting, where the sender rarely knows the recipient's preferences.

`goody_products_search` renders an interactive picker. When it does, frame
the choice in one line and wait for the click — don't restate the products
as a numbered list or ask which one they want.

### 3. Pick a greeting card

`goody_cards_list` — **required**, and also a picker widget. There is no
default card and you must not choose one silently.

Filter by `occasion` when one fits (`goody_card_occasions_list`).

### 4. Write the message

Write it yourself or call `goody_messages_generate`.

Keep it short and specific to the reason for sending. Don't imply anything
the sender hasn't decided — a thank-you is not a commitment, and a gift
that reads like a promise causes real problems.

### 5. Preview

`goody_order_batches_preview` with the exact inputs you intend to send. It
charges nothing and sends nothing. Returns:

```
{ preview_url, draft_batch_id, cart_price, total_price }
```

Lead your reply with `preview_url` so the user can open the real assembled
gift, then give the total **with its shape** — "47 people × $25 = $1,175
plus fees, $1,283.50 total." A bare number invites a reflexive yes.

### 6. Confirm, then send

Show all of this and get a typed yes:

- the card (name + preview link)
- the message
- the product and per-person amount
- `send_method` and what it means for notifications
- `payment_method_id` (from `goody_payment_methods_list`)
- `from_name` — a person or team, rarely a bare company name
- expiration (defaults to 10 weeks if omitted — say so)

`send_method` has no default. Choose one and state its consequence:

- `link_single` — recipient is **not** notified; you get a link to share
  yourself.
- `email_and_link` — Goody emails each recipient.
- `direct_send` — ships a physical item, no email. Requires a complete
  `mailing_address` per recipient and is **incompatible with
  `contact_id`**. Ask for addresses; never invent one.

Then call `goody_order_batches_create` with `confirmed: true`.

`confirmed: true` means the user answered yes to the exact gift and total
you just showed. Having shown it is not approval. Praise is not approval.
A yes from before the gift changed is not approval — show it again.

**This charges real money and cannot be undone from here.**

### 7. Report

Give the batch id, and the gift links for `link_single`. If the send came
from a list in another system, write the result back there so the same
people aren't gifted twice on the next run — Goody won't dedupe for you.

## When something fails

- Bad or missing email → skip that person, send the rest, name the skips.
- Insufficient balance or a validation error → surface the message, don't
  retry blindly. A blind retry is how one intended send becomes two
  charges.
- Partial list → send to the confirmed subset and report the remainder.
  Better than sending nothing.

## Full parameter reference

`reference/parameters.md` — cart shapes, Gift of Choice amounts, swap and
pricing visibility, scheduled sends, expiry, international shipping, and
alcohol attestation.
