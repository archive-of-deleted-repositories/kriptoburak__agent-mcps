# agent-mcps

Canonical MCP server registry for the Oriz fleet. **Single source of truth** — every MCP server used by any Oriz agent lives here.

## What this is

`mcps.json` is a machine-readable registry of MCP servers, each tagged with:

- **Command + args + env** — how to spawn it
- **Categories** — web, coding, docs, diagrams, meta, etc.
- **Agents** — which fleet agents it applies to (`*` = all, or explicit list)
- **Provenance** — `_source`, `_added`, `_purpose` for auditability

## How it syncs

The umbrella script [`scripts/sync-mcp-to-agents.mjs`](https://github.com/chirag127/workspace/blob/main/scripts/sync-mcp-to-agents.mjs) reads `mcps.json` and writes per-agent MCP configs:

| Agent | Target |
|---|---|
| Claude Code (workspace) | `<workspace>/.mcp.json` |
| Claude Code (user) | `~/.claude.json` `mcpServers` |
| OpenCode | `~/.config/opencode/opencode.jsonc` `mcp` |
| Kilo Code | `~/.config/kilo/kilo.jsonc` |
| Antigravity | via `agy plugin install` (out of band) |
| GH Copilot CLI | `~/.config/gh-copilot/mcp.json` |

Runs at:
- CC SessionStart hook (silent, byte-diff)
- Windows Scheduled Task `Oriz-SyncMCPs` (logon + daily 09:00)
- Manual: `node scripts/sync-mcp-to-agents.mjs`

## Per-machine overrides

Anything in `~/.claude.json.local`, `~/.config/opencode/opencode.local.jsonc`, etc. is preserved. Sync only overwrites the shared block.

## Adding a new MCP

1. Add an entry to `mcps.json` under `servers`
2. Include `_source`, `_added`, `_purpose` for auditability
3. Set `agents` array (default `["*"]` if universal)
4. Add secrets to `.env.enc` if needed; reference via `${env:VAR_NAME}`
5. Commit + push — auto-sync fires on next SessionStart

## Environment variables

Secrets referenced as `${env:VAR}` are populated by the daily `env-to-system-env` scheduled task (per rule `env-to-system-env-daily`). Never hardcode API keys — the sync script rejects any config with literal secrets.

## Filtering

Every server has an `agents` array. `["*"]` means all; `["cc", "opencode"]` targets specific agents. The sync script filters before writing per-agent configs.

## Related

- [`chirag127/workspace`](https://github.com/chirag127/workspace) — umbrella that consumes this repo
- [`chirag127/agent-skills`](https://github.com/chirag127/agent-skills) — same canonical pattern for skills
- [`chirag127/agent-configs`](https://github.com/chirag127/agent-configs) — same canonical pattern for agent settings
- Rules: [`mcp-config-single-source-of-truth`](https://github.com/chirag127/workspace/blob/main/knowledge/rules/agent/mcp-config-single-source-of-truth.md), [`canonical-repo-pure-data-orchestrator-in-umbrella`](https://github.com/chirag127/workspace/blob/main/knowledge/rules/agent/canonical-repo-pure-data-orchestrator-in-umbrella.md)

## License

MIT — see [LICENSE](./LICENSE).
