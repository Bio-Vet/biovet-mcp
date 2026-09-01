# BioVet MCP Server

[![BioVet MCP server on Glama](https://glama.ai/mcp/servers/Bio-Vet/biovet-mcp/badges/score.svg)](https://glama.ai/mcp/servers/Bio-Vet/biovet-mcp)
[![smithery badge](https://smithery.ai/badge/pwnz13/biovet)](https://smithery.ai/servers/pwnz13/biovet)

**Remote MCP server for BioVet — a network of 20 veterinary clinics in Moscow, Russia (open 24/7).**

Lets AI assistants (Claude, agentic browsers, any MCP client) find a clinic, check live prices, see open appointment slots, **book a visit**, run a symptom urgency check (triage table approved by licensed veterinarians) and check whether a food or houseplant is safe for a pet.

- **Endpoint:** `https://bio.vet/mcp` (Streamable HTTP, stateless JSON-RPC 2.0, no auth required)
- **Manifest:** [`https://bio.vet/.well-known/mcp.json`](https://bio.vet/.well-known/mcp.json)
- **Language:** tools accept and answer in Russian (the clinics operate in Moscow)

## Quick start

Claude Code:

```bash
claude mcp add --transport http biovet https://bio.vet/mcp
```

Claude Desktop: *Settings → Connectors → Add custom connector* → `https://bio.vet/mcp`.

## Tools

| Tool | What it does |
|---|---|
| `find_clinic` | Nearest of 20 clinics by district / metro / street query or by coordinates, with live Yandex Maps ratings and `clinic_id` |
| `get_prices` | Live prices from the clinic network's price list (900+ services, same in every clinic) |
| `check_slots` | Doctors of a clinic with their open appointment times; if the requested specialty is absent, suggests real clinics in the network that have it |
| `book_visit` | Books a real appointment (name + phone + clinic + time). The assistant must show the user a summary and get explicit confirmation first |
| `triage` | Symptom urgency check — *go now / see a vet today / observe* — based on a sign table approved by the network's veterinarians. Routing only, never a diagnosis |
| `food_check` | Is this food/plant safe for a cat, dog, rodent or bird — vet-reviewed reference |

## Example

```bash
curl -s -X POST https://bio.vet/mcp -H 'Content-Type: application/json' -d '{
  "jsonrpc": "2.0", "id": 1, "method": "tools/call",
  "params": { "name": "find_clinic", "arguments": { "query": "Марьино" } }
}'
```

## Safety

- Booking is rate-limited per IP and every booking is tagged in the clinic's journal; the CRM may call the client back to confirm.
- No drug dosages and no medical advice beyond urgency routing are ever returned — triage always ends with "see a veterinarian".
- Read tools are free to call; there is no tracking of MCP clients.

## About

[BioVet](https://bio.vet/) — сеть ветеринарных клиник: 20 клиник в Москве и Реутове, все работают круглосуточно, собственная лаборатория. Этот MCP-сервер — официальный.

Открытые ветеринарные датасеты сети (CC BY 4.0): [github.com/Bio-Vet](https://github.com/Bio-Vet) · [bio.vet/open-data](https://bio.vet/open-data/)
