---
name: ghl-setup
description: >
  Set up the "GHL Wallet Autopilot" money loop for a GoHighLevel agency using
  Sequence. Runs in the terminal against the user's own Sequence account over
  the Sequence MCP (OAuth), creates the Money In / Money Out / Profit structure
  and the two automation rules, simulates the split, then hands off the
  dashboard-only steps (verify identity, issue the card, activate rules) as
  deep links. Final output: the card the user pastes into their HighLevel
  wallet. Use when the user says "/ghl-setup", "set up my GHL wallet",
  "connect Sequence to GoHighLevel", or "make my GHL wallet fund itself".
license: MIT
compatibility: >
  Requires access to the Sequence MCP over OAuth. Some setup steps require
  browser or dashboard access to Sequence and GoHighLevel. Tested with Claude
  Code; compatible agents must support the Agent Skills SKILL.md format.
metadata:
  author: Sequence
  version: "1.0.0"
  product: sequence
  category: financial-workflow
  integrations: sequence-mcp, gohighlevel
  risk: high
  side_effects: creates-accounts, creates-paused-rules, runs-simulations
  requires_human_approval: true
  real_money_allowed: false
---

# GHL Wallet Autopilot — setup skill

Wires a GoHighLevel agency's cash flow through Sequence so the GHL agency wallet never runs dry and profit is pulled automatically. Revenue lands in a **Money In** pod, a rule tops up a **Money Out** pod (the pod behind the card that funds the GHL wallet), and everything left sweeps to the agency's **checking (Profit)**. A weekly rule backstops the Money Out float from checking.

This skill talks to Sequence through the **Sequence MCP over OAuth** — there is no API key to paste and no backend. Some steps can only happen in the Sequence dashboard (identity/KYB, issuing a card, activating rules, connecting a bank); for those the skill hands the user an exact **deep link**. See `reference/deep-links.md` and `reference/rules.md`.

## What's automated vs a deep link (verified against the API)

- **Automated via MCP:** `listBeneficiaries`, `listAccounts`, `createAccount`, `createRule` (rules are always created paused), `triggerRule` (simulation), `getRuleExecution`, `listTransfers`.
- **Deep link only (the MCP tools for these return app URLs, not headless actions):** Add Business / KYB, connect a bank, issue a card, activate a rule.

## Host detection (do this first)

Pick the deep-link host from the connected MCP:
- Staging MCP (`mcp__sequence-staging__*`) → `https://staging.getsequence.io`
- Production MCP (`mcp__sequence__*`) → `https://app.getsequence.io`

Use that host everywhere below as `{HOST}`. Default to production if only one Sequence MCP is connected and it isn't the staging one.

## How to talk to the user (read this — it governs every screen)

This is a smooth, human, five-minute setup. The user is an agency owner, not an engineer. Narrate *outcomes and intent*, never mechanics.

**Never say things like** (these are the tool-call play-by-play — keep them silent):
- "Let me confirm the rule schema before I build the rules."
- "Both rules created (paused)." / "Creating the pods now." / "Calling listAccounts."
- Any dump of account/rule IDs or pod internals, e.g. "Money In → 08b6b743…". That's technical noise the user doesn't need.
- Cents, field names, `savingsTargetInCents`, `ON_FUNDS_TRANSFERRED`, execution IDs, etc.

**Instead:**
- Do the tool calls silently and speak only the result in plain language.
- Talk about **Money In**, **your card / GHL wallet**, **your checking / profit** — the money, not the plumbing.
- Keep each turn short: what just happened, what you need from them, or what you're about to do.

**Framing (important — this is what keeps it clear):** always describe the loop starting from the **real event**, not the pod name. Lead with *"every time a client pays you"* / *"when a payment comes in"*, then say where it lands. **Never** write the circular *"when money lands in Money In"* — the user doesn't yet know what Money In is. On first mention, gloss it: *"your **Money In** account — where every client payment lands."*

**Be precise about the card — this is the part people get wrong.** Sequence does **not** top up the GHL wallet directly. Your GHL wallet recharges itself by **pulling from a card on file**; that card is a **dedicated Sequence account (the buffer)**. Sequence's job is to keep **that buffer card loaded** to $[buffer] so the wallet's pull always clears and it never runs dry. So never say *"your GHL card gets topped up"* or *"Sequence tops up your wallet."* Say: *"the card your GHL wallet pulls from stays loaded."*

The canonical one-liner, reused everywhere (fill in the real buffer + checking name):

