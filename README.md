# Goody skills

Gifting recipes for the [Goody](https://www.ongoody.com) MCP server —
automate candidate and employee appreciation from the tools you already
use.

## Get started

1. Open **[Connect AI tools](https://www.ongoody.com/plus/account/ai-tools)**
   in Goody and copy your MCP server URL.
2. Paste it into Claude, ChatGPT, Cursor, or any assistant that speaks
   MCP — into its connector settings, or straight into the chat and ask it
   to connect.
3. Sign in to Goody and approve the access it asks for.

That's the whole setup. Your assistant can now browse products, draft
cards, and send gifts on your behalf.

> Running from a script, cron job, or n8n / Zapier / Make instead? Mint a
> token on the **[Personal MCP token](https://www.ongoody.com/plus/account/api/mcp)**
> page — those need a token as well as the URL.

## The recipes

Each folder in `skills/` is a plain markdown recipe. Ask your assistant to
read one, or fork it and make it yours.

| Recipe | What it does |
|---|---|
| [`candidate-milestone-gifting`](skills/candidate-milestone-gifting) | Gifts candidates when they hit a recruiting stage — take-home done, onsite, offer, accepted, declined. |
| [`ats-contact-sync`](skills/ats-contact-sync) | Pulls people from a Notion database, ATS, or spreadsheet into Goody contacts and lists. |
| [`recurring-milestone-autogifts`](skills/recurring-milestone-autogifts) | Always-on rules for birthdays, work anniversaries, and onboarding. |
| [`gifting-budget-guardrails`](skills/gifting-budget-guardrails) | Keeps automated spend inside what you meant to spend. |

**Which one?** It depends on what triggers the gift. Goody's autogift
engine fires on exactly three events — birthdays, work anniversaries, and
onboarding — and runs those for you. Anything else, including any ATS
stage, has to be spotted by your own system, which then tells Goody to
send. There is no custom autogift trigger.

## Using them in Claude Code

```
/plugin marketplace add ongoody/goody-skills
/plugin install goody@goody-skills
```

This installs the recipes and the Goody connection together — no separate
setup needed.

To vendor one into your own project instead, copy the folder into
`.claude/skills/`:

```sh
git clone https://github.com/ongoody/goody-skills
cp -r goody-skills/skills/candidate-milestone-gifting .claude/skills/
```

## Before you point an agent at a payment method

Every recipe here previews the gift, shows the total, and waits for you to
say yes. Sending charges real money and can't be undone from the
assistant. Please keep that shape when you fork.

Then set a **Daily AI gifting limit** on the
[Connect AI tools](https://www.ongoody.com/plus/account/ai-tools) page. It
caps what connected assistants can spend in any rolling 24 hours and
defaults to $5,000 — gifts you send yourself are never counted against it.
It's the only limit that still holds when a query returns 200 people
instead of the 20 you expected.

## More

- [MCP documentation](https://developer.ongoody.com/mcp/overview)
- Bugs and rough edges: `mcp@ongoody.com`, or ask your assistant to send
  feedback mid-conversation
- Account, billing, or an existing gift: Goody's help center, live chat, or
  `support@ongoody.com`

MIT licensed.
