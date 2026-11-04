# Role models and effort

Use [model-defaults.json](model-defaults.json) as the initial preference policy.
Inbox uses Sonnet 5.5 with medium effort. Implementer uses Opus 5.5 with high
effort. Reviewer and Verifier use Sol 6.1 with high reasoning effort. These effort
settings are initial factory choices, not claims about the providers' defaults.
Orchestrator, Coordinator, and Outbox inherit their parent selection unless
configured otherwise. A scheduled inbox scan uses the Inbox preference too.

These are configuration preferences. This file does not configure live threads,
create schedules, reserve models, or guarantee availability on another machine.

## Resolve a selection

Read preferences in this order, with the highest priority first:

1. Explicit user selection for this role run.
2. Saved task role override.
3. Repository-local `.codswallop/model-overrides.json`.
4. Configured factory role default.
5. Bundled role preference.

## Repository-local overrides

Read `.codswallop/model-overrides.json` at the target repository root, not relative
to the Orchestrator's current directory. Find that root through Git. Read only
this one repository file; do not search parent folders or merge nested files.
If no repository is selected yet, use factory and bundled defaults until it is.

The file uses the bundled `version` and `roles` structure, but lists only the
roles and fields that need changes. For example:

```json
{
  "version": 1,
  "roles": {
    "implementer": {
      "options": { "effort": "max" }
    },
    "reviewer": {
      "options": { "reasoningEffort": "xhigh" }
    }
  }
}
```

With the bundled policy, this keeps Opus 5.5 for implementation and Sol 6.1 for
review. It changes only their effort. All other roles retain their defaults.
Apply the merge rules below; a generic recursive JSON merge is not sufficient
when changing models or selecting inheritance.

Missing file means no repository override. Invalid JSON, unsupported versions,
unknown roles or fields, and invalid selection combinations must produce a
configuration error. Do not silently ignore a broken local preference. Resolve
model availability and options using the current target environment catalog.

### Keep it local

Creating a `.codswallop` folder does not make it private or ignored. During local
setup, add `/.codswallop/` to this repository's local Git exclude file, whose path
is returned by `git rev-parse --git-path info/exclude`. Preserve existing entries
and avoid adding the same pattern twice. Do not edit the shared `.gitignore` or
`AGENTS.md` for personal model preferences.

Verify the override is untracked and ignored before calling it local-only. Local
excludes do not hide a tracked file. If it is already tracked, report that fact
and resolve the intended setup rather than removing it from the index silently.
Do not commit or include the local file in PRs. Keep credentials out of it.

The file is local to its checkout and is not fetched with Git. For a new worktree
or another execution machine, do not assume the file exists there. Resolve the
policy from the user's selected source checkout at assignment time, record its
path, content hash, and effective settings in the task, and pass those settings
explicitly to role runs. They remain stable for that assignment. Do not replace
them with destination-machine preferences during a transfer without a recorded
policy change. Local setup on each independent source checkout remains explicit;
do not copy or publish the file automatically.

## Merge and authority rules

The Orchestrator records policy changes and their source. The Coordinator can
apply configured choices but cannot silently change policy, increase effort, or
switch providers after a failure. A user can authorize a bounded escalation or
fallback policy in advance. Without one, report the unavailable choice and retain
the task for resolution instead of substituting a model.

Resolve fields from the lowest layer upward. Keep a lower layer's model when an
override changes only effort for that same model. Merge options by option ID only
when the driver and model remain unchanged. A new driver or model clears earlier
options and provider-instance bindings; apply only options explicitly supplied
with the new selection. Do not carry Claude effort or fast-mode settings into
Codex, or Codex reasoning and service-tier settings into Claude.

`mode: inherit` replaces the lower selection with the launching parent's confirmed
model and options. It cannot be combined with explicit model or options in the
same preference. For a top-level role with no parent, resolve the configured T3
project defaults. If those cannot be established, report the missing selection.
Resolve inheritance to concrete settings before recording and launching the run.

An empty `options` object explicitly resets prior options; it is not an instruction
to inherit them. Determine the selected model's provider defaults from current
capabilities, pass the runtime's explicit clear/reset value, and verify the result.
Never assume omitting options clears inherited settings.

## Validate on the selected machine

Use current T3 capability discovery under the runtime's discovery rules. Resolve
the preference's `driverKind` to an enabled provider instance in that environment;
provider-instance IDs are not portable machine identifiers. If multiple instances
match, use an explicitly configured instance or report the ambiguous selection.
Search the catalog when the initial capability response omits the requested model.

Check the exact model ID, supported option IDs and values, delegation support,
and permission constraints. The current option names differ: Claude uses
`effort`; Codex uses `reasoningEffort`. Do not translate one by guessing. Model
names and available effort values can change, so use the live catalog as evidence.

When using `delegate_task`, place the resolved provider instance, model, and
options in `target`, with effort in `target.options`. Use the appropriate
model-selection structure for other runtime tools. Read the current tool schema.
Choose the runtime's cross-provider delegation mechanism when required; native
delegation must not silently substitute a provider or omit requested settings.

Record the requested selection before dispatch. Read back the actual model and
options from the launch result or thread configuration. If they differ, report
the mismatch and reconcile before accepting that run's evidence. A higher model
or effort setting does not grant more task authority.

## Existing threads and scheduled roles

Do not claim that mentioning a model in a prompt changes the runtime selection.
An existing role thread keeps its configuration until an authorized runtime
configuration change is applied and confirmed. Do not reconfigure an active turn
mid-run. Apply a requested change at an idle boundary or to a fresh authorized run.

For a scheduled role, ensure the destination thread or schedule's supported model
configuration has the resolved role selection. If a schedule cannot select a
model directly, configure and verify its bound role thread during authorized
setup. Do not claim that saving the preference alone changed scheduled execution.

## Examples

An explicit request for the Implementer to use Opus 5.5 at `max` overrides its
bundled `high` effort for that run. It does not change future implementation runs.

A task override for Reviewer with `options: { reasoningEffort: "xhigh" }` keeps
Sol 6.1 unless a higher-priority preference selects another model. Record it in
the task and apply it to subsequent review runs for that task.

A factory default for Outbox with Sonnet 5.5 and `effort: low` replaces inheritance
for future Outbox runs. It does not reconfigure the existing Outbox thread until
the runtime change is applied and read back.
