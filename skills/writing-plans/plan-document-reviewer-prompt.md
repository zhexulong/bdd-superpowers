# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, preserves behavior scenarios where applicable, and has proper task decomposition.

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
    | Behavior Coverage | If the spec has Behavior Evaluation, the plan turns those behavior scenarios into observable evidence/oracles, failure signals, correction paths, and cross-task invariants |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable |
    | Behavior Coverage | Tasks reference behavior coverage only when they implement, observe, correct, or preserve a declared scenario/invariant; unrelated tasks are technical-only |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Behavior Coverage Guidance

    A behavior scenario is an acceptance scenario or concrete example turned into a reviewable control point. It helps catch cases where local implementation is correct but the overall behavior or flow drifts. It is not a task, module boundary, milestone, approval gate, or replacement for TDD.

    If the spec includes Behavior Evaluation, the plan should include:
    - Behavior scenarios derived from the spec's concrete examples or acceptance scenarios
    - Observable evidence or oracle for each scenario
    - Failure signal for each scenario
    - Correction path for failed evidence
    - Automation / observation split
    - Cross-task invariants when applicable

    Task-level behavior coverage should use only:
    - `implements Scenario N`
    - `observes Scenario N`
    - `corrects Scenario N`
    - `preserves Invariant N`
    - `technical-only`

    Do not require every task to have a behavior scenario. Flag tasks that invent user behavior just to avoid `technical-only`. Technical-only tasks still need TDD, regression testing, or static verification as appropriate.

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    missing behavior coverage for a behavior spec, contradictory steps,
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
