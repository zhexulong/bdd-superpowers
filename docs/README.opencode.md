# BDD Superpowers for OpenCode

Complete guide for using BDD Superpowers with [OpenCode.ai](https://opencode.ai).

BDD Superpowers is a fork of Superpowers. It keeps the original skill workflow and adds Behavior Evaluation in specs plus Behavior Coverage in implementation plans.

## Installation

Add BDD Superpowers to the `plugin` array in your `opencode.json` (global or project-level):

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

Restart OpenCode. The plugin installs through OpenCode's plugin manager and
registers all skills.

Verify with the smoke-test conversation:

```bash
opencode run 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'
```

Check that `writing-plans` explains when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

### Migrating from upstream Superpowers

Use a delete-then-install flow:

1. Remove the upstream `obra/superpowers` plugin entry.
2. Remove any old symlink-based OpenCode install.
3. Install BDD Superpowers from the git URL above.
4. Refresh stale cache if the loaded skill text still looks like upstream Superpowers.

Do not keep upstream Superpowers and BDD Superpowers enabled together. They expose the same `superpowers` namespace.

Replace any plugin entry that points at upstream `obra/superpowers` with the BDD Superpowers entry above.

If the loaded skill text still looks stale after reinstalling, clear the cache described in [Refreshing Stale Superpowers Caches](cache-refresh.md).

OpenCode uses its own plugin install. If you also use Claude Code, Codex, or
another harness, install Superpowers separately for each one.

### Migrating from the old symlink-based install

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

## Usage

### Finding Skills

Use OpenCode's native `skill` tool to list all available skills:

```text
use skill tool to list skills
```

### Loading a Skill

```text
use skill tool to load superpowers/brainstorming
```

The namespace remains `superpowers` for compatibility.

### Personal Skills

Create your own skills in `~/.config/opencode/skills/`:

```bash
mkdir -p ~/.config/opencode/skills/my-skill
```

Create `~/.config/opencode/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: Use when [condition] - [what it does]
---

# My Skill

[Your skill content here]
```

### Project Skills

Create project-specific skills in `.opencode/skills/` within your project.

**Skill Priority:** Project skills > Personal skills > BDD Superpowers skills

## Updating

OpenCode installs Superpowers through a git-backed package spec. Some OpenCode
and Bun versions pin that resolved git dependency in a lockfile or cache, so a
restart may not pick up the newest Superpowers commit. If updates do not appear,
clear OpenCode's package cache or reinstall the plugin.

To pin the current branch explicitly:

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

## How It Works

The plugin does two things:

1. **Injects bootstrap context** via the `experimental.chat.messages.transform` hook, adding superpowers awareness to every conversation.
2. **Registers the skills directory** via the `config` hook, so OpenCode discovers all superpowers skills without symlinks or manual config.

### Tool Mapping

Skills speak in actions rather than naming any one runtime's tools. On OpenCode these resolve to:

- "Create a todo" / "mark complete in todo list" → `todowrite`
- `Subagent (general-purpose):` template → OpenCode's `task` tool with `subagent_type: "general"` (or `"explore"` for codebase exploration)
- "Invoke a skill" → OpenCode's native `skill` tool
- "Read a file" → `read`
- "Create a file" / "edit a file" / "delete a file" → `apply_patch`
- "Run a shell command" → `bash`
- "Search file contents" / "find files by name" → `grep`, `glob`
- "Fetch a URL" → `webfetch`

(Verified against the installed OpenCode CLI's tool inventory.)

## Troubleshooting

### Plugin not loading

1. Check OpenCode logs: `opencode run --print-logs "hello" 2>&1 | grep -i superpowers`
2. Verify the plugin line in your `opencode.json` is correct
3. Make sure you're running a recent version of OpenCode

### Installed upstream by mistake

1. Search `opencode.json` for `obra/superpowers`
2. Replace upstream entries with the BDD Superpowers git URL
3. Clear stale OpenCode package cache if the loaded skill text still looks like upstream Superpowers

### Windows install issues

Some Windows OpenCode builds have upstream installer issues with git-backed
plugin specs, including cache paths for `git+https` URLs and Bun not finding
`git.exe` even when it works in a normal terminal. If OpenCode cannot install
the plugin, try installing with system npm and pointing OpenCode at the local
package:

```powershell
npm install superpowers@git+https://github.com/zhexulong/bdd-superpowers.git --prefix "$HOME\.config\opencode"
```

Then use the installed package path in `opencode.json`:

```json
{
  "plugin": ["~/.config/opencode/node_modules/superpowers"]
}
```

### Skills not found

1. Use OpenCode's `skill` tool to list available skills
2. Check that the plugin is loading (see above)
3. Each skill needs a `SKILL.md` file with valid YAML frontmatter

### Bootstrap not appearing

1. Check OpenCode version supports `experimental.chat.messages.transform` hook
2. Restart OpenCode after config changes

## Getting Help

- BDD Superpowers: https://github.com/zhexulong/bdd-superpowers
- Upstream Superpowers: https://github.com/obra/superpowers
- OpenCode docs: https://opencode.ai/docs/
