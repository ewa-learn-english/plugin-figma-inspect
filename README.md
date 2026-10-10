# plugin-figma-inspect

Standalone Codex and Claude Code plugin for finding components across configured
Figma teams and inspecting implementation-ready contracts. The plugin is
host-agnostic and only invokes the separately installed CLI.

## Prerequisites

- Node.js 24 or newer
- `figma-inspect-cli` 0.9.0 or newer on `PATH`
- A Figma personal access token with access to the configured teams

```sh
npm install --global figma-inspect-cli@latest
export FIGMA_API_TOKEN="figd_..."
export FIGMA_TEAMS='{"design":"<team_id>","marketing":"<team_id>"}'
figma-inspect --preflight --json
```

Aliases in `FIGMA_TEAMS` are user-defined. `FIGMA_TEAM_ID` remains available as
a single-team fallback. Set `FIGMA_INDEX_ROOT` only when the default
`~/.figma-inspect-cli/indexes` location is unsuitable.

Never commit tokens to this repository or another project configuration file.

## Install in Codex

From this repository root:

```sh
codex plugin marketplace add .
codex plugin add plugin-figma-inspect@plugin-figma-inspect
```

## Install in Claude Code

From this repository root:

```sh
claude plugin marketplace add .
claude plugin install plugin-figma-inspect@plugin-figma-inspect
```

## Inspect a supplied URL

For an overview or actual property values, save the complete response and query
the relevant JSON fields:

```sh
figma-inspect --inspect-file-node --url "<figma-url>" --json > ./figma-node.json
```

For implementation contracts or a section index:

```sh
figma-inspect --export-contract --url "<figma-url>" --output-dir ./figma-contract
```

URL operations need only the token; they do not require team selection or an
index refresh. SECTION exports provide an index, separate child contracts, and
the complete original response. Add `--export-preview` for a requested image or
a visual question that node data cannot resolve. Use preflight to diagnose
access failures rather than before every inspection.

The shared `skills/figma-inspect` directory can also be consumed directly by a
host that installs external skills and prepares its CLI environment.

## Indexes and search

```sh
figma-inspect --index-status --json
figma-inspect --refresh-index --json
figma-inspect --search-components --name "Button" --json
```

One SQLite index is stored per team. Package/plugin reinstalls do not erase the
indexes because they live under `FIGMA_INDEX_ROOT`, outside both installations.
When a request needs an index that is missing, unfinished, or older than 24
hours, the plugin refreshes it; it does not run a scheduler or background
process. A refresh downloads only changed files, so it is cheap after the first
build.

Without `--team`, managed commands operate on all `FIGMA_TEAMS` entries. Use
`--team <alias>` to narrow an operation to one configured team.

## Layout

- `.codex-plugin/plugin.json` — Codex manifest
- `.claude-plugin/plugin.json` — Claude manifest
- `.agents/plugins/marketplace.json` — Codex marketplace entry
- `.claude-plugin/marketplace.json` — Claude marketplace entry
- `skills/figma-inspect/SKILL.md` — shared agent instructions

## License

MIT
