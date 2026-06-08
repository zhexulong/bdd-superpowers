# Design Direction Review Workflow

## Overview

Reduce the repetitive "present many design sections and ask for approval after each one" pattern in `brainstorming` while preserving the core Superpowers human-in-the-loop philosophy.

The new workflow keeps Superpowers as the base:

```text
brainstorming -> design/spec -> writing-plans -> TDD -> review -> verification
```

The change is inside `brainstorming`: replace repeated section-by-section approval with one short design direction before the final spec is written. The design direction summarizes the direction the final spec will take, exposes architecture risks using the review-3 lens, and asks the human for one directional approval before writing the spec document.

This is not Kiro/Spec Kit goal-mode. The agent must not generate a full spec first and then ask the human to review a finished artifact. The human sees the intended spec shape before the spec becomes durable.

## Problem

Current Superpowers brainstorming asks the agent to present design sections and get approval after each section. In practice this can cause:

- Repeated low-value confirmations.
- Long conversations where the user is asked to approve prose structure rather than design direction.
- A late final spec that may already feel over-shaped by the agent.
- Weak integration of design review: reviewer prompts exist, but they are support templates unless the workflow explicitly dispatches them.

The user uses the spec to judge whether the agent has understood their intent. If review only happens after the spec is written, the workflow shifts toward a more autonomous "agent writes the spec, then user corrects it" model.

## Goals

- Preserve Superpowers' human approval before implementation planning.
- Keep the user able to judge whether the spec will align with their intent before the durable spec is written.
- Borrow Matt Pocock's lightweight skill shape: ask focused questions until shared understanding, then write from existing context; do not keep interviewing during document writing.
- Integrate the review-3 architecture lens without turning every brainstorm into a heavy review ceremony.
- Keep Behavior Evaluation in specs and Behavior Coverage in plans as the horizontal behavior/control harness.
- Make document review prompts actually reachable from the workflow.

## Non-Goals

- Do not replace Superpowers with Trellis, Kiro, Spec Kit, GSD, or Matt Pocock's workflow.
- Do not introduce a new repo-local task database, state machine, or issue tracker requirement.
- Do not force BDD/Behavior Coverage onto purely technical tasks.
- Do not make machine review the final approval authority.
- Do not add more section-by-section approval points.

## References

- Matt Pocock `grill-me`: one question at a time, recommended answer, inspect code/docs instead of asking when possible.
- Matt Pocock `to-prd`: synthesize existing conversation context; do not continue interviewing while producing the durable artifact.
- Matt Pocock `to-issues`: vertical tracer-bullet slices for issue decomposition. This is optional later reference for delegation or issue decomposition only; it is not part of the default Superpowers path.
- Matt Pocock `review`: separate Spec and Standards axes so one concern does not mask another.
- OpenReverse `review-3.md`: hidden ownership, accidental architecture, over-bound structure, and support/eval/debug artifacts becoming product contract.
- GSD sketch/refinement pattern: an approved sketch is the authoritative boundary before full planning.
- Kiro/Spec Kit: useful reference for spec-driven review points, but too autonomous/heavy for this fork's default philosophy.

## Relationship to Superpowers and Matt Pocock Skills

This design keeps Superpowers as the operating system. Matt Pocock's skills contribute interaction shapes, not a replacement workflow:

- `grill-me` supports focused shared-understanding questions before writing durable artifacts.
- `to-prd` supports the transition from conversation context to a written document without restarting the interview.
- `to-issues` may inform later delegation or issue decomposition, but issue creation and delegation labels are outside the default Superpowers path.
- `review` supports separating requirement/spec alignment from general standards review.

The design direction step is therefore this fork's adaptation. It is not a Matt Pocock primitive and should not be described as one.

## Why Superpowers Used Section Review

The original section-by-section design review exists for a real reason: it reduces drift in large or ambiguous designs by letting the human correct architecture, components, data flow, error handling, and testing before implementation planning.

The problem is not human review itself. The problem is making every normal design pass through many approval points even when one short design direction would give the user enough control.

This fork changes the default, not the safety valve:

- Default: one short design direction before durable spec writing.
- Fallback: use section-by-section review when the work spans multiple subsystems, has complex UI/UX shape, contains unresolved architecture trade-offs, or the user asks for detailed incremental review.
- Never: write the full spec first and treat human review as only post-hoc correction.

## Proposed Workflow

### 1. Explore Context

Unchanged from current `brainstorming`.

The agent reads relevant files, docs, recent commits, and conventions before asking questions.

### 2. Clarify Toward Shared Understanding

Replace broad section-confirmation with focused questions inspired by Matt Pocock:

- Ask one question at a time.
- Provide a recommended answer and why it matters.
- If code/docs can answer the question, inspect them instead of asking.
- Stop when the design direction is clear enough to present, not when every implementation detail is known.

Questions should focus on route-changing decisions:

- User behavior and success/failure signals.
- Architecture ownership and boundaries.
- Behavior examples, invariants, and correction paths.
- Terminology conflicts.

