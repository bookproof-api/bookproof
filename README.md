# Bookproof

Bookproof is a read-only pre-trade execution guard for Polymarket taker bots. It turns a Polymarket URL, slug, CLOB token ID, or market search into fee- and depth-aware execution data before a bot submits a BUY or FOK order.

> Official Bookproof technical project. Public responses may be AI-assisted and are supervised by the project owner.

## Request one free sample quote

**[Request one free sample quote](https://github.com/bookproof-api/bookproof/issues/new?template=request-sample-quote.yml)**

Give us one market URL, outcome, and a 50–500 USDC BUY/FOK budget. We’ll
return one free sample showing VWAP, worst fill, fees, price impact, and whether
the order should be accepted, resized, or rejected. No wallet credentials
required. The sample is for evaluation only; continued automated calls use the
paid API.

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

## 100 USDC historical execution demo

**Historical demo; not a live quote, customer test, paid call, or revenue.** A
fee-enabled public Polymarket order-book snapshot recorded on `2026-07-19`
shows how a seemingly profitable small order can become too weak after fees and
book walking:

| BUY/FOK scenario | Result |
| --- | ---: |
| Market / outcome | Spain vs. Argentina: Argentina O/U 1.5 / `Over` |
| All-in budget | `100.00 USDC` |
| Best ask | `0.280000` |
| Book-walk VWAP | `0.288597` |
| Worst fill | `0.290000` |
| Modeled taker fee | `3.434642 USDC` |
| Price impact vs. best ask | `307.04 bps` |
| Strategy headline profit | `10.714286 USDC` |
| Expected profit after execution cost | `3.726810 USDC` |
| Decision with a 5% minimum after-cost return | **RESIZE** |

The illustrative strategy input is a `0.31` fair probability; it is not a
forecast or trading claim. The first ask level alone preserves a modeled
`6.87%` after-cost return, while filling the entire budget falls to `3.73%`.
See the [levels, fee formula, calculations, provenance, and resize result](./EXAMPLE_QUOTE_100_USDC.md).

## Large-order stress test

The earlier `500,000 USDC` example remains available only as a large-order
stress test. Its recorded market had fees disabled, so it is not the primary
small-bot sales example. The order must cross from `0.995` to `0.996` and is
therefore rejected at a `0.995` FOK price cap. See the [stress-test levels and
provenance](./EXAMPLE_QUOTE.md).

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
