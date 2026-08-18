# Diavgis MCP server

Greek public-procurement data for AI agents, over the
[Model Context Protocol](https://modelcontextprotocol.io/).

```
https://mcp.diavgis.gr/mcp
```

JSON-RPC over HTTP. Registered as `gr.diavgis/procurement`.

---

## Configuration

### Claude Desktop / Claude Code

```json
{
  "mcpServers": {
    "diavgis": {
      "type": "http",
      "url": "https://mcp.diavgis.gr/mcp",
      "headers": { "Authorization": "Bearer YOUR_DIAVGIS_KEY" }
    }
  }
}
```

Get a key at <https://diavgis.gr/account> — registering creates a free one automatically.
The free tier is enough to try every tool below.

### Anything else

The server implements the standard MCP HTTP transport. Point your client at the URL above and send the
key as `Authorization: Bearer …`. The protocol version travels in `_meta` and in the
`MCP-Protocol-Version` header; `server/discover` is supported, and errors come back as `-32022`/`-32020`
on HTTP 400.

---

## Tools

| Tool | Answers |
|---|---|
| `search_tenders` | «What Greek public tenders exist for X?» — across Διαύγεια, ΚΗΜΔΗΣ and TED, cross-linked |
| `expiring_contracts` | «Which contracts are running now and expire within 0/7/30/180 days?» |
| `get_act` | «Show me this specific record» — by ΑΔΑ, `kimdis:<ΑΔΑΜ>` or `ted:<id>` |
| `get_entity` | «Who is this company / public body?» — by 9-digit ΑΦΜ |
| `get_chain` | «Show the full lifecycle» — commitment → tender → award → contract → payment |
| `rank_entities` | «Who are the biggest buyers / suppliers?» |
| `spend_by_cpv` | «How much public money goes to this category?» |
| `entity_counterparties` | «Who does this buyer pay most?» / «Who does this supplier sell to?» |

---

## Worked examples

**Find open medical-equipment tenders**

```json
{ "name": "search_tenders",
  "arguments": { "cpv": "33", "status": "tender", "page_size": 10 } }
```

CPV `33` is the EU category for medical equipment and pharmaceuticals. The CPV code is the international
key — you do not need to know Greek to filter by sector.

**Contracts expiring in the next 30 days, so you can bid on the renewal**

```json
{ "name": "expiring_contracts",
  "arguments": { "horizon": 30, "cpv": "33", "region": "EL30" } }
```

`EL30` is the NUTS code for Attica.

**Follow one procurement end to end**

```json
{ "name": "get_act", "arguments": { "id": "9ΨΩΞ46ΜΤΛ6-Θ7Γ" } }
```

The response carries `chain_id`; pass it to `get_chain` to see every linked stage with the confidence of
each link.

---

## Things worth knowing before you build on this

**Coverage starts 2026-03-01.** There is no earlier data. A query about 2024 returns nothing because
nothing was harvested, not because nothing happened.

**Amounts are gross.** Διαύγεια publishes a single VAT-inclusive figure; `amount_net` is only genuinely
net on ΚΗΜΔΗΣ rows. Do not mix the two bases in one total.

**Spending is counted once per chain, not once per record.** The same procurement is published at several
stages; summing records double-counts it. Tender budgets are estimates, not money paid.

**Links carry a confidence level.** Chains are reconstructed — partly from explicit ΑΔΑ/ΑΔΑΜ references,
partly by matching. `get_chain` tells you which, per link.

**Natural persons are redacted.** Where a counterparty is an individual, personal data is removed as far
as the source allows. Do not attempt re-identification — it is prohibited by the
[terms](https://diavgis.gr/terms).

---

## Free and open alternatives

If you want the whole corpus rather than per-query access, take the
[bulk dataset](https://huggingface.co/datasets/diavgis/greek-public-procurement) (CC-BY-4.0) or the
[OCDS 1.1 feed](https://api.diavgis.gr/ocds), which needs no key at all.
