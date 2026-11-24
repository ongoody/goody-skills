# Goody send flow — parameter reference

The three tools that matter, in order. All three take the same cart,
recipient, and template inputs, so the preview matches what gets sent.

| Tool | Charges? | Sends? | Use for |
|---|---|---|---|
| `goody_order_batches_price` | no | no | the total, the `from_name` that will be used, and the payment method |
| `goody_order_batches_preview` | no | no | a real link the user can open, plus the total |
| `goody_order_batches_create` | **yes** | **yes** | committing |

Use both before a send: `preview` for the link, then `price` with the
same inputs and `payment_method_id` for the From name and funding source,
which the preview doesn't return. Totals are a low-to-high range because
tax is estimated.

Each preview replaces the last one at the same `preview_url`. Neither
call checks variant names or whether a product ships to an address;
check those with `goody_products_get`.

## Saved templates

```json
{ "template_id": "TEMPLATE_UUID", "template_version": 7 }
```

Find one with `goody_templates_list` (`include_catalog: true` adds Goody's
own templates). Read it with `goody_templates_get` and pass the `version`
it returns. The template fills in the cart, card, message, branding,
custom form, and settings. Anything you pass explicitly overrides it.

Use the same version and overrides for preview and create. If the
template changed in between, the send is rejected; read it again and
re-preview. A template never sets recipients, `send_method`, or
`payment_method_id`.

## Cart — pick exactly one shape (unless a template supplies it)

Single product:

```json
{ "product_id": "PRODUCT_UUID", "quantity": 2 }
```

With a chosen option — copy the exact name from `goody_products_get` →
`variants`, or leave `variants` out to let the recipient choose:

```json
{ "product_id": "PRODUCT_UUID", "variants": ["Red / Large"] }
```

Gift of Choice or a variable gift card — amount in **cents**:

```json
{ "product_id": "GOC_UUID", "variable_price": 5000 }
```

The amount must fall inside the product's `[price_min, price_max]`
(`price_max: null` means no limit). Gift of Choice has a minimum, usually
$15 / `1500`. Below it, the call is rejected.

A collection is sent like any product, using the collection's
`product_id` from `goody_collections_list`.

Multi-item cart — an **object with an `items` array**, not a bare array:

```json
{ "cart": { "items": [
  { "product_id": "UUID_A", "quantity": 1 },
  { "product_id": "UUID_B", "quantity": 2, "variable_price": 5000 }
] } }
```

Unknown keys inside `cart` or `cart.items[]` are rejected.

## Prices in search

`goody_products_search` filters `price_min` / `price_max` on product price
**plus shipping**, but shows `price_cents` without shipping. A $50 cap
can hide a $45 item with $8 shipping. Preview for the real total.

## Recipients — pick exactly one shape

Single existing contact:

```json
{ "contact_id": "CONTACT_UUID" }
```

Inline list (the usual case):

```json
{ "recipients": [
  { "name": "Ada Lovelace", "email": "ada@example.com" },
  { "first_name": "Grace", "last_name": "Hopper", "email": "grace@example.com" },
  { "contact_id": "CONTACT_UUID" },
  { "email": "lin@example.com" }
] }
```

- `name` splits on the **first** space only, so surnames like "van Dijk"
  and "de la Cruz" stay whole. Pass `first_name` / `last_name` for full
  control.
- A `contact_id` inside `recipients[]` is how you send to several
  existing contacts.
- Email alone works; Goody makes up a placeholder name from it.
- `phone` is optional on each recipient.

## send_method

No default on create. Choose one and tell the user what it means before
passing `confirmed: true`. (Preview falls back to `link_single`.)

| Value | Recipient notified? | Requires |
|---|---|---|
| `link_single` | No. You share the link yourself. | — |
| `email_and_link` | Yes. Goody emails them. | an email on each recipient |
| `direct_send` | No email. The physical item ships. | a full `mailing_address` on **every** recipient |

`direct_send` can't use the top-level `contact_id` shorthand, because
Goody contacts store no mailing address. Use inline `recipients[]`, each
with:

```json
{ "mailing_address": {
  "address_1": "...", "address_2": "...", "city": "...",
  "state": "...", "postal_code": "...", "country": "FR"
} }
```

`country` is a 2-letter ISO code. `state` and `postal_code` are needed
only where the country uses them. International phone numbers are
accepted. `mailing_address` is rejected for any other send method.

## swap — show or hide prices

| Value | Behavior |
|---|---|
| `single` | Prices **hidden**. The recipient may swap for one item of equal value. The overall default. |
| `multiple` | Prices **shown**. The recipient shops several items within the budget. The classic Gift of Choice. |
| `disabled` | No swapping. |

