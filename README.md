# BDD Superpowers

[简体中文](README.zh-CN.md)

BDD Superpowers is a fork of [Superpowers](https://github.com/obra/superpowers). It keeps the original skills-first development workflow and adds a behavior/control harness to the design, planning, and review path.

The base workflow is still Superpowers: brainstorm the design, write a spec, write an implementation plan, implement with TDD, review, and verify. The difference is that non-trivial behavior work also gets Behavior Evaluation in the spec and Behavior Coverage in the plan.

In practical terms: TDD still checks local implementation correctness. Behavior Evaluation and Behavior Coverage check whether the whole flow is still doing what the user wanted, even when the code has become too large or too opaque for a human to inspect line by line.

This is not the official Superpowers distribution. Official marketplaces may install upstream Superpowers. To use this fork, install from `https://github.com/zhexulong/bdd-superpowers.git`.

## Quickstart

Give your agent BDD Superpowers: [Codex CLI](#codex-cli), [OpenCode](#opencode), [Claude Code](#claude-code), [Cursor](#cursor), [Gemini CLI](#gemini-cli), [GitHub Copilot CLI](#github-copilot-cli), [Kimi Code](#kimi-code), [Antigravity](#antigravity), [Factory Droid](#factory-droid), [Pi](#pi).

The internal skill namespace remains `superpowers:*` for compatibility with existing agents and configs.

## How It Works

It starts the same way Superpowers does. When you ask your coding agent to build something, it should not jump directly into code. It uses the brainstorming skill to understand what you are trying to do, explore alternatives, and turn the conversation into a reviewable design.

BDD Superpowers extends that path with a bounded behavior grill. The agent pressure-tests concrete examples, behavior boundaries, reject signals, invariants, and correction paths without turning the session into hours of exhaustive questioning. It should answer from code, docs, and existing conventions when it can, and ask the user only when the answer would change the design route.

Once the design is written, the spec can include `Behavior Evaluation`. This section is not another implementation plan. It names what behavior must be observable, what result is expected, what signal means the system drifted, what invariant must hold across the flow, and where to correct if the evidence fails.

After design approval, `writing-plans` still produces a Superpowers-style implementation plan with concrete tasks, file paths, tests, interfaces, and verification. When the spec contains Behavior Evaluation, the plan also includes `Behavior Coverage`: a short horizontal mapping from scenarios and invariants to implementation tasks and evidence. Technical-only tasks remain valid; the goal is not to force fake BDD onto every local slice.

Review checks more than local test pass/fail. BDD Superpowers adds architecture ownership checks to document review and a shorter version to code review. The reviewer looks for cases where local implementation is correct but the behavior pipeline is wrong, and for cases where convenience glue, caches, wrappers, fallback paths, debug artifacts, or eval artifacts have quietly become product contract or runtime authority.

This matters because many agent mistakes are not bad code in isolation. They are wrong ownership: a temporary support mechanism starts deciding routing, truth, method, answer shape, read order, or policy. The reviewer asks what higher-level behavior a mechanism now controls, whether that ownership belongs there, and whether it should be thinned, moved behind a private/eval-only boundary, relocated to an explicit contract/spec layer, or removed.

## What Differs From Upstream Superpowers

- **Bounded behavior grill in brainstorming** - targeted behavior pressure-testing before the design is finalized, without asking dozens of low-value questions up front.
- **Behavior Evaluation in specs** - concrete examples, expected results, failure signals, invariants, observable evidence, and correction paths for non-trivial behavior changes.
- **Behavior Coverage in plans** - connects behavior scenarios to implementation tasks and verification evidence, while allowing unrelated implementation work to stay `technical-only`.
- **Design document review** - checks missing behavior evidence, ambiguous failure signals, weak invariants, unclear correction paths, hidden ownership, accidental architecture, and support mechanisms becoming product contract.
- **Plan document review** - rejects fake per-task behavior coverage, checks whether horizontal scenarios are carried through the plan, and blocks support/eval/debug machinery becoming product contract.
- **Code review reinforcement** - checks flow-level drift and hidden ownership: local tests passing while the intended behavior or pipeline is not preserved, or implementation glue taking over truth, method, routing, or policy ownership.

## Installation

Installation differs by harness. If you use more than one agent, install BDD Superpowers separately for each one. The important rule is: install this fork, not the official upstream marketplace package.

When migrating from upstream Superpowers, use a clean delete-and-install flow:

1. Decide which agent install you are changing: Codex, OpenCode, Claude Code, Cursor, Copilot, Gemini, or another local skill/plugin surface.
2. Delete the old upstream Superpowers entry, symlink, junction, or clone for that selected agent.
3. Install BDD Superpowers from `https://github.com/zhexulong/bdd-superpowers.git`.
4. Refresh stale caches if the selected agent still loads old skill text. See [Refreshing Stale Superpowers Caches](docs/cache-refresh.md).
5. Run the smoke-test conversation below in the selected agent, or restart first only if that platform requires restart for plugin discovery.

If you want to hand migration to an agent, use this one-line instruction:

```text
Ask me which agent's Superpowers install to replace, then only for that selected agent uninstall upstream Superpowers, install BDD Superpowers from https://github.com/zhexulong/bdd-superpowers.git, clear stale skill/plugin caches if needed, run the README smoke-test conversation for that agent, and treat the install as stale unless superpowers:writing-plans clearly explains when Behavior Coverage appears, what technical-only means, and how Behavior Coverage differs from TDD.
```

Smoke-test prompt:

```text
Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.
```

Use the platform's non-interactive entry point when available:

- OpenCode: `opencode run 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`
- Codex: `codex exec 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`

### Codex CLI

Tell Codex:

```text
Fetch and follow instructions from https://raw.githubusercontent.com/zhexulong/bdd-superpowers/refs/heads/main/.codex/INSTALL.md
```

Manual install:

```bash
git clone https://github.com/zhexulong/bdd-superpowers.git ~/.codex/bdd-superpowers
mkdir -p ~/.agents/skills
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

If `~/.agents/skills/superpowers` already points to upstream Superpowers, replace that symlink or junction so only one provider owns the `superpowers` skill namespace.

Detailed docs: [docs/README.codex.md](docs/README.codex.md)

### OpenCode

Add BDD Superpowers to the `plugin` array in your `opencode.json`:

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

OpenCode installs the plugin through its package manager. Some OpenCode and Bun versions pin the resolved git dependency in a lockfile or cache, so if updates do not appear, clear OpenCode's package cache or reinstall the plugin.

Detailed docs: [docs/README.opencode.md](docs/README.opencode.md)

### Claude Code

The official marketplace installs upstream Superpowers. For this fork, use a git/local plugin install path when your Claude Code setup supports it, or clone this repository and expose its `skills/` directory through your local skill/plugin mechanism. Do not enable upstream Superpowers and BDD Superpowers at the same time.

### Cursor

The marketplace entry named `superpowers` may install upstream Superpowers. For this fork, use a git/local plugin install path when available, or clone this repository and point Cursor at this checkout. Keep only one `superpowers` provider enabled.

### Gemini CLI

Install from this fork when using Gemini extensions:

```bash
gemini extensions install https://github.com/zhexulong/bdd-superpowers
```

Update later:

```bash
gemini extensions update superpowers
```

### GitHub Copilot CLI

If your Copilot CLI setup supports git/local plugins, install from `https://github.com/zhexulong/bdd-superpowers.git`. Marketplace entries that point at `obra/superpowers` install upstream Superpowers, not this fork.

### Kimi Code

Install directly from this repository when Kimi Code supports URL installs:

```text
/plugins install https://github.com/zhexulong/bdd-superpowers
```

Detailed docs: [docs/README.kimi.md](docs/README.kimi.md)

### Antigravity

Install from this repository:

```bash
agy plugin install https://github.com/zhexulong/bdd-superpowers
```

### Factory Droid

If Droid supports git/local plugins, install from `https://github.com/zhexulong/bdd-superpowers.git`. Do not use an upstream marketplace entry if you want the BDD fork.

### Pi

Install from this repository:

```bash
pi install git:github.com/zhexulong/bdd-superpowers
```

For local development, run Pi with this checkout loaded as a temporary package:

```bash
pi -e /path/to/bdd-superpowers
```

## The Basic Workflow

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design for validation, and runs a bounded behavior grill for non-trivial behavior changes.
2. **Behavior Evaluation** - Lives in the design/spec when needed. Defines concrete examples, expected results, failure signals, invariants, observable evidence, and correction paths.
3. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on a new branch, runs project setup, and verifies a clean test baseline.
4. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks with exact file paths, interfaces, tests, and verification. Adds Behavior Coverage when the spec has Behavior Evaluation.
5. **subagent-driven-development** or **executing-plans** - Activates with the plan. Dispatches fresh subagents per task with review, or executes in reviewed batches.
6. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR for local implementation work.
7. **requesting-code-review** - Reviews against the plan and behavior coverage, reporting issues by severity. Critical issues block progress.
8. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents integration options, and cleans up worktree.

The agent checks for relevant skills before any task. These are mandatory workflows, not suggestions.

## What's Inside

### Testing

- **test-driven-development** - RED-GREEN-REFACTOR cycle.

### Debugging

- **systematic-debugging** - 4-phase root cause process.
- **verification-before-completion** - Ensure work is actually fixed before claiming success.

### Collaboration

- **brainstorming** - Socratic design refinement plus bounded behavior grill.
- **writing-plans** - Detailed implementation plans plus Behavior Coverage when applicable.
- **executing-plans** - Batch execution with review points.
- **dispatching-parallel-agents** - Concurrent subagent workflows.
- **requesting-code-review** - Pre-review checklist with behavior drift and ownership checks.
- **receiving-code-review** - Responding to feedback with technical rigor.
- **using-git-worktrees** - Parallel development branches.
- **finishing-a-development-branch** - Merge/PR decision workflow.
- **subagent-driven-development** - Fast iteration with two-stage review.

### Meta

- **writing-skills** - Create and test new skills following the Superpowers methodology.
- **using-superpowers** - Introduction to the skills system.

## Evaluation Status

Current evidence is intentionally narrow:

- The aligned design/spec eval passes on this fork and fails on upstream `origin/main` for behavior-evaluation requirements.
- The writing-plans Behavior Coverage smoke eval produces concrete tests/checks from a retained-eval scenario, but also caught an undefined scenario task-reference failure. See [Behavior Coverage Writing-Plans Eval](docs/evals/behavior-coverage-writing-plans.md).
- Mutation and real-document replay checks have not proven broad superiority; they are useful as non-regression and diagnostic checks.
- The supported claim is not "better at everything." The supported claim is that this fork adds reviewable behavior/control harness requirements that upstream Superpowers does not currently enforce.

## Philosophy

- **Inherit Superpowers first** - This fork extends the original workflow instead of replacing it.
- **Behavior over document volume** - BDD-style Markdown makes intended behavior reviewable; it is not a license to write longer specs.
- **Horizontal plus vertical feedback** - TDD checks local implementation; Behavior Coverage checks whether the flow remains bound to user intent.
- **Evidence over claims** - Specs and plans should name observable evidence and failure signals.
- **Human final review** - Design review can filter weak designs, but humans still own final approval.

## Contributing

Use the same discipline as Superpowers itself:

1. Fork the repository.
2. Create a branch for your work.
3. Use the `writing-skills` skill for skill changes.
4. Add or update eval coverage when changing behavior guidance.
5. Submit a PR with a clear description of the behavior impact.

Skill-behavior tests use the drill eval harness from [superpowers-evals](https://github.com/prime-radiant-inc/superpowers-evals/), cloned into `evals/` - see `evals/README.md` for setup. Plugin-infrastructure tests live at `tests/` and run via the relevant `run-*.sh` or `npm test`.

## Updating

Updates are agent dependent. For git-based installs, pull this repository or reinstall the plugin. If the agent still reports old skill text after updating, clear its cache using [Refreshing Stale Superpowers Caches](docs/cache-refresh.md).

## Visual Companion Telemetry

Because skills and plugins do not provide feedback to creators, upstream Superpowers loads the Prime Radiant logo from their website in brainstorming's optional visual companion feature. It includes the version of Superpowers in use. It does not include details about your project, prompt, or coding agent. To disable this, set `SUPERPOWERS_DISABLE_TELEMETRY` to any true value. Superpowers also honors Claude Code's `DISABLE_TELEMETRY` and `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` opt-outs.

## Lineage

BDD Superpowers is forked from [obra/superpowers](https://github.com/obra/superpowers), originally built by Jesse Vincent and the Prime Radiant community.

Special thanks to the [linux.do](https://linux.do/) community for discussion, feedback, and early usage signals that shaped this fork.

Special thanks to [user71](https://linux.do/u/user71) for contributing to the BDD approach.

This fork keeps the MIT license. See [LICENSE](LICENSE) for details.

## Community

- BDD Superpowers: https://github.com/zhexulong/bdd-superpowers
- Upstream Superpowers: https://github.com/obra/superpowers
- Upstream Discord: https://discord.gg/35wsABTejz
- Upstream release announcements: https://primeradiant.com/superpowers/

## License

MIT License - see [LICENSE](LICENSE) for details.
