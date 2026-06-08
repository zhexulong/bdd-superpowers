# Review-3 Architecture Lens Smoke Eval

This eval checks whether the spec document reviewer catches the core `review-3.md` failure mode: a support/eval/debug artifact becoming product contract or runtime authority.

It is intentionally small. It is a smoke eval for reviewer behavior, not proof that the reviewer catches every architecture ownership problem.

## Pressure Scenario

The intentionally flawed spec proposes a `docs/evals/debug-route-cache.json` support artifact that:

- stores routing decisions for eval stability
- is read by production runtime
- overrides normal route rules when they disagree
- becomes the only oracle in Behavior Evaluation
- uses exact request text as a cache key

This should be rejected because a debug/eval fixture becomes the owner of production routing behavior.

## Prompt

```text
Smoke eval only. Do not write files, do not run shell commands, and do not claim anything is saved. Read skills/brainstorming/spec-document-reviewer-prompt.md and use it to review this intentionally flawed spec. Return only the review result.

Spec to review:
# Debug Cache Authority Design

## Overview
Add a debug cache that stores the last successful routing decision. If normal routing cannot decide, production runtime will read the debug cache and use its stored route as the source of truth.

## Requirements
- The cache key is the exact request text.
- The cache file is docs/evals/debug-route-cache.json.
- Tests and runtime both read this file so eval behavior and production behavior stay aligned.
- If the cache contains a route, the router must trust it even when the normal route rules disagree.

## Architecture
The debug cache is introduced as a support artifact for eval stability, but it also decides runtime routing during fallback. There is no separate product contract for route ownership because the cache is easier to update.

## Behavior Evaluation
Example: A request with ambiguous route text is evaluated.
Expected result: The debug cache route is used.
Failure signal: Runtime ignores the cache.
Invariant: Eval and production use the same route answer.
Evidence/oracle: Inspect docs/evals/debug-route-cache.json.
Correction path: Update the cache file.
```

## Pass Criteria

- Reviewer returns `Issues Found`.
- Reviewer identifies that a support/eval/debug artifact became production authority or product contract.
- Reviewer identifies hidden ownership or accidental architecture.
- Reviewer identifies that Behavior Evaluation validates the mechanism rather than the intended behavior.
- Reviewer recommends relocating authority to an explicit contract/spec/rule layer or moving the artifact behind a private/eval-only boundary.
- Reviewer does not rewrite the design or approve the spec.

## Result

Run:

```bash
timeout 180s opencode run --log-level ERROR --format json "<prompt above>"
```

Observed output included:

```text
## Spec Review

**Status:** Issues Found
```

The reviewer reported:

- the debug/eval support artifact becomes the production source of truth for routing
- `docs/evals/debug-route-cache.json` becomes mutable product authority without product contract, schema, ownership model, or review path
- route ownership is hidden inside a convenience cache because it is easier to update
- the Behavior Evaluation validates cache usage, not routing correctness
- the oracle only proves the artifact contains a value, not that production routing is correct
- exact request text becomes a hidden global contract

Recommended dispositions included moving the cache behind an eval-only/private boundary and relocating routing authority to an explicit route contract/spec/rule layer.

Supported conclusion:

- The current `spec-document-reviewer-prompt.md` is sufficient to catch this review-3 architecture ownership failure mode in a smoke scenario.
- This eval does not prove broad review quality or plan/code review coverage.

Note: the outer `timeout` command returned 124 after the JSON event stream produced the final review and `step-finish` event. Treat the behavioral output as valid smoke evidence, but rerun with a longer timeout if command-level success is required.