For a Gift of Choice, default to `multiple` and tell the user, so they
can switch to hidden.

`allow_recipient_topup` (gift upgrade) lets the recipient pay the
difference to pick something above the budget. It requires
`swap: "multiple"`. Off by default; only some accounts have it, and it's
rejected on the others.

## Printed gift notes

Only for products whose brand prints a note in the box
(`printed_gift_note_enabled` on the product).

| `printed_gift_note_mode` | Prints |
|---|---|
| `verbatim` | the card `message` (or `printed_gift_note_text` if given) |
| `auto_shortened` | a shortened card message; pass the short text in `printed_gift_note_text` |
| `custom` | `printed_gift_note_text` |
| `disabled` | no note |

Leave it out to keep the brand's usual note. The text may use
`{{firstname}}` / `{{fullname}}`. A note over the brand's limit is
rejected with the number of characters to cut.

## Meeting requests

`meeting_setting`: `no_meeting` (default), `gift_first` (accept, then
asked to book), or `meeting_required` (must book to accept). Needs a
connected Calendly link (`goody_me_get` →
`meeting_scheduling.calendar_connected`). If the user doesn't say,
follow `meeting_scheduling.default_setting` and state it. If that default
gates a gift that isn't prospecting (a thank-you, say), point it out and
ask. Pass the same
value to preview and create. Not valid with `direct_send`.

## Custom forms

`custom_form_id` attaches an existing form of questions the recipient
answers when accepting. It must be one the user owns or one shared with
the workspace; these tools can't create forms. `null` clears a form a
template set. Ignored for `direct_send`.

## Timing and expiry

- `scheduled_send_on` — ISO 8601, in the future, at most 3 months out.
  Base it on `goody_me_get`'s `server_time`, never your own clock.
- `expires_at` (ISO 8601 datetime) or `expires_on` (`YYYY-MM-DD`) — pass
  one, not both. This is the deadline to **accept** the gift; gifts that
  were opened but not accepted still expire.
- Leave both out and it defaults to **10 weeks** from the send date,
  same as the Goody web app. Tell the user rather than passing a date
  they didn't ask for.

## International

- `international_shipping_tier`: `standard` (default — US & Canada plus
  passthrough and digital, no extra fees), `disabled` (US & Canada only),
  `full` (140+ countries, up to $55/order in taxes, tariffs, freight).
  Use `full` when a recipient may be outside the US or Canada.
- `international_gift_cards_enabled`: lets recipients outside supported
  shipping regions swap for a gift card.
- When searching, pass `shipping_country` so results can reach the
  recipient.

## Alcohol

All three must hold, or the send is rejected. Check them before building
the gift:

1. The account must have alcohol turned on. If it's off, the error says
   so. Tell the user to contact Goody support; don't retry.
2. Pay with a card. Balance (`BALANCE` / `WORKSPACE_BALANCE`) can't pay
   for alcohol.
3. `alcohol_age_verification_attested: true`, only after the user
   confirms they are over 21. Never attest for them.

`goody_products_search` leaves alcohol out unless `exclude_alcohol` is
false. Only set that when the user asked for alcohol.

## The confirmation gate

`confirmed: true` on `goody_order_batches_create` must be the literal
`true`; anything else is rejected.

Pass it only when the user has said yes to the exact gift and total you
showed. Not because you showed it. Not because they said it looked good.
Not on a yes from before the gift changed.

For automations with nobody watching — a scheduled send, a saved agent
token — approval happens when the automation is **set up**, not at each
run. Show a sample gift and the expected spend, and confirm before
turning it on.

## Retries — `idempotency_key`

Make one UUID per approved send and pass it to
`goody_order_batches_create`.

| What happened | What to do |
|---|---|
| Timeout or unclear result | Retry with the **same** key and the same inputs. A finished send returns its original result. |
| "This idempotency_key was already used with different inputs" | The inputs changed since that key was used. Check whether the first send went through before approving anything new. |
| Still unclear after that | Stop. Check `goody_order_batches_list`, or contact support. Never switch keys. |
| Rejected before anything was sent (bad input, changed template) | Fix the inputs, preview, get a new yes. |
| A failure returned by the send itself | Saved against that key. Once you're sure nothing was sent, fix, preview, get a new yes, and use a **new** key. |

## Spend limit

Each user has a **Daily AI gifting limit** (default $5,000, over any 24
hours) set on Goody's Connect AI tools page. Sends through AI tools count
against it; gifts sent in Goody itself don't. When a send goes over it,
tell the user. Don't split the send to get under it.
