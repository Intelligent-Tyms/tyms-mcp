# [Tyms](https://tyms.ai) Market Intelligence — MCP servers

Tyms Market Intelligence is a family of 12 hosted, read-only [MCP](https://modelcontextprotocol.io) servers giving AI agents curated, verification-statused market data for East Africa — Uganda, Kenya, Tanzania, Rwanda, and pan-African capital flows. Every record carries a verification status, source attribution and evidence links, and every dataset answers honestly about its own coverage through dedicated coverage and change-feed tools.

## Bundles

All endpoints are Streamable HTTP MCP servers at `https://mi-api.tyms.ai/mcp/{domain}/{region}`.

| Bundle | Endpoint | Tools | What it covers | Cursor |
|---|---|---|---|---|
| Tyms Procurement — Uganda | `https://mi-api.tyms.ai/mcp/procurement/ug` | 46 | Uganda public procurement: notices, awards, contracts, buyers, suppliers, debarments | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-procurement-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9wcm9jdXJlbWVudC91Zz92ZW51ZT1naXRodWIifQ) |
| Tyms Business News — Uganda | `https://mi-api.tyms.ai/mcp/business-news/ug` | 5 | Curated Uganda business news from national sources, with verification status and coverage | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3VnP3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Business News — Kenya | `https://mi-api.tyms.ai/mcp/business-news/ke` | 5 | Curated Kenya business news from national sources, with verification status and coverage | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-ke&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL2tlP3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Business News — Tanzania | `https://mi-api.tyms.ai/mcp/business-news/tz` | 5 | Curated Tanzania business news from national sources, with verification status and coverage | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-tz&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3R6P3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Business News — Rwanda | `https://mi-api.tyms.ai/mcp/business-news/rw` | 5 | Curated Rwanda business news from national sources, with verification status and coverage | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-business-news-rw&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9idXNpbmVzcy1uZXdzL3J3P3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Financial Markets — Uganda | `https://mi-api.tyms.ai/mcp/financial-markets/ug` | 10 | Bank of Uganda FX rates, T-bills and money-market series with coverage metadata | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-financial-markets-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9maW5hbmNpYWwtbWFya2V0cy91Zz92ZW51ZT1naXRodWIifQ) |
| Tyms Financial Markets — Kenya | `https://mi-api.tyms.ai/mcp/financial-markets/ke` | 10 | Kenya financial-market series: FX, yields and macro data with coverage metadata | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-financial-markets-ke&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9maW5hbmNpYWwtbWFya2V0cy9rZT92ZW51ZT1naXRodWIifQ) |
| Tyms Logistics — East Africa | `https://mi-api.tyms.ai/mcp/logistics/ea` | 16 | East Africa logistics: corridor KPIs, fuel prices, trade flows and policy documents | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-logistics-ea&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9sb2dpc3RpY3MvZWE_dmVudWU9Z2l0aHViIn0) |
| Tyms Capital Deployment — Africa | `https://mi-api.tyms.ai/mcp/capital-deployment/africa` | 8 | African capital deployment: projects, deals, commitments and reports with evidence links | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-capital-deployment-africa&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9jYXBpdGFsLWRlcGxveW1lbnQvYWZyaWNhP3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Mining — Uganda | `https://mi-api.tyms.ai/mcp/mining/ug` | 8 | Uganda mining licences and grant events with evidence and coverage tracking | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-mining-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9taW5pbmcvdWc_dmVudWU9Z2l0aHViIn0) |
| Tyms Land & Property — Uganda | `https://mi-api.tyms.ai/mcp/land-property/ug` | 16 | Uganda land and property: public notices, price indices and market snapshots | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-land-property-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9sYW5kLXByb3BlcnR5L3VnP3ZlbnVlPWdpdGh1YiJ9) |
| Tyms Coffee Prices — Uganda | `https://mi-api.tyms.ai/mcp/coffee-prices/ug` | 10 | Uganda coffee prices: UCDA farmgate and export series, World Bank benchmarks, with coverage | [Install in Cursor](https://cursor.com/en/install-mcp?name=tyms-coffee-prices-ug&config=eyJ1cmwiOiJodHRwczovL21pLWFwaS50eW1zLmFpL21jcC9jb2ZmZWUtcHJpY2VzL3VnP3ZlbnVlPWdpdGh1YiJ9) |

Tool counts are **real-route full-scope catalogue totals**: every task, specialist, compatibility and feedback tool served by the production routes, excluding only the two onboarding tools (`about_dataset`/`getting_started`). `send_feedback` and full-scope diagnostic tools are included, so procurement is 46 here and 44 with normal read scopes (48 and 46 including the two onboarding tools). Because tools are scope-filtered per caller, a connected client with narrow scopes may see fewer.

## Quickstart

### Claude (ai / Desktop / Code)

Add a custom connector with any bundle endpoint, for example:

```
https://mi-api.tyms.ai/mcp/procurement/ug?venue=github
```

Claude discovers the OAuth flow automatically and prompts you to sign in with Tyms.

### VS Code

Add to your `mcp.json`:

```json
{
  "servers": {
    "tyms-procurement-ug": {
      "type": "http",
      "url": "https://mi-api.tyms.ai/mcp/procurement/ug?venue=github"
    }
  }
}
```

Repeat with any endpoint from the table above.

### Cursor

Use the **Install in Cursor** links in the table, or add a server manually in Cursor settings with the endpoint URL.

### Gemini CLI

```sh
gemini extensions install https://github.com/Intelligent-Tyms/tyms-mcp
```

This installs all 12 servers from `gemini-extension.json`. OAuth is auto-discovered on first use.

### Other MCP clients

Any client that speaks Streamable HTTP MCP can connect: paste the endpoint URL and complete the Tyms OAuth sign-in when prompted. See [llms-install.md](llms-install.md) for agent-readable install steps.

## Authentication

Sign-in is Tyms OAuth (issuer `https://auth.tyms.ai`), discovered automatically by MCP clients via the RFC 9728 protected-resource challenge — no API keys to copy. A free tier is included; credit packs scale usage.

## Legal

- Privacy policy: https://tyms.ai/privacy
- Terms of Service: https://tyms.ai/terms

## Demo queries

Ten tried-and-tested example queries per bundle (what to ask, which tool serves it, what good output looks like) live in [`demos/`](demos/).

## Registry metadata

`server.json` holds the official MCP Registry drafts (`ai.tyms/*` namespace) for the 12 registry-listed bundles. This repo is a rendered copy — see [MAINTAINING.md](MAINTAINING.md) before editing.

## License

[MIT](LICENSE)