### 3. Present Design Direction

Before writing the spec file, present the design direction:

```markdown
## Design Direction

### Goal
<What behavior or outcome the user is trying to control.>

### Recommended Approach
<The recommended design direction in 3-6 bullets.>

### Behavior Evaluation
- Example:
- Expected result:
- Failure signal:
- Invariant:
- Evidence/oracle:
- Correction path:

### Architecture Notes
- Intended owner:
- Ownership or over-binding risks:
- Recommended disposition:

### Out of Scope
- <Explicit exclusions.>
```

This should usually fit in one screen. It is not a full spec and should not repeat every future section heading.

Ask one question:

```text
Does this design direction match the spec you want me to write?
```

If the user asks for changes, revise the design direction and ask again. Do not write the durable spec until the direction is approved.

### 4. Write the Spec

After direction approval, write the design spec to:

```text
docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md
```

The spec expands the approved direction into the normal Superpowers design doc, including Behavior Evaluation when the work is a non-trivial behavior change.

Writing the spec should synthesize existing context. It should not restart questioning unless the approved direction contains an unresolved blocking question.

### 5. Spec Document Review

After the spec file is written, run the normal spec self-review. Then run the spec document reviewer using `skills/brainstorming/spec-document-reviewer-prompt.md`.

If the platform supports subagents or reviewer dispatch, dispatch it as a separate reviewer. Otherwise, run the same reviewer prompt inline and state that no separate reviewer was available.

This review is a sanity check, not first contact with the design direction.

It checks:

- Completeness and consistency.
- Whether Behavior Evaluation was preserved.
- Whether the review-3 architecture lens reveals blocking ownership mistakes.
- Whether the written spec drifted from the approved design direction.

If the reviewer reports issues:

```text
fix spec -> re-review -> repeat until approved or human decision needed
```

Machine approval only means "ready for human review." It never replaces human approval.

### 6. Human Reviews Spec

Keep the existing human review gate:

```text
Spec written and reviewed at <path>. Please review it and tell me if you want changes before we write the implementation plan.
```

### 7. Plan and Behavior Coverage

After human approval, `writing-plans` creates the implementation plan.

If the spec includes Behavior Evaluation, the plan includes Behavior Coverage. Behavior Coverage maps scenarios/invariants to concrete tests, checks, observations, expected outputs, failure signals, and correction paths.

Technical-only tasks remain valid when they do not implement, observe, correct, or preserve a declared behavior scenario/invariant.

### 8. Plan Document Review

After the plan is written, run the normal plan self-review. Then run the plan document reviewer using `skills/writing-plans/plan-document-reviewer-prompt.md`.

If the platform supports subagents or reviewer dispatch, dispatch it as a separate reviewer. Otherwise, run the same reviewer prompt inline and state that no separate reviewer was available.

The reviewer checks:

- Spec alignment.
- Behavior Coverage concreteness and declared-scenario bookkeeping.
- Architecture ownership and over-binding.
- Whether tasks are actionable without turning support machinery into product contract.

If issues are found:

```text
fix plan -> re-review -> repeat until approved or human decision needed
```

## Behavior Evaluation

### Scenario: User Can Validate the Spec Direction Before Durable Spec Writing

**Example:** The user proposes a BDD/review workflow change. The agent finishes clarifying and presents one design direction before writing the spec.

**Expected result:** The user can tell whether the intended spec direction matches their thinking without reading a full generated spec or approving many sections.

**Failure signal:** The agent writes the spec before direction approval, or asks for approval after multiple long design sections instead of one short design direction.

**Invariant:** Durable spec writing requires prior directional approval from the human.

**Evidence/oracle:** The conversation contains a short design direction and an explicit user approval or requested revision before the spec file is written.

**Correction path:** Return to `brainstorming` and revise the design direction; do not proceed to spec writing.

### Scenario: Review-3 Lens Filters Architecture Risk Before Plan Lock-In

**Example:** A proposed design introduces a support cache, eval artifact, route grammar, or fallback path that could become the owner of product behavior.

**Expected result:** The design direction surfaces the ownership risk before the final spec is written, and the spec either removes it, thins it, moves it behind a private/eval-only boundary, or makes the real contract explicit.

**Failure signal:** The plan later treats support/eval/debug machinery as product authority without the spec explicitly making that decision.

**Invariant:** Support/eval/debug artifacts must not become product contract by accident.

**Evidence/oracle:** The design direction/spec contains architecture notes with disposition for any such mechanism, and the plan reviewer rejects tasks that violate it.

**Correction path:** Return to spec if the owner is wrong; return to plan if the spec is clear but tasks over-bind the mechanism.

### Scenario: Behavior Coverage Remains Horizontal, Not Per-Task Ceremony

**Example:** A plan has a mix of user-behavior work and small technical implementation tasks.

**Expected result:** Behavior Coverage maps only declared scenarios/invariants to tests or observations. Unrelated technical tasks use `technical-only` and still follow TDD or static verification.

