# Diavgis API

Greek public procurement as structured data — a **REST API**, an **MCP server** for AI agents, an
**OCDS 1.1** feed, and a free bulk dataset.

Diavgis harvests three official Greek/EU registries, normalises them into one schema and **links**
records into contract lifecycle chains, so a procurement is counted once rather than once per document.

> **Independent project.** This is not the Greek government's Διαύγεια portal (diavgeia.gov.gr), nor
> ΚΗΜΔΗΣ or ΕΣΗΔΗΣ, and it is not owned, operated or endorsed by the Greek State.

| | |
|---|---|
| Sources | Διαύγεια (transparency portal) · ΚΗΜΔΗΣ (procurement registry) · EU TED |
| Coverage | from **2026-03-01** — there is no earlier data |
| Website | <https://diavgis.gr> |
| Docs | <https://diavgis.gr/en/docs> (English) · <https://diavgis.gr/docs> (Greek) |

---

## What is in this repository

This repo holds the **public interface contract** — the same artefacts the live service already serves,
kept here so they can be diffed, versioned and referenced by API directories and registries.

| File | What it is | Live equivalent |
|---|---|---|
| [`openapi.json`](openapi.json) | OpenAPI 3.1 description of the REST API | <https://api.diavgis.gr/openapi.json> |
| [`ocds-index.json`](ocds-index.json) | OCDS publication metadata: prefix, licence, coverage, rate limit, pagination | <https://api.diavgis.gr/ocds> |
| [`llms.txt`](llms.txt) | Machine-readable site summary for AI agents | <https://diavgis.gr/llms.txt> |
| [`ai-catalog.json`](ai-catalog.json) | Agent-discovery catalogue (ARD-style draft) | <https://diavgis.gr/.well-known/ai-catalog.json> |
| [`mcp/`](mcp/) | MCP server card and usage examples | <https://mcp.diavgis.gr/mcp> |

**The application source code is not published here.** What is published is what the service already
exposes to any caller.

---

## REST API

Base URL `https://api.diavgis.gr`. Authenticate with an API key from your
[account](https://diavgis.gr/account) — a free key is created automatically when you register.

```bash
curl -H "Authorization: Bearer $DIAVGIS_KEY" \
  "https://api.diavgis.gr/v1/tenders?cpv=33&status=tender&page_size=5"
```

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/tenders` | Search acts — filter by CPV, region, amount, date, procurement signals |
| `GET` | `/v1/acts/{ada}` | One act, with its entities, CPVs and lifecycle chain |
| `GET` | `/v1/entities/{afm}` | Buyer or supplier profile with spend/receipt statistics |
| `GET` | `/v1/chains/{chain_id}` | A reconstructed procurement lifecycle |
| `GET` | `/v1/me` | Your plan, billing period, usage and remaining quota |

Identifiers: Διαύγεια acts use the **ΑΔΑ** (`9ΨΩΞ46ΜΤΛ6-Θ7Γ`), ΚΗΜΔΗΣ uses `kimdis:<ΑΔΑΜ>`, TED uses
`ted:<publication-number>`. Entities are keyed by **ΑΦΜ**, exactly nine digits, for both buyers and
suppliers.

Full reference, quotas and error semantics: <https://diavgis.gr/en/docs>.

---

## MCP server — for AI agents

Endpoint `https://mcp.diavgis.gr/mcp` (JSON-RPC over HTTP). Eight tools:

| Tool | What it answers |
|---|---|
| `search_tenders` | Search acts across the three linked sources |
| `expiring_contracts` | Contracts running now that expire within 0/7/30/180 days |
| `get_act` | Full detail of one act |
| `get_entity` | Profile of a company or public body by ΑΦΜ |
| `get_chain` | A reconstructed lifecycle chain |
| `rank_entities` | Top buyers or suppliers |
| `spend_by_cpv` | Committed value by CPV division |
| `entity_counterparties` | Who an entity buys from, or sells to |

See [`mcp/README.md`](mcp/README.md) for client configuration and worked examples.

---

## OCDS 1.1

Diavgis publishes to the [Open Contracting Data Standard](https://standard.open-contracting.org/).
A Diavgis chain **is** a contracting process, so `chain_id` is the `ocid` and each linked act is a release.

```bash
curl https://api.diavgis.gr/ocds                          # publication metadata
curl https://api.diavgis.gr/ocds/releases/2026-08-14      # one day, as a release package
```

* **ocid prefix** `ocds-r3r7e7`, registered with the Open Contracting Partnership.
* **No API key.** The OCDS feed is open, per OCP hosting guidance.
* **Rate limit** 30 requests / 10 seconds per IP → `HTTP 429` with `Retry-After`.
* **Pagination** uses the seek method — follow `links.next`, which carries an opaque `cursor`.
  Do not construct paged URLs by hand.
* Packages hold at most 1,000 releases. Days older than 30 are settled and cached for a week;
  more recent days are still filling — re-crawl them.

⚠ **Amounts.** `value` uses the gross basis `coalesce(amount_gross, amount_net)`: Διαύγεια publishes a
single VAT-inclusive figure, so `amount_net` is only genuinely net on ΚΗΜΔΗΣ rows.

---

## Bulk dataset

The whole corpus, free under **CC-BY-4.0**, as Parquet:

**<https://huggingface.co/datasets/diavgis/greek-public-procurement>**

Five tables — `acts`, `entities`, `chains`, `act_cpv`, `act_suppliers`. Use this instead of paginating
the API if you want everything. Read the dataset README before summing any amount column.

---

## Licences

| What | Licence |
|---|---|
| Bulk dataset and the browsable catalogue | [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/) — attribute to “Diavgis” |
| The files in this repository | CC-BY-4.0, as published metadata |
| Application source code | Not published |

Each upstream source keeps its own licence — Διαύγεια under Law 4305/2014, EU TED under the Open Data
Directive, ΓΕΜΗ under ODC-BY-1.0. Full attribution: <https://diavgis.gr/terms#sources>.

Personal data of natural persons is redacted as far as the source allows; we publish legal entities.
See <https://diavgis.gr/en/about#gdpr>.

---

## Contact

<https://diavgis.gr/contact> · issues on this repository are welcome for anything about the API surface,
the OCDS feed or the dataset.
