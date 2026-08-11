# Sequence Skills

Guidance for coding agents working in the public Sequence Skills repository.

## Repository purpose

This repository contains portable agent skills that teach coding agents how to
use [Sequence](https://getsequence.io) with external products and integrations.

Skills are instructions, references, examples, and validation assets. They are
not a replacement for the Sequence MCP, the Sequence application, or the
user's own approvals.

## Working principles

- Keep the public repository easy to understand for a first-time visitor.
- Prefer one small, coherent change per commit.
- Keep every commit in a usable state.
- Preserve the existing user-facing workflow unless the change explicitly
  updates that workflow.
- Write for both humans browsing GitHub and agents loading files selectively.
- Prefer plain language, concrete examples, and direct instructions.
- Keep procedural instructions in `SKILL.md` and move detailed reference
  material into `references/`.
- Avoid duplicating the same rule in multiple files. Link to the canonical
  source instead.

## Skill contract

Every skill must be a self-contained directory with:

```text
skill-name/
├── SKILL.md              # Required: frontmatter and agent instructions
├── README.md             # Optional: human-facing overview
├── references/           # Optional: detailed material loaded on demand
├── examples/             # Optional: representative tasks and outcomes
├── scripts/              # Optional: safe, repeatable helpers
└── tests/                # Optional: fixtures and workflow checks
```

Every `SKILL.md` must include YAML frontmatter with:

- `name`: lowercase, hyphenated, and unique within the repository
- `description`: what the skill does and when an agent should use it

When a skill has meaningful compatibility, permission, integration, or side
effect requirements, document them in frontmatter or the skill's canonical
metadata. Do not hide operational requirements in a distant reference file.

## Agent instructions

Before changing a skill:

1. Read the complete `SKILL.md`.
2. Read the references it links to.
3. Check whether the skill calls out human approval gates or safety limits.
4. Search the repository for duplicated names, links, examples, and claims.
5. Identify the smallest change that solves the request.

After changing a skill:

1. Check frontmatter syntax and required fields.
2. Check every internal link and referenced file.
3. Confirm that examples still match the workflow.
4. Confirm that safety boundaries remain explicit.
5. Report the exact verification performed.

## Sequence and financial safety

Sequence skills may describe workflows involving accounts, cards, rules, and
external money movement. Treat those workflows as high-risk.

- Never add instructions that move real money without an explicit human gate.
- Preserve simulation-first behavior wherever the underlying workflow supports
  simulation.
- Preserve paused-rule behavior until the user activates a rule themselves.
- Never introduce secret collection when OAuth or an existing connector is the
  intended authentication path.
- Keep account identifiers, card details, tokens, and personal data out of
  examples, tests, logs, and commits.
- Distinguish MCP-automated work from dashboard-only work.
- State what the agent cannot verify or perform.
- Do not weaken an approval gate to make a demo or test pass.

## Public repository quality

This is a public product repository. New public-facing content should be:

- Specific enough for a user to act on
- Short enough for an agent to load without unnecessary context
- Free of internal shorthand and unexplained identifiers
- Consistent with the Sequence product terminology
- Tested against the workflow it describes
- Safe to copy into a real project or agent configuration

Use descriptive names, stable links, and examples that do not depend on a
maintainer's local filesystem. Do not commit generated artifacts, credentials,
machine-specific paths, or temporary debugging output.

## Commit discipline

Keep commits narrow and name them after one user-visible or repository-level
change. Good examples:

```text
docs: define the skill contract
feat: add the canonical skill catalog
test: add GHL workflow fixtures
ci: validate skill metadata
```

Before committing, check:

- `git diff --check`
- The relevant validation or test command
- The final diff for unrelated changes
- The commit scope against the current plan

Do not combine documentation, repository moves, plugin packaging, and runtime
behavior changes in one commit unless the changes cannot work independently.

## Scope boundaries

An agent may make local repository changes requested by the user. An agent must
ask before:

- Publishing or releasing a package
- Modifying an external Sequence account
- Activating a financial rule
- Moving real money
- Adding a new external integration with production credentials
- Changing repository ownership, licensing, or public security policy

When a task is ambiguous, preserve existing behavior and describe the smallest
safe interpretation in the final handoff.
