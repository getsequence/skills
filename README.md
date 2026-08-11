# Sequence Skills

Portable skills for using [Sequence](https://getsequence.io) with coding
agents, integrations, and real-world money workflows.

The first skill sets up a self-funding [GoHighLevel](https://www.gohighlevel.com/)
wallet through Sequence. More product and integration skills will live here as
they become ready.

## Install in one command

The easiest path works across Claude Code, Codex, Cursor, GitHub Copilot,
Cline, OpenCode, Gemini CLI, Windsurf, and the other agents supported by the
open Agent Skills ecosystem:

```bash
npx skills add getsequence/skills --skill ghl-setup
```

Install globally for the agent you use every day:

```bash
npx skills add getsequence/skills \
  --skill ghl-setup \
  --global \
  --agent claude-code \
  --yes
```

The [`skills` CLI](https://github.com/vercel-labs/skills) can install the same
skill into multiple agents, choose project or global scope, update it later,
and list the agents available on your machine.

## Choose your coding agent

Run the same command with the agent name that matches your tool:

```bash
# Claude Code
npx skills add getsequence/skills --skill ghl-setup -g -a claude-code -y

# Codex
npx skills add getsequence/skills --skill ghl-setup -g -a codex -y

# Cursor
npx skills add getsequence/skills --skill ghl-setup -g -a cursor -y

# GitHub Copilot
npx skills add getsequence/skills --skill ghl-setup -g -a github-copilot -y

# Cline
npx skills add getsequence/skills --skill ghl-setup -g -a cline -y

# OpenCode
npx skills add getsequence/skills --skill ghl-setup -g -a opencode -y

# Gemini CLI
npx skills add getsequence/skills --skill ghl-setup -g -a gemini-cli -y

# Windsurf
npx skills add getsequence/skills --skill ghl-setup -g -a windsurf -y

# Kiro CLI
npx skills add getsequence/skills --skill ghl-setup -g -a kiro-cli -y
```

To see every agent supported by the installer:

```bash
npx skills add getsequence/skills --list
```

To install into more than one agent in one command:

```bash
npx skills add getsequence/skills \
  --skill ghl-setup \
  --global \
  --agent claude-code \
  --agent codex \
  --agent cursor \
  --yes
```

Use `--copy` instead of symlinks when your environment does not support them.

## Start the skill

### Claude Code

After installation, open Claude Code and run:

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

Ask the agent to set up your GHL wallet with Sequence, or refer to the skill by
name:

```text
Use the ghl-setup skill to set up my GoHighLevel wallet through Sequence.
```

The skill's frontmatter tells compatible agents when to activate it. If an
agent does not auto-discover installed skills, include the repository's
`ghl-setup/SKILL.md` in the agent's instructions or use the manual installation
path below.

### Claude desktop or Cowork

When the product supports custom skill uploads, download the `ghl-setup`
folder and upload it as a skill. If uploads are unavailable, give the agent the
raw [`ghl-setup/SKILL.md`](./ghl-setup/SKILL.md) link and include the linked
reference files when the workflow needs them.

## Manual installation

Use this fallback when the `skills` CLI is unavailable:

```bash
git clone https://github.com/getsequence/skills.git /tmp/sequence-skills
```

Copy the `ghl-setup` folder into the agent's skills directory:

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

Example for Claude Code:

```bash
cp -R /tmp/sequence-skills/ghl-setup ~/.claude/skills/ghl-setup
```

The `skills` CLI maintains the current list of supported agents and their
paths. Use it when your agent is not listed above.

## Available skills

| Skill | Purpose | Integrations |
| --- | --- | --- |
| [`ghl-setup`](./ghl-setup) | Set up a self-funding GoHighLevel wallet with a Sequence buffer card, revenue routing, profit sweep, and weekly backstop | Sequence MCP, GoHighLevel |

## What `ghl-setup` does

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

Update an installed skill with:

```bash
npx skills update ghl-setup
```

Remove it from the global scope with:

```bash
npx skills remove ghl-setup --global
```

## Repository layout

```text
ghl-setup/
├── SKILL.md              # Agent workflow and safety rules
├── README.md             # Human-facing skill overview
└── reference/
    ├── deep-links.md     # Dashboard-only Sequence steps
    └── rules.md          # Rule definitions and simulation details
```

The repository will add a catalog, plugin manifests, and additional
integration skills in later releases. The current layout remains intentionally
small and easy to copy.

## Contributing

Read [`CONTRIBUTING.md`](./CONTRIBUTING.md) before opening a pull request. For
security issues, follow [`SECURITY.md`](./SECURITY.md) instead of opening a
public issue.

## License

[MIT](./LICENSE)
