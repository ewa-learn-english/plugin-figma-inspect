---
name: figma-inspect
description: Use when a task references Figma, a Figma URL, a design-system component, visual specs, layouts, variants, or implementation from an existing design. Find components by name across configured teams and export compact node or section contracts.
---

# Figma Inspect

Use the installed `figma-inspect` CLI directly. The workflow is host-agnostic
and has no nested-agent or MCP dependency.

## Runtime access

Use `figma-inspect` directly when it is installed and the environment is already
prepared. Version `0.8.0` or newer is required. `FIGMA_API_TOKEN` provides access;
`FIGMA_TEAMS` is needed for team discovery, not exports by URL. `FIGMA_INDEX_ROOT`
is optional. Do not prepare credentials again when the host has provided them.

For a supplied Figma URL, start with the inspection below. If a command fails because
of installation or access, diagnose with `command -v figma-inspect`,
`figma-inspect --version`, and `figma-inspect --preflight --json`. For a known
team, pass its configured alias to preflight. Do not check every team before
inspecting a known URL. Missing or old installations need
`npm install --global figma-inspect-cli@latest` through the host's normal setup.

Do not print environment values or place the token on a command line. Preflight
output is safe to report. A `failed` result means credentials or team access
must be fixed. An `unavailable` result means Figma could not be reached; local
index operations may still be used.

## Find a component

For name-based discovery, use the persistent indexes:

1. Run `figma-inspect --index-status --json`.
2. Refresh an index when it is missing or `ageSeconds` is greater than `86400`.
3. Search by the user's component name or the most specific known substring.
4. Present ambiguous matches with their team, project, file, type, and URL
   before choosing one.

```sh
figma-inspect --index-status --json
figma-inspect --refresh-index --json
figma-inspect --search-components --name "Button" --json
```

With `FIGMA_TEAMS`, omitting `--team` searches or refreshes every configured
team. If the prompt, attached context, or Canvas names a team that exactly maps
to a configured alias, pass `--team <alias>`. Otherwise search all teams; do not
guess an id or invent aliases.

Indexes live outside the plugin under `FIGMA_INDEX_ROOT`, defaulting to
`~/.figma-inspect-cli/indexes`, so plugin and npm reinstalls do not erase them.
There is no background refresh. If refresh fails but a stale index exists, use
the stale index only when it can still answer the request and state its age.

## Inspect or export a match

For a supplied URL, skip index discovery. Choose the representation the task
needs, and reuse the saved response for follow-up questions.

For a description or questions about structure, dimensions, colors, or
typography, save the node response and query the requested fields with Python 3
or Node.js JSON parsing. This includes actual values alongside variable bindings
and the contents of every screen in a section. Keep the complete file; send only
the fields relevant to the question to model context.

```sh
figma-inspect --inspect-file-node --url "<figma-url>" --json > ./figma-node.json
```

For implementation contracts or a section index, export into a task-owned
directory. Use a separate directory for each source URL. URL exports need no
team selection.

```sh
figma-inspect --export-contract --output-dir ./figma-contract --url "<figma-url>"
```

Read `section.index.yaml` first for SECTION exports. It links to separate child
contracts and `section.source.json`, the complete original response. Child
contracts can be large for deeply nested screens; inspect the requested
properties rather than concatenating every file. Python 3 or Node.js can query
the JSON source directly, including annotations and actual typography values.
Use child meta, geometry, visuals, structure DSL, and layout-risk files when
implementing the selected screen or component.

`--variables <path>` is optional. Without it, contracts preserve unresolved
variable aliases and the CLI warns; use source values when the task needs
actual numbers rather than token names.

Add `--export-preview` when the user requests an image or the visual question
cannot be resolved from node data. Open the resulting image and share it through
the host's ordinary file tools when requested. A property overview normally needs
only the structured data.

## Rules

- Treat Figma as read-only; the CLI does not modify Figma documents.
- Never expose `FIGMA_API_TOKEN` in output, logs, files, or error reports.
- Do not refresh an index that is present and at most 24 hours old unless the
  user explicitly requests fresh data.
- Use `--team` only with a configured alias, never a raw team name guessed from
  natural language.
- Report the selected Figma URL and any unresolved ambiguity with the result.
