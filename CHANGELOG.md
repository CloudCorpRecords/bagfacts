# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Koopscan MCP server. Both captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `lookup_address`, `crime_sources` — a Dutch address lookup returning BAG building facts, the registered EP-Online energy label and a summary of reviewed news-based crime signals for the postcode with a provenance block, and the list of news articles behind those signals for a postcode.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `bagfacts error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `KOOPSCAN_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
