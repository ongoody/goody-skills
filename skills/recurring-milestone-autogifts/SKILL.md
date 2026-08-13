---
name: recurring-milestone-autogifts
description: Set up Goody autogift rules that fire automatically on birthdays, work anniversaries, and new-hire onboarding. Use when asked to automate recurring employee gifting, set up always-on celebration gifts, or stop manually sending the same milestone gift every month.
---

# Recurring autogift rules

Autogift is Goody's always-on engine: define a rule once, and it sends on
its own whenever a person on the bound contact list hits the event.

## What autogift can and cannot trigger on

**Only three event types exist:**

| `event_type` | Fires on |
|---|---|
| `birthday` | contact's `birthday` |
| `work_anniversary` | contact's `work_anniversary`, optionally filtered by tenure |
| `onboarding` | a new hire, keyed off `start_date` or `added_to_hris_date` |

There is **no** custom or arbitrary event type. Recruiting stages, deal
closes, project completions, and anything else your own system knows
about cannot drive an autogift rule. Detect those yourself and send with
`goody_order_batches_create` — see the `candidate-milestone-gifting`
skill.

Say this plainly when a user asks for an ATS-triggered autogift. Do not
approximate it with a birthday rule.

## Before anything else

1. `goody_me_get`, then `goody_workspaces_list` — pass `workspace_id`
   explicitly on every call.
2. `goody_autogift_rules_list` — check for an existing rule on the same
   event and cohort. Two active rules on one event send two gifts.

Creating and activating a rule both require the `mcp.gifts` scope —
activation is the money-committing call.

## Flow

### 1. Bind the audience

Either `contact_list_id` (an existing list — see `ats-contact-sync`) or
`contact_ids[]`, which creates an implicit list named after the first
contact. Prefer a real, deliberately named list; the implicit name is
unhelpful when someone audits rules later.

Contacts need the relevant date populated or the rule never fires for
them. A birthday rule over a list where half the contacts have no
`birthday` silently covers half the team. Check coverage and report it.

### 2. Define the gift — pick exactly one path

**Path A — clone a previous send.** Pass `source_order_batch_id` and the
cart, card, message, from_name, payment method, swap type, and shipping
tier are all pulled from that batch. Best when the user already sent
something they liked.

Cloning is guarded: if the source batch's greeting card carries occasions
that contradict `event_type` — a Birthday card cloned into a
`work_anniversary` rule — the request is **rejected**, so nobody gets
"Happy birthday!" on their work anniversary. `allow_occasion_mismatch:
true` overrides it, and is only right for a deliberately generic card.
Ask before overriding.

**Path B — build from scratch.** Pass a cart (`product_id` shorthand or
`cart.items[]`) plus `card_id`, `message`, and `payment_method_id`.

Passing both paths, or neither, is an error.

`payment_method_id` is technically optional but the rule **cannot be
activated without it** — it resolves to the autopay method the rule
charges. Always pass it.

### 3. Timing

- `anchor_date_send_option`: `send_on_anchor_date`,
  `send_before_anchor_date`, or `send_after_anchor_date`.
- `anchor_date_number_of_days_delta`: required for before/after.
- `anchor_date_type`: `start_date` or `added_to_hris_date` — **only** for
  `onboarding`. These are not event types.
- `tenure_min` / `tenure_max`: inclusive year bounds, meaningful for
  `work_anniversary`. This is how you build tiered programs — a $50 gift
  at years 1–4, something better at 5+.

Onboarding gifts usually want a small positive delta so the gift lands
after the first day rather than before the person has a badge.

### 4. Preview, confirm, then activate

`goody_autogift_rules_create` always returns a rule in **`paused`**
status. Nothing sends yet.

Call `goody_autogift_rules_preview` and show the user:

- the card and message
- the products
- `expected_spend_per_send`
- how many contacts are bound and how many have the date populated

Then state the recurring commitment in plain terms — expected spend per
send times the number of people who will trigger it over a year — and get
an explicit yes.

**Activation is what commits the money, not creation.** Because nobody is
present when the rule actually fires, this one approval stands in for
every future send. That is exactly why it has to be a real yes to a real
preview, not a yes to a description.

Only then call `goody_autogift_rules_activate`.

### 5. Pausing

`goody_autogift_rules_pause` stops future sends. Reach for it when a
budget is exhausted, a cohort is wrong, or the user wants to revise the
gift — edit-then-reactivate is safer than deleting and rebuilding, since
rebuilding loses the rule's history.

## Alcohol

`alcohol_age_verification_attested` must be `true` when the cart contains
alcohol. Confirm with the user; never attest for them. For a recurring
rule this attestation covers every future send, which is more weight than
a one-off — say so.
