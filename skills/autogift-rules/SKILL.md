---
name: autogift-rules
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
`goody_order_batches_create` — see the `event-triggered-gifting`
skill.

Say this plainly when a user asks for an ATS-triggered autogift. Do not
approximate it with a birthday rule.

## Before anything else

1. `goody_me_get`, then `goody_workspaces_list` — pass `workspace_id`
   explicitly on every call.
2. `goody_autogift_rules_list` — check for an existing rule on the same
   event and list. Two active rules on one event send two gifts. The list
   shows event, list, tenure, and status, but not timing or payment
   method.

Creating and activating a rule both require the `mcp.gifts` scope —
activation is the money-committing call.

## Flow

### 1. Bind the audience

Either `contact_list_id` (an existing list — see `contact-sync`) or
`contact_ids[]`, which creates an implicit list named after the first
contact. Prefer a real, deliberately named list; the implicit name is
unhelpful when someone audits rules later.

Contacts need the relevant date filled in or the rule never fires for
them. A birthday rule over a list where half the contacts have no
`birthday` silently covers half the team.

- **A list you're building from known contacts:** call `goody_contacts_get`
  on each one (search results don't include dates) and report how many
  have the date. A year of `1900` means only the month and day are known,
  which is fine.
- **An existing list:** these tools can't show who is on it. Say so, and
  ask the user to check coverage in the Goody web app.

Birthdays and work anniversaries can be set with these tools
(`goody_contacts_create` / `_update`). Onboarding start dates can't: they
come from a connected HRIS. Only bind an onboarding rule to a list synced
from an HRIS (its `kind` in `goody_contact_lists_list`). The create tool
accepts any list without warning, and on a hand-built list the rule never
fires.

### 2. Define the gift — pick exactly one path

**Path A — clone a previous send.** Pass `source_order_batch_id` and the
cart, card, message, from_name, payment method, swap type, and shipping
tier are all pulled from that batch. Best when the user already sent
something they liked. Find it with `goody_order_batches_list`, always
with a `query`, `product_id`, or `created_after` and a small `per_page`
(about 5); unfiltered results are very large. Or use the batch you just
sent.

Cloning is guarded: if the source batch's greeting card carries occasions
that contradict `event_type` — a Birthday card cloned into a
`work_anniversary` rule — the request is **rejected**, so nobody gets
"Happy birthday!" on their work anniversary. `allow_occasion_mismatch:
true` overrides it, and is only right for a deliberately generic card.
Ask before overriding. The error doesn't name the card's occasions; look
the card up in `goody_cards_list` to explain the clash.

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

Call `goody_autogift_rules_preview`. It returns the card, message, From
name, products, and spend. Show the user those, plus what you passed to
create, which the preview doesn't repeat:

- the card and message. The message is shown as written; tags like the
  first name fill in per person at send time.
- the products
- the spend per send: `expected_spend_per_send.est_group_total_low` to
  `est_group_total_high`, in cents. Despite the name, it's for one person.
- the timing, tenure range, and payment method
- the date coverage from step 1

Then state the recurring commitment in plain terms — expected spend per
send times the number of people who will trigger it over a year — and get
an explicit yes.

**Activation is what commits the money, not creation.** Because nobody is
present when the rule actually fires, this one approval stands in for
every future send. That is exactly why it has to be a real yes to a real
preview, not a yes to a description.

Only then call `goody_autogift_rules_activate`.

### 5. Pausing and changing a rule

`goody_autogift_rules_pause` stops future sends. Reach for it when a
budget runs out, the audience is wrong, or the user wants a different
gift. `goody_autogift_rules_activate` turns it back on, and re-checks the
rule first: it fails if the product was discontinued, the payment method
was removed, or the card, message, or From name is missing.

These tools can't edit a rule. To change the gift, either edit it in the
Goody web app, or pause the old rule and create a new one. Never leave
both active on the same people.

## Alcohol

An alcohol gift needs an account with alcohol turned on (ask Goody
support), a card rather than balance as the payment method, and
`alcohol_age_verification_attested: true`. Confirm the age with the user;
never attest for them. For a recurring rule this one attestation covers
every future send, which carries more weight than a one-off. Say so.
