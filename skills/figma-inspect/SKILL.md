---
name: figma-inspect
description: Find component sets and standalone components by name across configured Figma teams, inspect Figma nodes and component properties, and export compact implementation contracts. Use when a task references Figma, a Figma URL, a design-system component, visual specs, layouts, variants, or implementation from an existing design.
---

# Figma Inspect

Use the installed `figma-inspect` CLI directly. The workflow is host-agnostic
and has no nested-agent or MCP dependency.

## Preflight

Before the first API-backed operation in a session, verify the CLI and access:

```sh
command -v figma-inspect
figma-inspect --version
figma-inspect --preflight --json
```

Version `0.7.0` or newer is required. If the executable is missing or older,
ask the user to install or update it with:

```sh
npm install --global figma-inspect-cli@latest
```

The environment must provide `FIGMA_API_TOKEN` and normally `FIGMA_TEAMS`, a
JSON object of user-defined aliases to team ids. `FIGMA_TEAM_ID` is supported
as a legacy single-team fallback. `FIGMA_INDEX_ROOT` is optional.

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

Use a result URL directly. For raw node data:

```sh
figma-inspect --inspect-file-node --url "<figma-url>" --json
figma-inspect --inspect-component-set --url "<component-set-url>" --json
```

For implementation work, prefer a compact contract in a task-owned temporary
directory. Pass the search result's team alias for component sets when more
than one team is configured:

```sh
output_dir=$(mktemp -d)
figma-inspect --export-contract --output-dir "$output_dir" --team "<alias>" --url "<figma-url>"
```

`--variables <path>` is optional. Without it, contracts remain usable but
variable aliases are unresolved and the CLI prints a warning. Read the generated
meta, geometry, visuals, structure DSL, and layout-risk files relevant to the
task; do not dump raw Figma payloads into context when a contract is available.

If the user supplies a Figma URL, skip index discovery unless the URL is
insufficient or related component search is requested.

## Rules

- Treat Figma as read-only; the CLI does not modify Figma documents.
- Never expose `FIGMA_API_TOKEN` in output, logs, files, or error reports.
- Do not refresh an index that is present and at most 24 hours old unless the
  user explicitly requests fresh data.
- Use `--team` only with a configured alias, never a raw team name guessed from
  natural language.
- Report the selected Figma URL and any unresolved ambiguity with the result.
