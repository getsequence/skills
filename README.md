# Sequence Skills

Claude Code skills for automating real money movement with [Sequence](https://getsequence.io).

Each skill is a self-contained folder with a `SKILL.md` that teaches Claude Code how to wire up a specific money loop for you, end to end, from the terminal. Skills talk to Sequence over the **Sequence MCP using OAuth**, so there is no API key to paste and no backend to run. Anything that can only happen in the Sequence dashboard (verifying your identity, connecting a bank, issuing a card, activating a rule) is handed back to you as a direct link.

Nothing here ever moves real money on its own. Skills create accounts, build automation rules in a paused state, and run simulations. Real money only moves once you activate the rules yourself in the app.

## Available skills

| Skill | What it does |
|-------|--------------|
| [`ghl-setup`](./ghl-setup) | Sets up "GHL Wallet Autopilot" for a GoHighLevel agency: revenue lands in a Money In account, a rule keeps the card your GHL wallet pulls from topped up, and everything left sweeps to your checking as profit. |

## Install

You need [Claude Code](https://claude.com/claude-code) and a Sequence account.

### Option A: copy a skill into Claude Code

```bash
git clone https://github.com/getsequence/skills.git
cp -R skills/ghl-setup ~/.claude/skills/
```

Then open Claude Code and run the skill by name, for example:

```
/ghl-setup
```

To install a skill for a single project instead of globally, copy it into that project's `.claude/skills/` directory.

### Option B: point Claude Code at your own copy

Fork or clone this repo wherever you keep your tooling, then symlink the skills you want into `~/.claude/skills/`:

```bash
ln -s "$(pwd)/skills/ghl-setup" ~/.claude/skills/ghl-setup
```

## How a skill is structured

```
ghl-setup/
  SKILL.md          # the instructions Claude Code follows, with YAML frontmatter
  README.md         # a short human-readable overview of the skill
  reference/        # supporting docs Claude loads only when it needs them
```

The `SKILL.md` frontmatter (`name` and `description`) is what Claude Code uses to decide when the skill applies. The body is the full workflow.

## Contributing

Add a new skill as a top-level folder with its own `SKILL.md`, add a row to the table above, and open a pull request.

## License

[MIT](./LICENSE)
