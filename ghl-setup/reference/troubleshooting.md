# Troubleshooting GHL Wallet Autopilot

Use this reference when setup cannot continue. Keep the user informed about
the next required action, and do not repeat a write until the existing state
has been checked.

## Sequence MCP is missing

Symptoms:

- Sequence tools do not appear
- `/mcp` does not list the Sequence connection
- Tool calls fail before authentication

Recovery:

1. Add the Sequence MCP using the command in the main workflow.
2. Quit and reopen the coding agent.
3. Run `/mcp` and complete OAuth.
4. Re-run `/ghl-setup`.

Do not ask the user to paste an API key when OAuth is available.

## No verified beneficiary

Symptoms:

- `listBeneficiaries` returns no usable entity
- The business or individual entity appears pending in the app

Recovery:

- Provide the beneficiary verification deep link.
- Explain that the entity must own the accounts used in the workflow.
- Wait for the user to confirm approval before checking again.

Do not create accounts under a pending or ambiguous beneficiary.

## Multiple verified beneficiaries

Recovery:

- Show the names in plain language.
- Ask which entity owns the checking account and payment flow.
- Use only the beneficiary the user selects.

Do not choose by position, creation date, or an internal identifier alone.

## Checking account is missing or wrong

Recovery:

- Show the connected checking account name and institution.
- Ask the user to confirm it.
- If it is wrong or missing, provide the connect-account deep link.
- Re-check after the user returns.

If the user wants a bank-free test, use a Profit pod only when the workflow
supports it and say clearly that no external bank is connected.

## Duplicate accounts or rules

Before creating anything:

- List existing accounts and rules.
- Match by purpose, beneficiary, and workflow name.
- Reuse an exact match.
- Ask before treating a similar name as the same resource.

Never solve an uncertain match by creating another account or rule.

## Simulation does not complete

Recovery:

1. Check the rule execution status.
2. Poll only within a bounded interval.
3. Report a timeout plainly if the execution does not finish.
4. Preserve the rule's paused state.
5. Offer a retry only after checking whether the first simulation completed.

Never switch from simulation to a live execution as a fallback.

## A dashboard step is unavailable

Common causes include missing Writer/Admin permission, an unverified
beneficiary, or using the wrong environment host.

Check:

- Production versus staging MCP
- Beneficiary approval state
- Account role
- Correct deep-link host

Tell the user which condition needs attention. Do not imply that the MCP
completed a dashboard-only action.
