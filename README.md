# bagfacts

Before you judge a Dutch home by its listing photos, check what the registers say about it. bagfacts is a small forwarding client for the hosted Koopscan address service: the tool list is answered from your own machine using captured definitions, and each call travels upstream, with the service's answer handed back exactly as it arrived.

Both of the service's captured tools are exposed. Look up one residential address and get the resolved address, building facts from the Dutch BAG register (construction year, number of units), the dwelling's registered energy label from EP-Online, a summary of human-reviewed, news-based crime signals for the postcode with a provenance block, and school counts for the area — or list the actual news articles behind a postcode's crime signals, newest first.

The lookup key is always a place — a street address or a postcode — never a person. The crime material is a published dataset of reviewed news reports about an area, returned as-is with its sources; nothing here computes a risk score or a verdict about anyone, and where coverage does not reach, the upstream says so rather than implying an area is safe.

## The tools

- **`lookup_address`** — Look up one Dutch residential address (street + house number + city, or postcode + house number). Returns the BAG building facts, the registered EP-Online energy label, a summary of reviewed crime signals for the postcode (exact PC6, otherwise the 4-digit area, last 24 months) with its provenance block, and section links into the neighbourhood page. An optional `language` argument (`nl` by default, or `en`) selects the language of labels, the disclaimer and the linked pages.
- **`crime_sources`** — List the news articles behind the crime signals for a Dutch postcode (PC6 like `3024EW`, or the 4-digit area like `3024`): date, category, headline, source and link, newest first, at most 25, plus the same provenance block as `lookup_address`. Also takes the optional `language` argument.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/bagfacts mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "bagfacts": {
      "command": "npx",
      "args": ["-y", "@reneturcios/bagfacts", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/bagfacts
docker run --rm ghcr.io/cloudcorprecords/bagfacts crime_sources '{"postcode":"3024EW"}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
bagfacts lookup_address '{"address":"Herengracht 1, 1015 BA Amsterdam","language":"en"}'
bagfacts crime_sources '{"postcode":"3024EW"}'
bagfacts --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `KOOPSCAN_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://mcp.koopscan.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The Koopscan registry record lists no headers for its remote, so bagfacts never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `bagfacts error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, bagfacts repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged for both tools.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
