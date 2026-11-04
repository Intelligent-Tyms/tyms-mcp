# Tyms Market Intelligence — agent install guide

Machine-readable install instructions for the 12 Tyms Market Intelligence MCP servers. If you are an AI agent installing these for a user, follow the steps for the user's client.

## Facts

- Transport: Streamable HTTP (remote MCP; nothing runs locally).
- Endpoint grammar: `https://mi-api.tyms.ai/mcp/{domain}/{region}`.
- Auth: OAuth 2.0 (Tyms OAuth, issuer `https://auth.tyms.ai`). Unauthenticated calls return 401 with an RFC 9728 `WWW-Authenticate` challenge, so compliant clients discover and run the OAuth flow themselves. The user signs in once per client; a free tier is included.
- All servers are read-only. Every tool is annotated with `readOnlyHint`/`destructiveHint`.

## Servers

| Server name | URL | Coverage |
|---|---|---|
| tyms-procurement-ug | https://mi-api.tyms.ai/mcp/procurement/ug | Uganda public procurement: notices, awards, contracts, buyers, suppliers, debarments |
| tyms-business-news-ug | https://mi-api.tyms.ai/mcp/business-news/ug | Curated Uganda business news from national sources, with verification status and coverage |
| tyms-business-news-ke | https://mi-api.tyms.ai/mcp/business-news/ke | Curated Kenya business news from national sources, with verification status and coverage |
| tyms-business-news-tz | https://mi-api.tyms.ai/mcp/business-news/tz | Curated Tanzania business news from national sources, with verification status and coverage |
| tyms-business-news-rw | https://mi-api.tyms.ai/mcp/business-news/rw | Curated Rwanda business news from national sources, with verification status and coverage |
| tyms-financial-markets-ug | https://mi-api.tyms.ai/mcp/financial-markets/ug | Bank of Uganda FX rates, T-bills and money-market series with coverage metadata |
| tyms-financial-markets-ke | https://mi-api.tyms.ai/mcp/financial-markets/ke | Kenya financial-market series: FX, yields and macro data with coverage metadata |
| tyms-logistics-ea | https://mi-api.tyms.ai/mcp/logistics/ea | East Africa logistics: corridor KPIs, fuel prices, trade flows and policy documents |
| tyms-capital-deployment-africa | https://mi-api.tyms.ai/mcp/capital-deployment/africa | African capital deployment: projects, deals, commitments and reports with evidence links |
| tyms-mining-ug | https://mi-api.tyms.ai/mcp/mining/ug | Uganda mining licences and grant events with evidence and coverage tracking |
| tyms-land-property-ug | https://mi-api.tyms.ai/mcp/land-property/ug | Uganda land and property: public notices, price indices and market snapshots |
| tyms-coffee-prices-ug | https://mi-api.tyms.ai/mcp/coffee-prices/ug | Uganda coffee prices: UCDA farmgate and export series, World Bank benchmarks, with coverage |

## Install per client

### Claude (claude.ai, Desktop, Claude Code)

1. Open connector settings and choose "Add custom connector".
2. Paste one server URL from the table, e.g. `https://mi-api.tyms.ai/mcp/procurement/ug`.
3. Complete the Tyms OAuth sign-in when the browser opens.
4. Repeat per bundle the user wants, then call `about_dataset` to confirm the connection.

### VS Code (`mcp.json`)

```json
{
  "servers": {
    "tyms-procurement-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/procurement/ug?venue=github" },
    "tyms-business-news-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/business-news/ug?venue=github" },
    "tyms-business-news-ke": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/business-news/ke?venue=github" },
    "tyms-business-news-tz": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/business-news/tz?venue=github" },
    "tyms-business-news-rw": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/business-news/rw?venue=github" },
    "tyms-financial-markets-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/financial-markets/ug?venue=github" },
    "tyms-financial-markets-ke": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/financial-markets/ke?venue=github" },
    "tyms-logistics-ea": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/logistics/ea?venue=github" },
    "tyms-capital-deployment-africa": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/capital-deployment/africa?venue=github" },
    "tyms-mining-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/mining/ug?venue=github" },
    "tyms-land-property-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/land-property/ug?venue=github" },
    "tyms-coffee-prices-ug": { "type": "http", "url": "https://mi-api.tyms.ai/mcp/coffee-prices/ug?venue=github" }
  }
}
```

### Cursor

One-click install deeplinks (checklist 1.5; config URLs carry `?venue=cursor`
attribution — the README table's links carry `?venue=github` instead):

- Tyms Procurement — Uganda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-procurement-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9wcm9jdXJlbWVudC91Zz92ZW51ZT1jdXJzb3IifQ)
- Tyms Business News — Uganda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3VnP3ZlbnVlPWN1cnNvciJ9)
- Tyms Business News — Kenya: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-ke&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL2tlP3ZlbnVlPWN1cnNvciJ9)
- Tyms Business News — Tanzania: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-tz&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3R6P3ZlbnVlPWN1cnNvciJ9)
- Tyms Business News — Rwanda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-rw&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3J3P3ZlbnVlPWN1cnNvciJ9)
- Tyms Financial Markets — Uganda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-financial-markets-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9maW5hbmNpYWwtbWFya2V0cy91Zz92ZW51ZT1jdXJzb3IifQ)
- Tyms Financial Markets — Kenya: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-financial-markets-ke&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9maW5hbmNpYWwtbWFya2V0cy9rZT92ZW51ZT1jdXJzb3IifQ)
- Tyms Logistics — East Africa: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-logistics-ea&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9sb2dpc3RpY3MvZWE_dmVudWU9Y3Vyc29yIn0)
- Tyms Capital Deployment — Africa: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-capital-deployment-africa&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9jYXBpdGFsLWRlcGxveW1lbnQvYWZyaWNhP3ZlbnVlPWN1cnNvciJ9)
- Tyms Mining — Uganda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-mining-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9taW5pbmcvdWc_dmVudWU9Y3Vyc29yIn0)
- Tyms Land & Property — Uganda: [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-land-property-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9sYW5kLXByb3BlcnR5L3VnP3ZlbnVlPWN1cnNvciJ9)

The `config` value is base64url of `{"url":"<endpoint>?venue=cursor"}`. Or
add any endpoint URL from the table above manually under Cursor's MCP settings.

### Gemini CLI

```sh
gemini extensions install https://github.com/Intelligent-Tyms/tyms-mcp
```

The extension manifest (`gemini-extension.json` in this repo) registers all 12 servers via `httpUrl`; OAuth is auto-discovered.

### Cline / other generic clients

Use the same JSON shape as VS Code (`mcpServers` or `servers` depending on the client), with `"type": "http"` (or `"streamable-http"`) and a URL from the table.

## Verify the install

After connecting, call `about_dataset` on the server. A healthy connection returns the dataset description, surfaces and coverage semantics. Then call `getting_started` for the recommended first queries, or pick from the per-bundle query lists in `demos/`.

## Troubleshooting

- 401 responses are expected before sign-in; run the OAuth flow rather than looking for an API key.
- If a tool returns an error result, the message is the real upstream error (Tyms servers surface errors as tool results, not protocol errors).
