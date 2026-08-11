---
name: sequence
description: >
  Help a user choose and use the right Sequence skill for financial workflows,
  product integrations, accounts, cards, rules, simulations, and dashboard
  handoffs. Use when the user asks how to use Sequence, wants to connect a
  product to Sequence, or describes a Sequence outcome without naming a
  specific workflow.
license: MIT
compatibility: >
  Works with agents that support the Agent Skills SKILL.md format. Individual
  workflows may require the Sequence MCP, OAuth, browser access, or an
  external product account.
metadata:
  author: Sequence
  version: "1.0.0"
  product: sequence
  category: router
  integrations: sequence-mcp
  risk: high
  requires_human_approval: true
---

# Sequence

Use this skill as the entry point for Sequence requests. Route the user to the
most specific available workflow, then load only that workflow's instructions
and references.

## Route by intent

- GHL or GoHighLevel wallet setup → use [`ghl-setup`](../ghl-setup/SKILL.md).
- A named skill → use that skill directly after checking its requirements.
- A new product integration → explain what is currently supported and ask for
  the product outcome before proposing a workflow.
- A question about what Sequence can automate → describe the available skill
  and integration metadata, then distinguish MCP actions from dashboard-only
  actions.
- An ambiguous financial request → ask which account, product, or outcome the
  user means before choosing a workflow.

## Common requirements

Before a workflow begins, check its `SKILL.md` for:

- Required MCP, CLI, browser, or connector access
- Authentication and permission requirements
- External product accounts
- Simulation or dry-run support
- Human approval gates
- Actions the workflow explicitly forbids

Do not invent a workflow that the repository does not contain. Tell the user
when a requested integration is not available yet.

## Safety boundary

Sequence workflows may involve financial accounts, cards, and automation
rules. Keep consequential actions under explicit human control:

- Preserve simulation-first behavior.
- Preserve paused-rule behavior.
- Never create a real transfer as a shortcut or fallback.
- Never present a dashboard-only action as completed by the MCP.
- Confirm ambiguous accounts, beneficiaries, amounts, and activation choices.

## Final response

After routing, the selected workflow owns the interaction. Do not repeat its
full instructions or expose internal identifiers that the selected workflow
keeps hidden from the user.
