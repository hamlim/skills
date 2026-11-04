---
name: software-factory
description: Run a software factory in T3 Code with an Orchestrator that receives requests, manages schedules, coordinates implementation, review and verification, and reports delivery. Use for factory role assignments and task cycles, not ordinary coding or PR work outside the factory.
---

# software-factory

The software factory is named **Codswallop**.

Take sufficiently defined tasks through implementation, review, verification,
and authorized delivery. Resolve routine decisions without user input. Keep task
state outside the conversation so another run can recover it.

## Select a role

For existing work, read the task record before acting. Use the role assigned by
the caller or stored in the record. Use Orchestrator for factory requests without
an assigned role, including new ideas, user replies, scheduling, and recovery.
The user submits work and receives results in the Orchestrator thread. Do not
create or use separate Inbox or Outbox roles or pinned threads. Do not assume
control of an existing task that has another owner.

Read [task-record.md](references/task-record.md) and
[transitions.md](references/transitions.md), then only the relevant role reference:

| Role | Read when | Responsibility |
| --- | --- | --- |
| [Orchestrator](references/orchestrator.md) | Receiving requests, reporting results, or managing the factory | Define briefs; own schedules, assignments, state, and user decisions |
| [Coordinator](references/coordinator.md) | Managing one task cycle | Direct work and decide its next action |
| [Implementer](references/implementer.md) | Producing or correcting a change | Return code, tests, and a specific commit |
| [Reviewer](references/reviewer.md) | Independently evaluating a change | Return findings against a specific commit |
| [Verifier](references/verifier.md) | Checking acceptance criteria | Return observed behavior and acceptance evidence |

Roles are responsibilities, not five permanent processes. Use separate role
threads when independent judgment or context is useful. Do not start extra agents
for checks that a command can perform. This skill authorizes task-scoped role
delegation within the user's existing scope; it does not authorize unrelated work.

For a scheduled scan of an Everything inbox, also read
[inbox-scan.md](references/inbox-scan.md). Scanning is an optional Orchestrator
mode. Direct conversation is the normal intake path. The Orchestrator owns scan
configuration, task creation, duplicate checks, acknowledgement, and scheduling.

Before selecting or launching a role model, read
[model-selection.md](references/model-selection.md). It defines role defaults,
task and run overrides, effort options, and capability checks. The bundled
[model-defaults.json](references/model-defaults.json) supplies the initial role
preferences. Repository-local `.codswallop/model-overrides.json` files override only
the specified settings without changing shared repository instructions. Neither
file changes the model of an already running thread by itself.

## Shared rules

- Follow the task brief, repository instructions, and runtime permission limits.
  Use required repository task systems, such as Dodo, and link their task IDs.
  Do not replace those systems with a second backlog.
- Use a dedicated Everything factory vault as the intended task store. Discover
  its current schema and API before using it. A vault is not a lock. Do not claim
  conflict protection, notifications, or atomic updates without verification.
- The Orchestrator is the only writer of task ownership, stage, waiting state,
  and outcome. Other roles return structured results to it through the
  Coordinator when one is assigned, or directly otherwise. The Orchestrator
  records intake and reports results in its own thread. Do not
  infer a saved transition from a chat message.
- Record facts and evidence separately from plans and assumptions. A completed
  model turn is not proof of completed work. Inspect actual Git, PR, and check
  state before reporting delivery.
- Let the Orchestrator select machines from available capabilities and task
  restrictions. There is no fixed default machine. Do not send code, context,
  secrets, or artifacts to a machine outside the task's allowed environments.
- Skills supply instructions. T3 supplies execution and wakeups. Neither a skill
  nor an idle thread is a continuously running scheduler. Set up scheduled work
  only when requested; do not invent a polling daemon.

## Authority and acceptance

`work` tasks may reach a draft PR without further input. Do not mark that PR ready
for review, enable automatic merge, merge it, or otherwise advance it beyond draft
without explicit user authorization for that task. Record the authorization.

`personal` tasks may reach merge into the target branch when the required checks
pass, review findings are resolved, and the Coordinator and Verifier identify no
unresolved decision that needs the user. They may instead wait at a draft PR.
Permission to merge is not permission to bypass branch protection or deploy.

Use repository classification defaults when they are configured. Missing or
conflicting classification uses the `work` authority limit until resolved. Do not
change a task to `personal` to bypass a limit. Store explicit task permissions
separately from classification, including any deployment authority.

The Coordinator and Verifier assess the need for user input for each task. First
investigate uncertainty or run a useful additional check. Ask when a material
requirement remains unresolved, evidence conflicts, an action exceeds authority,
or the task cannot progress within its agreed scope or retry limit. Do not weaken
criteria, conceal failures, or ask for routine acceptance merely because a stage
has ended.

## Begin or resume

1. Read the task, its revision, owner, last events, and any pending action. Check
   the actual environment before starting work.
2. Identify your role, allowed action, and expected output. Resolve material
   ambiguity through the Coordinator rather than silently changing the brief.
3. Perform the bounded assignment. Report evidence, commit identity, findings,
   unresolved decisions, and the recommended next action.
4. Have the Orchestrator persist and read back the result before advancing.
   Follow the recovery rules if the save or external action has an uncertain
   outcome. Never start a second owner solely because the first is unreachable.
