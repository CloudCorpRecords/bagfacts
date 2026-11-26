# BUILD REPORT — bagfacts

Clean-room rebuild of `com.koopscan/mcp` v0.7.0
(remote-only MCP server at `https://mcp.koopscan.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/bagfacts`
0.1.0, registry name `io.github.CloudCorpRecords/bagfacts`.

## Surface built

**FULL surface: all 2 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `lookup_address` | `address` |
| `crime_sources` | `postcode` |

Each served tool carries exactly the keys `name`,
`description`, `inputSchema` — the capture carries NO
`title` key, so the fixture carries none and none is
served. Each `inputSchema` carries its captured `title`
(`lookup_addressArguments` / `crime_sourcesArguments`),
`type: object`, the captured `required` array, and NO
`$schema` marker and NO `additionalProperties` key —
none was captured, none was invented. Both tools carry
the optional `language` property exactly as captured
(enum `nl`/`en`, default `nl`, title `Language`), and
the key properties are plain strings (`address` titled
`Address`, `postcode` titled `Postcode`). The tool
literals were generated programmatically from
`harness/fixtures/koopscan-tools-original.json` by the
script at `/tmp/laneb9-bagfacts-scratch/gen.py`
(parse + re-serialise, spliced into `src/forward.ts`
between the TOOL_DEFS_GENERATED markers) — never
retyped — and the generator's round-trip check confirmed
the spliced block parses back deep-equal to the
fixture's 2 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

The surface is a keyed lookup into published Dutch
registers and a published, human-reviewed news dataset:
for one residential address, the BAG register's
building facts (construction year, units) and the
dwelling's registered EP-Online energy label, plus a
summary of reviewed news-based crime signals for the
postcode with a provenance block; and the list of the
news articles behind those signals.

**DISTINCTION (recorded from the spec, for the publish
gate's visibility):** the lookup key is a PLACE (a
street address or postcode), not a named person or
entity; the returned crime material is a published
dataset of reviewed news reports about an area,
returned as-is with its sources — the freezeledger
published-register shape, and of a piece with
beetlewatch's published municipal dataset reads. No
risk score, verdict or eligibility judgment about any
person is computed or returned, and the upstream
descriptions themselves forbid inferring that an area
is safe. This is NOT the caller-named-entity screening
family. No flag is added; the reasoning is recorded
here as the spec directs.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools'
names, descriptions, the `required` arrays (`address`;
`postcode`), the absence of any `title` key on the tool
objects, and the property shapes (including the
`title` keys that DO appear inside each `inputSchema`
and on its properties, served verbatim).

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://mcp.koopscan.com/mcp`.
- Env override `KOOPSCAN_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `KOOPSCAN_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `KOOPSCAN_TOKEN` variables present.
- Error rendering `bagfacts error:` text for upstream failures
  — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

**None.** No write-guard workaround was needed in this session:
the lockfile, `npm ci`, `tsc` build and test suite all ran
through the normal routes (direct `node_modules/.bin/tsc` /
`tsx` binaries), and `dist/` is the direct output of
`tsc` on the validated sources.

## Lockfile

Generated clean from birth (Route 3, clean dir, no
`node_modules`, no pre-existing lockfile):
`npm install --package-lock-only --ignore-scripts --offline`
→ success. Parses as JSON; `name` == `@reneturcios/bagfacts`;
exactly **128** package entries; sha256
`d239a5a3c298619e776dee45b47880e20eb4a54c97936dfb1c2ee31f0e56babf`.
`npm ci --ignore-scripts --offline` then succeeded and the
lockfile sha256 was re-verified **unchanged** afterwards.

## server.json

- Description exactly: "Dutch address facts: BAG building data, energy labels and reviewed crime-signal sources." — **88 characters** (measured from the written file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/bagfacts:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/bagfacts"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/bagfacts`
  and its smoke test asserts the server name `bagfacts`.

## Verification (run 2026-10-28, builder session)

1. `npm install --package-lock-only --ignore-scripts --offline`
   → success; lockfile parses, name `@reneturcios/bagfacts`,
   128 package entries (see Lockfile).
2. `npm ci --ignore-scripts --offline` → success; lockfile
   sha256 unchanged.
3. `tsc --noEmit` (direct `node_modules/.bin/tsc`) → exit 0.
4. `tsc` build → exit 0; `node dist/cli.js --version` → `0.1.0`.
5. Full suite (`tsx --test test/*.test.ts`, direct
   `node_modules/.bin/tsx`) → **61 tests,
   61 pass, 0 fail** (registration-integrity suite deep-equals
   the served tools/list against the fixture's 2 tools, in
   order, at both the registry level and through an
   in-memory-transport MCP `tools/list`; plus per-tool schema
   detail assertions including the no-title check, the
   captured inputSchema titles, and the absence of `$schema` /
   `additionalProperties`; session handling/reuse, env override,
   the no-Authorization-header assertions under every
   environment, verbatim argument forwarding for both tools and
   result pass-through including upstream isError, SSE parsing,
   error rendering, MCP-layer validation, and CLI behaviour).
6. Parity (harness, tools-file mode):
   `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/koopscan-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/bagfacts/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/bagfacts/parity-out/`
   → **verdict PASS, parity 100.0** (2/2 identical, 0 missing,
   0 extra, no schema diffs). Output in `parity-out/`.
