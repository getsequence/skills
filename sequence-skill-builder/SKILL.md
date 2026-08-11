---
name: sequence-skill-builder
description: >
  Add or update an agent skill in a public skills repository. Use when creating
  a new skill, adding an integration workflow, reviewing a skill repository
  before editing it, updating README or catalog metadata, validating an agent
  skill, preparing a focused commit, or checking upstream changes before a
  repository update.
license: MIT
compatibility: >
  Requires a cloned Git repository, Git, and Node.js with npx available for
  Agent Skills discovery checks. The repository may use a different catalog,
  plugin, or documentation layout; inspect it before applying this workflow.
metadata:
  author: Sequence
  version: "1.0.0"
  category: repository-maintenance
  integrations: git, agent-skills
  risk: medium
  side_effects: edits-repository-files, creates-commits, may-push-branches
  requires_human_approval: true
---

# Add a skill to a public skills repository

Use this workflow to create a skill that agents can discover, humans can
understand, and maintainers can update without chasing stale references.

The repository is the product surface. A new `SKILL.md` is only complete when
its documentation, metadata, examples, validation, and release path agree with
it.

## 1. Inspect before editing

Start read-only. Do not assume the repository still matches an earlier plan.

```bash
pwd
git status --short --branch
git remote -v
git branch --show-current
git fetch --all --prune
git log --oneline --decorate -12
rg --files | sort
```

Identify the default branch and the repository's actual source of truth. Look
for these surfaces, using the paths that exist:

- Root `README.md`
- Root `AGENTS.md` or equivalent agent instructions
- `CONTRIBUTING.md` and `SECURITY.md`
- Skill directories containing `SKILL.md`
- Skill and integration catalogs
- Plugin or marketplace manifests
- Agent-specific metadata such as `agents/openai.yaml`
- Scripts, tests, fixtures, CI, and release workflows

### Review upstream changes

If the repository tracks an upstream product repository, inspect it before
editing:

```bash
git fetch origin main
git log --oneline HEAD..origin/main
git diff --stat HEAD..origin/main
git diff HEAD..origin/main -- README.md AGENTS.md CONTRIBUTING.md SECURITY.md
```

Use the actual default branch if it is not `main`. Read upstream changes that
affect skill layout, naming, installation, catalogs, manifests, validation, or
safety policy. Reconcile the new skill with the upstream shape instead of
building against stale local assumptions.

If upstream has changes that the user has not asked you to merge, stop before
overwriting or force-pushing. Report the divergence and ask which base to use.

## 2. Define the new skill

Write down the following before creating files:

- User outcome
- Trigger phrases and explicit non-triggers
- Required tools, permissions, and external products
- Integrations and side effects
- Human approval gates
- Simulation or dry-run behavior
- Failure and retry behavior
- Expected final response

Search existing skills and catalogs before choosing a name:

```bash
rg -n "name:|description:|<product-or-workflow-keyword>" .
```

Add a new skill only when its responsibility is distinct. If an existing skill
already owns the workflow, extend it or sharpen its routing boundary. Do not
create a second skill that repeats the same product setup with different prose.

Use a short lowercase hyphenated name, such as `sequence-stripe-payouts` or
`sequence-rule-simulation`.

## 3. Create the skill package

Use the repository's established layout. If no layout exists, create only what
the skill needs:

```text
<skill-name>/
├── SKILL.md
├── agents/openai.yaml       # When the repository uses agent UI metadata
├── references/              # Detailed material loaded only when needed
├── examples/                # Realistic prompts and expected behavior
└── tests/                   # Fixtures or deterministic checks
```

Keep `SKILL.md` procedural and under 500 lines. Its frontmatter must include a
clear `name` and a description that states both what the skill does and when
an agent should use it. Add compatibility, risk, integration, side-effect,
and approval metadata when the workflow needs them.

The body should usually contain:

1. Outcome and scope
2. When to use and when not to use
3. Requirements and capability detection
4. Ordered workflow or decision tree
5. Validation checkpoints
6. Failure recovery
7. Side effects and approval gates
8. References and examples

Move detailed schemas, API facts, variants, and long troubleshooting material
into one-level-deep reference files linked from `SKILL.md`. Do not add a second
README inside the skill unless the repository requires one for every skill.

