# GHL Wallet Autopilot (Sequence × GoHighLevel)

A Claude Code skill that sets up a self-funding money loop for a GoHighLevel agency using
[Sequence](https://getsequence.io): revenue lands, your GHL wallet card stays topped up, and your
profit is swept to your bank automatically — no manual top-ups, no wallet ever running dry.

It runs in your terminal and talks to **your own** Sequence account over the Sequence MCP (OAuth).
There is no API key to paste and nothing sent to anyone else's server. The steps Sequence only
allows in its dashboard (verifying your business, issuing the card, activating rules) are handed to
you as one-click deep links.

## What it sets up

- **Money In** pod — where your agency revenue lands
- **Money Out** pod — backs the virtual card you paste into your GHL wallet; kept topped up
- **Profit** — your connected checking, where everything left over sweeps automatically
- **Rule A** (on revenue): top the card float up, then sweep the remainder to profit
- **Rule B** (weekly): backstop the card float from checking

Rules are created **paused** and simulated for you before anything goes live. You activate them.

## Install

1. Copy this folder to your Claude Code skills directory:
   ```
   git clone https://github.com/<org>/ghl-setup ~/.claude/skills/ghl-setup
   ```
2. Add the Sequence MCP server to Claude Code and authenticate (one-time OAuth):
   ```
   claude mcp add --transport http sequence https://app.getsequence.io/api/mcp
   ```
   Then run `/mcp` in Claude Code and complete the Sequence sign-in. (A freshly added MCP may need a
   Claude Code reload before its tools appear.)
3. Run it:
   ```
   /ghl-setup
   ```

## Requirements

- A Sequence account with a **verified business** (the skill deep-links you to set this up if you
  haven't) and a connected business checking account.
- **Writer or Admin** permission on the Sequence account (needed to add a business, connect a bank,
  and issue a card).
- A GoHighLevel agency account with wallet auto-recharge.

## Safety

The skill never moves real money. It only creates accounts, creates **paused** rules, and runs
**simulations**. Real money moves only after you activate the rules yourself in the Sequence app.
