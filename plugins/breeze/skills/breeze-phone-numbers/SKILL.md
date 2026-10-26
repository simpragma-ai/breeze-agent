---
name: breeze-phone-numbers
description: Use when a Breeze customer needs a phone number — before their first launch to a country, when a launch is refused with CALLER_ID_REQUIRED, when they want people to call an agent (inbound), or when they ask which number their calls come from. Covers list_phone_numbers / search_phone_numbers / buy_phone_number / connect_phone_number. Buying costs money and always needs a human's yes.
---

# Phone numbers — buy your own, use it both ways

Every Breeze customer calls customers from **their own number**, in their own country (Breeze
calls domestically only, so one number is normally enough). In **test mode** no number is needed —
test calls and calls to the person's verified phones go out without one.

- **Outbound:** calls to a country go out from your number in that country (or the one set as
  the campaign's caller ID, if you own it). A run with contacts in a country where you have
  no number is **refused at launch** with `CALLER_ID_REQUIRED`, listing the missing countries.
- **Inbound:** the same number can ring one of your agents (campaigns) when people call it back.

Countries: **US**, **UK** (`GB`), **India** (`IN`).

## The flow

```
list_phone_numbers()                                  # what you already own — check first
search_phone_numbers({country: 'US', area_code: '415'})
buy_phone_number({phone_number: '+14155550123'})      # → quote, nothing bought yet
   … show the quote to the human, wait for an explicit yes …
buy_phone_number({phone_number: '+14155550123', confirm: true[, campaign_id]})
connect_phone_number({phone_number, campaign_id})     # inbound, if not done at purchase
```

## Rules

1. **Never set `confirm: true` on your own.** The first `buy_phone_number` call returns a quote.
   Show it — number, country, type, monthly rental — and buy only after a human says yes in this
   conversation. "Set things up so we can launch" is not a yes to spend money. One number per
   yes; don't buy spares.
2. **Check before you buy.** `list_phone_numbers()` first; a customer who already has a GB number
   doesn't need another to call the UK.
3. **Prefer a local area code** near the people you call — answer rates are higher. Search with
   `area_code` (US: `415`, `212`; UK: `20` London, `161` Manchester).
4. **Buying needs the `dial` scope**; searching and listing need `read`, connecting needs `write`.
   If a call is refused for scope, tell the human which permission the key needs; don't look for
   a way round it.
5. `monthly_cost` in search results is the carrier's figure. What the workspace pays is the
   number rent in `get_balance`, taken from its credit each month; never quote another price.

## Country notes

- **US:** any local number works. Carriers label unregistered business numbers as spam over time
  — keep call volume per number moderate and calls wanted (see `breeze-compliance`).
- **UK:** Ofcom rules apply to the number's use; some geographic numbers require a UK address on
  file. If the purchase is rejected for regulatory reasons, say so and point the customer to
  Breeze support — don't retry with a different number hoping it passes.
- **India:** commercial calls must come from a DLT-registered **140-series** (promotional) or
  **160-series** (service/transactional) number, obtained by the customer through TRAI/DLT
  registration — not something an agent can buy (1600-series for banks, insurers and other
  regulated financial firms). An ordinary Indian number found by search is not enough: Breeze
  does not place a call from the wrong series. Point the customer to their Breeze contact.

## When launch says CALLER_ID_REQUIRED

The error lists `missing_countries`. For each one: `list_phone_numbers()` (perhaps it was just
bought and is still activating), else search → quote → human yes → buy, then re-run the launch
preflight (`launch_run` without confirm). Never remove contacts to get the launch through without
asking.
