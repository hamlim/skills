---
name: new-workspace
description: Create a directory under a given base path, clone a list of repositories into it, and start a new T3 Code session with the user's goal. Use for a new workspace with one or more repos, not a worktree of the current repo.
---

# new-workspace

Create the workspace and start one new top-level T3 Code thread in it. Keep each
repo as a separate clone. Do the goal work in the new session.

## Read the request

Accept these inputs from natural language:

- **Base path:** The parent directory, such as `~/src`. Expand `~` and use an
  absolute path on the current machine.
- **Repos:** One or more remote URLs, `owner/repo` names, or short repo names.
- **Goal:** The prompt for the new session. Keep the user's wording, conditions,
  order of work, and requested PR states.

Use a short name from the goal for the new directory and thread title. Accept an
explicit name if supplied. Ask only for missing inputs or a repo identity that
cannot be resolved from available evidence.

Treat a repo URL as data, not shell code. For short names, check known project
roots and their Git remotes, or use authenticated repository search. A local
checkout can identify a remote; it is not the clone source. Do not assume that
the current GitHub account owns every repo. If several repos match, ask which
one to use. Check access to each resolved remote before creating the workspace.
Do not print credentials from remote URLs.

Invoking this skill authorizes workspace creation and a new session for the
goal. An example in a request to write or change this skill does not authorize
execution of that example.

## Prepare the workspace

1. Discover the live schemas for `t3_project_create`, `t3_project_read`,
   `t3_project_list`, `t3_thread_launch`, and `t3_thread_read`. Check that they can
   register a folder and bind a new thread to it. If the required tools are not
   available, stop before cloning and report the missing capability. Do not
   substitute a different agent or a thread in the current project.
2. Select a distinct child directory under the base path. Use a goal name with a
   unique suffix, for example `~/src/audit-bend-imports-<unique-suffix>`. Create
   the base path if needed. Create the child directory exclusively; never reuse
   an existing directory. Pass paths as quoted arguments or through an argument
   array. Do not insert the goal into a shell command.
3. Clone each resolved remote into a separate child directory with `git clone`.
   Use a fresh, full clone of the remote default branch unless the user requests
   another ref. Do not copy a local checkout or use a Git worktree, shared object
   store, or shallow clone. If repo names collide, use distinct names such as
   `<owner>-<repo>` and record the mapping.
4. Check each clone's top-level path, origin, branch, HEAD commit, and clean
   working tree. If a clone fails, stop before session launch. Report the failed
   repo and partial workspace path. Keep partial files for recovery; do not
   delete the workspace automatically.
5. Write `WORKSPACE.md` in the parent directory. Include the original goal,
   repo-to-directory map, remote identities without credentials, and starting
   branches and commits. Keep launch state in `.new-workspace.json`: absolute
   workspace path, a new `clientRequestId`, and project/thread IDs and link as
   they become available. Generate the request ID once before launch.

The parent directory is a container for the clones. Do not initialize it as a
Git repo or add the clones as submodules. Do not install dependencies or execute
the goal before the new session starts.

## Start the T3 Code session

Register the **parent directory** with `t3_project_create`. Set `title` to the
goal name and `workspaceRoot` to the absolute parent path. Supply the path
explicitly: omitting it can create a different repo. Read the returned project
with `t3_project_read` and check its workspace root before launch. Save its ID.

Call `t3_thread_launch` with:

- The new `projectId`, a short `title`, and the saved `clientRequestId`.
- `workspaceStrategy: { "type": "root" }`, to bind the session to the parent
  directory that contains all the clones.
- A `message` with the full original goal, the absolute workspace path, and the
  repo-to-directory map. Tell the session to read `WORKSPACE.md` and each repo's
  applicable instructions before edits, and to run Git commands in the intended
  clone. Preserve the requested order and PR states. Do not add merge or deploy
  authority.

Keep inherited model and permission settings unless the user specifies others.
Do not use `scratch`, `create_threads`, `delegate_task`, or a worktree strategy
for this launch.

Read the returned thread with `t3_thread_read`. Check that its project ID is the
new project's ID and that its state does not show launch failure. Save the
thread ID and returned link in `.new-workspace.json`. Report the workspace path,
clone names, and the thread's `link` verbatim. Distinguish session start from
completion of the goal. There is no need to wait for the goal work to finish.

## Recover from an uncertain result

Keep the workspace and launch state if registration or launch fails. If a
project-create reply is lost, list projects and match the exact absolute
workspace root before another create call. Do not repoint an unrelated project.
If a launch reply is lost, retry with the **same** `clientRequestId` and launch
arguments. Do not generate a second request ID for that workspace. If recovery
cannot establish the result, report the saved path and unresolved launch state.

## Example

```text
$new-workspace ~/src include bend-hubspot and nbd repos, the goal is to audit
current imports from Bend packages within nbd, see if they're all exported from
the bend-amarillo package and if not open a PR in Bend to re-export those things
from the bend-amarillo package and then prepare a draft PR in the nbd repo
updating all imports/dependencies to depend only on bend-amarillo
```

Resolve the two remote identities first. Create a parent directory under
`~/src` with both clones. Start one session at that parent directory with the
complete goal. The new session must preserve the Bend PR work before the nbd
draft PR work.
