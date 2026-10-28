---
name: make-pr
description: Create a pull request from existing work. Derive a clear title and a description that explains the intended result, reason for the work, main changes, verification, and follow-up work. Use when asked to open or prepare a PR, not to implement a new feature or review a PR.
---

# make-pr

Create a pull request (PR) that a reviewer can understand without the chat history.
Use the existing work as the source of truth. Make the intended result clear before
you explain the implementation.

## Inspect the work

1. Read the repository instructions and PR template. Check the current branch,
   working tree, remote, and any existing PR for this branch.
2. Identify the correct base branch from the user request, repository rules, or
   branch history. Inspect the full proposed diff against that base, including
   committed and uncommitted work. Do not use commit messages alone as evidence.
3. Read the changed code and relevant tests or documents. Determine who benefits,
   what they can do after the change, and why the work is needed. Use issues or
   specifications when available. Distinguish stated requirements from assumptions.
4. Check that the proposed PR contains the intended work. Preserve unrelated local
   changes. Ask a focused question only if an unresolved choice changes the scope,
   base branch, or intended result.

## Derive the title

Use a short, specific title that states the main result or behavior change.
Use an active verb where practical. Follow the repository's title conventions.
Use an issue prefix or a conventional commit prefix only when required or useful.

For example, prefer `Preserve callout content when changing its type` to
`Update editor components`. For internal work, name the concrete technical result
and its value. Do not invent a user-facing feature for maintenance work.

Rewrite the title if the final scope differs from the original plan.

## Write the description

Use the repository's PR template when present. Keep its required fields and cover
the subjects below in the appropriate sections. Otherwise, use this structure.
Replace each instruction with specific facts before you submit the PR.

```markdown
## Goal

State the intended result after this PR. Explain the experience of the user or
developer: what they can now do, what works better, or what this change enables.
For a fix, describe the trigger and the behavior before and after the change.

## Why

Explain the problem or need that makes this work useful. State what it enables
and include relevant issue or specification links. Do not repeat the goal.

## Changes

Describe the main changes at a high level. Group them by behavior or purpose.
Explain design choices only when they help the reviewer assess the change.
Briefly explain material trade-offs and alternative approaches that were
considered, including why the chosen approach fits. Do not invent alternatives.

## Verification

State what was tested or checked, how it was checked, and the result. Include
material limits, failed checks, and checks that were not run, with reasons.

## Additional notes

Include known limits, rollout or migration needs, and planned follow-up work.
Link existing follow-up issues when available. Omit this section if it is empty.
```

Keep every PR description concise, useful, and easy to read. Use simple language
and include only the detail a reviewer needs to understand and assess the change.
A small PR needs only a few precise sentences. Add context for larger changes
only where it helps review. Omit repetition, chat history, abandoned plans, and
a file-by-file account unless they explain a material decision.

Support claims with the diff or other evidence. Do not claim that planned work
is complete, that passing local checks prove deployment, or that a benefit was
measured without results. State material uncertainty directly.

## Verify and create the PR

1. Run the repository's required checks and the checks appropriate to the change.
   Use existing results only when they apply to the current work. Record actual
   commands, outcomes, and any manual checks. Do not invent successful results.
2. Review the final diff, title, and description together. Confirm that the PR
   covers the intended scope and that each major claim has evidence. If the diff
   changes after verification, repeat the affected checks.
3. If asked only to prepare or draft text, return the title and description
   without creating a PR. If asked to create a PR, use the `gh` CLI. Default to
   `gh pr create --draft` unless the user asks for a PR that is ready for review.
   Create a branch, commit the intended changes, and push as needed within
   the user's authorized scope. Do not force-push, include unrelated changes, or
   bypass repository checks. Resolve an existing PR for this branch before
   creating a duplicate; update it when that matches the request.
4. Write the description to a file and pass it with `--body-file` to preserve real
   newlines and literal characters. If a required check cannot pass, report the
   cause and keep the PR in draft. Do not mark the work ready for
   review while a required check remains unresolved.
5. Read back the saved PR to verify the title, description, base branch, and head
   branch. Follow any environment rules for PR links. Return the PR URL and any
   material verification limits. Distinguish PR creation from checks, approval,
   merge, and release. Stop after the requested PR work; do not merge it unless
   the user asks.
