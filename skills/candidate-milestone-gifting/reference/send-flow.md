# Goody send flow — parameter reference

The three tools that matter, in order. All three share the same cart and
recipient shorthands, so the preview reflects exactly what will be sent.

| Tool | Charges? | Sends? | Use for |
|---|---|---|---|
| `goody_order_batches_price` | no | no | a total, nothing else |
| `goody_order_batches_preview` | no | no | a real link the user can open, plus the total |
| `goody_order_batches_create` | **yes** | **yes** | committing |

Prefer `preview` over `price`. It costs the same round trip and gives the
user something to actually look at.

## Cart — pick exactly one shape

Single product:

```json
{ "product_id": "PRODUCT_UUID", "quantity": 2 }
```

Gift of Choice / variable gift card — amount in **cents**:

```json
{ "product_id": "GOC_UUID", "variable_price": 5000 }
```

The amount must fall inside the product's `[price_min, price_max]`
(`price_max: null` means unlimited). Gift of Choice has a floor, typically
$15 / `1500`. Below it, the call is rejected.

Multi-item cart — an **object with an `items` array**, not a bare array:

```json
{ "cart": { "items": [
  { "product_id": "UUID_A", "quantity": 1 },
  { "product_id": "UUID_B", "quantity": 2, "variable_price": 5000 }
] } }
```

Unknown keys inside `cart` or `cart.items[]` are rejected outright.

## Recipients — pick exactly one shape

Single existing contact:

```json
{ "contact_id": "CONTACT_UUID" }
```

Inline list (the normal case for candidates):

```json
{ "recipients": [
  { "name": "Ada Lovelace", "email": "ada@example.com" },
  { "first_name": "Grace", "last_name": "Hopper", "email": "grace@example.com" }
] }
```

`name` splits on the **first** whitespace only, so particle surnames
("van Dijk", "de la Cruz") survive intact. Pass explicit
`first_name`/`last_name` when you need full control.

Each `recipients[]` entry may carry its own `contact_id` instead of
name/email — that is how you build a multi-recipient send from existing
contacts.

## send_method

No default. You must choose, and you must tell the user the notification
consequence before passing `confirmed: true`.

| Value | Recipient notified? | Requires |
|---|---|---|
| `link_single` | No — you share the link yourself | — |
| `email_and_link` | Yes, Goody emails them | email on each recipient |
| `direct_send` | No email; the physical item ships | complete `mailing_address` on **every** recipient |

`direct_send` cannot be combined with the top-level `contact_id`
shorthand — Goody contacts store no mailing address. Use inline
`recipients[]`, each with:

```json
{ "address_1": "...", "address_2": "...", "city": "...",
  "state": "...", "postal_code": "...", "country": "US" }
```

`country` is a 2-letter ISO code. `mailing_address` is rejected on any
non-`direct_send` method.

## swap — the show-vs-hide-pricing control

| Value | Behavior |
|---|---|
| `single` | Prices **hidden**; recipient may swap for one item of equal value. Overall default. |
| `multiple` | Prices **shown**; recipient shops multiple items against the budget. The classic Gift of Choice experience. |
| `disabled` | No swapping. |

For a Gift of Choice, default to `multiple` — and tell the user that is
the default so they can switch to hidden if they prefer.

`allow_recipient_topup` lets the recipient pay the difference to upgrade
past the budget. It requires `swap: "multiple"` and is rejected otherwise.
Off by default; it is a limited-availability feature and will be rejected
if the account does not have it.

## Timing and expiry

- `scheduled_send_on` — ISO 8601, must be future, at most 3 months out.
  Anchor against `goody_me_get`'s `server_time`, never your own clock.
- `expires_at` (ISO 8601 datetime) or `expires_on` (`YYYY-MM-DD`) — pass
  one, not both. This is the deadline to **accept** the gift; opened-but-
  unaccepted gifts still expire.
- Omit both and it defaults to **10 weeks** from the send date, matching
  the Goody web app. State that default to the user rather than passing a
  value they did not ask for.

## International

- `international_shipping_tier`: `standard` (default — US & Canada plus
  passthrough and digital, no extra fees), `disabled` (US & Canada only),
  `full` (140+ countries, up to $55/order in taxes, tariffs, freight).
  Use `full` when a recipient may be outside the US/Canada.
- `international_gift_cards_enabled`: lets recipients outside supported
  shipping regions swap for a gift card.

## Alcohol

If the cart contains alcohol, `alcohol_age_verification_attested` must be
`true` or the create is rejected. Confirm with the user first — do not
attest on their behalf. `goody_products_search` excludes alcohol by
default (`exclude_alcohol` defaults true); only set it false when the user
explicitly asked.

## The confirmation gate

`confirmed: true` on `goody_order_batches_create` is a literal-true gate
checked before anything else runs. Anything other than `true` is an
invalid-input error.

Pass it only when the user has answered yes to the exact gift and total
you showed. Not because you showed it. Not because they said the flow
looked good. Not on a yes that predates a change to the gift.

For automations that run with no human present — a scheduled send, a
saved agent token — approval is captured when the automation is **set
up**, not at each run. Show a representative gift and the expected spend
and confirm before enabling it.
