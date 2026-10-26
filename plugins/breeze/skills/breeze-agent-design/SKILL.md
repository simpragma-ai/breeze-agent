---
name: breeze-agent-design
description: Use when building or changing a Breeze agent — turning someone's description into create_agent (role, goal, flow, facts, greeting, outcomes), choosing create_agent_from_template instead, editing an agent with update_agent / set_agent_facts / set_agent_languages, or fixing an agent that sounds robotic, asks two things at once, says the wrong thing to the wrong person, promises a transfer, or hangs up in the wrong place. Not for outcome codes in depth (breeze-outcomes), test calls (breeze-testing) or campaigns (breeze-campaigns).
---

# Designing a Breeze agent

The agent speaks down a phone line to a real person who cannot re-read anything and will hang
up if it sounds like a form. Your job is to turn what the business owner told you into a short,
clear definition. Breeze turns that into the full prompt and adds its own rules.

## What Breeze already does — don't write it yourself

Every agent gets Breeze's shared call rules on every call, whatever its own text says:

- It says it is an **AI assistant** and that the **call is recorded** (added to the greeting if
  yours doesn't say it), and answers truthfully if asked.
- **Right person first:** no personal or account detail until the named person confirms; nothing
  at all to anyone else.
- **Short replies, one question per turn**, acknowledge before asking, numbers and dates spoken
  naturally, no symbols.
- **It cannot transfer a call** and never says it will; it offers a callback instead.
- **Stop / don't call me:** it ends politely at once, and the number goes on the workspace's
  do-not-call list.
- **People in a difficult situation** (money trouble, illness, bereavement, disability, caring,
  confused, distressed, a child): it stops the task, takes no sale or promise, offers a callback
  from a person; the call is marked `needs_care`.
- **Hang-up, hold, "can you hear me?"** and unclear speech are handled.
- **Sales calls** offer an opt-out; **debt collection** gets its own script rules (no threats or
  false consequences, agree what the person can manage, disputes noted not argued, and the
  debt-collector disclosure when collecting for another company).
- **The call type** (sales, service or debt collection) is worked out by Breeze from the agent
  itself — never ask the person for it, and there is no parameter for it. It decides the
  calling hours and which of these rules apply. `get_agent` shows it.

So your definition is about **this business and this call**, not safety boilerplate. If you put
"never transfer" or "say you are an AI" in the flow, it does no harm, but it adds nothing.

## create_agent — the normal path

Use it for anything specific (most things). `create_agent_from_template` only when
`list_templates` has a template that genuinely matches — read it with `get_template` first.

```
create_agent({
  name: "Rent reminders",
  role: "Sam, calling for Oakfield Lettings, a letting agent in Leeds",
  goal: "Remind the tenant their rent is due, find out when they will pay, and note any problem they mention.",
  flow: [
    "Say the rent amount and the date it is due",
    "Ask when they expect to pay",
    "If they mention a problem with the property, ask briefly what it is",
    "Confirm what was agreed once, then say goodbye"
  ],
  facts: [
    {name: "company_name", source: "agent", must: true},
    {name: "tenant_name", source: "call", must: true, description: "Tenant's name", example: "James Whitfield"},
    {name: "rent_amount", source: "call", must: true, description: "Rent due", example: "£950"},
    {name: "due_date", source: "call", must: true, description: "Date the rent is due", example: "1 November"}
  ],
  greeting: "Hello, this is Sam calling from {company_name}. Am I speaking with {tenant_name}?",
  outcomes: {positive: ["WILL_PAY_ON_TIME", "ALREADY_PAID"], callback: ["CALLBACK_REQUESTED", "PROPERTY_ISSUE"], failure: ["WONT_PAY"]},
  company_name: "Oakfield Lettings"
})
```

### role — one sentence
Who it is and who it calls for. A first name and the business make it sound like a person.

### goal — what a good call achieves
One or two sentences. Say what counts as done, so it knows when to close.

### flow — the steps, one ask each
- Short imperative steps, in order; Breeze numbers them and enforces one ask per turn.
- Put the branch inline where it applies ("If they already paid, thank them and close").
- No identity-check step (Breeze does that), no transfer step, no "say you are an AI" step.
- Take a rough answer: "next week" is enough unless the goal truly needs a date.

### facts — what it knows
Decide each by one question: **does it differ between two people in the same campaign?**

| | source | where the value comes from |
|---|---|---|
| Company name, agent name, opening hours, payment options | `"agent"` | `company_name` / `set_agent_facts` — same on every call |
| The person's name, amount, date, reference | `"call"` | each contact's `variables` in `add_contacts`, or once for a whole run with `create_run({facts})` |

Mark `must: true` on anything the call cannot happen without. A person missing a must-have is
not called and is listed back to you. Never write a per-person value into role, goal or flow —
those are spoken to everyone.

### greeting — optional
One short sentence with `{fact}` placeholders. A sentence whose fact is missing is dropped, so a
blank name never produces "…am I speaking with?". Omit it for a sensible default.

### outcomes — required
Codes for how each call ended, grouped positive / callback / failure, designed for **this**
goal. WRONG_PERSON, NO_RESPONSE and OTHER are added for you. See **breeze-outcomes**.

### languages
UK and US workspaces default to English; India to all six (en, hi, ta, te, kn, ml). One agent
switches when the person asks. Change with `set_agent_languages`; check with
`get_agent_languages`.

## After creating it

1. `get_agent` — tell the person in two or three lines what it will say, what it needs per
   person, and its call type.
2. `get_agent_facts` — anything in `missing_on_agent` must be set with `set_agent_facts` before
   contacts can be added.
3. A test call — **breeze-testing**. Before the first launch Breeze also runs an automatic
   **go-live check**; `launch_run` returns `REVIEW_BLOCKED` with the issues if something must
   change (for example a sales agent written as service, or a prompt that asks for card numbers).

## Changing an agent

| Change | Tool |
|---|---|
| Name, or its whole prompt | `update_agent({campaign_id, name?, custom_system_prompt?})` — `null` prompt goes back to the template's script |
| Company name, opening hours, other agent facts; must-have on/off | `set_agent_facts` |
| Languages | `set_agent_languages` |

A changed prompt re-checks the call type and re-runs the go-live check before the next launch.
When you write a full `custom_system_prompt`, keep the same discipline: no per-person values in
it (they arrive in a silent "[Context — …]" note on each turn), one ask per step, a clear goal
and a clear close. Breeze's shared rules are still added after it and win if they conflict.

## Fixing what you hear on a test call

| Symptom | Usual fix |
|---|---|
| Says the wrong name / amount | the fact is wrong or missing on the contact — fix the data, not the prompt |
| Two questions at once, long replies | split the flow step |
| Argues or repeats after a no | add the branch to the step: "If they say no, thank them and close" |
| Doesn't know when to finish | sharpen the goal and make the last step "confirm once, then say goodbye" |
| Says something it can't know (prices, availability, offers) | add a step or branch: "If asked about X, say a colleague will follow up" |
| Wrong register ("awesome", "reach out") on a UK call | say the register in the role: "warm, plain British English" |

Change one thing, call again, compare.
