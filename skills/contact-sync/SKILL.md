---
name: contact-sync
description: Import people into Goody contacts and contact lists from wherever they live — a spreadsheet, CRM, ATS, HRIS export, Notion database, or CSV. Use when asked to import, sync, or build a gifting list of customers, employees, candidates, or event attendees held in another tool.
---

# Getting people into Goody

Turns rows in another system into Goody contacts and a named list you can
gift to repeatedly.

## When you need this — and when you don't

Create contacts when the relationship **recurs**: customers, employees,
anyone an autogift rule will fire on, anyone you'll gift more than once.

Skip it for one-off sends. `goody_order_batches_preview` and `_create`
accept inline `recipients: [{ name, email }]`, and storing a one-time
recipient as a permanent contact is often not what the user wants. Ask
before persisting people whose relationship may not continue.

## Before anything else

1. `goody_me_get` — confirm the connected account.
2. `goody_workspaces_list` — pass `workspace_id` explicitly on every call.

Needs `mcp.read` + `mcp.write`. No `mcp.gifts` required — this skill never
spends.

## Flow

### 1. Read the source and map the columns

Works the same whether the source is a CSV, a spreadsheet, a CRM export, a
Notion database, or an HRIS report.

| Goody field | Typical source column |
|---|---|
| `first_name` (**required**) | Name, First Name, Contact |
| `last_name` | often needs splitting from a single Name column |
| `email` | Email, Work Email |
| `phone` | needs the country code (see below) |
| `company` | Company, Account, Employer |
| `title` | Title, Role, Position |
| `birthday` | `YYYY-MM-DD` |
| `work_anniversary` | Start Date, Hire Date — `YYYY-MM-DD` |

Phone numbers are converted to international format (E.164). A full
number like `(415) 555-0100` or `+44 20 7946 0958` converts correctly. A
short local number like `555-0199` is saved as junk with no error. Leave
out numbers you can't make complete.

A start date maps to `work_anniversary`. It doesn't set up a first-day
(onboarding) gift; those dates come from a connected HRIS.

Show the user the mapping you inferred and let them correct it before you
write anything. A wrong column is cheap to fix now and tedious to unwind
after 200 contacts exist.

Splitting a single name column: split on the **first** whitespace so
particle surnames ("van Dijk", "de la Cruz") stay intact.

For `birthday` and `work_anniversary`, use year **1900** when only month
and day are known. Never guess a real year.

### 2. Check what already exists

`goody_contacts_search` before creating anything. Re-importing the same
source twice is the normal failure mode, and it produces duplicate contacts
that then receive duplicate gifts.

For each row:

- No match → create.
- Match, same data → leave alone.
- Match, changed data → `goody_contacts_update`.

Search results don't include `birthday` or `work_anniversary`. To compare
dates, call `goody_contacts_get` for each match.

`goody_contacts_create` refuses an email or phone that already exists in
the workspace (email ignores capitals), so a missed match on those fails
safely. Count it as "already there", not as an error. A contact with
neither email nor phone has no such guard: running the import again
creates a second copy.

### 3. Create

`goody_contacts_create` per person. Only `first_name` and `workspace_id`
are required, but a contact with no email can't receive an emailed gift.
Skip rows with no email and list them, unless the user says to create
them anyway. If they do, search by full name before creating on every
run, since nothing else stops a duplicate.

Report counts as you go: created, updated, skipped, failed. On a partial
failure keep going and report at the end. Don't abort the whole import
because row 40 has a malformed email.

### 4. Build the list

`goody_contact_lists_create` with a `name` and `contact_ids[]`.

Name it after the cohort and the period, not the tool it came from — "Q3
2026 enterprise customers" ages better than "Salesforce sync". Autogift
rules bind to this list, so the name gets read back months later.

## Keeping it current

None of this is a live connection. A contact list holds the
`contact_ids` passed when it was made. These tools can't add people to an
existing list or remove them, so later rows in the source don't appear on
their own.

Tell the user that up front when the list will drive an autogift rule.
New hires added next month won't be on it. The options:

- Add new people to the list in the Goody web app.
- Use one of Goody's native integrations if the source has one (below).
- Re-run this skill to create the new contacts. Write the Goody contact
  id back to each source row, so the next run can tell new people from
  ones already synced without searching each row.

Check `goody_contact_lists_list` before making another list. Names don't
have to be unique, so running the import again quietly makes a second
list with the same name. The listing shows each list's name and size, not
who is on it.

## Goody's native integrations

If the source is a supported HRIS or platform, Goody may already sync it
natively with real ongoing updates instead of a snapshot. These MCP tools
cover only part of what Goody does, so the absence of a tool here is not
evidence the integration doesn't exist — point the user at Goody support to
confirm rather than asserting either way.
