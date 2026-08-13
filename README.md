# Goody skills

Gifting recipes for the [Goody](https://www.ongoody.com) MCP server —
starting points for automating candidate and employee appreciation from
your own systems.

These are meant to be **forked**. Copy one, change the budget bands, the
message tone, and the milestones to match how your team actually gifts.

## Step 1 — connect Goody

Goody is not in any AI client's built-in connector directory. You add the
server URL yourself. Which page you start from depends on how you'll call
it.

### Interactive AI clients — Claude Desktop, claude.ai, Claude Code, Cursor, ChatGPT

Go to **[Connect AI tools](https://www.ongoody.com/plus/account/ai-tools)**
(Goody → Account → Connect AI tools), copy the MCP server URL, and paste
it into your client's connector settings. You'll sign in to Goody and
approve the scopes the client asks for. No token to copy.

That page also lists every connected client with a **Disconnect** button,
and sets your **Daily AI gifting limit** — see below.

### Scripts, cron jobs, n8n / Zapier / Make, headless agents

Anywhere a browser OAuth flow isn't possible, mint a token at
**[Personal MCP token](https://www.ongoody.com/plus/account/api/mcp)**
(Goody → Account → API → Personal MCP token). You need **both** the server
URL and the token in your MCP client config.

The token plaintext is shown **once**, at creation. Copy it then — Goody
never displays it again.

### Scopes

| Scope | Grants |
|---|---|
| `mcp.read` | Read-only — search, list, get |
| `mcp.write` | Mutate non-spend resources — contacts, lists, previews |
| `mcp.gifts` | **Send gifts and activate autogift rules — spends money** |

The skills here need `mcp.gifts` to complete a send.
`candidate-milestone-gifting` and `recurring-milestone-autogifts` will
otherwise get as far as a priced preview and then fail at the commit step.
`ats-contact-sync` needs only `mcp.read` + `mcp.write`.

Grant `mcp.gifts` deliberately, and pair it with a daily limit.

## Step 2 — install the skills

### Claude Code

```
/plugin marketplace add ongoody/goody-skills
/plugin install goody@goody-skills
```

`marketplace add` points Claude Code straight at this repo — no directory
listing or approval is involved. The plugin brings the skills **and** the
Goody server config, so this covers step 1 too.

### Vendor into your own repo

Copy any skill folder into `.claude/skills/` in your project, or
`~/.claude/skills/` for personal use:

```sh
git clone https://github.com/ongoody/goody-skills
cp -r goody-skills/skills/candidate-milestone-gifting .claude/skills/
```

Keep the frontmatter `metadata` block when you fork, so you can tell which
upstream version you started from.

### Claude Desktop / claude.ai

Download a skill zip from
[Releases](https://github.com/ongoody/goody-skills/releases) and upload it
under **Settings → Capabilities → Skills**. Workspace admins can push
skills to everyone in the org, which is usually what you want for a team
of recruiters who shouldn't each be installing things.

Connect the server separately via **Connect AI tools**, above.

## Full MCP documentation

[developer.ongoody.com/mcp/overview](https://developer.ongoody.com/mcp/overview)

## Skills

| Skill | For |
|---|---|
| `candidate-milestone-gifting` | One-off sends triggered by recruiting-pipeline stages: take-home complete, onsite, offer, accept, decline. |
| `ats-contact-sync` | Importing people from a Notion database, ATS, or spreadsheet into Goody contacts and lists. |
| `recurring-milestone-autogifts` | Always-on rules for birthdays, work anniversaries, and onboarding. |
| `gifting-budget-guardrails` | Keeping automated spend inside its intended envelope. |

## Which one do I need?

The dividing line is **what triggers the gift**.

Goody's autogift engine fires on exactly three events — `birthday`,
`work_anniversary`, `onboarding`. If your trigger is one of those, use
`recurring-milestone-autogifts` and Goody runs it for you.

Anything else — a candidate reaching an ATS stage, a deal closing, a
project shipping — has to be detected in **your** system, which then calls
`goody_order_batches_create`. That is `candidate-milestone-gifting`. There
is no custom autogift event type, and no amount of configuration produces
one.

## The rule every skill enforces

`goody_order_batches_create` charges real money and cannot be undone from
the MCP tools. Every skill here previews the gift, shows the total, and
requires an explicit human yes before committing.

For unattended automation, that approval is captured when the automation
is **set up** — with a representative gift and the expected spend on
screen — because nobody is present when it fires.

Please keep that shape when you fork.

## The backstop

Set a **Daily AI gifting limit** on the
[Connect AI tools](https://www.ongoody.com/plus/account/ai-tools) page. It
caps what connected AI tools can spend on gifts in any rolling 24-hour
window, and defaults to **$5,000**. Gifts you send yourself on Goody are
never counted against it.

Set it to what you actually intend to spend in a day before you point an
agent at a payment method. It is the only limit that holds when a query
returns 200 rows instead of the 20 you expected.

## Feedback

Bugs and rough edges: `mcp@ongoody.com`, or ask Claude to call
`goody_feedback_create` mid-conversation.

For account, billing, or existing gifts, use Goody's normal support
channels — help center, live chat, or `support@ongoody.com`.

## License

MIT
