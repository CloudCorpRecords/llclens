# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted LLC Lane MCP server. All seven captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `dataset_columns`, `dataset_search`, `dataset_row`, `dataset_compare`, `dataset_stats`, `dataset_top`, `dataset_provenance` — the complete captured 7-tool surface, no omissions.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `llclens error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LLCLANE_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
