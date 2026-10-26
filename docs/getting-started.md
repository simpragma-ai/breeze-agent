# Getting started: Breeze in your AI assistant

Breeze lets you build and run AI phone agents — appointment reminders, rent and payment
reminders, renewals, surveys — by asking your AI assistant in plain language. You can create your
Breeze account in the browser window that opens when you first connect; there is no key to copy.

**The address your assistant connects to:** `https://studio.simpragma.com/mcp`

---

## 1. Connect your assistant

Pick the one you use. The first time, a browser window opens where you sign in or create your
account and approve (step 2).

| Assistant | How |
|---|---|
| **Claude Code** | `/plugin marketplace add simpragma-ai/breeze-agent`, then `/plugin install breeze@simpragma`. Restart, run `/mcp`, choose **breeze** → *Authenticate*. The plugin also teaches Claude how to design, test and run agents well. |
| **Claude** (web and desktop) | Settings → **Connectors** → *Add custom connector* → name `Breeze`, URL above → *Connect*. On Team and Enterprise plans an owner can add it once for the whole organisation. |
| **ChatGPT** (Plus, Pro, Business, Enterprise, Edu) | Settings → Security and login → turn on **Developer mode**; then add a connector with the URL above and choose OAuth. |
| **Microsoft Copilot Studio** | In your agent: *Tools* → *Add a tool* → **Model Context Protocol** → server URL above, authentication **OAuth 2.0 → Dynamic discovery**. |
| **GitHub Copilot in VS Code** | Command palette → *MCP: Add Server* → **HTTP** → URL above, name `breeze`. Or in `.vscode/mcp.json`: `{"servers": {"breeze": {"type": "http", "url": "https://studio.simpragma.com/mcp"}}}` |
| **Cursor** | Settings → MCP → *Add*: `{"mcpServers": {"breeze": {"url": "https://studio.simpragma.com/mcp"}}}` |
| **Codex** | `codex mcp add breeze --url https://studio.simpragma.com/mcp`, then `codex mcp login breeze` |
| **Gemini CLI** | In `settings.json`: `{"mcpServers": {"breeze": {"httpUrl": "https://studio.simpragma.com/mcp"}}}`, then `/mcp auth breeze` |

Anything else that supports remote MCP servers with OAuth works the same way.

## 2. Sign in (or sign up) and approve

1. On **Sign in or create your Breeze account to connect**:
   - already have an account — sign in;
   - new — click **Create one**, then **Continue with Google / Microsoft** with your **work**
     account, or enter your work email and a password. (Personal addresses such as gmail.com
     aren't accepted.) Breeze emails you a link to verify your address; click it before your first
     campaign.
2. **First person from your company?** Your workspace is created there and then, and you are its
   admin. Answer **Tell us about your business**: your business name, the country you'll call
   customers in (this sets your currency and calling hours, and can't be changed later) and the
   kind of business.
   **Colleague already on Breeze?** You're asked to join their workspace (an admin approves you
   under **Invitations**) or to create a new one.
3. On the approval screen, check the workspace and choose what the assistant may do:

   | Permission | Lets the assistant | Default |
   |---|---|---|
   | **View** | see agents, campaigns, results, transcripts and spend | always on |
   | **Build** | create and change agents, campaigns and lists | on |
   | **Launch calls** | call your own phone to test, start / pause / resume campaigns | **off** — tick it to try calls |

4. Click **Allow**. You're back in your assistant, connected.

> **Microsoft 365 companies:** if your IT department blocks users from approving new apps, you'll
> see "Need admin approval". Ask IT to approve the **Breeze** app for your organisation once
> (Microsoft Entra admin center → Enterprise applications).

## 3. Try it

Your workspace starts with free credit, in **test mode**: it can call only your own verified phone
and test numbers. Ask your assistant, in your own words:

1. *"What's my Breeze balance?"*
2. *"Verify my phone number +44 7…"* — Breeze calls you and reads a 4-digit code; tell your
   assistant the code.
3. *"Build me an agent that reminds tenants their rent is due, asks when they'll pay, and notes
   any problem they mention."* — Breeze works out the kind of call (here, debt collection) and
   applies the right rules. You don't choose it.
4. *"Give me a test call from the agent."* — your phone rings; talk to it like a customer would.
   Ask for changes ("make it shorter") and call again.
5. *"Start a campaign that calls me with name Mate, amount £120, due Friday. Allow calls today."*
   — you see who will be called, when, the rules that apply and a short lawful-calling statement.
   Nothing is dialled until you say yes.
6. *"How did it go? Show me the transcript and export the results."*
7. *"Show my billing summary."*

## 4. What keeps you safe

- **Test mode** until you ask Simpragma to take your workspace live.
- **Every launch shows a summary and waits for your yes** — the assistant can't skip it.
- **Calling hours, public holidays, call limits and opt-outs** are enforced by Breeze itself,
  whatever the assistant asks for. New agents call Monday–Saturday 09:00–19:00. Anyone who asks
  not to be called again, or replies STOP to a text, is never contacted by your workspace again.
- **Every call** says it's an AI assistant and that it's recorded, checks it's the right person
  before saying anything personal, and stops kindly if someone is in a difficult situation.
- **Do-not-call registers are your responsibility.** Only upload people you may lawfully call,
  screened against the registers that apply (US National Do Not Call Registry, UK TPS / CTPS,
  India NCPR / DND). Breeze doesn't check these registers, and every launch asks you to confirm.
- **Prepaid credit:** calls stop when credit runs out; you can be emailed before it does. Add
  credit in Breeze Studio → **Billing**.
- **The assistant only gets the permissions you ticked**, only for the workspace you picked.

See [Calling rules and compliance](https://breeze.simpragma.com/docs/calling-rules) for the hours
by country.

## 5. Manage access

- **Disconnect an assistant:** Breeze Studio → **API keys** → *Connected apps* → *Disconnect*
  (workspace admins). It stops working immediately.
- **Someone leaves:** remove them from the workspace; every assistant they connected stops working.
- **Scripts and CI:** create a scoped API key on the same page and send it to the same address as
  `Authorization: Bearer bk_…` — see the [README](../README.md#scripts-and-ci-api-keys).

## Troubleshooting

| You see | Do this |
|---|---|
| "That sign-in was refused" | Use your work email. With Microsoft, your organisation must have verified your email — ask IT, or sign up with your email and a password instead. |
| "An account with this email already exists" | Sign in the way you did before (for example with your password). |
| *Create or join a workspace* | Someone from your company is already on Breeze: ask to join, or create your own workspace. |
| "This connection request is not valid" | Go back to your assistant and connect again. |
| Tools stopped working | Your access was disconnected or changed: reconnect (Claude Code: `/mcp` → breeze → *Authenticate*). |
| "requires the dial scope" | You didn't tick **Launch calls**. Disconnect and connect again, ticking it. |
| Launch refused: email not verified | Click the link in the email Breeze sent you. |
| Launch refused: test mode | Only your verified phone and test numbers can be called until your workspace is live. |
| The campaign doesn't call | Ask your assistant why — usually it's waiting for the allowed days or hours. |

Questions: your Simpragma contact.
