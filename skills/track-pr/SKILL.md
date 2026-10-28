---
name: track-pr
description: Track a pull request in Everything by creating a page with the pull request trait and review properties. Use when asked to add or track a PR in Everything.
---

# track-pr

Before making any changes, check for an available and usable Everything MCP. If it
is not available, check whether Executor MCP provides access to Everything. Verify
the connection and required capabilities with read-only calls. If neither route is
available or can perform the required work, stop and tell the user what is missing.
Do not use a browser, CLI, or another service as a fallback for Everything.

Identify the PR from the request or current context. Get its current title, URL,
repository name, and PR number from a reliable source. Ask for the PR if it cannot
be identified. Use the live
Everything schema to find the intended vault, pull request trait, properties, status,
tag, and daily note. Search for an existing entry for the PR before creating one; if
one exists, report it instead of creating a duplicate.

Use the current date and time in `America/New_York`, including its daylight saving
offset. Create the entry with:

- The PR title as the page title and the pull request trait applied.
- An alias in the format `PR-<repo-slug>--<pr-number>`, such as `PR-projects--1500`.
- The PR URL in the trait's corresponding property, when supported.
- Created set to the current Eastern date and time, in the format the property requires.
- Day created linked to the daily note for the current Eastern date. Use the
  periodic-note capability to find or create that note if needed.
- Status set to `in review` and the `work` tag applied.

Use the actual property types and options returned by Everything; do not invent IDs
or silently omit a requested property. If a required property or relationship is
not supported, stop before creating the entry and explain the limitation. If a
write has an uncertain outcome, check the existing data before retrying. Read back
the created entry to verify its properties, then return its link and report any
failure or partial result accurately.
