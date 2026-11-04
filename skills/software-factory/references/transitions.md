# Task transitions

The Coordinator proposes task-cycle transitions. The Orchestrator checks the
current record, authority, assignment generation, and evidence, then saves the
transition and reads it back. The Orchestrator reports the saved result in its thread.

## Stages

| From | To | Required evidence or action |
| --- | --- | --- |
| New request | `shaping` | Request recorded with a stable ID |
| `shaping` | `queued` | Objective, scope, criteria, authority, and material assumptions are sufficiently defined |
| `queued` | `implementing` | Dependencies satisfied; one Coordinator assigned; model, environment, checkout, and retry limit selected |
| `implementing` | `reviewing` | Specific commit available; implementation report and required checks recorded |
| `reviewing` | `implementing` | Blocking findings require code changes |
| `reviewing` | `verifying` | Review covers the current commit; blocking findings resolved |
| `verifying` | `implementing` | Implementation fails an acceptance criterion |
| `verifying` | `delivering` | Required criteria pass against the current commit; Coordinator accepts the evidence |
| `delivering` | `implementing` | Delivery checks reveal a defect that needs code changes |
| `delivering` | `done` | Authorized delivery outcome confirmed and recorded |
| `done` | `delivering` | User authorizes further delivery of the same verified result, such as advancing a work draft |
| Any unfinished stage | `shaping` | Material scope or criteria change; dependent evidence is invalidated |
| Any unfinished stage | `cancelled` | Cancellation authorized; active runs stopped or recorded as still stopping |

Use sequential review then verification in the initial cycle. The Coordinator may
run independent checks in parallel when useful, but delivery still requires all
required reports against the same final change. A new commit requires an explicit
assessment of which evidence remains valid; rerun affected checks and record the
reason for any reused evidence. Do not treat old reports as current by default.

## Waiting is separate from stage

Set `waitingReason` to one of `none`, `waiting_for_user`,
`waiting_for_dependency`, `waiting_for_machine`, or `waiting_for_recovery`.
Preserve the stage, pending action, exact blocker, and required resolution.

When resolved, clear the waiting reason and resume the recorded stage. If the
answer changes scope, return to `shaping`. If it changes code, return to
`implementing`. A wait does not automatically create a new Coordinator.

Review findings and test failures normally lead to correction, not user approval.
Repeated failure, material ambiguity, or exceeded authority can require input.
The Coordinator and Verifier make this assessment using the available evidence.

## Delivery outcomes

| Classification and condition | Action and recorded result |
| --- | --- |
| `work`, verified and within scope | Open or update a draft PR; confirm draft state; finish with `draft_pr_delivered` |
| `work`, further action explicitly authorized | Perform only the authorized action; record its confirmed outcome |
| `personal`, verified with no unresolved decision | Open or update the PR, satisfy required host checks and reviews, and merge; finish with `merged` |
| Either, user decision needed to finish the task | Preserve useful work in a draft PR where permitted; retain the current stage with `waiting_for_user` |

A work draft delivered at its authorized boundary is complete for this cycle. A
personal draft awaiting a decision is not complete. Record PR state separately
from task stage. Missing classification uses the work authority limit.

Before merge, check the current PR head, target branch, required checks, review
state, merge conflicts, and scope. If the head changes, reassess evidence. Follow
the repository's update and merge rules. Never bypass protections or use an admin
override simply to complete a task. If checks are pending, use the runtime's
supported watch or wakeup mechanism and save the pending delivery action.

After merge, confirm the actual merged result. Opening a PR, enabling automatic
merge, or requesting a merge is not proof that it merged. If a post-merge defect
is found, create linked corrective work under the repository rules; do not
silently rewrite the completed task's history. Reporting in the Orchestrator
thread is tracked separately so an interrupted report cannot repeat delivery or
merge.
