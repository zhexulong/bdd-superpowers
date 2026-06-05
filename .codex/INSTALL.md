# Installing BDD Superpowers for Codex

Enable BDD Superpowers skills in Codex via native skill discovery. Clone this fork and symlink its `skills/` directory.

## Prerequisites

- Git
- OpenAI Codex CLI with native skill discovery

## Installation

1. **Clone the BDD Superpowers repository:**
   ```bash
   git clone --branch feature/bdd-control-harness https://github.com/zhexulong/superpowers.git ~/.codex/bdd-superpowers
   ```

   After the repository is renamed, use:
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

If `~/.agents/skills/superpowers` already points to upstream Superpowers, replace it so Codex discovers BDD Superpowers instead.

```bash
rm ~/.agents/skills/superpowers
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

If you installed the old bootstrap block, remove it from `~/.codex/AGENTS.md`; any block referencing `superpowers-codex bootstrap` is no longer needed.

## Verify

```bash
ls -la ~/.agents/skills/superpowers
```

You should see a symlink or junction pointing to `~/.codex/bdd-superpowers/skills`.

Then restart Codex and ask for brainstorming guidance. The loaded brainstorming skill should mention `Behavior Evaluation` or `Behavior Coverage`.

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
