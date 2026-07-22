# Bookproof

Bookproof is a read-only pre-trade execution guard for Polymarket taker bots. It turns a Polymarket URL, slug, CLOB token ID, or market search into fee- and depth-aware execution data before a bot submits a BUY or FOK order.

> Official Bookproof technical project. Public responses may be AI-assisted and are supervised by the project owner.

## What the paid calls add

Polymarket's public APIs expose market metadata and raw order books. Bookproof packages the execution decision layer into stable JSON:

- exact-size/full-budget book walking;
- executable shares and VWAP;
- worst consumed price level;
- modeled taker fee and all-in cost;
- price impact, unfilled budget and insufficient-depth state;
- book timestamp, hash and freshness flags.

Bookproof never places an order, holds trading keys, or promises execution or profit.

| Endpoint | Price | Purpose |
| --- | ---: | --- |
| `GET /api/v1/resolve-market` | Free | Resolve a URL, slug, token ID or search phrase |
| `GET /api/v1/market-health` | 0.005 USDC | Spread, depth, imbalance, fee and freshness snapshot |
| `POST /api/v1/execution-quote` | 0.015 USDC | Budget-aware shares, VWAP, worst fill, fee and impact |

Payments use x402 v2 USDC on Base. There is no subscription or customer account.

## Free market resolution

```bash
curl --get 'https://bookproof-api.sn95wjq846.chatgpt.site/api/v1/resolve-market' \
  --data-urlencode 'input=Fed rate cut July 2026'
```

## Execution-quote request

```json
{
  "market": "https://polymarket.com/event/wti-up-or-down-on-july-22-2026",
  "outcome": "Up",
  "budgetUsd": 100
}
```

The documented example walks the visible asks for a 100 USDC all-in budget and reports 219.78022 shares, 0.445 VWAP, 2.17 USDC modeled fee and 348.84 bps price impact. These values are a dated example response, not a current quote; callers must request a fresh snapshot immediately before their own decision.

## Real public order-book demo

**Historical demo; not a live quote, customer test, paid call, or revenue.** A
public Polymarket Gamma/CLOB snapshot recorded on `2026-06-16` shows why
`budget / bestAsk` is not an execution quote:

| BUY/FOK scenario | Result |
| --- | ---: |
| Budget | `500,000 USDC` |
| Best ask | `0.995` |
| Book-walk VWAP | `0.995315` |
| Worst fill | `0.996` |
| Fee | `0.00 USDC` (fees disabled in the snapshot) |
| Visible depth | `20,358,472.91 shares` |
| Decision at a `0.995` cap | **REJECT** |

The first ask could not absorb the full budget. The quote crossed into `0.996`,
so an FOK order capped at the displayed best ask must be rejected even though
total depth was sufficient. See the [levels, provenance, calculation, and safety
labels](./EXAMPLE_QUOTE.md).

## JavaScript (x402)

```js
import { wrapFetchWithPayment } from "@x402/fetch";
import { x402Client } from "@x402/core/client";
import { ExactEvmScheme } from "@x402/evm/exact/client";
import { privateKeyToAccount } from "viem/accounts";

const signer = privateKeyToAccount(process.env.EVM_PRIVATE_KEY);
const client = new x402Client();
client.register("eip155:*", new ExactEvmScheme(signer));
const paidFetch = wrapFetchWithPayment(fetch, client);

const response = await paidFetch(
  "https://bookproof-api.sn95wjq846.chatgpt.site/api/v1/market-health?" +
  new URLSearchParams({ market: "Fed rate cut July 2026" })
);
console.log(await response.json());
```

## Python (x402)

```python
# pip install "x402[httpx]"
import asyncio
import os
from eth_account import Account
from x402 import x402Client
from x402.http.clients import x402HttpxClient
from x402.mechanisms.evm import EthAccountSigner
from x402.mechanisms.evm.exact.register import register_exact_evm_client

async def main():
    client = x402Client()
    account = Account.from_key(os.environ["EVM_PRIVATE_KEY"])
    register_exact_evm_client(client, EthAccountSigner(account))
    async with x402HttpxClient(client) as http:
        response = await http.get(
            "https://bookproof-api.sn95wjq846.chatgpt.site/api/v1/market-health",
            params={"market": "Fed rate cut July 2026"},
        )
        print(response.json())

asyncio.run(main())
```

## MCP

```json
{
  "mcpServers": {
    "bookproof": {
      "type": "streamable-http",
      "url": "https://bookproof-api.sn95wjq846.chatgpt.site/mcp"
    }
  }
}
```

Tools: `resolve_market` (free), `market_health` (0.005 USDC), and `execution_quote` (0.015 USDC).

## Machine-readable integration

- [OpenAPI 3.1](https://bookproof-api.sn95wjq846.chatgpt.site/openapi.json)
- [MCP manifest](https://bookproof-api.sn95wjq846.chatgpt.site/server.json)
- [x402 discovery](https://bookproof-api.sn95wjq846.chatgpt.site/.well-known/x402)
- [Anonymous demand report](https://bookproof-api.sn95wjq846.chatgpt.site/api/v1/demand-report)

The current `chatgpt.site` hostname may be blocked by Cloudflare on some networks in China. That is tracked as a production-access risk; it does not change the Base settlement or anonymous revenue classification rules.

Bookproof is independent and unaffiliated with Polymarket. Output is informational and may be stale or incomplete. It is not investment advice, an order, or a promise of execution or profit.
