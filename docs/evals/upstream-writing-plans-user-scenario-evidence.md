# Upstream Eval Packet: Writing Plans Preserve User Scenario Evidence

## Decision

Build the upstream proposal around one narrow eval before proposing any
`writing-plans` skill change.

The eval should prove a specific behavior gap:

> Given an approved design/spec with one concrete user scenario, the
> implementation plan should carry that scenario into observable evidence.
> It should not plan only local implementation tests that can pass while the
> end-to-end user flow drifts.

This deliberately avoids upstream-hostile framing:

- It does not require the words `BDD`, `Behavior Coverage`, or `control harness`.
- It does not require every implementation task to carry scenario coverage.
- It does not reject technical-only/local tasks.
- It does not ask Superpowers to adopt this fork's branding or README framing.

## Implemented Artifact

Created a draft quorum scenario in the upstream eval harness checkout:

```text
/home/prosumer/agent/design-review-bdd-lab/forks/superpowers-evals/
  scenarios/writing-plans-preserves-user-scenario-evidence/
    story.md
    setup.sh
    checks.sh
```

Static validation:

```bash
cd /home/prosumer/agent/design-review-bdd-lab/forks/superpowers-evals
TMPDIR=/tmp XDG_CACHE_HOME=/tmp/bun-xdg-cache BUN_INSTALL_CACHE_DIR=/tmp/bun-install-cache \
  bun run quorum check scenarios/writing-plans-preserves-user-scenario-evidence
```

Result:

```text
ok   writing-plans-preserves-user-scenario-evidence
ok   credentials
```

`setup.sh` is executable and `checks.sh` is not executable, matching quorum's
scenario contract.

## Scenario Shape

The fixture creates a small existing project with:

- `src/csvImporter.js` producing normalized task objects
- `src/reportRenderer.js` rendering normalized tasks
- `docs/superpowers/specs/2026-06-25-json-import-pipeline-design.md`

The spec asks for a JSON importer that preserves the existing report behavior:

```json
[
  { "title": "Pay rent", "completed": true },
  { "title": "Book train", "completed": false }
]
```

Expected user-visible result:

- The generated report includes `Pay rent` as done.
- The generated report includes `Book train` as not done.
- JSON parsing stays in the importer/normalization boundary.
- The renderer does not become JSON-aware.

The Gauntlet-Agent asks only:

```text
Use superpowers:writing-plans to create an implementation plan for
docs/superpowers/specs/2026-06-25-json-import-pipeline-design.md.
Do not implement the feature. Create only the plan file under
docs/superpowers/plans/.
```

It explicitly does not mention `BDD`, `Behavior Coverage`, `control harness`,
or any desired section name.

## Acceptance Criteria

The LLM-graded acceptance criteria require:

- `superpowers:writing-plans` was loaded or clearly followed.
- A plan file was created under `docs/superpowers/plans/`.
- No application implementation was performed.
- The plan preserves the user scenario from the spec.
- The plan includes observable evidence for importer-to-renderer behavior,
  not only parser-local tests.
- The evidence has concrete expected results for both task titles and the
  done/not-done distinction.
- The plan names a failure signal or correction path.
- Technical/local tasks remain allowed.
- The plan does not invent undefined labels such as `Scenario 2`.

The deterministic `checks.sh` independently asserts:

- `superpowers:writing-plans` was called.
- A plan file exists.
- `src/jsonImporter.js` and `test/jsonImporter.test.js` do not exist, proving
  the agent did not implement code.
- The plan mentions `Pay rent`, `Book train`, done/completed state, and
  report/rendering evidence.
- The plan mentions a failure/correction pattern such as parser tests passing
  while report behavior fails, importer-to-renderer behavior, or the
  normalization boundary.
- The plan does not mention undeclared `Scenario 2` or `Scenario 3`.

## Why This Eval Is Credible

The eval checks the failure mode we care about, not our preferred vocabulary.

Credibility points:

- **Neutral fixture:** JSON importer + report renderer is a generic pipeline,
  not a project-specific BDD Superpowers domain.
- **No term leakage:** The user prompt does not ask for BDD or Behavior
  Coverage.
- **Observable output:** The required evidence is concrete: two task titles and
  done-state semantics in a report.
- **TDD boundary:** Parser-local tests are allowed, but they are insufficient
  alone. This captures the exact gap: local correctness can pass while the
  user flow is wrong.
- **No fake ceremony:** Technical/local tasks are explicitly allowed.
- **Independent checks:** The LLM acceptance criteria and shell post-checks
  assert overlapping facts.
- **Negative guard:** Undefined scenario labels are rejected because prior local
  smoke evals found that failure mode.

