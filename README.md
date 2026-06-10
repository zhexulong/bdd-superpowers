# BDD Superpowers

[简体中文](README.zh-CN.md)

BDD Superpowers is a fork of [Superpowers](https://github.com/obra/superpowers) that keeps the original skills-first software development workflow and adds behavior evaluation to the design and planning path.

The base workflow is still Superpowers: brainstorm the design, write a spec, write an implementation plan, implement with TDD, review, and verify. The change is that non-trivial behavior work now carries a horizontal behavior/control harness alongside the normal vertical implementation slices.

In practical terms: TDD still checks local implementation correctness. Behavior Evaluation and Behavior Coverage check whether the whole flow is still doing what the user wanted, even when the code is too large or too opaque for a human to inspect line by line.

This is not the official Superpowers distribution. It is a Superpowers-derived fork focused on BDD-style behavior review, pipeline-level constraints, and design-plan-code alignment.

## How it works

It starts the same way Superpowers does. When you ask your coding agent to build something, it should not jump directly into code. It uses the brainstorming skill to understand what you are trying to do, explore alternatives, and turn the conversation into a reviewable design.

BDD Superpowers extends that step with a bounded behavior grill. The agent pressure-tests concrete examples, behavior boundaries, failure signals, invariants, and correction paths without turning the session into hours of exhaustive questioning. It should answer from code, docs, and existing conventions when it can, and ask the user only when the answer would change the design route.

Once the design is written, the spec can include a `Behavior Evaluation` section. This section is not another implementation plan. It describes what behavior must be observable, what results are expected, what signals mean the system drifted, what invariants must hold across the flow, and where to correct if the evidence fails.

After design approval, the writing-plans skill still produces a Superpowers-style implementation plan with concrete tasks, file paths, tests, and verification. When the spec contains Behavior Evaluation, the plan also includes `Behavior Coverage`: a short horizontal mapping from scenarios and invariants to implementation tasks and evidence. Technical-only tasks remain valid; the goal is not to force fake BDD onto every local slice.

Finally, review checks more than local test pass/fail. BDD Superpowers adds architecture ownership checks to document review and a shorter version to code review. The reviewer looks for cases where local implementation is correct but the behavior pipeline is wrong, and for cases where convenience glue, caches, wrappers, fallback paths, debug artifacts, or eval artifacts have quietly become product contract or runtime authority.

This matters because many agent mistakes are not "bad code" in isolation. They are wrong ownership: a temporary support mechanism starts deciding routing, truth, method, answer shape, read order, or policy. The reviewer asks what higher-level behavior a mechanism now controls, whether that ownership belongs there, and whether it should be thinned, moved behind a private/eval-only boundary, relocated to an explicit contract/spec layer, or removed.

## What differs from upstream Superpowers

- **Bounded behavior grill in brainstorming** - adds targeted pressure-testing before the design is finalized, without asking dozens of low-value questions up front.
- **Behavior Evaluation in specs** - captures concrete examples, expected results, failure signals, invariants, observable evidence, and correction paths for non-trivial behavior changes.
- **Behavior Coverage in plans** - connects the spec's behavior scenarios to plan tasks and verification evidence, while allowing unrelated implementation steps to stay `technical-only`.
- **Design document self-review** - the spec reviewer now checks for missing behavior evidence, ambiguous failure signals, weak invariants, unclear correction paths, hidden ownership, accidental architecture, and support mechanisms becoming product contract.
- **Plan document review** - the plan reviewer rejects fake per-task behavior coverage, checks whether horizontal scenarios are actually carried through the plan, and blocks plans that make support/eval/debug machinery into product contract.
- **Code review reinforcement** - code review checks flow-level drift and hidden ownership: local tests passing while the intended behavior or pipeline is not preserved, or implementation glue quietly taking over truth, method, routing, or policy ownership.

## Installation

Installation differs by platform. The important rule is: install this fork, not the official Superpowers marketplace package.

When migrating from upstream Superpowers, use a clean delete-and-install flow:

First decide which agent install you are changing: Codex, OpenCode, Claude Code, Cursor, Copilot, Gemini, or another local skill/plugin surface. Many users have more than one installed. Only uninstall and reinstall Superpowers for the selected agent.

1. Delete the old upstream Superpowers entry, symlink, junction, or clone.
2. Install BDD Superpowers from the git URL below.
3. Refresh stale caches for that selected agent. See [Refreshing Stale Superpowers Caches](docs/cache-refresh.md).
4. Run the smoke-test conversation below in the selected agent, or restart it first if that platform requires restart for plugin discovery. Treat the install as stale if the answer does not clearly explain when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

If you want to hand the migration to an agent, use this one-line instruction:

```text
Ask me which agent's Superpowers install to replace, then only for that selected agent uninstall upstream Superpowers, install BDD Superpowers from the git URL below, clear stale skill/plugin caches, run the README smoke-test conversation for that agent, or tell me if that platform requires restart first, and treat the install as stale unless `superpowers:writing-plans` clearly explains when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.
```

Smoke-test prompt:

```text
Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.
```

Use the platform's own non-interactive entry point when available:

- OpenCode: `opencode run 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`
- Codex: `codex exec 'Use superpowers:writing-plans. Answer only with three bullets: when does the plan include Behavior Coverage, what does technical-only mean, and how is Behavior Coverage different from TDD? If the loaded skill does not mention Behavior Coverage, say STALE SUPERPOWERS CACHE.'`

Use the BDD Superpowers repository:

```text
https://github.com/zhexulong/bdd-superpowers.git
```

The internal skill namespace currently remains `superpowers:*` for compatibility with existing agents and configs.

### OpenCode

Add BDD Superpowers to the `plugin` array in your `opencode.json`:

```json
{
  "plugin": ["superpowers@git+https://github.com/zhexulong/bdd-superpowers.git"]
}
```

Restart OpenCode if needed for plugin discovery. Then run the smoke test above and check that `writing-plans` explains when `Behavior Coverage` appears, what `technical-only` means, and how it differs from TDD.

If you previously installed official Superpowers, delete the old plugin entry before adding this one. Do not keep both official Superpowers and BDD Superpowers enabled at the same time; they expose the same skill names. If OpenCode or another agent still loads old skill text after the change, clear stale caches using [the cache refresh guide](docs/cache-refresh.md).

### OpenAI Codex CLI

Clone this fork and symlink its skills into Codex native skill discovery:

```bash
git clone https://github.com/zhexulong/bdd-superpowers.git ~/.codex/bdd-superpowers
mkdir -p ~/.agents/skills
ln -s ~/.codex/bdd-superpowers/skills ~/.agents/skills/superpowers
```

Restart Codex after installing. To update:

```bash
cd ~/.codex/bdd-superpowers && git pull
```

If you already have `~/.agents/skills/superpowers` pointing to official Superpowers, replace that symlink so it points to this fork.

After replacing an old install, clear stale plugin caches if the loaded skill text still looks like upstream Superpowers. See [Refreshing Stale Superpowers Caches](docs/cache-refresh.md).

### Claude Code, Cursor, Copilot, and Gemini

The official marketplaces install upstream Superpowers, not this fork. For now, do not use the official marketplace entries if you want BDD Superpowers.

Use a git-based install path when the platform supports it, or clone this repository and expose its `skills/` directory through the platform's local skill/plugin mechanism. Keep only one provider for the `superpowers` skill namespace enabled at a time.

## The Basic Workflow

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation, and runs a bounded behavior grill for non-trivial behavior changes.

2. **Behavior Evaluation** - Lives in the design/spec when needed. Defines concrete examples, expected results, failure signals, invariants, observable evidence, and correction paths.

3. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on a new branch, runs project setup, verifies clean test baseline.

4. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks with exact file paths, tests, and verification. Adds Behavior Coverage when the spec has Behavior Evaluation.

5. **subagent-driven-development** or **executing-plans** - Activates with the plan. Dispatches fresh subagents per task with review, or executes in reviewed batches.

6. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR for local implementation work.

7. **requesting-code-review** - Reviews against the plan and behavior coverage, reporting issues by severity. Critical issues block progress.

8. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents integration options, and cleans up worktree.

The agent checks for relevant skills before any task. Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle.

**Debugging**
- **systematic-debugging** - 4-phase root cause process.
- **verification-before-completion** - Ensure work is actually fixed before claiming success.

**Collaboration**
- **brainstorming** - Socratic design refinement plus bounded behavior grill.
- **writing-plans** - Detailed implementation plans plus Behavior Coverage when applicable.
- **executing-plans** - Batch execution with review points.
- **dispatching-parallel-agents** - Concurrent subagent workflows.
- **requesting-code-review** - Pre-review checklist with behavior drift checks.
- **receiving-code-review** - Responding to feedback with technical rigor.
- **using-git-worktrees** - Parallel development branches.
- **finishing-a-development-branch** - Merge/PR decision workflow.
- **subagent-driven-development** - Fast iteration with two-stage review.

**Meta**
- **writing-skills** - Create and test new skills following the Superpowers methodology.
- **using-superpowers** - Introduction to the skills system.

## Evaluation Status

Current evidence is intentionally narrow:

- The aligned design/spec eval passes on this fork and fails on upstream `origin/main` for behavior-evaluation requirements.
- The writing-plans Behavior Coverage smoke eval produces concrete tests/checks from a retained-eval scenario, but also caught an undefined `Scenario 2` task-reference failure. See [Behavior Coverage Writing-Plans Eval](docs/evals/behavior-coverage-writing-plans.md).
- Mutation and real-document replay checks have not proven broad superiority; they are useful as non-regression and diagnostic checks.
- The supported claim is not "better at everything." The supported claim is that this fork adds reviewable behavior/control harness requirements that upstream Superpowers does not currently enforce.

## Philosophy

- **Inherit Superpowers first** - This fork extends the original workflow instead of replacing it.
- **Behavior over document volume** - The point of BDD-style Markdown is to make intended behavior reviewable, not to write longer specs.
- **Horizontal plus vertical feedback** - TDD checks local implementation; Behavior Coverage checks whether the flow remains bound to user intent.
- **Evidence over claims** - Specs and plans should name observable evidence and failure signals.
- **Human final review** - Design review can filter weak designs, but humans still own final approval.

## Lineage

BDD Superpowers is forked from [obra/superpowers](https://github.com/obra/superpowers), originally built by Jesse Vincent and the Prime Radiant community.

Special thanks to the [linux.do](https://linux.do/) community for discussion, feedback, and early usage signals that shaped this fork.

This fork keeps the MIT license. See [LICENSE](LICENSE) for details.

## Contributing

Use the same discipline as Superpowers itself:

1. Fork the repository.
2. Create a branch for your work.
3. Use the `writing-skills` skill for skill changes.
4. Add or update eval coverage when changing behavior guidance.
5. Submit a PR with a clear description of the behavior impact.

## Community and Issues

- Upstream Superpowers: https://github.com/obra/superpowers
- BDD Superpowers: https://github.com/zhexulong/bdd-superpowers
