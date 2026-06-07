# Installing BDD Superpowers for OpenCode

BDD Superpowers is a fork of Superpowers. Install this repository directly; the official marketplaces install upstream Superpowers instead.

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Add BDD Superpowers to the `plugin` array in your `opencode.json` (global or project-level):

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/superpowers.git#feature/bdd-control-harness"]
}
```

After the repository is renamed, use:

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

Restart OpenCode. The plugin auto-installs and registers all skills.

Verify with the smoke-test conversation:

```bash
opencode run 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'
```

Check that `writing-plans` explains when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

## Migrating from upstream Superpowers

Use a delete-then-install flow:

1. Remove any upstream `obra/superpowers` plugin entry.
2. Remove any old symlink-based OpenCode install.
3. Install BDD Superpowers from the git URL above.
4. Clear stale OpenCode package cache if the loaded skill text still looks like upstream Superpowers.

Do not enable upstream Superpowers and BDD Superpowers at the same time. They expose the same `superpowers` skill namespace.

If your `opencode.json` contains an upstream `obra/superpowers` entry, replace that entry with the BDD Superpowers entry above.

If you previously installed superpowers using `git clone` and symlinks, remove the old setup:

```bash
# Remove old symlinks
rm -f ~/.config/opencode/plugins/superpowers.js
rm -rf ~/.config/opencode/skills/superpowers

# Optionally remove the cloned repo
rm -rf ~/.config/opencode/superpowers

# Remove skills.paths from opencode.json if you added one for superpowers
```

Then follow the installation steps above.

If the loaded skill text still looks stale after reinstalling, clear the OpenCode package cache described in [docs/cache-refresh.md](../docs/cache-refresh.md).

## Usage

Use OpenCode's native `skill` tool:

```text
use skill tool to list skills
use skill tool to load superpowers/brainstorming
```

The skill namespace remains `superpowers` for compatibility.

## Updating

BDD Superpowers updates automatically when you restart OpenCode.

To pin the current branch explicitly:

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/superpowers.git#feature/bdd-control-harness"]
}
```

## Troubleshooting

### Plugin not loading

1. Check logs: `opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. Verify the plugin line in your `opencode.json`
3. Make sure you're running a recent version of OpenCode

### Installed upstream by mistake

1. Search `opencode.json` for `obra/superpowers`
2. Replace upstream entries with the BDD Superpowers git URL
3. Restart OpenCode so it refreshes the plugin

### Skills not found

1. Use `skill` tool to list what's discovered
2. Check that the plugin is loading (see above)

### Tool mapping

When skills reference Claude Code tools:
- `TodoWrite` -> `todowrite`
- `Task` with subagents -> `@mention` syntax
- `Skill` tool -> OpenCode's native `skill` tool
- File operations -> your native tools

## Getting Help

- This fork: https://github.com/zhexulong/superpowers/tree/feature/bdd-control-harness
- Planned renamed repository: https://github.com/zhexulong/bdd-superpowers
- Upstream Superpowers: https://github.com/obra/superpowers
