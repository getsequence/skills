# Contributing to Sequence Skills

Sequence Skills is a public repository of portable instructions for coding
agents. Contributions should make a real Sequence workflow easier to discover,
understand, verify, or use safely.

## Before you start

Read:

- [`AGENTS.md`](./AGENTS.md) for repository and agent guidance
- The complete `SKILL.md` you plan to change
- Every reference file linked by that skill

For a new integration or workflow, open an issue first when the change affects
financial behavior, permissions, authentication, or a public installation
path. Small documentation fixes can go directly into a pull request.

## Skill requirements

Every skill must:

- Live in its own directory
- Include a `SKILL.md`
- Use a unique lowercase hyphenated `name`
- Describe both what it does and when to use it
- State required integrations and capabilities
- Identify human approval gates
- Keep detailed reference material in linked files
- Avoid secrets, personal data, account identifiers, and machine-specific paths

Skills that interact with Sequence must preserve these boundaries:

- Simulate wherever the workflow supports simulation
- Keep newly created rules paused until the user activates them
- Never create or suggest an unapproved real-money transfer
- Distinguish MCP-automated actions from dashboard-only actions
- Tell the user when a step requires their approval or credentials

## Making a change

Keep each change focused. Prefer a sequence of small commits over one broad
rewrite. A good commit changes one understandable thing and leaves the
repository usable.

Before opening a pull request:

1. Check `git diff --check`.
2. Validate frontmatter and internal links when tooling is available.
3. Run the tests relevant to the changed skill.
4. Review examples against the current workflow.
5. Check the README, catalog, manifests, and indexes for affected public
   references.
6. Inspect the final diff for unrelated changes or sensitive data.

## Pull requests

The pull request description should explain:

- What changed
- Why it changed
- Which skills, integrations, or agent runtimes are affected
- What verification you ran
- Whether the README, catalog, plugin metadata, or compatibility matrix needs
  an update
- Any follow-up work that remains

Keep documentation examples copyable. Use placeholders such as `{HOST}` or
`{ACCOUNT_ID}` instead of realistic credentials or identifiers.

## Adding an integration

Document the integration as a complete boundary:

- What the external product does
- What Sequence does
- Which side owns authentication
- Which actions are automated
- Which actions require a dashboard or browser
- What permissions are required
- What can be simulated
- What happens when setup is interrupted

Add a regression case for every important failure mode you introduce or fix.

## Commit names

Use a short conventional prefix:

```text
docs: explain Cursor installation
feat: add Stripe integration skill
fix: clarify paused rule behavior
test: cover missing beneficiary setup
ci: validate skill metadata
```

## License

By contributing, you agree that your contribution is released under this
repository's [MIT License](./LICENSE).
