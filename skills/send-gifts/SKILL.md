---
name: send-gifts
description: Send business gifts through Goody — to one person or a few hundred. Covers choosing a gift (catalog search, Gift of Choice, personalized picks, collections, or a saved template), picking a greeting card, writing the message, previewing, and confirming before anything is charged. Use whenever someone wants to send a gift, thank-you, swag, or gift card to customers, employees, candidates, partners, or event attendees.
---

# Sending gifts with Goody

The core send flow. Every other Goody skill ends up here. The tool
descriptions cover each input; this covers the order of steps, the
judgment calls, and what the tools don't tell you.

Load a reference file only when the gift needs it:

| When | Read |
|---|---|
| The user mentions a template or "the gift we always send" | `reference/templates.md` |
| Shipping a physical item (`direct_send`), or anyone outside the US/Canada | `reference/shipping.md` |
| A meeting request, a printed note in the box, or a question form | `reference/extras.md` |
| The cart has alcohol | `reference/alcohol.md` |
| A create call times out, errors, or you're retrying | `reference/retries.md` |

## Before anything else

`goody_me_get` gives the account, `default_workspace`, `server_time`
(use it for all date math), and `meeting_scheduling`. Then
`goody_workspaces_list`; if there's more than one workspace, ask which
before spending.

Sending needs the `mcp.gifts` scope. Without it, search, preview, and
price still work. The fix is reconnecting with that permission, not
retrying.

## 1. Gather the recipients

Use inline `recipients` for one-off sends. Create contacts first only for
people you'll gift again (see `contact-sync`). Never invent an email; say
who you skipped.

If the list came from a query, restate the filter and the count before
pricing. A count that doesn't match what the user described is the most
common way a send goes wrong.

## 2. Choose the gift

**One specific person, no gift named yet?** Ask how they want to choose,
with three options and no recommendation: a Gift of Choice (the person
picks), personalized picks for that person, or a catalog search. Skip
the question if they already asked for ideas (get picks) or named a gift,
and don't ask when sending to many people.

What the tools don't tell you:

- **Personalized picks** — one person at a time, never looped over a
  list. Picks have no reason attached; don't invent one. If none come
  back, follow the result's `note` and move to a search.
- **Gift of Choice** — if `suggested_variable_price` is null, the budget
  is under the gift's minimum. Tell the user the minimum; don't raise
  their spend yourself.
- **Catalog search** — ask about the recipient's interests, occasion, and
  budget before searching. The price filter includes shipping, but the
  prices shown don't, so a $50 cap can hide a $45 item with $8 shipping.
  Pass `shipping_country` whenever you know it.
- **Variants** — copy exact names from `goody_products_get` → `variants`
  ("True Navy / M", not "Navy / Medium"). Nothing checks a wrong name
  before sending. Leave `variants` out to let the recipient choose.
- **Collections** — send with the collection's `product_id`, not its `id`.

When a picker widget appears, frame the choice in one line and wait for
the click. Don't list the items again. With no widget, list them as text
with links and ask.

## 3. Card and message

A card is required and there's no default; never choose one without
asking. For the message, offer the choice: none, theirs, or a few drafts
from `goody_messages_generate`.

Keep the message short and tied to the reason for sending. Don't imply
anything the sender hasn't decided. A thank-you is not a promise, and a
gift that reads like one causes real problems.

## 4. Preview and price

1. `goody_order_batches_preview` with the exact inputs. Lead your reply
   with `preview_url`. Each preview replaces the last one at the same
   link, so share only the newest.
2. `goody_order_batches_price` with the same inputs plus
   `payment_method_id`. The preview doesn't return the From name or the
   funding source; price does.

Give the total **with its shape**. Totals are a low-to-high range
because tax is estimated: "47 people × $25 + $1.25 fee each = $1,233.75,
up to $1,357 with tax." A bare number invites a reflexive yes.

## 5. Confirm, then send

Show all of this and get a typed yes:

- the card (name + link) and the message
- the gift and the amount per person, and the total range
- the send method and whether recipients get an email:
  `link_single` = no, you share the link; `email_and_link` = yes;
  `direct_send` = no email, the item ships
- the payment method
- the From name. If the message is signed by someone else, ask which
  name they want.
- expiry: 10 weeks unless they pick a date
- for a Gift of Choice: prices shown by default, gift upgrade off. Say
  both so they can change them.
- a meeting request, if their default adds one (`reference/extras.md`)

Then `goody_order_batches_create` with `confirmed: true` and a new
`idempotency_key` UUID for this approved send.

A yes counts only for the exact gift and total you just showed. If
anything changed, show it again. **This charges real money and can't be
undone from here.**

## 6. Report

Lead with `preview_url`, then the `notification_summary`. Never call a
gift "sent" without saying whether recipients were reached. Give the
batch id. For `link_single`, give each person's link from
`orders_preview[].individual_gift_link`; the top-level `gift_link_url`
is only the first recipient's.

Then offer a next step when it fits: save inline recipients as contacts
(only with a yes), or repeat the gift yearly with `autogift-rules`, which
can copy this send. If the list came from another system, write the
result back there; Goody won't dedupe across runs.

For checking on gifts later or cancelling one, see `manage-sent-gifts`.

## When something fails

- **Timeout or unclear result** — see `reference/retries.md` before doing
  anything. Never retry with a new key.
- **Bad or missing email** — skip that person, send the rest, name the
  skips.
- **Insufficient balance, or over the daily AI gifting limit** — say so.
  Don't retry, and don't split the send to get under the limit.
