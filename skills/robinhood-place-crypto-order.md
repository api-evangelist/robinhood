---
name: Place a Robinhood crypto order
description: Check buying power and market price, then place a crypto order idempotently and confirm its state.
api: openapi/robinhood-crypto-trading-openapi.yml
operations: [getCryptoTradingAccount, getBestBidAsk, getEstimatedPrice, placeOrder, getOrder]
---

# Place a Robinhood crypto order

Use the Robinhood Crypto Trading API (`https://trading.robinhood.com`). Every request must carry
`x-api-key`, `x-timestamp`, and `x-signature` headers — the signature is a base64 Ed25519 signature over
`api_key + timestamp + path + method + body`.

## Steps

1. **Confirm the account** — `getCryptoTradingAccount` (`GET /api/v1/crypto/trading/accounts/`). Read
   `status` (must be `active`) and `buying_power`.
2. **Price the trade** — `getBestBidAsk` (`GET /api/v1/crypto/marketdata/best_bid_ask/?symbol=BTC-USD`) for
   the current quote, and/or `getEstimatedPrice` (`GET /api/v1/crypto/marketdata/estimated_price/`) with
   `side` and `quantity` to estimate the fill. Verify the notional fits `buying_power`.
3. **Place the order idempotently** — `placeOrder` (`POST /api/v1/crypto/trading/orders/`). Generate a fresh
   UUID for `client_order_id` and set `side`, `symbol`, `type`, and the matching config object
   (`market_order_config` / `limit_order_config` / `stop_loss_order_config` / `stop_limit_order_config`).
   Reuse the SAME `client_order_id` on any retry — Robinhood returns the original order instead of placing a
   duplicate.
4. **Confirm** — `getOrder` (`GET /api/v1/crypto/trading/orders/{id}/`) and poll `state`
   (`open` -> `partially_filled` -> `filled`, or `canceled` / `failed`).

## Rules

- This order is `physical`/`safety-critical` (moves real money) — require human confirmation before step 3.
- Never place a market order without pricing it first (step 2).
- On HTTP 400 check the auth headers; on 401 recompute the signature; on 429 back off and retry.