**Failure signal:** The plan invents scenario labels to avoid `technical-only`, references undeclared scenarios, or treats every implementation slice as BDD.

**Invariant:** Behavior Coverage is a horizontal behavior/control harness; TDD remains the vertical local implementation feedback loop.

**Evidence/oracle:** The plan reviewer can trace each behavior reference to a declared scenario/invariant, and every automated behavior check has a concrete expected result.

**Correction path:** Return to plan and repair Behavior Coverage; return to spec if the scenario/invariant was missing from Behavior Evaluation.

## Skill Changes

### `skills/brainstorming/SKILL.md`

Change the design presentation section:

- Replace repeated approval after each design section with one short design direction.
- Add the review-3 architecture lens as an internal check before presenting the design.
- After direction approval, write the spec.
- After spec writing, dispatch the spec document reviewer.
- Keep human spec review after machine initial review.

### `skills/brainstorming/spec-document-reviewer-prompt.md`

Update purpose:

- It verifies the written spec against the approved design direction.
- It does not create the first design direction.
- It remains advisory and initial-filtering only.

Add an input placeholder:

```text
Approved design direction: [DESIGN_DIRECTION_TEXT_OR_PATH]
```

### `skills/writing-plans/SKILL.md`

Keep existing Behavior Coverage guidance, but ensure plan creation follows human-approved spec only.

After plan writing, dispatch the plan document reviewer instead of leaving the prompt as support-only material.

### `skills/writing-plans/plan-document-reviewer-prompt.md`

Keep current Behavior Coverage and architecture ownership checks.

Add explicit failure for:

- Undeclared scenario/invariant references.
- Support/eval/debug artifacts becoming product contract in tasks.
- Behavior Coverage rows with no concrete evidence, expected result, or correction path.

### `skills/requesting-code-review/code-reviewer.md`

Keep the review-3 lens short at code-review time:

- Does support/eval/debug/helper code become runtime authority?
- Could local tests pass while the declared behavior scenario drifts?
- Do implementation changes preserve declared invariants?

Do not paste the full review-3 prompt into code review.

## User Experience

Before:

```text
Ask questions -> propose approaches -> present many sections -> approve section 1 -> approve section 2 -> approve section 3 -> write spec
```

After:

```text
Ask focused questions -> present one design direction -> user approves direction -> write spec -> machine initial review -> human spec review
```

The user reviews fewer intermediate artifacts, but gets earlier control over the final spec direction.

## Risks

### Risk: The Design Direction Becomes Another Long Spec

Mitigation:

- State that the design direction should usually fit in one screen.
- It should contain decisions and risks, not full prose.
- If it grows too large, ask another grill question or split the work.

### Risk: Agent Skips Direction Approval and Writes the Spec

Mitigation:

- Make direction approval a hard gate in `brainstorming`.
- Smoke-eval the skill with prompts that ask for feature design and check whether spec writing happens only after direction approval.

### Risk: Review-3 Lens Over-Blocks Normal Designs

Mitigation:

- Use calibration from existing reviewer prompts.
- Block only when wrong ownership would cause the plan to build the wrong thing.
- List advisory architecture concerns separately.

### Risk: Workflow Becomes Too Heavy

Mitigation:

- Do not add Trellis-style state files or a task database.
- Do not require full document review for trivial technical-only changes.
- Keep the design direction short.

## Eval Plan

### Static Checks

- `git diff --check`
- Existing Superpowers test/smoke scripts where applicable.

### Smoke Eval: Design Direction Appears Before Spec Writing

Prompt an agent to use `superpowers:brainstorming` for a non-trivial workflow change.

Pass criteria:

- It asks focused questions or answers from repo context.
- It presents a short design direction before writing the spec.
- It does not ask for approval after many long sections.
- It does not write the spec before user approval of the design direction.

### Smoke Eval: Review-3 Risk Is Surfaced Early

Use a prompt that proposes a support/eval artifact as the behavior authority.

Pass criteria:

- The design direction identifies the hidden ownership risk.
- The proposed disposition is keep/thin/private-eval-only/relocate/remove.
- The final spec does not silently make the support artifact product contract.

### Smoke Eval: Plan Carries Behavior Coverage Without Over-Binding

Use a spec with one declared behavior scenario and several technical tasks.

Pass criteria:

- Plan includes Behavior Coverage with concrete evidence and expected result.
- Technical-only tasks are allowed.
- No undeclared scenario labels appear.
- Plan reviewer would reject fake or invented scenario mappings.

## Decision

Adopt a short **design direction** step as the proposed workflow concept for now.

Do not implement goal-mode. Do not write the full spec before the human sees and approves the intended direction.

Use Matt Pocock's skills as interaction shape, not as a replacement:

- `grill-me` supplies focused shared-understanding conversation.
- `to-prd` supplies discipline for writing from existing context.
- `to-issues` supplies optional later reference for delegation or issue decomposition.
- `review` supplies separate review axes.

Use Superpowers as the backbone, and add BDD/control-harness and review-3 architecture ownership checks as normal review points.
