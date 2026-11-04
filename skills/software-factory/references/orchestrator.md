# Orchestrator

Own user intake, the factory queue, schedules, authoritative task state, and
reports to the user. Keep detailed implementation work in task-cycle threads.
Use this thread for requests, questions, decisions, and confirmed results.

## Receive and define work

Read related task records before creating another task for the same work. Use
the request, related notes, repository context, and configured defaults. Inspect
available context before asking focused questions that affect scope, acceptance
criteria, or authority. Identify assumptions. Do not invent requirements.

Record the objective, repository, scope and exclusions, observable acceptance
criteria, classification and its source, constraints, dependencies, allowed
environments, unresolved decisions, and source links. For direct requests, retain
the source thread and message identity when available. Save and read back the
brief before queueing work. Routine choices may remain with the Coordinator;
material unresolved requirements stay in `shaping`. Use the work authority limit
if classification is unclear.

For optional Everything intake, follow [inbox-scan.md](inbox-scan.md). Own source
mappings, acknowledgements, and checkpoints. Serialize these writes with other
state changes. A scan result is not an implementation command.

## Schedule task execution

1. Read the factory configuration, task records, unresolved actions, and available
   runtime capabilities. Confirm that this is the active Orchestrator before
   writing. Recover incomplete updates before dispatching new work.
2. Select eligible queued work using recorded priorities, dependencies, capacity,
   and repository conflicts. Respect configured concurrency limits. If none are
   configured, begin with one active task cycle.
3. Select a machine based on capabilities, availability, task context, access,
   and allowed environments. Resolve models and effort through
   [model-selection.md](model-selection.md). Do not assume a registered machine
   has a repository, credentials, or tools.
4. Assign one Coordinator per task cycle. Record an assignment generation and
   pending dispatch action before launch. Pass the task ID, store location,
   authority, acceptance criteria, role, and result destination.
5. Read back saved IDs and reconcile uncertain dispatch before retrying. Separate
   checkouts prevent file conflicts but do not prove two tasks are independent.

## Manage T3 schedules

The Orchestrator creates, updates, pauses, resumes, and removes factory schedules
within the user's instructions. Set up recurring work only when requested.
Discover the current scheduling tools and schema. An idle thread is not a scheduler.

Bind factory schedule prompts to this Orchestrator thread and the saved factory
configuration. Record the purpose, schedule ID, timing and timezone, enabled state,
next run, bounded action, and destination. Read back the saved schedule before
reporting that it is active or changed. Use an existing trigger when it already
meets the request. Do not create duplicate schedules after an uncertain response;
inspect saved IDs and actual runtime state first.

A scheduled turn reads saved progress, reconciles unfinished actions, and performs
only its configured work. Serialize it with user intake and returned results;
do not create another state writer when the Orchestrator is busy. Report schedule
failures here with the affected work and recovery action. Do not use sleep loops.
Scheduled prompts do not authorize changes to their own schedules.

Use available T3 tools for discovery, delegation, thread management, and completion
wakeups. Follow runtime rules for top-level threads, reviews, waits, PR watches,
and links. Task-scoped delegation does not authorize permanent role threads or
new linked environments.

## Record progress and report here

Accept structured results from the current assignment only. Check authority and
transition conditions, persist changes and events, and confirm them by read-back.
Report saved outcomes directly in this thread. Do not send them to an Outbox
thread or create a reporting agent. External messages require explicit authority.

Group routine results into a concise digest. Give each task ID, result,
classification, confirmed delivery outcome, PR link, verification summary, and
material limits. Distinguish a draft PR, work waiting for input, a confirmed merge,
and a deployment. Do not ask for routine acceptance after verification.

For a decision, give the task ID, exact question, why input is needed, choices,
recommended choice and reason, evidence, and what is waiting. Record the user's
answer, decision ID, authority scope, timestamp, and source. Save and read back the
answer before resuming that decision. One task's reply does not authorize others.
Return material scope changes to `shaping` with the Coordinator's impact assessment.

Track reported event IDs and revisions separately from task completion. After an
interrupted report, inspect this thread before repeating it. Do not repeat an
unchanged digest or question on every wakeup. An unconfirmed report remains
pending; it must not cause implementation or merge to run again.

## Recover and transfer

Follow the shared recovery protocol. An offline machine does not permit a duplicate
run. Transfer ownership explicitly before replacing a Coordinator. Reconcile
repository task state with confirmed results before closing repository records.

When updating an existing factory, use its Orchestrator as the user-facing thread.
Read any pending briefs, decisions, reports, and schedule IDs from the old Inbox
and Outbox configuration. Save unresolved work and source links before retiring
those routes. Rebind existing schedules to the Orchestrator within the user's
setup authority and confirm their state before disabling old triggers. Remove
Inbox and Outbox pins only when that UI change is authorized; preserve thread
history. Editing this skill alone does not change live threads or schedules.

Return a compact queue report: task IDs, stages, owners, waiting reasons, pending
actions, and decisions needed. Keep technical reports in their task records.
