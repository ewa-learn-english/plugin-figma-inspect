# plugin-figma-inspect

Standalone Codex and Claude Code plugin for finding components across configured
Figma teams and inspecting implementation-ready contracts. The plugin is
host-agnostic and only invokes the separately installed CLI.

## Prerequisites

- Node.js 24 or newer
- `figma-inspect-cli` 0.7.0 or newer on `PATH`
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

## Indexes and search

```sh
figma-inspect --index-status --json
figma-inspect --refresh-index --json
figma-inspect --search-components --name "Button" --json
```

One SQLite index is stored per team. Package/plugin reinstalls do not erase the
indexes because they live under `FIGMA_INDEX_ROOT`, outside both installations.
The plugin refreshes missing indexes and indexes older than 24 hours when a
request needs them; it does not run a scheduler or background process.

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
