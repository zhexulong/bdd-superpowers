# Document Reviewer Loop Loading Smoke Eval

This eval checks whether runtime skill loading exposes the automatic document reviewer loop guidance.

It is intentionally small. It verifies the loaded skill text, not full multi-turn reviewer behavior.

## What It Checks

The loaded `superpowers:brainstorming` skill should say:

- run the spec document reviewer after spec self-review
- use subagent/reviewer dispatch when supported
- use the same reviewer prompt inline when no separate reviewer is available
- fix reviewer issues and run the reviewer again
- repeat until reviewer approval or human decision

The loaded `superpowers:writing-plans` skill should say the same for the plan document reviewer.

## Prompt

```text
Use the skill tool to load superpowers:brainstorming and superpowers:writing-plans. Answer exactly one JSON object with keys brainstorming_reviewer_loop and writing_plans_reviewer_loop. Set each to true only if the loaded skill says to run the document reviewer, fix reviewer issues, and run the reviewer again until approval or human decision. If either loaded skill does not contain that guidance, set that key to false. Do not write files.
```

## Pass Criteria

```json
{"brainstorming_reviewer_loop":true,"writing_plans_reviewer_loop":true}
```

## Result

Run:

```bash
timeout 180s opencode run --log-level ERROR --format json "<prompt above>"
```

Observed result on 2026-06-09:

```json
{"brainstorming_reviewer_loop":true,"writing_plans_reviewer_loop":true}
```

Supported conclusion:

- OpenCode runtime skill loading sees the automatic reviewer loop wording in both `brainstorming` and `writing-plans`.
- This does not prove a full end-to-end document review loop with a real fix and re-review. That should be tested separately if the loop mechanics change.
