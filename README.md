# @pipeworx/cn-mof-audit-oversight

Which audit firms and asset-appraisal firms has China's Ministry of Finance (财政部) penalised, inspected or named, and why? Rows come from the Ministry's Supervision and Evaluation Bureau (监督评价局) bulletins: fiscal and accounting supervision inspection bulletins with typical cases, administrative penalty decisions against audit firms, annual lists of firms selected for practice-quality inspection, and serious-dishonesty and delivery notices. Firms only.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2006+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `cn_mof_bulletins_list(query?, kind?, page?)` -- one page (about 10 rows, newest first). Each row gives the title, date, bulletin type, the firm in the title where there is one, and the link to the official bulletin. `kind` is one of `inspection_bulletin`, `inspection_list`, `penalty_decision`, `service_notice`, `credit_blacklist`, `case_roundup`, `other`. `query` is Chinese text; both filters match titles on the page read only (the publisher has no keyword search). `page` goes back in time and the response gives `pages_total`. Also returns `searched`, `individual_rows_omitted` and `data_as_of`.
- `cn_mof_bulletin_get(bulletin_url)` -- one bulletin from a list row: its text in the Ministry's own wording, the firms named in it (audit firms, appraisal firms, audited companies), the date and type, and `person_clauses_omitted`.

## Auth

Keyless.

## Data sources

- Ministry of Finance of China (财政部), Supervision and Evaluation Bureau: fiscal and accounting supervision inspection bulletins, administrative penalty decisions and practice-quality inspection lists for accounting and asset-appraisal firms. Read from the publisher on each call; nothing is stored.

## Individuals are not returned

The bulletins also name certified public accountants, appraisers and company officers. Clauses that name them (and aggregate-free sentences about a signing accountant) are cut from the text and counted in `person_clauses_omitted`; a list row whose title makes a natural person the party is left out and counted in `individual_rows_omitted`. Every response carries a `note` saying so; the tests go red if the cut is removed.

## Behaviour worth knowing

- The bulletins cover cases the Ministry chose to publish. Provincial finance departments penalise most firms and publish on their own sites, so no match is never evidence that a firm was not penalised.
- Firm names mostly appear in the bulletin text, not the title: a title filter only finds titles. Open the bulletins (`cn_mof_bulletin_get`) and read `firms_named`.
- `firms_named` is extracted by pattern; it can miss or truncate a name. The text is authoritative. On the practice-quality inspection lists it returned 39 of the 40 firms in the 2026 list.
- The publisher's front end sometimes answers 502 for a page for minutes at a time. The pack retries twice, then reports `upstream_down`; it never reports an unreadable page as an empty result.
- Text is in simplified Chinese.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "cn-mof-audit-oversight": {
      "url": "https://gateway.pipeworx.io/cn-mof-audit-oversight/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/cn-mof-audit-oversight/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2006+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/cn_mof_bulletins_list \
  -H 'Content-Type: application/json' \
  -d '{"kind":"penalty_decision","page":1}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/cn_mof_bulletins_list`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-cn-mof-audit-oversight` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/cn-mof-audit-oversight/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "cn-mof-audit-oversight": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-cn-mof-audit-oversight"]
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
ask_pipeworx({ question: "your question about Cn Mof Audit Oversight data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
