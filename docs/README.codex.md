# BDD Superpowers for Codex

Guide for using BDD Superpowers with OpenAI Codex via native skill discovery.

BDD Superpowers is a fork of Superpowers. It keeps the original skill workflow and adds Behavior Evaluation in specs plus Behavior Coverage in implementation plans.

## Quick Install

Tell Codex:

```text
Fetch and follow instructions from https://raw.githubusercontent.com/zhexulong/superpowers/refs/heads/feature/bdd-control-harness/.codex/INSTALL.md
```

After the repository is renamed:

```text
Fetch and follow instructions from https://raw.githubusercontent.com/zhexulong/bdd-superpowers/refs/heads/main/.codex/INSTALL.md
```

## Manual Installation

### Prerequisites

- OpenAI Codex CLI
- Git

### Steps

1. Clone the repo:
   ```bash
   git clone --branch feature/bdd-control-harness https://github.com/zhexulong/superpowers.git ~/.codex/bdd-superpowers
   ```

   After the repository is renamed:
   ```bash
   git clone https://github.com/zhexulong/bdd-superpowers.git ~/.codex/bdd-superpowers
   ```

2. Create the skills symlink:
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
   ```

   The symlink name remains `superpowers` for compatibility.

3. Restart Codex.

Verify with the smoke-test conversation:

```bash
codex exec 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'
```

Check that the loaded `writing-plans` skill explains when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

4. **For subagent skills** (optional): Skills like `dispatching-parallel-agents` and `subagent-driven-development` require Codex's multi-agent feature. Add to your Codex config:
   ```toml
   [features]
   multi_agent = true
   ```

### Windows

Use a junction instead of a symlink:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\bdd-superpowers\skills"
```

## Migrating from upstream Superpowers

Use a delete-then-install flow:

1. Remove the old `~/.agents/skills/superpowers` link or junction.
2. Install BDD Superpowers from the git URL above.
3. Refresh stale cache if the loaded skill text still looks like upstream Superpowers.

If `~/.agents/skills/superpowers` already points to upstream Superpowers, replace it:

```bash
rm ~/.agents/skills/superpowers
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

Do not expose both upstream Superpowers and BDD Superpowers under skill discovery at the same time.

If the loaded skill text still looks stale after reinstalling, clear the cache described in [Refreshing Stale Superpowers Caches](cache-refresh.md).

## How It Works

Codex has native skill discovery. It scans `~/.agents/skills/` at startup, parses `SKILL.md` frontmatter, and loads skills on demand. BDD Superpowers skills are made visible through a single symlink:

```text
~/.agents/skills/superpowers/ -> ~/.codex/bdd-superpowers/skills/
```

The `using-superpowers` skill is discovered automatically and enforces skill usage discipline. No additional bootstrap is needed.

## Usage

Skills are discovered automatically. Codex activates them when:

- You mention a skill by name, such as "use brainstorming"
- The task matches a skill's description
- The `using-superpowers` skill directs Codex to use one

### Personal Skills

Create your own skills in `~/.agents/skills/`:

```bash
mkdir -p ~/.agents/skills/my-skill
```

Create `~/.agents/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

The `description` field is how Codex decides when to activate a skill automatically. Write it as a clear trigger condition.

## Updating

```bash
cd ~/.codex/bdd-superpowers && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/superpowers
```

**Windows (PowerShell):**
```powershell
Remove-Item "$env:USERPROFILE\.agents\skills\superpowers"
```

Optionally delete the clone: `rm -rf ~/.codex/bdd-superpowers`.

## Troubleshooting

### Skills not showing up

1. Verify the symlink: `ls -la ~/.agents/skills/superpowers`
2. Check skills exist: `ls ~/.codex/bdd-superpowers/skills`
3. Restart Codex; skills are discovered at startup

### Installed upstream by mistake

1. Check whether `~/.agents/skills/superpowers` points to `obra/superpowers`
2. Replace the symlink with the BDD Superpowers path above
3. Restart Codex

### Windows junction issues

Junctions normally work without special permissions. If creation fails, try running PowerShell as administrator.

## Getting Help

- This fork: https://github.com/zhexulong/superpowers/tree/feature/bdd-control-harness
- Planned renamed repository: https://github.com/zhexulong/bdd-superpowers
- Upstream Superpowers: https://github.com/obra/superpowers
