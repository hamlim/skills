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
3. Configured repository role default.
4. Configured factory role default.
5. Bundled role preference.

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
