# Polymarket BTC Agent Blueprint

A security-first blueprint for a Bonereaper-inspired Polymarket BTC Up/Down trading system with:

- calibrated settlement-probability modelling;
- fee-, spread-, slippage-, and latency-aware entry filters;
- deterministic risk and execution controls;
- realistic dry-run and shadow-mode validation;
- daily Hermes reviews delivered through Telegram; and
- explicit, hash-bound approval before any proposed configuration change.

This repository is a strategy and operating blueprint. It is not proof of profitability and does not contain a production trading implementation.

## Recommended architecture

```text
Market data -> Probability model -> Net-EV filter -> Risk engine
            -> Execution engine -> Immutable journal
            -> Hermes review -> Telegram proposal -> Owner approval
            -> Paper-test gate -> Versioned deployment
```

The execution service and review agent must be isolated:

- The trading service may access wallet-signing and CLOB credentials.
- Hermes may access only sanitized journals and proposal files.
- Hermes must never access the private key or apply live changes directly.

## Repository contents

- [`docs/STRATEGY.md`](docs/STRATEGY.md) — trading model and risk design.
- [`docs/OPERATIONS.md`](docs/OPERATIONS.md) — wallet, dry-run, review, approval, and rollout process.
- [`prompts/hermes-trading-review.md`](prompts/hermes-trading-review.md) — copy-paste Hermes instruction.
- [`config/trading.env.example`](config/trading.env.example) — trading-service settings.
- [`config/review.env.example`](config/review.env.example) — secret-isolated Hermes settings.

## Safe starting point

1. Use a dedicated wallet with a limited test balance.
2. Start with BTC 5-minute markets only.
3. Run fixed $1 simulated orders.
4. Collect at least 2,000–5,000 eligible observations across at least 30 days.
5. Run shadow mode for 7–14 additional days.
6. Enable $1 live orders only after all acceptance gates pass.

Do not enable Kelly sizing, ETH, multiple timeframes, long-shot bets, and automated parameter deployment at the same time.

## Important warning

Prediction-market trading can lose the entire amount committed to a position. Public account activity does not reveal a trader's private algorithm, and this design must not be presented as a reconstruction of Bonereaper's system.

