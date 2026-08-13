---
name: candidate-milestone-gifting
description: Send Goody gifts when candidates hit recruiting milestones — completing a take-home, reaching onsite, receiving an offer, accepting, or being declined. Use when gifting from an ATS, applicant tracker, recruiting pipeline, or candidate database, or when a request names a hiring stage as the reason to send something.
---

# Candidate milestone gifting

Turns a recruiting-pipeline event into a sent Goody gift, with a preview
and an explicit human approval in between.

## Scope

This skill covers **one-off, externally triggered sends** — you (or a
scheduled job) noticed candidates crossed a stage, and you are gifting
them now.

It does **not** use Goody autogift rules. Autogift only fires on
`birthday`, `work_anniversary`, and `onboarding` — there is no ATS-stage
trigger. Recruiting milestones must be detected on your side and sent
through `goody_order_batches_create`. See `recurring-milestone-autogifts`
for the cases autogift does cover.

## Before anything else

1. `goody_me_get` — confirm the connected account and capture
   `server_time` (use it, not your own clock, for any date math).
2. `goody_workspaces_list` — pick the workspace and pass `workspace_id`
   explicitly on every later call. Never let it default.

If the user has more than one workspace, ask which one before spending.

Sending requires the `mcp.gifts` scope. Without it you can still search,
preview, and price — the commit is what fails. If it does, the fix is a
reconnection or a new token with that scope, not a retry.

## Flow

### 1. Resolve the candidates

Pull the cohort from the source system (Notion database, ATS export,
spreadsheet). You need a name and an email for each.

Decide whether they should become Goody contacts:

- **One-off send** — pass them inline as `recipients: [{ name, email }]`.
  Nothing is stored in Goody. This is the default for candidates.
- **Recurring relationship** — create contacts first so you can gift them
  again later. See the `ats-contact-sync` skill.

Never fabricate an email. If one is missing, list who was skipped and why.

### 2. Choose the gift

Match the budget to the milestone. Confirm the tier with the user if they
have not named one.

| Milestone | Typical band | Notes |
|---|---|---|
| Take-home / assessment complete | $15–25 | Coffee, snacks. High volume, keep it light. |
| Onsite or final round | $25–50 | |
| Offer extended | $50–100 | |
| Offer accepted | $75–150 | Often shipped physically — see `direct_send` below. |
| Candidate declined / not moving forward | $15–25 | Goodwill. Keep the message plainly appreciative, never consoling. |

Find products with `goody_products_search` (`price_min` / `price_max` in
**cents**). A **Gift of Choice** — discover via `goody_gift_of_choice_list`
— is usually the better default for candidates: you set an amount with
`variable_price` (cents) and they pick what they want, so you are not
guessing taste for someone you barely know.

`goody_products_search` renders an interactive picker widget. When it
does, say one line framing the choice and wait for the click. Do not
restate the products as a numbered list and do not ask which one they
want.

### 3. Pick a greeting card

`goody_cards_list` — required, and it also renders a picker widget. There
is no default card and you must not choose one silently; `card_id` is
required by both preview and create.

Filter by `occasion` where one fits. Recruiting milestones rarely map to a
stock occasion, so a clean congratulations or thank-you card is usually
right.

### 4. Write the message

Either write it yourself or call `goody_messages_generate`.

Keep it short, specific to the milestone, and free of anything that reads
as a commitment. Do not imply an outcome the recruiter has not decided —
a gift after a take-home is thanks for the time spent, not a signal that
an offer is coming. This is the single most common way an automated
recruiting gift causes a problem.

### 5. Preview — always

Call `goody_order_batches_preview` with the same inputs you intend to
send. It charges nothing and sends nothing. It returns:

```
{ preview_url, draft_batch_id, cart_price, total_price }
```

Lead your reply with `preview_url` so the user can open the real
assembled gift, then state `total_price` and the recipient count.

### 6. Confirm, then send

Show all of this and get a typed yes **to this gift and this total**:

- the card (name + preview link)
- the message
- the product and per-recipient amount
- `send_method` and its notification consequence
- `payment_method_id` (from `goody_payment_methods_list`)
- `from_name` — for candidates set this to a person or the team, not a
  bare company name
- expiration (defaults to 10 weeks if you omit it — say so)

`send_method` has no default and you must choose one:

- `link_single` — recipient is **not** notified; you get a link to send
  yourself. Right when the recruiter wants to deliver it personally in an
  existing email thread.
- `email_and_link` — Goody emails each recipient. Right for bulk stage
  sends.
- `direct_send` — ships a physical product; no email. Requires a complete
  `mailing_address` on every recipient, and is **incompatible with
  `contact_id`** because Goody contacts do not store addresses. Ask for
  addresses; never invent one.

Only then call `goody_order_batches_create` with `confirmed: true`.

`confirmed: true` means the user answered yes to the exact gift and total
you just showed. Having shown it is not approval. Praise is not approval.
A yes from before the gift changed is not approval — show it again.

**This charges real money and cannot be undone from here.**

## After sending

Report the batch id and, for `link_single`, the gift links. Write the
result back to the source system (e.g. stamp the Notion row) so the same
candidate is not gifted twice on the next run — Goody will not dedupe
this for you.

## Failure handling

- Missing or malformed email → skip that recipient, send the rest, report
  the skips explicitly. Do not silently drop them.
- Insufficient balance → surface the error text; do not retry blindly.
- Partial cohort → it is always better to send to the confirmed subset and
  report the remainder than to send nothing.

## Detailed send-flow reference

`reference/send-flow.md` has the full parameter-by-parameter walkthrough
of preview → price → create, including Gift of Choice amounts, swap
behavior, and international shipping.