## 4. Model integrations honestly

For each integration, document:

- Authentication owner
- Available MCP, CLI, API, or browser capabilities
- Dashboard-only actions
- Required permissions
- Side effects
- Simulation or dry-run support
- Failure and retry behavior
- Human approval points

Separate explanation skills from execution workflows only when the distinction
is real. An integration skill should not duplicate an end-to-end setup skill.

For financial or destructive workflows:

- Preserve simulation-first behavior.
- Keep rules paused until the user activates them.
- Never use a live action as a fallback for a failed simulation.
- Keep credentials, account numbers, card details, and personal data out of
  source files, examples, fixtures, and logs.
- Never claim that a dashboard-only action completed through an MCP.

## 5. Update every repository surface

After the skill exists, inspect and update the surfaces this repository uses:

- The skill's `SKILL.md`
- Root README skill catalog and installation guidance
- Machine-readable skill catalog or registry
- Agent UI metadata such as `agents/openai.yaml`
- Integration catalog when an integration is new or changed
- Examples, references, tests, and evaluation cases
- Plugin or marketplace manifests
- Changelog or version metadata
- Agent, contribution, or security policy when the public contract changes

For the current Sequence repository, the important surfaces are:

```text
README.md
catalog/skills.json
catalog/integrations.json
<skill-name>/SKILL.md
<skill-name>/agents/openai.yaml
```

Confirm these paths after inspection because the upstream repository may evolve.

Update README content when any of these facts change:

- Skill name or path
- Installation command
- Supported agent
- Skill category or purpose
- Integration requirement
- Manual installation path
- Update or removal command

Keep README prose collection-level. Put detailed workflow behavior in the
skill. Keep catalog entries machine-readable and synchronized with real paths.

## 6. Add examples and regression coverage

Add at least three realistic cases for a non-trivial workflow:

1. Normal successful request
2. Incomplete, ambiguous, or interrupted request
3. Request that must pause for approval or refuse an unsafe action

For financial workflows, add negative cases proving that the skill does not:

- Move real money
- Activate a rule automatically
- Skip account or beneficiary confirmation
- Expose credentials or financial identifiers
- Treat a simulation as a live result

Grade outcomes and safety behavior, not one rigid internal implementation path.

## 7. Validate before committing

Run the checks available in the repository. For the current Sequence layout:

```bash
git diff --check
npx skills add . --list
jq empty catalog/skills.json
jq empty catalog/integrations.json
```

Also verify:

- Every `SKILL.md` has valid frontmatter with `name` and `description`.
- Every catalog path exists.
- Every README and skill link resolves.
- New names are unique.
- New integrations appear in the integration registry when required.
- Agent metadata matches the skill's name and purpose.
- No secrets or private data appear in the diff.

If the repository provides a validator, test runner, formatter, or CI command,
run it instead of inventing a parallel check. If a validator is unavailable,
report the missing dependency and run the strongest equivalent checks.

## 8. Review the diff as a public release

```bash
git diff --cached --stat
git diff --cached --check
git diff --cached
```

Ask:

- Can a new user discover the skill from the README?
- Can an agent route to it from the description?
- Can a maintainer identify the canonical source?
- Does the catalog describe reality?
- Does the skill state its limits and approval gates?
- Did any stale name, path, command, or integration survive?
- Did upstream review reveal a newer repository convention?

Keep the commit narrow. Good messages include:

```text
feat: add Stripe payout workflow skill
docs: document skill creation workflow
test: cover paused-rule safety
fix: correct skill metadata
```

## 9. Commit and push safely

Do not commit or push merely because the work is ready. Commit and push only
when the user has asked for publication.

Before committing:

1. Confirm the intended branch and remote.
2. Confirm the diff contains only the requested skill change.
3. Confirm all repository surfaces are synchronized.
4. Run validation and record the results.

After pushing, report the commit hash, remote and branch, changed public
surfaces, validation results, and any known limitation.

Never force-push over upstream changes without explicit approval. If the user
requests a history rewrite, verify the target commit and branch first, explain
the scope, and use the narrowest safe command.
