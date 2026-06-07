# Behavior Coverage Writing-Plans Eval

## Purpose

Check whether `superpowers:writing-plans` turns a spec's `Behavior Evaluation` scenario into concrete implementation-plan checks instead of only restating the behavior.

This eval is intentionally small. It is a smoke eval for the skill guidance, not proof that BDD Superpowers is broadly better than upstream Superpowers.

## Pressure Scenario

Ask an agent to use `superpowers:writing-plans` and produce only a `Behavior Coverage` section plus one task excerpt for this spec excerpt:

```text
Goal: A retained bootstrap-quality report must not claim retained eval success just because command execution and fixed post-generation question eval pass.

Behavior Evaluation:

Scenario: Narrow no-graph report answers fixed questions but lacks runtime coverage.
Example: The report has `Status: pass`, `Question eval status: pass`, one accepted reading page about request syntax, and explicit uncovered claims for runtime execution, configuration, sessions, transport, auth, plugins, uploads, output, and downloads.
Expected result: `Retained eval false-positive guard: triggered`; `Retained eval outcome: fail`; top metadata must not let `Status: pass` imply retained success.
Failure signal: report only shows prominent `Status: pass`, or missing runtime probe is mentioned in a lower section but does not affect retained outcome.
If it fails: fix report outcome aggregation and report placement, not generation or question pack.
```

The prompt should explicitly ask whether Behavior Coverage turns the scenario into concrete tests/checks with commands and expected outputs.

## Passing Criteria

- The output includes a Behavior Coverage scenario or equivalent row derived from the spec's scenario, with observable evidence, expected result, failure signal, and correction path.
- The automated check is concrete enough to implement: exact command, expected output or expected failure, and a test/eval/report artifact to inspect.
- The task excerpt follows TDD-style concreteness for any automated check: write the failing test/check, run it, expected failure, then implementation.
- Task-level behavior coverage references only declared scenarios or invariants. If the output says `observes Scenario 2`, there must be a declared Scenario 2.
- The output does not treat generic `tests pass` as enough to prove Behavior Coverage.

## Failing Patterns

- Restates the scenario without a test, command, eval, report output, or human observation.
- Provides a command without expected output or expected failure.
- Adds fake per-task behavior coverage to unrelated technical work instead of using `technical-only`.
- Splits one scenario into extra checks, then references those checks as undeclared scenarios in task mapping.
- Uses placeholder paths or APIs as if they were known current repo facts.

## Latest Result

Date: 2026-06-08

Current skill invocation:
- Loaded the updated `writing-plans` skill text.
- Produced concrete pytest-style tests, fixture content, exact command, and expected pre-implementation failure.
- Failed the declared-scenario-reference criterion by writing `observes Scenario 2` even though the Behavior Coverage section declared only one scenario. This appears to be a task-mapping error: the agent split the original scenario into a second check about top metadata ordering, but did not declare it as a separate scenario.

Old-guidance ablation:
- Also produced a regression-test-shaped answer with a command and expected pass/fail text.
- Was less concrete: no full test code and a more placeholder-like test command.

Supported conclusion:
- The updated guidance is sufficient to produce concrete tests/checks in this pressure scenario.
- The eval does not prove the new sentence is the only cause.
- Future runs should treat undefined scenario references as a failure, even when the generated tests are otherwise concrete.
