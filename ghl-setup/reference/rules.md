# Rule definitions (verified against the Sequence rule engine)

Both rules are created **paused** (`createRule` always returns `status: DISABLED`) and return a
`...?action=activateRule` URL. The user activates them in the app.

Substitute the real IDs:
- `{MONEY_IN}` — the Money In pod id
- `{MONEY_OUT}` — the Money Out pod id
- `{PROFIT}` — the business's connected checking (`EXTERNAL_ACCOUNT`), or a Profit pod stand-in for a bank-less test
- `{FLOAT_CENTS}` — the Money Out float target in cents (default 50000 = $500)

## Shape notes (learned the hard way)

- All actions for a rule live in **one step's `actions` array**, ordered by `groupIndex` — not multiple steps.
- Every action must include the common fields: `type`, `source {id,type}`, `destination {id,type}`, `groupIndex`, `upToEnabled`, `limit` (null ok), `achDescription` (null ok), `isDirectDeposit` (false). On **create**, `source`/`destination` take `{id, type}` only (no `name`).
- `TOP_UP` also needs `amountInCents` (top the destination *up to* this balance) plus the nulls `nextPaymentMinimumAccount` / `currentBalanceAccount` / `lastStatementBalanceAccount`.
- `PERCENTAGE` needs `percentageValue` (0–100) and `percentageTarget`: `INCOMING_AMOUNT` (% of the triggering deposit) or `SOURCE_ACCOUNT` (% of the source balance *at that action's execution time*). For "sweep the remainder," use `SOURCE_ACCOUNT` 100% as the action *after* the top-up — verified: a $2,000 inflow → $500 top-up, $1,500 remainder to profit.
- **SCHEDULED** trigger requires `accountId`, and it must equal the action **source** account. `scheduleType: WEEKLY` needs a `startDate` (ISO 8601).

## Rule A — Refill + Sweep (event-driven)

```json
{
  "name": "GHL — Refill + Sweep",
  "trigger": { "type": "ON_FUNDS_TRANSFERRED", "accountId": "{MONEY_IN}" },
  "steps": [
    {
      "conditions": null,
      "actions": [
        {
          "type": "TOP_UP",
          "source": { "id": "{MONEY_IN}", "type": "POD" },
          "destination": { "id": "{MONEY_OUT}", "type": "POD" },
          "groupIndex": 0,
          "upToEnabled": true,
          "limit": null,
          "achDescription": null,
          "isDirectDeposit": false,
          "amountInCents": "{FLOAT_CENTS}",
          "nextPaymentMinimumAccount": null,
          "currentBalanceAccount": null,
          "lastStatementBalanceAccount": null
        },
        {
          "type": "PERCENTAGE",
          "source": { "id": "{MONEY_IN}", "type": "POD" },
          "destination": { "id": "{PROFIT}", "type": "EXTERNAL_ACCOUNT" },
          "groupIndex": 1,
          "upToEnabled": false,
          "limit": null,
          "achDescription": null,
          "isDirectDeposit": false,
          "percentageValue": 100,
          "percentageTarget": "SOURCE_ACCOUNT"
        }
      ]
    }
  ]
}
```

(If `{PROFIT}` is a stand-in pod rather than a bank, set its `destination.type` to `POD`.)

## Rule B — Weekly Float Top-up (backstop)

```json
{
  "name": "GHL — Weekly Float Top-up",
  "trigger": {
    "type": "SCHEDULED",
    "scheduleType": "WEEKLY",
    "startDate": "2026-08-17T14:00:00.000Z",
    "accountId": "{PROFIT}"
  },
  "steps": [
    {
      "conditions": null,
      "actions": [
        {
          "type": "TOP_UP",
          "source": { "id": "{PROFIT}", "type": "EXTERNAL_ACCOUNT" },
          "destination": { "id": "{MONEY_OUT}", "type": "POD" },
          "groupIndex": 0,
          "upToEnabled": true,
          "limit": null,
          "achDescription": null,
          "isDirectDeposit": false,
          "amountInCents": "{FLOAT_CENTS}",
          "nextPaymentMinimumAccount": null,
          "currentBalanceAccount": null,
          "lastStatementBalanceAccount": null
        }
      ]
    }
  ]
}
```

Set `startDate` to the next occurrence you want; `accountId` and the action `source` must both be `{PROFIT}` (the account funds are pulled from).

## Simulate before handing off

```
triggerRule(id = RuleA, simulation = true, simulatedIncomingFunds = 200000)
→ getRuleExecution(ruleId, executionId)         # expect status EXECUTED, transfersCompleted 2
→ listTransfers(accountIds=[MoneyIn,MoneyOut,Profit], ruleExecutionId, executionMode="SIMULATION")
```

Verified result for a $2,000 inflow with a $500 float: two INTERNAL transfers — 50000 to Money Out, 150000 to Profit. No real money moves (`executionMode: SIMULATION`).
