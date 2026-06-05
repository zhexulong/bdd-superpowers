# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, preserves behavior scenarios where applicable, keeps architecture ownership clear, and has proper task decomposition.

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
    | Architecture Ownership | Tasks do not turn convenience glue, eval/debug artifacts, caches, wrappers, fallback paths, or support docs into product contract unless the spec explicitly makes them authority |
    | Over-Binding | Plan avoids unnecessary fixed file names, fixed read order, exact choreography, historical scaffolding, or hidden route policy that would be harder to evolve than the spec requires |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Behavior Coverage Guidance

    A behavior scenario is a concrete example or user-observable flow turned into a reviewable control point. It helps catch cases where local implementation is correct but the overall behavior or flow drifts. It is not a task, module boundary, milestone, approval gate, or replacement for TDD.

    If the spec includes Behavior Evaluation, the plan should include:
    - Behavior scenarios derived from the spec's concrete examples or observable flows
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

    ## Architecture Review Lens

    Review whether the plan will preserve or worsen accidental architecture from the spec or existing system.

    Look for plan tasks that:
    - make a local convenience mechanism, wrapper, adapter, cache, fallback, index, ranking, debug artifact, eval artifact, or support document the owner of product behavior
    - implement hidden glue that controls method, answer shape, read order, route grammar, policy, or truth ownership
    - create a second authority surface instead of relocating behavior to an explicit contract, schema, spec, skill, or authority doc
    - hardcode local choreography, fixed artifact names, fixed task order, historical scaffolding, or reference-specific assumptions without spec justification
    - keep a mechanism that comparable systems avoid, isolate differently, or solve with less machinery, without explaining why this plan needs it

    For each architecture finding, state:
    - which task or step introduces or preserves the mechanism
    - what local problem it tries to solve
    - what higher-level decision, workflow, or contract it would now control
    - why that ownership is risky for implementation or future evolution
    - the clearer alternative
    - disposition: keep as-is, thin, move behind private/eval-only boundary, relocate to explicit contract/spec/skill layer, or remove

    Do not fail a plan merely for having support code. Fail it when support code becomes hidden authority, product contract, or a stricter workflow owner than the spec.

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the spec,
    missing behavior coverage for a behavior spec, contradictory steps,
    placeholder content, tasks so vague they can't be acted on, or architecture ownership mistakes that would cause the wrong thing to be built.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
