# 100 USDC historical execution-quote demo

> **Demo only — not a live quote, customer request, external test, paid call, or revenue.**
> The public order-book snapshot was recorded at `2026-07-19T12:07:44.108Z`.
> It must not be used as a current trading input.

This example is intentionally sized for a small Polymarket bot. It shows how
book walking and a fee-enabled market can reduce a headline edge before an
immediate BUY/FOK order reaches the order path.

## Scenario

- Market: `Spain vs. Argentina: Argentina O/U 1.5`
- Market slug: `fifwc-esp-arg-2026-07-19-team-total-away-1pt5`
- Outcome: `Over`
- Action: `BUY / FOK`
- All-in budget: `100.00 USDC`
- Illustrative strategy fair probability: `0.31`
- Illustrative decision rule: require at least `5%` expected return after
  visible-book execution cost
- Snapshot provenance: [WorldCup2026 public Polymarket snapshot](https://github.com/yanyw/WorldCup2026/blob/bda1b3f7200c73ea1e068933a669c2b2f9348928/yanyawei/20260719/data/polymarket_snapshot_0719.json)

The `0.31` probability and `5%` threshold are demo inputs, not a forecast,
recommendation, or profit claim.

## Fee model recorded in the snapshot

The market metadata says fees are enabled and records `rate=0.05`,
`exponent=1`, and `takerOnly=true`. For each BUY fill, the calculation uses
Polymarket's documented structure:

`fee = shares × rate × price × (1 - price)`

See [Polymarket's fee documentation](https://docs.polymarket.com/trading/fees).
The recorded fee schedule is used here because fee parameters can vary by
market and time.

## Ask levels consumed

| Ask | Visible shares | Shares used | Raw cost | Modeled fee | All-in cost |
| ---: | ---: | ---: | ---: | ---: | ---: |
| `0.280000` | `46.94` | `46.940000` | `13.143200` | `0.473155` | `13.616355` |
| `0.290000` | `11,774.45` | `287.662614` | `83.422158` | `2.961487` | `86.383645` |
| **Total** |  | **`334.602614`** | **`96.565358`** | **`3.434642`** | **`100.000000`** |

The next visible levels were not needed for the 100 USDC budget:
`0.30 × 27,101.89`, `0.31 × 18,527.45`, `0.32 × 19,872.55`,
`0.33 × 3,572.00`, and `0.34 × 18.18` shares.

## Full-budget quote result

| Field | Result |
| --- | ---: |
| Best ask | `0.280000` |
| Filled shares | `334.602614` |
| Raw book-walk cost | `96.565358 USDC` |
| Modeled taker fee | `3.434642 USDC` |
| All-in cost | `100.000000 USDC` |
| VWAP | `0.288597` |
| Worst fill | `0.290000` |
| VWAP price impact vs. best ask | `307.04 bps` |
| Insufficient depth | `false` |

## Headline profit versus after-cost profit

A headline calculation that assumes every share fills at the best ask and
ignores fees reports:

`(100 / 0.28) × (0.31 - 0.28) = 10.714286 USDC`

Walking the book, charging the recorded fee schedule, and constraining the
total spend to 100 USDC gives:

`334.602614 × 0.31 - 100 = 3.726810 USDC`

| Profit view | Expected profit | Expected return |
| --- | ---: | ---: |
| Headline best-ask calculation | `10.714286 USDC` | `10.71%` |
| Full-budget after execution cost | `3.726810 USDC` | `3.73%` |

The visible execution costs reduce the modeled profit by `6.987475 USDC`, or
about `65.22%` of the headline result.

## Decision: resize

Under the illustrative `5%` minimum after-cost return rule, the full 100 USDC
order should not be accepted. It is not rejected for insufficient depth;
instead, it is **resized to the first ask level**:

| Resized order | Result |
| --- | ---: |
| Shares | `46.940000` |
| Raw cost | `13.143200 USDC` |
| Modeled fee | `0.473155 USDC` |
| All-in budget | `13.616355 USDC` |
| VWAP / worst fill | `0.280000` |
| Expected profit at the illustrative `0.31` fair value | `0.935045 USDC` |
| Expected return after execution cost | `6.87%` |
| Final status | **RESIZE** |

This is the decision layer Bookproof sells: not a prediction, but a
deterministic answer about whether a proposed budget still satisfies the
caller's own threshold after visible depth and fees.

## Request one free sample quote

[Request one free sample quote](https://github.com/bookproof-api/bookproof/issues/new?template=request-sample-quote.yml)

Give us one market URL, outcome, and a 50–500 USDC BUY/FOK budget. We’ll return
one free sample showing VWAP, worst fill, fees, price impact, and whether the
order should be accepted, resized, or rejected. No wallet credentials required.
