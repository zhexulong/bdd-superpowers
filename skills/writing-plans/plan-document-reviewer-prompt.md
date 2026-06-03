# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, preserves behavior checkpoints, and has proper task decomposition.

**Dispatch after:** The complete plan is written.

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Verify this plan is complete and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers spec requirements, no major scope creep |
    | Behavior Pipeline | If the spec has Behavior Evaluation, the plan has behavior checkpoints, observable evidence/oracles, reject signals, correction paths, and cross-task invariants |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable |
    | BDD Mapping | Tasks reference BDD only when they implement, observe, correct, or preserve a declared checkpoint/invariant; unrelated tasks are technical-only |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Behavior Pipeline Guidance

    A behavior checkpoint is an acceptance scenario or concrete example turned into a reviewable control point. It is not a task, module boundary, milestone, or approval gate.

    If the spec includes Behavior Evaluation, the plan should include:
    - Behavior checkpoints derived from concrete examples or acceptance scenarios
    - Observable evidence or oracle for each checkpoint
    - Reject signal for each checkpoint
    - Correction path for failed evidence
    - Automation / observation split
    - Cross-task invariants when applicable

    Task-level BDD relationships should use only:
    - `implements BDD-N`
    - `observes BDD-N`
    - `corrects BDD-N`
    - `preserves INV-N`
    - `technical-only`

    Do not require every task to have a BDD scenario. Flag tasks that invent user behavior just to avoid `technical-only`.

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    missing behavior-control coverage for a behavior spec, contradictory steps,
    placeholder content, or tasks so vague they can't be acted on.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
