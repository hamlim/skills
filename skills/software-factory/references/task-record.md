# Task record

Use one stable task ID for the full cycle, including retries and machine changes.
Map these logical fields to Everything's discovered schema. Do not assume that a
property, trait, vault, or API exists. Keep credentials out of records and reports.

## Required fields

| Group | Fields |
| --- | --- |
| Identity | ID, title, source request, repository, repository task ID if required |
| Intake | Source vault and note IDs, source revision or content hash, stable intake key, submission provenance, source-to-task link |
| Brief | Objective, scope, exclusions, acceptance criteria with stable IDs, constraints, unresolved assumptions |
| Authority | `work` or `personal`, classification source, target branch, allowed environments, explicit user authorizations |
| State | Stage, waiting reason, outcome, revision, created and updated timestamps |
| Ownership | Orchestrator thread, Coordinator thread, assignment generation, assigned environment, current role runs |
| Evidence | Current commit, base commit, branch, worktree, PR URL, review and verification reports |
| Recovery | Pending action, stable action ID, attempt count, retry limit, last known external status |
| Decisions | Question ID, exact question, decision owner, answer, authority source, decision timestamp |
| Delivery | Delivery event ID, reported revision, destination thread, delivery status |

Each role run records its role, environment ID, thread link and ID, run or task ID,
assignment generation, reviewed commit where applicable, status, and result link.
Record model and provider selection when available. Do not assume identifiers
are global across environments.

## Inbox scan configuration

Keep one factory configuration record with the source vault IDs, exact selection
predicate, allowed submitters or integrations, repository and classification
mapping, scan batch limit, pagination or checkpoint strategy, overlap policy,
acknowledgement method, and Inbox and Orchestrator destinations. A trait and tags
can define the predicate once their actual schema is discovered. Do not select
notes merely because their title contains the word inbox.

Record the configured schedule ID and interval only after setup is authorized
and confirmed. The Orchestrator owns scan state: current scan ID, progress,
pending intake actions, source-to-task mapping, and last durable checkpoint.
See [inbox-scan.md](inbox-scan.md) for processing and recovery rules.

Choose a finite retry limit when scheduling the task. A useful initial default is
three correction cycles, with a recorded reason for any change. A transient
transport retry is separate from another implementation cycle. Pause for a useful
decision after repeated identical failures; do not create unbounded agent loops.

## Evidence report

Every report identifies the task ID, assignment generation, role, input commit,
observations, findings, unresolved questions, and recommended next action. Review
and verification reports must say which commit they cover. Verification lists
each acceptance criterion as `pass`, `fail`, or `blocked`, with supporting output
or artifact links. Redact secret values from evidence.

## Updates and history

One Orchestrator owns state writes for the vault in the initial version. Other
roles send results; they do not perform read-modify-write updates to task state.
Do not start a second Orchestrator for the same vault without an explicit transfer.
Human edits are possible, so re-read before saving and reconcile unexpected edits.

For every accepted change, append an event containing event ID, task ID, previous
and new revision, actor, timestamp, changed fields, reason, and evidence links.
Keep existing events intact. Record the pending action before issuing a remote
mutation and record its confirmed result afterward.

Use conditional updates when the API supports them. If it does not, single-writer
operation is a convention, not a guarantee. Do not automate competing claims or
failover. If event and state writes cannot be atomic, keep a pending event marker,
write both with stable IDs, read them back, and reconcile a partial write before
issuing more work. Stop dispatch if consistency cannot be established.

If Everything is unavailable or its write result is uncertain, report the limit
and stop new state-changing dispatch. Already running agents may return results;
do not discard them or treat them as saved. Do not create an alternative task
store silently. Store adapter details in configured factory context once verified.

## Recovery

1. Read the saved pending action and inspect T3, Git, or the PR for its actual
   result. Reuse a retry key only where the API supports it. Do not assume every
   command is idempotent.
2. If the action succeeded, record the result without repeating it. If it failed
   before taking effect, retry within the limit. If uncertain, wait and reconcile.
3. Loss of contact does not prove a run stopped. Before replacement, establish
   that the previous run ended or was stopped. If this cannot be established,
   record `waiting_for_machine` or `waiting_for_recovery`.
4. After a confirmed transfer, increase the assignment generation. Keep late
   results from earlier generations as history; they cannot advance the task.
5. To move execution, preserve and push an authorized code checkpoint, record
   the commit and remaining work, fetch it on the target, and start a new run.
   A thread link does not transfer local files, ignored data, or a live process.

Link required repository records such as Dodo. They retain their repository
lifecycle rules. Factory records track execution, evidence, and cross-machine
recovery; do not copy the repository backlog into the factory vault.
