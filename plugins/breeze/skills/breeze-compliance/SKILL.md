---
name: breeze-compliance
description: Use before launching any Breeze campaign (launch_run's two steps and the lawful-calling attestation), when a person asks to call at night, on a Sunday or a holiday, or abroad, when a launch is refused (REVIEW_BLOCKED, SANDBOX_RESTRICTED, CALLER_ID_REQUIRED, EMAIL_NOT_VERIFIED, INSUFFICIENT_BALANCE, ATTESTATION_REQUIRED, INTERNATIONAL_NOT_ALLOWED), or when someone asks what Breeze does to keep calls lawful — calling hours and holidays by country, call types, call limits, do-not-call registers, opt-outs and people in a difficult situation.
---

# Calling lawfully with Breeze

These calls reach real people and are regulated. Breeze enforces the rules below on the server,
however a call is started — Studio, the API, you, or a CRM — and **none of them can be switched
off from a tool**. Your part: explain them plainly, plan within them, and get the person's own
yes before anything dials.

**If asked to get round a rule** (call at 22:00, on a bank holiday, abroad, skip the
attestation): say plainly it can't be done and why, and offer the nearest thing that can —
usually the next time calls are allowed. Don't retry, don't look for a workaround.

This is guidance, not legal advice.

## The call type decides the rules — and Breeze decides the call type

Every agent is **sales**, **service** (reminders, confirmations, updates to existing customers)
or **debt collection** (asking people to pay money they owe). Breeze works it out from the
agent's template, description and an AI check when it is created or its prompt changes; the
stricter answer wins. Nobody chooses it and no tool takes it. `get_agent` shows it — tell the
person what it is, and if they think it's wrong, the fix is a clearer agent description, not a
setting.

## When calls may be made

Checked in the **person's** local time. Sales and debt collection are never made on a public
holiday in the person's country.

| | Sales | Debt collection | Service |
|---|---|---|---|
| **UK** | 08:00–21:00 · no bank holidays | 08:00–21:00 · no bank holidays | 08:00–21:00 |
| **US** | Mon–Fri 10:00–19:00, Sat 10:00–18:00 · never Sundays or federal holidays | 08:00–21:00 · no federal holidays | 08:00–21:00 |
| **India** | 09:00–21:00 · no public holidays | 09:00–19:00 · no public holidays | 09:00–21:00 |

New agents call **Monday–Saturday, 09:00–19:00** in the workspace's time zone. A campaign can
widen its own days and hours (`create_run({calling_days, calling_hours})`), but never past the
table above. Holidays: UK bank holidays (all UK nations), US federal holidays, India's central
government holidays.

Launching outside the hours is fine: people are held until calls are allowed, not called early.
When someone says "launch it now" at a time calls can't go out, tell them when they will start.

## Every call, every country

- **Domestic only.** A workspace calls numbers in its own country (UK, US or India), never
  abroad or premium-rate (`INTERNATIONAL_NOT_ALLOWED`).
- **Said on every call:** that it is an AI assistant, that the call is recorded; for sales, how
  to opt out; in the US, the business's callback number.
- **Right person first**; nothing at all to anyone else.
- **Opt-outs:** "don't call me again" on a call, or STOP to a text, puts the number on the
  workspace's do-not-call list for good, across all its agents.
- **How often:** at most 7 calls to one person in 7 days across the workspace; debt collection
  at most once a day.
- **People in a difficult situation:** the agent stops the task, takes no sale or promise, offers
  a callback from a person, and the call is marked `needs_care`.
- **Debt collection:** no threats or false consequences, agree what the person can manage,
  disputes noted and not argued; collecting for another company adds the debt-collector
  disclosure.
- **Calls come from the business's own number** in that country.
- **If the voice line fails**, the person hears a short "technical problem, we'll call back"
  message in their language rather than silence.

## What the business is responsible for

Breeze does **not** check national do-not-call registers. Before each launch the person confirms,
for the countries in the list:

1. a lawful basis to call everyone — in the US, prior express consent (written, for marketing)
   for AI-voice calls; in the UK, a lawful basis under UK GDPR and PECR (consent for automated
   marketing calls); in India, consent or an existing customer relationship;
2. the list was screened against the registers that apply — US National DNC and state lists, UK
   TPS and CTPS, India NCPR (DND);
3. everyone who asked not to be contacted was removed.

India also needs the right number series: service calls from a DLT-registered 160-series number
(1600 for banks, insurers and other regulated financial firms), sales from 140-series. Breeze
won't place a call from the wrong series.

## Launching — always two steps, always the person's yes

```
launch_run({campaign_id, run_id})
```
returns a **preflight** and launches nothing: how many people, sample numbers, when calls will
go out, the rules that apply, cost estimate, test-mode status, and an **attestation**.

1. Show the preflight in plain words, and the attestation text **word for word**.
2. Ask whether it is true. The attestation is **their** statement — never accept it for them,
   and never treat "build it and launch it" as a yes to it.
3. Only after an explicit yes:
```
launch_run({campaign_id, run_id, confirm: true, attestation: "<preflight.attestation.id>"})
```
If they hesitate, or say the list hasn't been screened, don't launch. Their acceptance is stored
with the campaign.

## When a launch is refused

| Code | Means | Say / do |
|---|---|---|
| `REVIEW_BLOCKED` | Breeze's go-live check found something to change | show the issues; fix with `update_agent`; launch again |
| `REVIEW_UNAVAILABLE` | the check couldn't run just now | try again in a minute |
| `SANDBOX_RESTRICTED` | test mode: only their verified phones and test numbers | to call customers, they ask their Breeze contact to take the workspace live |
| `CALLER_ID_REQUIRED` | no number of their own in that country | **breeze-phone-numbers** |
| `EMAIL_NOT_VERIFIED` | the account's email isn't verified | click the link Breeze emailed them |
| `INSUFFICIENT_BALANCE` | under one minute of credit | a person adds credit in Studio → Billing |
| `ATTESTATION_REQUIRED` / `CONFIRMATION_REQUIRED` | the second step was incomplete | go back to the preflight with the person |
| `INTERNATIONAL_NOT_ALLOWED` | a number outside the workspace's country | remove those numbers; Breeze calls domestically only |
| `PLAN_LIMIT_EXCEEDED` | over the plan's limits | report the numbers; don't retry |

Never drop people from the list to get a launch through without asking.

## Pause first, then look

If results look wrong mid-campaign — a strange outcome mix, a transcript with a broken greeting,
complaints — `pause_run` straight away, then investigate. A paused campaign costs nothing.
