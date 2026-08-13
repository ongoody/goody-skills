---
name: ats-contact-sync
description: Sync people from an external system — a Notion database, ATS, HRIS export, or spreadsheet — into Goody contacts and contact lists. Use when asked to import, sync, or build a gifting list from candidates, new hires, or employees held in another tool.
---

# Syncing people into Goody contacts

Turns rows in someone else's system into Goody contacts and a named
contact list you can gift to repeatedly.

## When you need this — and when you don't

Create contacts when the relationship **recurs**: employees, new hires,
anyone an autogift rule will fire on, anyone you will gift more than once.

Skip it for one-off sends. `goody_order_batches_preview` and
`_create` accept inline `recipients: [{ name, email }]`, and storing a
rejected candidate as a permanent contact is usually not what the user
wants. Ask before persisting people who may not stay in scope.

## Before anything else

1. `goody_me_get` — confirm the connected account.
2. `goody_workspaces_list` — pass `workspace_id` explicitly on every call.

## Flow

### 1. Read the source

For a Notion database, query it and identify which properties map to:

| Goody field | Typical source property |
|---|---|
| `first_name` (**required**) | Name, Candidate, Full Name |
| `last_name` | — often needs splitting from a single Name field |
| `email` | Email, Work Email |
| `phone` | Phone — any format, Goody normalizes to E.164 |
| `company` | Company, Employer |
| `title` | Role, Title, Position |
| `birthday` | `YYYY-MM-DD` |
| `work_anniversary` | Start Date, Hire Date — `YYYY-MM-DD` |

Show the user the mapping you inferred and let them correct it before you
write anything. A wrong column mapping is cheap to fix now and tedious to
unwind after 200 contacts exist.

When splitting a single name field, split on the **first** whitespace so
particle surnames stay intact.

For `birthday` and `work_anniversary`, use year **1900** when only month
and day are known. Do not guess a real year.

### 2. Check what already exists

Call `goody_contacts_search` before creating. Re-importing the same
Notion database twice is the normal failure mode here and it produces
duplicate contacts that then receive duplicate gifts.

For each source row:

- No match → create it.
- Match with the same data → leave it alone.
- Match with changed data (new title, new email) → `goody_contacts_update`.

### 3. Create

`goody_contacts_create` per person. Only `first_name` and `workspace_id`
are required, but a contact with no email cannot receive an emailed gift —
flag those rather than creating a contact that will fail at send time.

Report counts as you go: created, updated, skipped, failed. On a partial
failure, keep going and report the failures at the end. Do not abort the
whole import because row 40 has a malformed email.

### 4. Build the list

`goody_contact_lists_create` with a `name` and `contact_ids[]`.

Name it after the cohort and the period, not the tool that produced it —
"Q3 2026 new hires" ages better than "Notion sync". The list id is what
autogift rules bind to, so this name will be read back months later.

## Keeping it in sync

Nothing here is a live connection. A contact list is a snapshot of the
`contact_ids` you passed at creation time; later rows in Notion do not
appear on their own.

For a recurring sync, the honest options are:

- Re-run this skill on a schedule, using `goody_contacts_search` to add
  only the new people.
- Write a Goody contact id back to each source row on creation, so the
  next run can tell new from already-synced without a search per row.

The second is worth the extra column once the cohort passes a few dozen
people.

## Goody's own HRIS integrations

If the source is a supported HRIS rather than a homegrown database, Goody
may already sync it natively, with real ongoing updates instead of a
snapshot. These MCP tools cover only part of what Goody does, so the
absence of a tool here is not evidence the integration does not exist —
point the user at Goody support to confirm rather than asserting either
way.