Credibility limits:

- Static validation only proves the scenario shape, not baseline behavior.
- The deterministic grep checks are intentionally broad; they can catch obvious
  omissions but cannot fully judge plan quality.
- A clever but weak plan could mention the right words without a useful test.
  The LLM acceptance criteria must catch that.
- A strong plan that describes the evidence using unexpected wording may need
  a small deterministic-check adjustment.

## RED/GREEN Plan

Run three stages before proposing a skill patch upstream.

### 1. Baseline RED

Checkout upstream Superpowers `dev` or latest target ref.

```bash
cd /home/prosumer/agent/design-review-bdd-lab/forks/superpowers-evals
export SUPERPOWERS_ROOT=/path/to/upstream-superpowers
bun run quorum run scenarios/writing-plans-preserves-user-scenario-evidence --coding-agent codex
```

Repeat on at least one more supported agent if credentials are available:

```bash
bun run quorum run scenarios/writing-plans-preserves-user-scenario-evidence --coding-agent claude
```

Expected baseline failure pattern:

- Plan focuses on JSON parser/unit tests only.
- Plan omits importer-to-renderer/report evidence.
- Plan does not name the report-level expected result or failure signal.

If baseline already passes consistently, do not patch upstream. The eval can
still be proposed as non-regression coverage, but it does not justify a skill
change by itself.

### 2. Minimal Patch GREEN

Patch only upstream `skills/writing-plans/SKILL.md` and, if necessary,
`skills/writing-plans/plan-document-reviewer-prompt.md`.

Use upstream language:

- `user scenario`
- `observable evidence`
- `expected result`
- `failure signal`
- `correction path`
- `local implementation tests`

Avoid fork language in the first PR:

- `BDD`
- `Behavior Coverage`
- `control harness`
- large new sections
- README or branding changes

Then rerun the same scenario. The patch is only credible if the same scenario
changes from fail to pass without making unrelated scenarios worse.

### 3. Regression Sweep

Run nearby scenarios:

```bash
bun run quorum run scenarios/triggering-writing-plans --coding-agent codex
bun run quorum run scenarios/cost-spec-plan-duplication --coding-agent codex
bun run quorum run scenarios/writing-plans-no-spec-conversational --coding-agent codex
```

The patch should not:

- make writing-plans trigger later
- cause plan/spec duplication
- force scenario/evidence ceremony when no approved spec exists
- increase task count dramatically for simple work

## Upstream Acceptance Strategy

Recommended PR sequence:

1. Open an issue or discussion with the problem statement and this eval.
2. Submit the eval-only PR to `prime-radiant-inc/superpowers-evals`.
3. After maintainer feedback, submit a minimal Superpowers PR to `obra/superpowers`
   targeting `dev`.

Do not submit this fork's full BDD patch upstream.

The strongest upstream framing:

> `writing-plans` can lose a spec's user scenario by producing only local
> implementation tests. This eval checks that a plan carries the scenario into
> observable evidence while still allowing local technical tasks.

Avoid framing:

> Add BDD Superpowers behavior coverage to upstream.

## Acceptance Probability

Estimated chance by approach:

- **Eval-only PR to `superpowers-evals`: medium-high**, if baseline evidence is
  real and the scenario remains terminology-neutral.
- **Minimal `writing-plans` wording PR with RED/GREEN evidence: medium**, if it
  changes only a few lines and does not disturb cost scenarios.
- **Full BDD Superpowers PR: low**, because upstream rejects fork-specific,
  bundled, or philosophy/branding changes without extensive eval evidence.

## Reviewer Checklist

Ask the reviewer to inspect these points:

- Does the scenario accidentally teach the Coding-Agent the expected answer?
- Are the acceptance criteria independent of BDD Superpowers terminology?
- Are the deterministic checks too broad, too narrow, or gameable?
- Would a parser-only plan fail?
- Would a correct plan with different section names pass?
- Does the scenario distinguish "plan only" from implementation?
- Does the baseline actually fail on upstream `dev`?
- Does the minimal patch pass without increasing cost or breaking nearby
  writing-plans scenarios?

## Current State

Completed:

- Cloned `prime-radiant-inc/superpowers-evals`.
- Added the draft scenario.
- Installed dependencies with writable Bun cache/temp paths.
- Ran static scenario validation successfully.

Not completed:

- No live quorum run has been executed yet.
- No upstream skill patch has been made.
- No GitHub issue or PR has been opened.

Do not claim this eval proves BDD Superpowers is better. At this stage it only
provides a fair pressure scenario for one proposed upstream behavior:
implementation plans should preserve concrete user scenarios as observable
evidence.
