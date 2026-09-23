# Troostwijk Auctions

European industrial equipment, vehicles, real estate and bankruptcy/
insolvency liquidation auction lots from
[troostwijkauctions.com](https://www.troostwijkauctions.com) — covering
NL/DE/BE/UK/PL/RO/IT and more.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1669+ live data sources.

## Tools

| Tool | What it returns |
|------|-----------------|
| `troostwijk_categories` | The full category tree (industrial equipment, vehicles, real estate, construction, metalworking, woodworking, transport, etc.) with the `category_path` each one needs for the other tools. |
| `troostwijk_search_lots` | Currently-open lots in one category: live current bid, bid count, location, close time. |
| `troostwijk_lot_detail` | Full detail for one lot within its auction: auction description, current/final bid, sale outcome, location. |
| `troostwijk_sold_comps` | Realized sale-price comps from recently-closed auctions — see below for how the closed-lot price question was resolved. |

## The closed-lot hammer-price question — resolved

Verified live 2026-09-13 against real closed auctions:

- `biddingStatus: "BIDDING_CLOSED"` + `saleTerm` of
  `CLOSED_RESERVE_PRICE_ALLOCATED` or `GUARANTEED_SALE_CLOSED` means the lot
  **sold**, and `currentBidAmount` **is** the real hammer price (sample:
  "Genelec HPW-100 T5 INS 50HZ Stroomgenerator", 42 bids, sold for
  EUR 5,500.00).
- `saleTerm: "UNSOLD"` or `"CLOSED_RESERVE_PRICE_ALLOCATION_DECLINED"` means
  it did **not** sell — `currentBidAmount` there is the highest bid received
  (or reserve floor), not a sale price.
- `saleTerm: "CLOSED_RESERVE_PRICE_ALLOCATION_PENDING"` means bidding closed
  but the seller has not yet accepted or declined — outcome unknown.

`troostwijk_sold_comps` returns **only** the first category (real, allocated
sales) by default, with every row's `outcome` field stated explicitly. It
never reports a pending or declined bid amount as a sale price.

## Locale

Requesting under `/en/...` with **any** locale's slug — even the native Dutch
one from `troostwijk_categories` — 307-redirects to the canonical English
URL, and `fetch()` follows it transparently. Category names and templated lot
titles (tractors, cars, etc.) come back in English this way. Free-text
auction/lot titles written by the seller (estate-sale and liquidation-lot
names) are **not** translated and stay in the source country's language —
expect Dutch/German/Polish/Romanian text mixed into results.

## Auth

None. Keyless, no login required to browse.

## Data sources

- `troostwijkauctions.com/en/c/{category_path}` — category browse pages;
  `pageProps.lotsData.results` in the page's `__NEXT_DATA__` blob.
- `troostwijkauctions.com/en/a/{auction_slug}` — auction detail pages;
  `pageProps.lots.results` (works for both single-lot and multi-lot
  auctions).
- `troostwijkauctions.com/en/auctions?auctionBiddingStatuses=BIDDING_CLOSED`
  — the closed-auctions listing `troostwijk_sold_comps` scans.

### robots.txt

Disallows only query-param SEARCH URLs (`*search*`, `*categoryLevel=*`,
`*totalSize=*`, `*countries=*`, `*brands=*`, `*auctions=*`, `*amounts=*`) —
category and lot/auction pages are unrestricted. Per Bruce's 2026-09-01
ruling, a Disallow on content already served without credentials is not a
blocker; this pack does use the `countries=` filter param since the
underlying data is public either way, but never touches the actual
`/search` path (category browsing serves the same tools without it).

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "troostwijk": {
      "url": "https://gateway.pipeworx.io/troostwijk/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/troostwijk/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1669+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/troostwijk_categories \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/troostwijk_categories`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "troostwijk": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-troostwijk"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-troostwijk
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Troostwijk data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
