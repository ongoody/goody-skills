---
name: manage-sent-gifts
description: Look up, check on, and cancel Goody gifts that were already sent. Use when someone asks whether a gift was opened or accepted, who hasn't claimed theirs, what was sent to a person or company before, which campaign drove acceptances, or wants to cancel a gift.
---

# Managing gifts already sent

Answers "what happened to the gifts we sent?" and cancels the ones that
should not go through.

Reading needs `mcp.read`. Cancelling needs `mcp.write`.

## Before anything else

1. `goody_me_get` — confirm the connected account. Use
   `default_workspace.id` unless the user means another workspace.
2. `goody_workspaces_list` — if the user has more than one workspace and
   doesn't say which, ask. A send from another workspace won't show up.

## How sends and gifts fit together

- A **send** (order batch) is one action: one gift, one card, one
  message, to one or many people.
- Each person in it gets their own **gift** (order), with its own status.

"Did Sam get it?" is a question about one gift. "How did the Q3 thank-you
go?" is about a send.

## Finding a send

`goody_order_batches_list`, newest first. Narrow it before paging:

- `query` — searches the send's name, sender name, message, and each
  recipient's name and email. "Acme" or "sam@acme.com" usually finds it.
- `created_after` / `created_before` — `YYYY-MM-DD`. Turn "last quarter"
  into dates using `goody_me_get`'s `server_time`.
- `product_id` — everything sent with one product.
- `send_status` — `pending`, `complete`, `failed`, or `canceled`. This
  describes the send step only. It stays `complete` even after every gift
  in it is cancelled or fails. To find cancelled or failed gifts, check
  each gift with `goody_orders_list`.

Each row is large, because it carries every gift in the send. Keep
`per_page` small (about 5–10) and filter instead of paging. `total_count`
tells you how many matched.

`goody_order_batches_get` gives one send in full: the cart, the card, the
message, who it went to, the scheduled date, the expiry, and its
`preview_url` (what the recipient sees). Lead with that link when the
user wants to see what was sent.

## Checking on gifts

`goody_orders_list` with `order_batch_id` lists every gift in a send,
one per person. Without it, it lists the newest gifts across the
workspace, including some with no send (`order_batch_id: null`, such as
gifts claimed through a shared link). A count for one send never includes
those. Keep `per_page` small here too.

What each `status` means:

| status | Meaning |
|---|---|
| `created` | Not opened yet, **or expired unopened**. Compare the send's `expires_at` with `server_time`. `view_count_recipient` above 0 means they looked at it |
| `opened` | Opened. Can also mean just accepted and still processing |
| `pending_user_approval` | Someone claimed a SmartLink that needs the sender's approval; approve it in Goody |
| `pending_payment` | Accepted, payment still going through |
| `paid` | Accepted and paid for |
| `ordered` | The item was ordered from the brand |
| `shipped` | On its way; tracking is in `shipments` |
| `delivered` | Delivered |
| `failed` | Didn't go through: declined, cancelled, couldn't be emailed or delivered, or not eligible |

`failed` covers several causes and the status alone doesn't say which.
Don't guess; say it failed and point the user to Goody's Track page or
support for the reason. A gift you cancelled also shows as `failed`;
call it cancelled.

**What did they pick?** If the recipient swapped the gift,
`is_swapped` is true, `cart` is what they chose, and `original_cart` is
what was sent.

When reporting on a send, count by status and name the people who matter
for the question: "40 of 47 accepted. Not opened yet: Ana, Raj, Lee."
Don't paste the whole list unless asked.

`total_count` is the number of gifts, not a breakdown by status. Page
through `goody_orders_list` until you've read `total_count` gifts before
giving any counts. For a very large send (hundreds of gifts), say how
many calls that takes and ask first, or say plainly that your counts
cover only the gifts you read.

Gifts expire if nobody accepts them in time, and an expired gift still
reads `created`. Check the send's `expires_at` before calling a gift
"waiting". If many are still `created` close to that date, mention it.

**Which campaign worked?** Each accepted gift may carry `utm_params`
(`utm_source`, `utm_campaign`, …) from the gift link the recipient
clicked. Group by them to answer "which campaign drove acceptances?". A
gift accepted without them has none; count those separately rather than
dropping them.

## Cancelling a gift

`goody_orders_cancel` cancels **one** gift. It can't be undone. It
releases the payment held for that gift and refunds any credit used.

1. Find the exact gift with `goody_orders_list`, and check its status.
   Only gifts the recipient hasn't accepted can be cancelled. Anything
   from `paid` onward can't, nor can one already cancelled. The refusal
   just says "Gift cannot be canceled", so explain why from the status
   you checked.
2. Show the user who it's for, what it is, and what cancelling does, and
   get a typed yes for that gift.
3. Cancel it, and tell the user it's cancelled. (Its status will read
   `failed`.)

To cancel a whole send, cancel each gift in it. List them first, say how
many can still be cancelled and how many can't, and get one yes for the
whole set before starting. If some fail partway, finish the rest and
report both lists.

Never cancel to "fix" a send — say, to change the message or the amount
— without the user agreeing to both the cancel and the new send. The
new send goes through the full `send-gifts` flow.

## What these tools can't do

Resending a notification, changing a gift's expiry, or changing the
message or gift after sending have no tool here. That doesn't mean Goody
can't do them. Point the user to Goody's Track page in the web app, or to
support@ongoody.com, instead of saying it's impossible.

A scheduled send that hasn't gone out yet may have no gifts listed. If
so, it can't be cancelled from here; send the user to the Track page.

## Feedback

When the user reacts to how a gift landed — good or bad — offer to pass
it on with `goody_feedback_create`. Use their own words as `message`, set
`sentiment` if it's clear, and pass `order_batch_id` so the team knows
which send it's about.
