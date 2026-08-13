# Goody skills

Gifting recipes for the [Goody](https://www.ongoody.com) MCP server — send
and automate business gifts from the tools you already use.

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

## Using a recipe

Once Goody is connected, hand your assistant a recipe. Two ways — both
take about ten seconds.

**Paste the recipe in.** Works in every assistant, no browsing needed.

1. Click a **Copy** link in the table below. You'll get plain text.
2. Select all (`Cmd/Ctrl+A`), copy (`Cmd/Ctrl+C`).
3. Paste it into your chat, then add what you want to send:

```
[paste the recipe here]

---

Follow the recipe above. Send a $25 coffee gift to the five people in
this list, from me, and show me a preview first.
```

**Or just give it the link**, if your assistant can browse the web:

```
Read this gifting recipe and follow it:
https://raw.githubusercontent.com/ongoody/goody-skills/main/skills/send-gifts/SKILL.md

Here's what I want to send:
```

Either way your assistant follows the recipe for the rest of that
conversation.

## The recipes

| Recipe | What it does | |
|---|---|---|
| [`send-gifts`](skills/send-gifts) | The core flow — pick a gift and card, write the message, preview, confirm, send. Start here. | [Copy](https://raw.githubusercontent.com/ongoody/goody-skills/main/skills/send-gifts/SKILL.md) |
| [`event-triggered-gifting`](skills/event-triggered-gifting) | Gift when something happens elsewhere — a deal closes, a candidate clears a stage, a customer renews, a ticket gets a great CSAT. | [Copy](https://raw.githubusercontent.com/ongoody/goody-skills/main/skills/event-triggered-gifting/SKILL.md) |
| [`contact-sync`](skills/contact-sync) | Import people from a spreadsheet, CRM, ATS, HRIS, or Notion database into Goody contacts and lists. | [Copy](https://raw.githubusercontent.com/ongoody/goody-skills/main/skills/contact-sync/SKILL.md) |
| [`autogift-rules`](skills/autogift-rules) | Always-on rules for birthdays, work anniversaries, and onboarding. | [Copy](https://raw.githubusercontent.com/ongoody/goody-skills/main/skills/autogift-rules/SKILL.md) |

**Which one?** If you're sending now, `send-gifts` is all you need. If the
gift should fire off something in another system, the dividing line is what
triggers it: Goody automates birthdays, work anniversaries, and onboarding
itself (`autogift-rules`), and everything else — any CRM stage, hiring
stage, or support event — has to be spotted by your system, which then
tells Goody to send (`event-triggered-gifting`).

## Making them permanent

Pasting works per conversation. To have your assistant reach for these on
its own, every time — one command:

```sh
npx skills add ongoody/goody-skills --all
```

That installs all four into Claude Code, Cursor, Cline, Copilot, Windsurf,
and 30+ other agents. Add `--global` to install for every project, or
`--skill send-gifts` to take just one. `npx skills add ongoody/goody-skills --list`
shows what's available first.

**Claude Code**, if you'd rather have the recipes and the Goody connection
together in one step:

```
/plugin marketplace add ongoody/goody-skills
/plugin install goody@goody-skills
```

You can also just paste a recipe into Claude Code and say *"save this as a
skill at `.claude/skills/send-gifts/SKILL.md`"* — it writes the file
itself.

**Claude Desktop / claude.ai** — this one is manual. Claude there can't add
skills to its own settings, so pasting and asking won't stick. Download a
recipe folder and upload it under Settings → Capabilities → Skills. Team
and Enterprise admins can push skills to everyone at once, which beats each
person installing their own.

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
