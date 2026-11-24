---
name: send-gifts
description: Send business gifts through Goody — to one person or a few hundred. Covers choosing a gift (catalog search, Gift of Choice, personalized picks, collections, or a saved template), picking a greeting card, writing the message, previewing, and confirming before anything is charged. Use whenever someone wants to send a gift, thank-you, swag, or gift card to customers, employees, candidates, partners, or event attendees.
---

# Sending gifts with Goody

The core send flow. Every other Goody skill ends up here.

## Before anything else

1. `goody_me_get` — confirm the connected account. It also returns:
   - `default_workspace` — use its `id` unless the user wants another.
   - `server_time` — use it for all date math, not your own clock.
   - `meeting_scheduling` — whether a Calendly link is connected (step 6).
2. `goody_workspaces_list` — call it before any write, and pass
   `workspace_id` explicitly on every later call. If the user has more
   than one workspace, ask which one before spending.

Sending requires the `mcp.gifts` scope. Without it you can still search,
preview, and price; only the send fails. A hidden or refused tool says
which permission it needs. The fix is reconnecting with that permission,
not retrying.

## Flow

### 1. Gather the recipients

Each person needs an email. A name is better; with only an email, Goody
makes up a placeholder name from it.

- **One-off send** — pass them inline as `recipients: [{ name, email }]`.
  Nothing is saved to Goody.
- **People you'll gift again** — create contacts first so they're
  reusable and can drive autogift rules. See the `contact-sync` skill.
  `goody_contacts_search` finds people who are already there.

Never invent an email address. If one is missing, say who you skipped.

If the list came from a query, restate what you filtered on and how many
people came back before you price anything. A count that doesn't match
what the user described is the most common way a send goes wrong.

### 2. Choose the gift

**Starting from a saved template?** If the user mentions a template or
"the gift we always send", use `goody_templates_list` and then
`goody_templates_get`. Pass `template_id` and its `version` as
`template_version` to preview, price, and create. The template fills in
the cart, card, message, branding, and settings; anything you pass
explicitly overrides it. A template never picks recipients, the send
method, or the payment method, and it never counts as approval.

**Gifting one specific person, with no gift named yet?** Ask how they want
to choose, and give three options without recommending one:

- a **Gift of Choice** — the person picks
- **personalized picks** for that person
- **searching the catalog** for a specific gift

Ask this before searching or suggesting anything. If the sender already
asked for ideas for that person, skip the question and get picks. Don't
ask when they've named a gift or are sending to many people.

The options:

- **Personalized picks** — `goody_personalized_picks_list` with the
  person's email and, for better picks, first and last name. Set
  `requested_by` to `sender_asked` or `assistant_offered`. It returns up
  to 5 picks, and more can arrive for a couple of minutes. Calling again
  the same day with the same details returns the saved picks. Picks come
  with no reason attached, so don't invent one. If none come back, follow
  the result's `note`: say so in one line and move to a catalog search.
  One person at a time; never loop it over a list.
- **Gift of Choice** — `goody_gift_of_choice_list`. Pass `budget_cents`
  and each result's `suggested_variable_price` is the amount to send as
  `variable_price`. If it's null, the budget is under that gift's minimum.
  Tell the user the minimum; don't raise their spend yourself. It's a
  safe default when the sender doesn't know the person's taste.
- **Catalog search** — `goody_products_search`. Before searching, ask a
  question or two about the recipient's interests, the occasion, and the
  budget. `price_min` / `price_max` are in **cents**. Pass
  `shipping_country` whenever you know where the gift is going; it's the
  only filter that guarantees the product can get there. Up to 5 results,
  no paging: refine the search instead. To stay inside one category, get
  its id from `goody_product_categories_list`; a plain `query` is usually
  better. `goody_products_get` gives full
  detail for the one they want.
- **Collections** — `goody_collections_list` shows the workspace's
  curated collections. Send one with its `product_id`, not its `id`. Drafts
  have no `product_id` and can't be sent.

