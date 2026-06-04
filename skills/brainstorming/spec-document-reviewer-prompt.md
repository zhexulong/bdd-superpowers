# Spec Document Reviewer Prompt Template

Use this template when dispatching a spec document reviewer subagent.

**Purpose:** Verify the spec is complete, consistent, behavior-grounded, and ready for implementation planning.

**Dispatch after:** Spec document is written to docs/superpowers/specs/

```
Task tool (general-purpose):
  description: "Review spec document"
  prompt: |
    You are a spec document reviewer. Verify this spec is complete and ready for planning. You are doing initial filtering only; human approval remains required.

    **Spec to review:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", incomplete sections |
    | Consistency | Internal contradictions, conflicting requirements |
    | Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
    | Scope | Focused enough for a single plan — not covering multiple independent subsystems |
    | Behavior Evaluation | For behavior changes, concrete examples, accept/reject scenarios, invariants, observable evidence/oracle, and correction paths |
    | Architecture Boundary | Components have behavior-backed responsibilities; no placeholder architecture or unsupported abstraction |
    | YAGNI | Unrequested features, over-engineering |

    ## Behavior Evaluation Guidance

    A behavior scenario is an acceptance scenario or concrete example turned into a reviewable control point. It is not a task, module boundary, milestone, or approval gate.

    For specs that change behavior someone relies on, verify the spec answers:
    - What concrete example exposes the intended behavior?
    - What result should be accepted?
    - What result must be rejected as a misunderstanding?
    - What invariant must remain true across implementation choices?
    - What observable evidence or oracle lets a test or human reviewer judge the behavior?
    - If that evidence fails, should correction return to the spec, plan, implementation, or human decision?

    ## Calibration

    **Only flag issues that would cause real problems during implementation planning.**
    A missing section, a contradiction, or a requirement so ambiguous it could be
    interpreted two different ways — those are issues. Minor wording improvements,
    stylistic preferences, and "sections less detailed than others" are not.

    Treat missing reject scenarios, missing oracle/evidence, or missing correction path for behavior changes as issues because they let planning proceed without a way to detect or correct behavioral drift.

    Approve unless there are serious gaps that would lead to a flawed plan. "Approved" means ready for the human to review and decide; it does not replace human approval.

    ## Output Format

    ## Spec Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters for planning]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
