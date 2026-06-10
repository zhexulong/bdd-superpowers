# Installing BDD Superpowers for Codex

Enable BDD Superpowers skills in Codex via native skill discovery. Clone this fork and symlink its `skills/` directory.

## Prerequisites

- Git
- OpenAI Codex CLI with native skill discovery

## Installation

1. **Clone the BDD Superpowers repository:**
   ```bash
   git clone https://github.com/zhexulong/bdd-superpowers.git ~/.codex/bdd-superpowers
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
   ```

   The symlink name remains `superpowers` for compatibility with existing skill references.

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\superpowers" "$env:USERPROFILE\.codex\bdd-superpowers\skills"
   ```

3. **Restart Codex** to discover the skills.

## Migrating from upstream Superpowers

Use a delete-then-install flow:

1. Remove the old `~/.agents/skills/superpowers` link or junction.
2. Install BDD Superpowers from the git clone above.
3. Refresh any stale plugin cache if the loaded skill text still looks like upstream Superpowers.

```bash
rm ~/.agents/skills/superpowers
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

If you installed the old bootstrap block, remove it from `~/.codex/AGENTS.md`; any block referencing `superpowers-codex bootstrap` is no longer needed.

If the loaded skill text still looks stale after reinstalling, clear the cache described in [docs/cache-refresh.md](../docs/cache-refresh.md).

## Verify

```bash
ls -la ~/.agents/skills/superpowers
```

You should see a symlink or junction pointing to `~/.codex/bdd-superpowers/skills`.

Then run the smoke-test conversation:

```bash
codex exec 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'
```

The loaded `writing-plans` skill should explain when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

## Updating

```bash
cd ~/.codex/bdd-superpowers && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/superpowers
```

Optionally delete the clone: `rm -rf ~/.codex/bdd-superpowers`.