For a product with options (size, flavor), pass `variants` to choose one,
or leave it out to let the recipient choose. Copy the exact names from
`goody_products_get` → `variants` (e.g. "True Navy / M", not "Navy /
Medium"); nothing checks a wrong name before sending.

For `direct_send`, check `goody_products_get` → `shipping_tier` and
`restrictions` before promising a gift can reach an address outside the
US. The preview won't warn you.

`goody_products_search`, `goody_personalized_picks_list`, and
`goody_cards_list` show a picker in hosts that support it. When one
appears, frame the choice in one line and wait for the click. Don't list
the items again or ask which one they want. If no picker appears, list
the items as text with links and ask.

### 3. Pick a greeting card

`goody_cards_list` — **required** unless a template supplies one. There is
no default card, so never choose one without asking. Filter by `occasion`
when one fits (`goody_card_occasions_list`).

### 4. Write the message

Offer the choice: no message, one they write, or a few drafts from
`goody_messages_generate` for them to pick from. Don't quietly skip it.

Keep it short and tied to the reason for sending. Don't imply anything
the sender hasn't decided. A thank-you is not a promise, and a gift that
reads like one causes real problems.

Some brands print a note in the box (`printed_gift_note_mode`). See
`reference/parameters.md`.

### 5. Preview

`goody_order_batches_preview` with the exact inputs you plan to send. It
charges nothing and sends nothing. It returns:

```
{ preview_url, draft_batch_id, cart_price, total_price }
```

Lead your reply with `preview_url` so the user can open the real gift.
Each preview replaces the last one at the same link, so share only the
newest.

Then give the total **with its shape**. `total_price` is a low-to-high
range (`est_group_total_low` / `_high`, in cents) because tax is
estimated: "47 people × $25 + $1.25 fee each = $1,233.75, up to $1,357
with tax." A bare number invites a reflexive yes.

The preview doesn't return the From name or the payment method. Call
`goody_order_batches_price` with the same inputs and the
`payment_method_id` to get the `from_name` that will be used and the
funding source, and show both.

The preview also doesn't check that a chosen variant exists or that the
gift can ship to each address. Check those with `goody_products_get`
first (below).

### 6. Confirm, then send

Show all of this and get a typed yes:

- the card (name + preview link) and the message
- the product and the amount per person
- `send_method`, and whether recipients get an email
- the payment method (from `goody_payment_methods_list`)
- `from_name` — a person or team, rarely a bare company name. If the
  message is signed by someone else, ask which name they want.
- expiration — 10 weeks from sending unless they pick another date. Say so.
- for a Gift of Choice: prices are shown by default (`swap: multiple`),
  and gift upgrade (`allow_recipient_topup`) is off. Say both so they can
  change them.
- a meeting request, if one applies (below)
- for anyone outside the US or Canada: international shipping

`send_method` has no default. Choose one and say what it means:

- `link_single` — the recipient is **not** notified. You get a link to
  share yourself.
- `email_and_link` — Goody emails each recipient.
- `direct_send` — ships a physical item with no email. Every recipient
  needs a full `mailing_address`. It works in supported countries, not
  only the US. It can't be used with the `contact_id` shorthand. Ask for
  addresses; never invent one.

**Meeting requests.** For prospecting or welcome gifts, `meeting_setting`
asks the recipient to book time: `gift_first` (they can accept, then are
asked to book) or `meeting_required` (they must book first). It needs a
connected Calendly link (`meeting_scheduling.calendar_connected`); if
there isn't one, send them to Goody's web settings. If the user didn't
mention a meeting, use their `meeting_scheduling.default_setting` and
say so. If that default gates the gift and this isn't a prospecting send
(a thank-you, say), point it out and ask. Use this instead of pasting a Calendly link into the message.
Not valid with `direct_send`.

Then call `goody_order_batches_create` with `confirmed: true` and a new
`idempotency_key` (a UUID made once for this approved send).

`confirmed: true` means the user said yes to the exact gift and total you
just showed. Showing it is not approval. Praise is not approval. A yes
from before the gift changed is not approval; show it again.

**This charges real money and can't be undone from here.**

### 7. Report

Lead with `preview_url`, then state `notification_summary` so the user
knows whether recipients were reached. Never call a gift "sent" without
saying that. Give the batch id. For `link_single`, give each person's
link from `orders_preview[].individual_gift_link`; the top-level
`gift_link_url` is only the first recipient's.

Then offer a next step when it fits:

- Save inline recipients as contacts. Sending doesn't do this; follow the
  result's `suggested_next_steps`, and only with a yes.
- Send the same gift every year on a birthday or work anniversary. See
  `autogift-rules`, which can copy this batch.

If the send came from a list in another system, write the result back
there so the same people aren't gifted again next run. Goody won't
dedupe across runs.

## When something fails

- **Timeout or unclear result** — retry with the **same**
  `idempotency_key` and the same inputs. If Goody already sent it, you
  get the original result back, not a second charge. Never switch keys to
  retry an uncertain send. If you still can't tell, check
  `goody_order_batches_list` before doing anything else.
- **Clear rejection** (nothing was sent) — fix the inputs, preview again,
  get a new yes, and use a **new** `idempotency_key`.
- **Bad or missing email** — skip that person, send the rest, name the
  skips.
- **Insufficient balance, or over the daily AI gifting limit** — say so
  plainly. Don't retry or split the send to get around it.

## After sending

Checking whether gifts were accepted, looking up past sends, and
cancelling a gift are in the `manage-sent-gifts` skill. Offer to pass any
reaction to the Goody team with `goody_feedback_create`.

## Full parameter reference

`reference/parameters.md` — cart shapes, templates, Gift of Choice
amounts, swap and price display, printed notes, meetings, custom forms,
scheduled sends, expiry, international shipping, alcohol, and retries.
