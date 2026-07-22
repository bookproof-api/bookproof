# Historical large-order stress test

> **Demo only — not a live quote, customer request, external test, paid call, or revenue.**
> The snapshot below was recorded from Polymarket's public Gamma and CLOB APIs at
> `2026-06-16T21:32:48.674Z`. It must not be used as a current trading input.

This example is retained only as a large-order stress test. It is not the
primary Bookproof sales demo because its `500,000 USDC` budget is outside the
50–500 USDC target-customer range and the recorded market had fees disabled.
For the small-bot example, see the [100 USDC fee-enabled historical
demo](./EXAMPLE_QUOTE_100_USDC.md).

## Scenario

- Market: [US x Iran permanent peace deal by June 15, 2026?](https://polymarket.com/event/us-x-iran-permanent-peace-deal-by/us-x-iran-permanent-peace-deal-by-june-15-2026-734-856-129)
- Outcome: `YES`
- Action: `BUY / FOK`
- All-in budget: `500,000 USDC`
- FOK worst-price cap: `0.995`
- Public snapshot provenance: [elizaOS recorded Polymarket fixture](https://github.com/elizaOS/eliza/blob/a7fc4bc8b296b74da97338bad56783715d6048b9/plugins/plugin-polymarket/src/__fixtures__/polymarket-real.recorded.json)
- Snapshot market volume: approximately `81.07M USDC`
- Snapshot market liquidity: approximately `4.28M USDC`
- Fee state: fees disabled in the recorded market metadata

## Ask levels consumed by the quote

The CLOB response stores asks highest-first. Bookproof sorts them cheapest-first
before walking the book.

| Ask | Visible shares | Shares used | Cost used |
| ---: | ---: | ---: | ---: |
| `0.995` | `344,089.41` | `344,089.41` | `342,368.96295 USDC` |
| `0.996` | `633,346.81` | `158,264.09342` | `157,631.03705 USDC` |

The remaining recorded asks were not needed for this budget:
`0.997 × 582,464.20`, `0.998 × 868,876.81`, and
`0.999 × 17,929,695.68` shares.

## Quote result

| Field | Result |
| --- | ---: |
| Best ask | `0.995` |
| Filled shares if the cap allowed both levels | `502,353.50342` |
| VWAP | `0.995315` |
| Worst fill | `0.996` |
| Modeled taker fee | `0.00 USDC` |
| All-in cost | `500,000.00 USDC` |
| VWAP price impact vs. best ask | `3.17 bps` |
| Worst-level impact vs. best ask | `10.05 bps` |
| Visible depth in the recorded five ask levels | `20,358,472.91 shares` / `20,332,804.23 USDC` |
| Insufficient depth | `false` |

A naive `budget / bestAsk` calculation predicts `502,512.56281` shares. Walking
the book returns `502,353.50342`, or `159.05939` fewer shares, because the first
ask level cannot absorb the whole budget.

## Decision

**Reject this FOK order at a `0.995` cap.** Depth is sufficient, but the exact
budget reaches the `0.996` ask, which is worse than the configured cap. A bot
that only reads `bestAsk=0.995` could incorrectly approve the order.

Bookproof does not place the order. It returns the deterministic fields needed
for the caller to allow, resize, re-quote, or reject the action.

## Try one public sample

Send a Polymarket market URL, outcome, and BUY/FOK budget, and Bookproof can
return one free sample quote showing VWAP, worst fill, fees, price impact, and
insufficient-depth status. No wallet keys or credentials required.

[Request a sample quote](https://github.com/bookproof-api/bookproof/issues/new?template=request-sample-quote.yml)
