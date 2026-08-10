# Deep links (the dashboard-only steps)

`{HOST}` = `https://app.getsequence.io` (production) or `https://staging.getsequence.io` (staging),
chosen from the connected MCP. These four steps cannot be done over the API — the corresponding
MCP tools (`addBusinessBeneficiary`, `connectExternalAccount`, `issueCard`) return guidance + these
URLs rather than performing the action.

| Step | Deep link | Notes |
|---|---|---|
| Add Business / KYB | `{HOST}/account/beneficiaries` | "Add Business" → KYB (legal name, EIN, owners). Takes a few minutes; entity is greyed/Pending until approved. Needs Writer/Admin. |
| Connect a bank | `{HOST}/account-list?action=connectAccount` | Opens the connect wizard on load. Business beneficiary must be verified first (Pending entities are greyed). Checking = depository via Plaid. |
| Issue the Money Out card | `{HOST}/map?node={MONEY_OUT_POD_ID}` | "Issue your debit card" → virtual → copy number/exp/CVV → paste into GHL wallet. Needs Writer/Admin + approved KYC. |
| Money In deposit details | `{HOST}/map?node={MONEY_IN_POD_ID}` | Read the pod's account + routing off the map to point revenue at it (not exposed via API). |
| Activate a rule | returned by `createRule` as `{HOST}/map/rules/{RULE_ID}?action=activateRule` | Rules are created paused; the user flips each on here. |

## HighLevel side (where the card goes)

HighLevel → **Agency Settings → Wallet → card on file** → paste the Money Out card → set the wallet
**auto-recharge** threshold + amount. GHL's native auto-recharge is what detects "low" and charges
the card; Sequence keeps the money behind that card topped up and sweeps profit first.
