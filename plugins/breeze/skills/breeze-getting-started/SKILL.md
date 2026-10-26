---
name: breeze-getting-started
description: Use when someone is new to Breeze or asks "what can you do with Breeze", "help me get started", "make my first call", or anything in the first-run path — checking the connection and credit, verifying their own phone, building a first agent, hearing it on a test call, running a first small campaign and reading the result. Points to the other breeze-* skills for each step in depth.
---

# First run: from nothing to a first real call

The person you are helping is a business user, usually not technical. Speak plainly, ask as
little as possible, and do the steps for them with the Breeze tools. Breeze works out what it
can by itself (the call type, sensible calling days and hours, made-up test details) — do not
ask them for those.

## 0. Check the connection and the workspace

```
get_balance()        # credit, currency, price per minute
list_my_phones()     # their verified phones, test calls left today
list_agents()        # anything already built
```

If the tools are missing or refused, the connection isn't authorised: tell them to run
`/mcp` → the Breeze server → **Authenticate** (in Claude Code) and sign in. A brand-new person
can create their Breeze account in that same browser window. If a tool says it "requires the
dial scope", they did not tick **Launch calls** when connecting — they need to connect again
and tick it.

A new workspace starts with free welcome credit and in **test mode**: it can call only the
person's own verified phones and fictional test numbers. That is what a first try needs.

## 1. Verify their phone (once)

Ask for their mobile number with the country code. Only a number they say is theirs.

```
verify_my_phone({phone: "+447…"})          # Breeze calls it now and reads a 4-digit code
   … tell them to answer; ask them for the code they heard …
confirm_my_phone({phone_id, code: "1234"})  # never guess the code
```

Own country only, three verification calls a day. A number already verified
(`list_my_phones`) needs no new code.

## 2. Build the agent from their description

Ask what the call is for in one or two sentences if they haven't said. Then build it with
`create_agent` — see **breeze-agent-design** for role, goal, flow, facts and outcomes. Use
`create_agent_from_template` only when `list_templates` has one that really fits.

Afterwards tell them, briefly: what the agent will say, what details it needs for each person,
and the call type Breeze worked out (`get_agent`). They don't choose the call type.

## 3. Let them hear it

```
test_call_me({campaign_id, phone_id})   # made-up details fill any fact you don't give
```

Say "your phone is about to ring". Encourage them to behave like a real customer —
interrupt, ask questions, say no. Then read the call back (`list_calls` → `get_call`) and fix
what they didn't like with `update_agent` or `set_agent_facts`, and call again.
See **breeze-testing**.

## 4. A first small campaign

```
create_run({campaign_id, run_name, facts?, calling_days?, calling_hours?})
add_contacts({campaign_id, run_id, contacts: [{phone, variables}]})
launch_run({campaign_id, run_id})          # preflight only — launches nothing
   … show the summary and the attestation word for word; wait for their yes …
launch_run({campaign_id, run_id, confirm: true, attestation: "<preflight.attestation.id>"})
```

In test mode the list can contain only their verified phone(s) and test numbers. New agents
call Monday–Saturday 09:00–19:00 in the workspace's time zone; if they want to try it now on a
Sunday, pass `calling_days` for this run (the law and public holidays still apply).
See **breeze-campaigns** and **breeze-compliance**.

## 5. Results and spend

```
get_run_stats({campaign_id, run_id})       # outcomes
export_run_results({campaign_id, run_id})  # one row per person
get_billing_summary()                      # balance, runway, spend per agent
```

Credit is added by a person in Breeze Studio → **Billing**, never by you.

## When they want to call real customers

Test mode is lifted by Simpragma — tell them to ask their Breeze contact. They will also need
their own phone number in their country (**breeze-phone-numbers**), and their email verified
(the link Breeze emailed them) before any launch.