> Every time a client pays you, the money lands in your **Money In** account. Sequence keeps your **buffer card** loaded at $[buffer] — that's the card your GHL wallet pulls from to recharge, so the wallet never runs dry — and sends whatever's left to your **checking** as profit.

**The beats, in order:**
1. They finish verification → you check silently and say **"You're good to go."**
2. You look at their connected accounts and **confirm the checking account** with them by name ("Looks like your checking is Plaid Checking at Citibank — that's where your profit lands, and it's also the account Sequence pulls from once a week to reload the buffer card if it ever gets low. Right account?").
3. **Confirm the buffer** and explain what it is in their terms: *"Sequence gives you a card you'll put on file in HighLevel — that's the card your GHL wallet pulls from whenever it recharges. The buffer is how much I keep loaded on that card at all times, so the wallet can always pull and never runs dry. I'd set it at $X — good?"*
4. Tell them **what you're about to do** (top the card up to the buffer on every payment, sweep the rest to profit, and a weekly safety top-up) and ask if they want you to **run a quick simulation** so they can see the split before anything's live.
5. Show the simulation as a simple before/after split, confirm no real money moved, then move to go-live.

## The flow

### Screen 1 — Welcome + create account
If no Sequence MCP is connected yet: **first add the MCP for the user** (don't make them type the command) by running it via Bash —

```
claude mcp add -s user --transport http sequence https://app.getsequence.io/api/mcp
```

— then show this and stop until they've signed in:

> 👋 **GHL Wallet Autopilot**
> Five minutes from now your GHL wallet runs itself. Here's the end result:
>
> ```
>  ┌───────────────┐                       ┌──────────────┐
>  │ Client payout │                       │  GHL Wallet  │
>  │   from GHL    │                       └──────────────┘
>  └───────────────┘                              ▲
>          │                                       │ pulls to recharge
>          ▼            reload card         ┌──────────────┐
>  ┌──────────┐ ─────────────────────────▶ │ Buffer card  │
>  │ Money In │ ─┐                          └──────────────┘
>  │(revenue) │  │                          ┌──────────────┐
>  └──────────┘  │        profit            │   Checking   │
>                └─────────────────────────▶└──────────────┘
> ```
>
> Every time a client pays you, the money lands in Sequence. Sequence keeps a dedicated card loaded — that's the card your GHL wallet pulls from to recharge, so it never runs dry — and sweeps the rest to your bank as profit. Automatically.
>
> **Step 1: create your Sequence account.** I've added Sequence for you — now just:
> 1. **Quit and reopen Claude Code.** (A newly added connection only shows up after a restart — it won't appear in `/mcp` until you do this.)
> 2. Run **`/mcp`**, open **sequence**, and sign in (this creates/opens your account via OAuth).
> 3. Re-run **`/ghl-setup`** and I'll take it from here.

**Auto-detect the return — do not ask the user to type anything else here.** Poll by attempting `listBeneficiaries` every few seconds. The moment it succeeds, continue to Screen 2. If the Sequence tools never appear, the MCP was added but not loaded — tell the user to reload Claude Code (quit/reopen or `/mcp` reconnect), then re-run `/ghl-setup`.

### Screen 2 — Verify identity (manual `ready`)
Call `listBeneficiaries`. Find a **verified** beneficiary — **either a business or a personal (individual) entity** — that owns the accounts already connected during OAuth. `listBeneficiaries` only returns verified entities; one still in review shows as Pending / greyed in the app and won't be usable. Everything in the loop (pods, card, rules, connected bank) must sit under this **one** beneficiary, so pick the entity that owns the checking account they already connected.

If there is **no verified beneficiary**, show:

> ✅ You're connected.
>
> Here's the loop we're about to wire up: **every time a client pays you, the money lands in Sequence. Sequence keeps a dedicated card loaded — the card your GHL wallet pulls from to recharge, so it never runs dry — and sends whatever's left to your checking as profit.**
>
> **Step 2: verify your identity.** Add the entity that owns the accounts you're connecting here: {HOST}/account/beneficiaries → **"Add Business"** if the wallet and bank are under your agency's business, or **"Add Individual"** if they're in your personal name (KYB/KYC — legal name, ID/EIN, owners; takes a few minutes). Just make sure it's the same entity that owns the checking account you already connected. When you're approved, come back and type **`ready`**.

Wait for the user to type `ready`, then re-check `listBeneficiaries`. If still Pending, tell them it's not approved yet and to try again shortly. If a verified beneficiary already exists, skip straight ahead. If **both** a business and a personal entity are verified, ask which one owns the accounts before continuing.

### Screen 3 — Provision (automated)
Capture the verified `beneficiaryId` (the business or personal entity chosen in Screen 2). Ask the user for their **float target** (the buffer kept on the card), defaulting to **$500** if they don't care.

1. **Confirm checking + float with the user.** `listAccounts` (type `EXTERNAL_ACCOUNT`) and find the depository owned by that same beneficiary (this is the Profit sink + weekly top-up source). Then **explicitly confirm two things with the user before building anything**: (a) that the account you found is the right checking account, named, and (b) the float target. Do not just assume the connected account is correct.

   **Show the diagram while you ask**, with their real checking account name and buffer filled in (no flow amounts yet — this is the shape they're confirming), so it's clear what they're signing off on:

   ```
    ┌───────────────┐                       ┌──────────────┐
    │ Client payout │                       │  GHL Wallet  │
    │   from GHL    │                       └──────────────┘
    └───────────────┘                              ▲
            │                                       │ pulls to recharge
            ▼            reload card         ┌──────────────┐
    ┌──────────┐ ─────────────────────────▶ │ Buffer card  │
    │ Money In │ ─┐                          │   $1,000     │
    │(revenue) │  │                          └──────────────┘
    └──────────┘  │        profit            ┌──────────────┐
                  └─────────────────────────▶│Plaid Checking│
                                             └──────────────┘
   ```

   Caption it in one plain line so the boxes make sense: *"Every time a client pays you, it lands in your Money In account. Sequence reloads your buffer card back to $1,000 — that's the card your GHL wallet pulls from to recharge — and the rest becomes profit in Plaid Checking."*

   - If they say the account is **wrong** (e.g. they connected several, or want a different one), let them tell you which to use and pick that one.
   - If they **need to add** the right account, hand them the deep link to connect it: {HOST}/account-list?action=connectAccount — the wizard opens on load — and wait for them to come back.
   - If there's **no depository at all**, same deep link. If they want to skip connecting a bank for a first test, use a **Profit pod** instead (create one) and tell them plainly that no real bank is in the loop yet.
2. **Create pods** (idempotent — reuse if a pod with the same name already exists in `listAccounts`):
   - `createAccount` → **Money In** (POD, under the chosen `beneficiaryId`)
   - `createAccount` → **Money Out** (POD, under the same beneficiary, `savingsTargetInCents` = the float, e.g. 50000)
3. **Create both rules, paused.** Build them exactly as in `reference/rules.md`, substituting the real account IDs. Rule A is `ON_FUNDS_TRANSFERRED` on Money In (top up Money Out, then sweep 100% of the remainder to checking/Profit). Rule B is `SCHEDULED` weekly (top up Money Out from checking). `createRule` returns an **activate URL** (`...?action=activateRule`) — save both.
4. **Simulate — but ask first.** Before running it, tell the user in one line what the loop will do (top the card up to the buffer on each payment, sweep the rest to profit) and ask if they want a quick simulation to see it. On yes, `triggerRule` on Rule A with `simulation: true` and `simulatedIncomingFunds` set to a sample payment (e.g. 200000 for $2,000). Poll `getRuleExecution`, then `listTransfers` with `executionMode: SIMULATION` and the `ruleExecutionId` — silently. Then show only the plain split, led by the event, e.g. "Here's what happens the next time a client pays you $2,000: $1,000 reloads your buffer card (the one your GHL wallet pulls from), $1,000 lands in your checking as profit — and no real money moved just now." No IDs, no cents fields.

   Illustrate it with this diagram (substitute the real sample payment, buffer, and profit amounts):

   ```
    ┌───────────────┐                       ┌──────────────┐
    │ Client payout │                       │  GHL Wallet  │
    │   from GHL    │                       └──────────────┘
    │   ($2,000)    │                              ▲
    └───────────────┘                              │ pulls to recharge
            │                              ┌──────────────┐
            ▼           reload  $1,000     │ Buffer card  │
    ┌──────────┐ ─────────────────────────▶│    $1,000    │
    │ Money In │ ─┐                         └──────────────┘
    │(revenue) │  │                         ┌──────────────┐
    └──────────┘  │   profit  $1,000        │   Checking   │
                  └────────────────────────▶└──────────────┘
   ```

   Then **stop and hand off with a single line** — do not spill the go-live steps yet:

   > That's the whole loop, and nothing's moved for real. Ready to go live? Just let me know and I'll issue you the card to paste straight into your HighLevel wallet.

   Wait for their go-ahead before Screen 4.

### Screen 4 — Connect to GHL + go live (two gated stops)

This screen is **two stops**, not a checklist. Give the user only what the current gate needs, then wait.

**Gate 1 — route your revenue in, and get the card into HighLevel.** On their go-ahead, **show the diagram** (so it's clear which box each move wires up), then hand them these three concrete moves (all required — the routing is what makes the loop fire, the card is what funds the wallet), then stop until they say it's done:

> ```
>  ┌───────────────┐                       ┌──────────────┐
>  │ Client payout │        ③ paste card   │  GHL Wallet  │
>  │   from GHL    │                       └──────────────┘
>  └───────────────┘                              ▲
>          │ ① point revenue here                  │ pulls to recharge
>          ▼            reload card         ┌──────────────┐
>  ┌──────────┐ ─────────────────────────▶ │ Buffer card  │ ② issue card
>  │ Money In │ ─┐                          │   $1,000     │
>  │(revenue) │  │                          └──────────────┘
>  └──────────┘  │        profit            ┌──────────────┐
>                └─────────────────────────▶│Plaid Checking│
>                                           └──────────────┘
> ```
>
> **1. Point your revenue at Money In (Sequence).** Open your Money In pod: {HOST}/map?node={moneyInPodId} → copy its **account + routing number**, and set that as the payout/deposit destination for your GHL (or Stripe) client payments. Every payment has to land here or the automation has nothing to split.
>
> **2. Issue your card (Sequence).** Open your card pod: {HOST}/map?node={moneyOutPodId} → **Issue debit card → Virtual** → reveal and copy the **card number, expiry, and CVV**.
>
> **3. Paste it into HighLevel.** Agency view → **Settings → Company Billing (Wallet)** → **add a payment method** → paste the card number, expiry, CVV and billing zip, save. Turn on **Auto-recharge** and set the threshold + recharge amount — that's what pulls from your Sequence buffer card to keep the wallet full.
>
> Tell me once revenue's pointed at Money In and the card's pasted in.

Substitute the real `{moneyInPodId}` and `{moneyOutPodId}`. Both the routing and the pasted card are required deliverables — don't let the user skip the Money In routing.

**Gate 2 — turn the automation on.** Once they confirm the card is pasted, hand them the two activate links, how to flip them on, and **show the diagram** so it's clear this is the loop they're switching live:

> Great — you're good to go. Just click here to turn it on:
> • Refill + Sweep (runs on every client payment): {ruleA activateUrl}
> • Weekly safety top-up (the backstop): {ruleB activateUrl}
>
> Each link opens the rule in Sequence — hit **Activate** (the toggle at the top) on both, and you're live. This is what's now running on its own:
>
> ```
>  ┌───────────────┐                       ┌──────────────┐
>  │ Client payout │                       │  GHL Wallet  │
>  │   from GHL    │                       └──────────────┘
>  └───────────────┘                              ▲
>          │                                       │ pulls to recharge
>          ▼            reload card         ┌──────────────┐
>  ┌──────────┐ ─────────────────────────▶ │ Buffer card  │
>  │ Money In │ ─┐                          │   $1,000     │
>  │(revenue) │  │                          └──────────────┘
>  └──────────┘  │        profit            ┌──────────────┐
>                └─────────────────────────▶│Plaid Checking│
>                                           └──────────────┘
> ```

Then close with the canonical one-liner, led by the event, e.g.: *"That's it — you're live. From now on, every time a client pays you, the money lands in your Money In account. Sequence reloads your buffer card back to $1,000 — the card your GHL wallet pulls from to recharge, so it never runs dry — and sweeps the rest to Plaid Checking as profit. Once a week Sequence also checks that buffer card and tops it up from checking if it's low, as a backstop."* No IDs.

## Guardrails

- **Never move real money.** This skill only creates accounts, creates paused rules, and runs simulations. Never call `triggerRule` without `simulation: true`, and never `createTransfer`. Real money only moves once the user activates the rules themselves.
- **Idempotent.** Before creating a pod or rule, list first and reuse anything already named for this loop, so re-running the skill doesn't duplicate.
- **Float sizing.** Because Rule A refills on each payment event, size the float above the worst-case wallet spend between two client payments. Rule B (weekly) is the backstop. Surface this; don't silently add extra scheduled rules.
- **Permissions.** Add Business, connecting a bank, and issuing a card all need Writer/Admin on the Sequence account. If a deep-linked step is missing in the app, that's usually the cause — tell the user.

## Examples

- [First-time setup](./examples/first-time-setup.md)
- [Resume an existing setup](./examples/resume-existing-setup.md)
- [Simulation-only test](./examples/simulation-only.md)
