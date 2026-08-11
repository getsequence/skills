# Security Policy

Sequence Skills contains instructions for workflows that may involve bank
accounts, cards, automation rules, and external products. Treat every change
that affects permissions, authentication, money movement, or approval gates as
a security-sensitive change.

## Supported versions

Only the latest release on the default branch receives security fixes. Pin a
release when you need reproducible behavior, and review the changelog before
upgrading.

## Report a vulnerability

Do not open a public GitHub issue for a vulnerability.

Report security issues privately through the contact method listed on the
[Sequence security page](https://getsequence.io/security), or contact the
Sequence team through the private security channel associated with this
repository.

Include:

- A short description of the issue
- The affected skill, file, and version or commit
- Reproduction steps or a minimal proof of concept
- The expected and observed behavior
- Any known impact or required permissions
- A safe contact address for follow-up

Please give the team a reasonable opportunity to investigate and release a fix
before disclosing the issue publicly.

## Credential and data rules

Never commit or paste into a skill:

- API keys, OAuth tokens, passwords, or private keys
- Card numbers, CVVs, bank account numbers, or routing numbers
- Personal identity, beneficiary, or business information
- Production MCP responses containing user data
- Local filesystem paths that expose private machine details

Use placeholders in examples and sanitized fixtures. OAuth and connector-based
authentication should remain the default when the product supports it.

## Financial safety rules

Sequence skills must keep the user in control of consequential actions.

- Never add a real transfer to a workflow that previously simulated one.
- Never activate a rule automatically unless the user explicitly requests and
  confirms that action in the current interaction.
- Preserve human confirmation for beneficiary, bank, float, card, and rule
  choices.
- Make dashboard-only steps visible instead of implying that MCP completed them.
- Keep simulation results clearly separate from live transaction results.
- Do not weaken a guardrail to make a test, example, or demo pass.

## Reviewing security-sensitive changes

Reviewers should check:

- Authentication and permission assumptions
- New tool calls and their side effects
- Human approval gates
- Simulation and paused-state behavior
- Logging and example data
- Failure recovery and partial setup behavior
- README, catalog, plugin, and compatibility claims

Security-sensitive changes should include a regression test or fixture that
proves the unsafe action does not occur.
