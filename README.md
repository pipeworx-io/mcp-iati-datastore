# @pipeworx/iati-datastore

International aid activities (IATI Datastore) — search every published aid and
development activity worldwide by country, sector, donor or keyword, and read
the individual commitments and disbursements behind it.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `iati_search_activities({ query, country, sector, reportingOrg, limit, offset, _apiKey })` — activities matching a Solr query, with convenience filters.
- `iati_activity({ iatiIdentifier, _apiKey })` — the full record for one activity.
- `iati_transactions({ iatiIdentifier, query, transactionType, limit, offset, _apiKey })` — the money actually moved: commitments vs disbursements, with dates and counterparties.

## Auth

Platform key (`PLATFORM_IATI_KEY`) with BYO override via `?_apiKey=`; sent
upstream as an `Ocp-Apim-Subscription-Key` header. Free self-service
registration at <https://developer.iatistandard.org/> — sign up, then subscribe
to the "IATI Datastore API" product. No manual approval step.

## Data sources

- <https://api.iatistandard.org/datastore/activity/select> — activity index.
- <https://api.iatistandard.org/datastore/transaction/select> — transaction index.

IATI is the standard donor governments, UN agencies, development banks and
large NGOs publish their aid spending to; the Datastore is the single queryable
index over all of it.

## Traps

- **`query` is Apache Solr syntax, not free text.** A bare word searches the
  default field only and quietly matches little; prefer an explicit field
  (`title_narrative:water`, `recipient_country_code:KE`). Every response echoes
  `query_sent` so you can see what actually ran.
- **An activity with a budget may have no transactions.** Not every publisher
  reports transaction-level detail, so an empty `iati_transactions` result is
  often a publisher gap, not a query error.
- Solr multi-valued fields come back as arrays and scalars do not, for the same
  field name across documents — the mapping normalises both.
- The populated payload is **unverified** — no fleet key exists. The Solr
  envelope (`response.numFound` / `response.docs`) is stable; document field
  names come from the published IATI field reference, and every tool returns
  `raw` so a caller with a real key is never blocked by a stale name here.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "iati-datastore": {
      "url": "https://gateway.pipeworx.io/iati-datastore/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/iati-datastore/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/iati_search_activities \
  -H 'Content-Type: application/json' \
  -d '{"country":"KE","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/iati_search_activities`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-iati-datastore` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/iati-datastore/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "iati-datastore": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-iati-datastore"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Iati Datastore data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
