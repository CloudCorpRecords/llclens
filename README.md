# llclens

A window onto one published dataset, without downloading it. llclens is a local forwarding client for the hosted LLC Lane MCP service: it answers `tools/list` on your own machine from the captured tool definitions and forwards every query to LLC Lane, returning the answers exactly as the service sends them back.

The surface is entirely read-only. You can inspect the dataset's shape and provenance, search its rows, look rows up by an exact key, line several rows up side by side, and ask for summary statistics or rankings over its numeric columns. Nothing is cached, stored or computed locally — the package is a pipe, not a copy of the data.

## The tools

- **`dataset_columns`** — The columns of the dataset, which of them are numeric, the row count and the provenance banner. Call this first to learn the schema. Takes no arguments.
- **`dataset_search`** — Rows whose cells contain the query text (case-insensitive), up to 50. `query` is required; `limit` caps the rows returned (1–50).
- **`dataset_row`** — The rows where a column equals a value exactly (case-insensitive). Both `column` and `value` are required.
- **`dataset_compare`** — The rows whose column is any of the given values, in the order given — built for "X vs Y" questions. `column` is required, and `values` takes 2–10 values.
- **`dataset_stats`** — count, min, max, mean, median and sum of a numeric column. Grouping commas and currency are handled; non-numeric rows are excluded and counted. `column` is required.
- **`dataset_top`** — The highest (or lowest) rows by a numeric column — "which is the most/least X". `column` is required; `limit` caps the rows (1–50) and `ascending: true` returns the lowest first instead of the highest.
- **`dataset_provenance`** — The source, the date the data was computed, the licence and the citation, so a figure can be attributed correctly. Takes no arguments.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/llclens mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "llclens": {
      "command": "npx",
      "args": ["-y", "@reneturcios/llclens", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/llclens
docker run --rm ghcr.io/cloudcorprecords/llclens dataset_columns '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
llclens dataset_columns '{}'
llclens dataset_search '{"query":"smith"}'
llclens dataset_row '{"column":"state","value":"CA"}'
llclens dataset_compare '{"column":"state","values":["CA","TX"]}'
llclens dataset_stats '{"column":"employees"}'
llclens dataset_top '{"column":"employees","limit":5}'
llclens dataset_provenance '{}'
llclens --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `LLCLANE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://llclane.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The LLC Lane registry record declares no headers on its remote, so llclens sends no authentication header of any kind — queries are keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `llclens error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, exercises the session handshake and its reuse, asserts that no `Authorization` header is sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
