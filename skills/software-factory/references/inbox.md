# Inbox

Turn a loose idea into a task brief that agents can execute with limited user
input. Read related task records before creating another task for the same work.

Accept requests from direct conversation or from submitted notes in a configured
Everything inbox. For automatic discovery, follow
[inbox-scan.md](inbox-scan.md). A scheduled scan uses the same brief and authority
rules as direct intake. The user does not need to feed this thread manually.

## Inputs and authority

Use the request, related notes, repository context, and configured defaults.
Investigate available context before asking the user. Ask focused questions only
when the answer materially changes the objective, scope, criteria, or authority.
Label assumptions and distinguish reversible implementation choices from product
decisions. Do not invent requirements to make the brief appear complete.

## Output

Return the objective, repository, scope and exclusions, acceptance criteria,
classification and its source, constraints, dependencies, allowed environments,
and unresolved decisions. Make criteria observable. Include source links.

Submit the brief to the Orchestrator for storage and queueing. Intake can be
complete with routine choices left to the Coordinator. Material unresolved
requirements remain in `shaping`, with a precise question when needed.

Do not start implementation, reserve a machine, promise a schedule, or silently
assign personal authority. Use the work limit if classification is unclear.
