# Operating procedure

## 1. Provide the Hermes prompt

Give Hermes the contents of `prompts/hermes-trading-review.md`. Do not append wallet credentials, seed phrases, API secrets, Telegram tokens, or private logs.

## 2. Set up a dedicated Polymarket wallet

1. Create a new signer used only for this bot.
2. Store its recovery material offline.
3. Use the current Polymarket CLOB V2 deposit-wallet flow for a new API integration (`POLY_1271`, signature type `3`), unless an existing account requires another documented wallet type.
4. Deploy the deposit wallet with the supported SDK/relayer.
5. Deposit only the planned test bankroll.
6. Configure the required pUSD and conditional-token allowances.
7. Synchronize CLOB balance and allowance state.
8. Derive L2 CLOB credentials locally with the official SDK.
9. Keep the signer and CLOB credentials exclusively in the trading-service environment.

Never paste a private key into Hermes or Telegram. CLOB L2 credentials authenticate requests, but order creation still requires valid local signing.

## 3. Configure isolated environments

Copy:

```text
config/trading.env.example -> private trading service .env
config/review.env.example  -> Hermes review service .env
```

Do not place the production secret files inside a shared repository. Ensure the Hermes process cannot read the trading environment or private logs.

## 4. Dry-run validation

### Connectivity and correctness

Verify market discovery, outcome-token mapping, timestamps, reference price, resolution source, fee schedule, tick size, minimum order size, and feed timestamps.

### Realistic fills

- Maker fills require the market to trade through the simulated quote and must allow partial fills.
- Taker fills must walk live depth and include fees and slippage.
- Midpoint fills are prohibited.

### Failure injection

Test disconnections, stale feeds, clock drift, missing reference prices, duplicate fills, partial-fill restarts, balance mismatches, stale orders, loss-limit breaches, and corrupted journals.

### Acceptance gates

Require:

- 2,000–5,000 eligible observations;
- at least 30 calendar days across several volatility regimes;
- zero unresolved reconciliation errors;
- positive net simulated EV after realistic execution costs;
- probability calibration better than the market-price baseline;
- acceptable drawdown; and
- stable walk-forward results.

Then run 7–14 days of shadow mode before limited $1 live trading.

## 5. Daily review and approval

The deterministic engine generates a sanitized report. Hermes interprets it and delivers the result through a scheduled Telegram job.

The report must include data integrity, opportunities, orders, fills, maker/taker mix, fees, slippage, P&L, drawdown, Brier score, log loss, calibration, bucketed performance, execution errors, and the current configuration hash.

### Approval protocol

A configuration proposal must contain:

- unique proposal ID;
- exact parameter diff;
- current and proposed hashes;
- statistical rationale and sample size;
- walk-forward results;
- risks and rollback configuration; and
- required paper-test period.

Valid commands are:

```text
APPROVE <proposal_id> <proposed_hash>
REJECT <proposal_id>
PROMOTE <proposal_id> <proposed_hash>
```

Approval is valid only from the allowlisted Telegram chat, for the exact unexpired proposal and hash. Approval starts a paper-test probation; it does not immediately deploy to live trading. Promotion requires a second explicit command after probation results are reported.

Never permit Telegram-driven increases to bankroll, order size, exposure, loss limits, Kelly activation, `LIVE_TRADING`, wallet addresses, or credentials.

## 6. Rollout

1. BTC 5-minute dry run.
2. BTC 5-minute shadow mode.
3. BTC 5-minute limited live mode at $1 maximum.
4. Independently test BTC 15-minute markets.
5. Independently test ETH.
6. Consider multi-timeframe allocation only after each component has a validated record.
7. Consider long-shot and Kelly modules last.

