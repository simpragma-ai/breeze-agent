# Breeze for your AI assistant

Build, test and run [Breeze](https://studio.simpragma.com) AI phone agents from the assistant you
already use — Claude, Claude Code, ChatGPT, Codex, Cursor, Copilot or Gemini CLI.

| | What it is | What it gives you |
|---|---|---|
| **The Breeze connection** (MCP server) | `https://studio.simpragma.com/mcp`, hosted by Breeze | Tools that *do* things: build an agent, call your own phone, load a list, launch a campaign, read results and spend |
| **The Breeze plugin** (skills) | `plugins/breeze/skills/` | Know-how that makes your assistant do them *well*: designing an agent, choosing outcomes, what to test, the calling rules |

The connection lets an assistant act; the skills stop it from building a technically valid agent
that sounds like a form, scores the wrong outcomes, or gets launched without the person's own yes.

**New here? Read [docs/getting-started.md](docs/getting-started.md).**

## Install

**Claude Code** — the connection and the skills in one plugin:

```
/plugin marketplace add simpragma-ai/breeze-agent
/plugin install breeze@simpragma
```

Restart Claude Code when asked, run `/mcp`, pick **breeze** → **Authenticate**. A browser opens:
sign in — or create your Breeze account right there — answer two questions about your business,
choose what the assistant may do, and click **Allow**. No keys to copy.

(Connection only, without the skills: `claude mcp add --transport http breeze https://studio.simpragma.com/mcp`.)

**Other assistants** get the connection; the setup for each is in
[docs/getting-started.md](docs/getting-started.md#1-connect-your-assistant). Skills are a Claude
feature, so elsewhere the files in [`plugins/breeze/skills/`](plugins/breeze/skills/) are plain
markdown you can read or paste in — they are written to make sense to a person too.

Workspace admins see and disconnect connected assistants in Breeze Studio → **API keys** →
*Connected apps*.

### Scripts and CI: API keys

Where no browser can open, use the same address with a scoped API key (Studio → **API keys**) as
a Bearer token:

```json
{ "mcpServers": { "breeze": { "type": "http", "url": "https://studio.simpragma.com/mcp",
  "headers": { "Authorization": "Bearer ${BREEZE_API_KEY}" } } } }
```

## Skills

| Skill | Use it for |
|---|---|
| `breeze-getting-started` | the first run: check credit, verify your phone, build an agent, hear it, first small campaign |
| `breeze-agent-design` | turning a description into a good agent; changing one |
| `breeze-outcomes` | the outcome codes an agent reports, and reading results |
| `breeze-testing` | test calls to your own phone, and what to try before going live |
| `breeze-campaigns` | lists, campaigns, days and hours, progress, results, credit and spend |
| `breeze-compliance` | calling hours and holidays, call types, launching, refusals |
| `breeze-phone-numbers` | your own number for calling customers, and inbound calls |

## Permissions: start narrow

| On the sign-in screen | Scope | Lets the assistant |
|---|---|---|
| **View** | `read` | see agents, campaigns, results, transcripts and spend — always on |
| **Build** | `write` | create and change agents, campaigns and lists |
| **Launch calls** | `dial` | test calls to your own phone, launch / pause / resume campaigns, buy a number — **off unless you tick it** |

Even with **Launch calls**, nothing reaches a customer without your explicit yes to a launch
summary, and a phone number is never bought without your yes to its price.

## The rules are on the server, not in the tools

Everything that matters is enforced by Breeze itself, however the request arrives — an
assistant, a script, a CRM or a person in Studio:

- **Calling hours and public holidays** by country and call type (sales, service, debt
  collection). Breeze works out the call type itself; there is no override.
- **Domestic calls only**, from the business's own number.
- **At most 7 calls to one person in 7 days** (debt collection: once a day).
- **Opt-outs** — "don't call me again" or STOP to a text — honoured for good.
- **AI and recording notices**, right-person checks and care for people in a difficult situation,
  on every call.
- **A go-live check** on every agent before its first campaign.
- **Test mode** for new workspaces: only your own verified phones and test numbers.
- **Prepaid credit**: no credit, no calls.
- **Launching is two steps.** The first `launch_run` returns a summary and an **attestation** —
  that you have a lawful basis to call everyone, screened the list against the do-not-call
  registers that apply (US National DNC, UK TPS/CTPS, India NCPR — Breeze doesn't check these),
  and removed opt-outs — and launches nothing. Only a second call with `confirm: true` and the
  attestation's id launches, after you've said it is true; your acceptance is recorded.

An assistant cannot talk its way past any of these, because none of them live in the tools.

## Layout

```
.claude-plugin/marketplace.json    the marketplace
plugins/breeze/
  .claude-plugin/plugin.json
  .mcp.json                        the Breeze connection
  skills/                          the seven skills above
docs/getting-started.md            the guide for business users
```

## Support

Your Simpragma contact.
