# @pipeworx/mozilla-observatory

MDN HTTP Observatory — grade any website's HTTP security headers and get the
specific fixes, from Mozilla's public scanner.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `observatory_scan(host)` — runs (or returns Mozilla's recent) scan and gives
  the letter grade A+..F, the numeric score, and the pass/fail counts.
- `observatory_results(host, only_failures?, include_history?, history_limit?)` —
  all 12 tests with pass/fail, score impact, Mozilla's own remediation prose,
  the observed response headers, and the host's grade history.
- `observatory_grade_distribution()` — how many scanned sites hold each grade,
  so a single grade can be placed against the web at large.

## Auth

Keyless.

## Data sources

- `POST https://observatory-api.mdn.mozilla.net/api/v2/scan?host=` — scan summary.
- `GET  https://observatory-api.mdn.mozilla.net/api/v2/analyze?host=` — full detail.
- `GET  https://observatory-api.mdn.mozilla.net/api/v2/grade_distribution` — histogram.

## Things the next person would otherwise rediscover

- **The detail endpoint is `/analyze`, not `/scan`.** `GET /api/v2/scan?host=`
  is a 404 — `/scan` exists only as a POST. `/api/v2/tests?scan=<id>` and
  `/api/v2/scan/<id>` are both 404 as well (all verified 2026-09-17). The
  histogram is `grade_distribution` with an underscore; the hyphenated spelling
  404s.
- **`tests_failed` and `pass: false` disagree, and both are right.** Mozilla
  counts only score-reducing failures in `tests_failed`, but sets `pass: false`
  on informational tests with a `score_modifier` of 0 too — mozilla.org on
  2026-09-17 reported `tests_failed: 2` with four tests marked not-passing. The
  pack returns a per-test `penalized` flag and a `tests_failing_with_score_impact`
  count rather than dropping the zero-impact rows, so `only_failures` returning
  more rows than `tests_failed` is expected and explained in the payload.
- **`recommendation` and `score_description` are HTML fragments.** Stripped to
  prose before returning.
- A host that does not resolve comes back as a 200 with a populated `error`
  field rather than an HTTP error; the pack raises on it.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "mozilla-observatory": {
      "url": "https://gateway.pipeworx.io/mozilla-observatory/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/mozilla-observatory/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/observatory_scan \
  -H 'Content-Type: application/json' \
  -d '{"host":"mozilla.org"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/observatory_scan`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "mozilla-observatory": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-mozilla-observatory"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-mozilla-observatory
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Mozilla Observatory data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
