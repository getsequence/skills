# Sequence Skills

Portable skills for using [Sequence](https://getsequence.io) with coding
agents, integrations, and real-world money workflows.

The repository currently includes one skill for a self-funding
[GoHighLevel](https://www.gohighlevel.com/) wallet. New Sequence product and
integration skills will use the same installation and documentation model.

## Install in one command

The easiest path works across Claude Code, Codex, Cursor, GitHub Copilot,
Cline, OpenCode, Gemini CLI, Windsurf, and the other agents supported by the
open Agent Skills ecosystem:

```bash
npx skills add getsequence/skills
```

Install only one skill when you know what you need:

```bash
npx skills add getsequence/skills --skill ghl-setup
```

Install globally for the agent you use every day:

```bash
npx skills add getsequence/skills \
  --global \
  --agent claude-code \
  --yes
```

The [`skills` CLI](https://github.com/vercel-labs/skills) can install the same
skill into multiple agents, choose project or global scope, update it later,
and list the agents available on your machine.

## Choose your coding agent

Install the full Sequence skill collection into a specific agent:

```bash
# Claude Code
npx skills add getsequence/skills -g -a claude-code -y

# Codex
npx skills add getsequence/skills -g -a codex -y

# Cursor
npx skills add getsequence/skills -g -a cursor -y

# GitHub Copilot
npx skills add getsequence/skills -g -a github-copilot -y

# Cline
npx skills add getsequence/skills -g -a cline -y

# OpenCode
npx skills add getsequence/skills -g -a opencode -y

# Gemini CLI
npx skills add getsequence/skills -g -a gemini-cli -y

# Windsurf
npx skills add getsequence/skills -g -a windsurf -y

# Kiro CLI
npx skills add getsequence/skills -g -a kiro-cli -y
```

Install only one skill into a specific agent by adding `--skill <skill-name>`:

```bash
npx skills add getsequence/skills \
  --skill ghl-setup \
  --global \
  --agent claude-code \
  --yes
```

To see every agent supported by the installer:

```bash
npx skills add getsequence/skills --list
```

To install into more than one agent in one command:

```bash
npx skills add getsequence/skills \
  --global \
  --agent claude-code \
  --agent codex \
  --agent cursor \
  --yes
```

Use `--copy` instead of symlinks when your environment does not support them.

## Start the skill

### Claude Code

After installing a skill, open Claude Code and invoke it by name. For the
current skill:

```text
/ghl-setup
```

If the Sequence MCP is not connected, the skill explains how to add it and
complete OAuth:

```bash
claude mcp add --transport http sequence https://app.getsequence.io/api/mcp
```

Restart Claude Code after adding a new MCP connection, then run `/mcp` and
complete the sign-in flow.

### Codex, Cursor, Copilot, and other agents

Ask the agent to use the workflow you need, or refer to a skill by name:

```text
Use the ghl-setup skill to set up my GoHighLevel wallet through Sequence.
```

Each skill's frontmatter tells compatible agents when to activate it. If an
agent does not auto-discover installed skills, include that skill's `SKILL.md`
in the agent's instructions or use the manual installation path below.

### Claude desktop or Cowork

When the product supports custom skill uploads, download the skill folder you
need and upload it as a skill. If uploads are unavailable, give the agent the
raw `SKILL.md` link and include that skill's linked reference files when the
workflow needs them.

## Manual installation

Use this fallback when the `skills` CLI is unavailable:

```bash
git clone https://github.com/getsequence/skills.git /tmp/sequence-skills
```

Copy one skill folder, or the complete collection, into the agent's skills
directory:

| Agent | Project directory | Global directory |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` |
| Cline | `.agents/skills/` | `~/.agents/skills/` |
| OpenCode | `.agents/skills/` | `~/.config/opencode/skills/` |
| Gemini CLI | `.agents/skills/` | `~/.gemini/skills/` |
| Windsurf | `.windsurf/skills/` | `~/.codeium/windsurf/skills/` |
| Kiro CLI | `.kiro/skills/` | `~/.kiro/skills/` |

Example for Claude Code with the current skill:

```bash
cp -R /tmp/sequence-skills/ghl-setup ~/.claude/skills/ghl-setup
```

To install the complete current collection manually:

```bash
cp -R /tmp/sequence-skills/* ~/.claude/skills/
```

The `skills` CLI maintains the current list of supported agents and their
paths. Use it when your agent is not listed above.

## Skill catalog

The catalog will grow as Sequence adds product and integration workflows.

| Skill | Category | Purpose | Integrations |
| --- | --- | --- | --- |
| [`ghl-setup`](./ghl-setup) | Workflow | Set up a self-funding GoHighLevel wallet with a Sequence buffer card, revenue routing, profit sweep, and weekly backstop | Sequence MCP, GoHighLevel |

Use the installer to inspect the live collection before choosing a skill:

```bash
npx skills add getsequence/skills --list
```

## Featured workflow: `ghl-setup`

The skill guides you through this loop:

```text
Client payment
      ↓
Money In in Sequence
      ├── refill the buffer card
      │       ↓
      │   GHL wallet auto-recharge
      └── sweep the remainder to checking as profit
```

It can:

- Find or confirm the Sequence beneficiary and checking account
- Create the Money In and Money Out accounts
- Create the automation rules in a paused state
- Run a simulation before anything goes live
- Provide direct links for dashboard-only steps
- Explain how to add the Sequence card to the GHL wallet

## Safety model

The skill never moves real money on its own. It creates accounts, creates
paused rules, and runs simulations. You activate the rules yourself in the
Sequence app.

The workflow keeps these steps human-controlled:

- Identity or business verification
- Bank connection and account selection
- Float-size confirmation
- Card issuance and card entry in GoHighLevel
- Rule activation

Read the full [security policy](./SECURITY.md) before adapting this workflow
for another integration.

## Requirements

- A Sequence account
- Access to the Sequence MCP over OAuth
- A verified business or individual beneficiary
- A connected checking account, unless you are testing with a Sequence pod
- Writer or Admin permission for beneficiary, bank, and card actions
- A GoHighLevel agency account with wallet auto-recharge

## Updating or removing the skill

Update all installed Sequence skills with:

```bash
npx skills update
```

Update one skill by name with:

```bash
npx skills update ghl-setup
```

Remove one skill from the global scope with:

```bash
npx skills remove ghl-setup --global
```

## Repository layout

```text
skills-repository/
├── <skill-name>/
│   ├── SKILL.md          # Agent workflow and safety rules
│   ├── README.md         # Optional human-facing overview
│   ├── reference/        # Optional detailed guidance
│   ├── examples/         # Optional representative tasks
│   └── tests/            # Optional workflow fixtures
└── README.md
```

The current repository keeps `ghl-setup` at the root for compatibility with
the first release. Future skills will follow the same self-contained layout;
the catalog and plugin manifests will arrive as the collection expands.

## Contributing

Read [`CONTRIBUTING.md`](./CONTRIBUTING.md) before opening a pull request. For
security issues, follow [`SECURITY.md`](./SECURITY.md) instead of opening a
public issue.

## License

[MIT](./LICENSE)
