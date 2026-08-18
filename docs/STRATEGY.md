# Trading strategy

## 1. Evidence boundary

Bonereaper's public profile has shown high-frequency BTC and ETH activity across 5-minute, 15-minute, hourly, and 4-hour markets; repeated partial fills; variable sizing; and occasional holdings on both outcomes. These observations suggest automated execution and inventory management, but they do not disclose the trader's model.

The design below borrows observable execution characteristics without claiming to reproduce Bonereaper's private strategy.

## 2. Initial scope

Start with BTC 5-minute markets only. Add BTC 15-minute markets as an independently evaluated strategy after the 5-minute system passes dry-run, shadow, and limited-live validation.

## 3. Settlement probability

Estimate the probability that the exact contract settles UP:

```text
p_up = P(S_T > K | S_t, K, time remaining, volatility, price path, order book)
```

Required features include:

- exact contract reference price;
- settlement-source price and timestamp;
- time remaining;
- realized volatility over several horizons;
- return and acceleration;
- distance from reference normalized by volatility;
- Binance/settlement-source basis;
- Polymarket order-book imbalance, depth, spread, and recent fills; and
- optional Markov or hidden-Markov regime state.

A Markov persistence probability is a feature. It is not automatically the contract's settlement probability.

## 4. Conservative net expected value

For an UP purchase:

```text
net_ev_up = p_lower - executable_up_ask - fee - slippage - latency_buffer
```

For DOWN:

```text
net_ev_down = (1 - p_upper) - executable_down_ask - fee - slippage - latency_buffer
```

`p_lower` and `p_upper` are conservative confidence bounds. Enter only when the applicable net EV exceeds `NET_EDGE_MIN` after all costs.

Never use midpoint prices as executable prices. Walk visible depth for taker simulations and model queue position conservatively for maker simulations.

## 5. Execution

Use:

- post-only orders when the signal is likely to survive queue time;
- IOC/FAK orders only when the edge exceeds fees and adverse-selection allowance; and
- SKIP when either data or executable edge is inadequate.

Scale in through at most three risk-approved tranches. Never add simply because a position is losing.

## 6. Both-side inventory

Holding both sides is permitted only for:

1. executable complementary arbitrage after all costs;
2. controlled reversal of a prior directional position; or
3. market-making inventory governed by explicit limits.

Unequal UP and DOWN holdings must be treated as net directional exposure. Buying the opposite side is not automatically a hedge or arbitrage.

## 7. Position sizing

Use fixed $1 orders during validation. Kelly sizing remains disabled until probability calibration and net out-of-sample EV have been established over a large sample.

If later enabled, use no more than 0.10–0.25 fractional Kelly based on a conservative probability bound, subject to tighter portfolio and liquidity caps.

## 8. Initial $100 risk limits

| Control | Limit |
|---|---:|
| Single order | $1.00 |
| One market | $1.50 |
| Aggregate BTC exposure | $2.00 |
| Concurrent markets | 2 |
| Daily loss stop | $3.00 |
| Daily drawdown | 3% |
| Weekly drawdown | 7% |
| Stale settlement feed | 2 seconds |
| Rejected orders per hour | 5 |

Any missing reference price, stale feed, material feed divergence, clock drift, reconciliation error, or breached limit must result in SKIP, CANCEL_ALL, or HALT.

## 9. Long-shot module

Cheap 3–5 cent outcomes are a separate convex strategy, not part of the core model. Keep the module disabled until it has its own backtest, calibration report, risk budget, and approval.

## 10. Evaluation

Measure:

- net P&L after realistic fees and fills;
- Brier score and log loss;
- calibration by probability bucket;
- net EV versus realized outcome;
- fill rate and adverse selection;
- performance by price, time remaining, volatility, and edge bucket;
- maximum drawdown; and
- stability across walk-forward folds.

Win rate alone is not an edge.

