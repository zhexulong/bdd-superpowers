# Refreshing Stale Superpowers Caches

When migrating from upstream Superpowers to BDD Superpowers, delete the old install, install this fork, and refresh any agent/plugin caches. Some agents load skills from cached plugin directories instead of the live git checkout. If that cache is stale, the `skill` tool can still load old `SKILL.md` files even after the new plugin entry is configured.

## When to do this

Run the relevant cache refresh step after:
- replacing upstream `obra/superpowers` with BDD Superpowers
- changing the git URL or branch for the `superpowers` plugin
- seeing a loaded skill that does not mention `Behavior Evaluation`, `Behavior Coverage`, behavior boundary checks, or architecture ownership checks

Restart the agent after clearing cache.

## Claude Code Plugin Cache

Windows PowerShell:

```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\plugins\cache\superpowers-dev\superpowers" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\plugins\cache\superpowers\superpowers" -ErrorAction SilentlyContinue
```

macOS / Linux:

```bash
rm -rf ~/.claude/plugins/cache/superpowers-dev/superpowers
rm -rf ~/.claude/plugins/cache/superpowers/superpowers
```

These commands remove only cached Superpowers plugin copies. Claude Code should rebuild the cache from the configured plugin source on restart.

## OpenCode Package Cache

Windows PowerShell:

```powershell
$paths = @("$env:USERPROFILE\.cache\opencode\packages", "$env:LOCALAPPDATA\opencode\packages")
foreach ($path in $paths) {
  if (Test-Path $path) {
    Get-ChildItem $path -Directory -Filter "superpowers*" | Remove-Item -Recurse -Force
  }
}
```

macOS / Linux:

```bash
find ~/.cache/opencode/packages -maxdepth 1 -type d -name 'superpowers*' -exec rm -rf {} + 2>/dev/null || true
```

Then restart OpenCode so it reinstalls the plugin from the BDD Superpowers git entry.

## Codex Native Skill Discovery

Codex native skills usually do not need a package cache refresh. The important thing is that the `superpowers` skill path points to this fork.

Windows PowerShell:

```powershell
cmd /c dir "%USERPROFILE%\.agents\skills\superpowers"
```

macOS / Linux:

```bash
ls -la ~/.agents/skills/superpowers
```

The target should point to `bdd-superpowers/skills`. If it points to an upstream clone such as `~/.codex/superpowers/skills`, delete the old link or junction and recreate it to point at this fork.

## Verify the Loaded Skill

After restarting, load or ask about `superpowers:writing-plans` or `superpowers:brainstorming`. The loaded text should include current BDD Superpowers language such as:
- `Behavior Evaluation`
- `Behavior Coverage`
- lightweight behavior boundary check
- architecture ownership check

If those terms are missing, the agent is still reading a stale cache or an upstream install.
