# źlab — Operational Decision Stress Test (MCP server)

Evidence-governed screening for material decisions about physical assets — facility
retrofits, equipment replacement, energy projects, vendor savings claims. Connect it
to your AI assistant and it gains a governed screen: what the evidence currently
supports, what could change the next action, and the justified posture — advance,
condition, compare, measure, defer, or stop.

**Remote MCP server** (streamable HTTP, OAuth sign-in):

```
https://mcp.zircular.io/mcp
```

Listed in the [official MCP registry](https://registry.modelcontextprotocol.io) as
**`io.zircular/zlab-stress-test`**.

> **New here? Start with [What źlab does](docs/WHAT_IS_ZLAB.md)** — a plain-language
> explanation of the problem it solves, what you get back, and a worked example.

## Install

**Claude (web / desktop)** — Settings → Connectors → *Add custom connector* → paste the URL. (Pro/Max)

**Claude Code**

```bash
claude mcp add --transport http zlab https://mcp.zircular.io/mcp
```

**Cursor** — add to `~/.cursor/mcp.json`:

```json
{ "mcpServers": { "zlab": { "url": "https://mcp.zircular.io/mcp" } } }
```

**VS Code** — add to your `mcp.json`:

```json
{ "servers": { "zlab": { "type": "http", "url": "https://mcp.zircular.io/mcp" } } }
```

**ChatGPT** — enable developer mode, then Settings → Connectors → *Create* with the URL above (OAuth).

Full instructions: [zircular.io/connect](https://zircular.io/connect)

## The connector (tools & output)

For the plain-language "what does this actually do" answer, see
[**What źlab does**](docs/WHAT_IS_ZLAB.md). In short:

Four tools: `screen_operational_decision` (submit; job-based, 3–10 min),
`get_stress_test_status`, `fetch_stress_test_result`, and a backward-compatible
alias. The result is an evidence-bounded governed read — never a categorical
yes/no, and financial magnitudes are withheld until site-level evidence supports
them. **A refusal is a product answer**: it names what is missing and how to lift
it, so your assistant can gather the named evidence and resubmit the same case to
raise the ceiling.

Host-facing documentation (also served in-band as MCP resources
`zlab://docs/tool-contract` and `zlab://docs/document-workflows`):

- [Tool contract for hosts](docs/MCP_TOOL_CONTRACT_FOR_HOSTS.md) — the four tools,
  job lifecycle, payloads, refusal/error handling.
- [Document workflows](docs/HOST_DOCUMENT_WORKFLOWS.md) — when to invoke from
  documents/spreadsheets, what each posture means in the sheet, the evidence loop.

## Privacy

What leaves the machine, what is stored, and what is never logged:
[zircular.io/privacy](https://zircular.io/privacy). Every run requires explicit
user authorization before public-data lookups (US Census, EIA, NOAA, EPA, and
related public sources).

## About

Built by [źlab](https://zircular.io) (Zircular LLC) — decision governance for
physical assets. Questions: davidl@zircular.io

> Note: this repository distributes the connector documentation. The analysis
> engine (64 governed motors) runs server-side and is not open source.
