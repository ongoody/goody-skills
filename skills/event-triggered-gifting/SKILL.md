---
name: event-triggered-gifting
description: Send Goody gifts automatically when something happens in another system — a deal closes in the CRM, a candidate clears a hiring stage, a customer renews, a project ships, a ticket gets a great CSAT, an attendee registers. Use when gifting should be driven by events in a CRM, ATS, project tracker, support desk, database, or spreadsheet.
---

# Gifting triggered by events elsewhere

Turns "X happened in our system" into a sent Goody gift, with a preview
and a human yes in between.

## What this covers, and what it doesn't

This is for events **your** systems know about. You detect them; you send.

Goody's own autogift engine handles exactly three recurring events —
`birthday`, `work_anniversary`, `onboarding` — and runs those on its own.
Everything else lives here. See the `autogift-rules` skill for the three
Goody automates.

Say that plainly when someone asks for an autogift rule on a CRM stage or
a support event. Do not approximate it with a birthday rule.

## Common triggers

| Where | Event | Typical gift |
|---|---|---|
| CRM | deal closed / won | $75–150, often shipped |
| CRM | renewal, upsell, contract anniversary | $50–100 |
| ATS | take-home submitted, onsite completed, offer accepted | $15–150 by stage |
| Support | outstanding CSAT, a save after an escalation | $25–50 |
| Product | milestone reached, big upgrade, beta feedback given | $25–75 |
| Events | registered, attended, spoke, staffed a booth | $15–50 |
| People ops | project shipped, peer recognition, referral hired | $25–100 |

Bands are a starting point. Ask what the team actually spends rather than
assuming these.

## Flow

### 1. Read the trigger source

Query the system for the people who crossed the event in the window the
user cares about. Restate what you filtered on and how many came back
before pricing anything.

You need a name and an email per person. Never invent one — list who you
skipped and why.

### 2. Decide whether they become contacts

- **One-off** — pass inline `recipients: [{ name, email }]`. Nothing is
  stored. Right for candidates, event attendees, one-time thank-yous.
- **Ongoing relationship** — create contacts so you can gift them again
  and drive autogift rules. See `contact-sync`.

Ask before persisting people whose relationship may not continue.

### 3. Send

Follow the `send-gifts` skill for the whole flow: choose the gift and
card, write the message, preview, confirm, then
`goody_order_batches_create`. Nothing about being event-triggered changes
the approval rules.

Match the message to the trigger, and keep it free of anything that reads
as a commitment. A gift after a hiring stage is thanks for the time spent,
not a signal an offer is coming. A gift after a renewal is appreciation,
not a discount. This is the most common way an automated gift causes a
problem.

### 4. Write the result back

Stamp the source record — a Notion property, a CRM field, a spreadsheet
column — with the batch id or a sent-on date. Goody will not dedupe for
you, and the next run will re-send to everyone still matching the filter.

This is the step people skip, and it is the one that causes duplicate
gifts.

## Running it unattended

For a scheduled job with nobody watching:

- Approval is captured when the automation is **set up**, not per run.
  Show a representative gift and the expected spend, and confirm before
  enabling it.
- Mint a token scoped to exactly what it needs. A job that only syncs
  contacts and builds previews for a human should get `mcp.read` +
  `mcp.write` and no `mcp.gifts` — a token that cannot spend beats a
  prompt that says not to.
- Make the source query narrow and the write-back reliable. An unattended
  send with a broad filter and no write-back is the worst combination.
