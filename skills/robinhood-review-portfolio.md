---
name: Review Robinhood crypto holdings and open orders
description: Read-only sweep of account, holdings, and open orders with current mark-to-market prices.
api: openapi/robinhood-crypto-trading-openapi.yml
operations: [getCryptoTradingAccount, getHoldings, getOrders, getBestBidAsk]
---

# Review Robinhood crypto holdings and open orders

Read-only. Uses the Robinhood Crypto Trading API (`https://trading.robinhood.com`) with the
`x-api-key` / `x-timestamp` / `x-signature` headers on every request.

## Steps

1. **Account** — `getCryptoTradingAccount` (`GET /api/v1/crypto/trading/accounts/`) for `status` and
   `buying_power`.
2. **Holdings** — `getHoldings` (`GET /api/v1/crypto/trading/holdings/`). Page with `limit` + `cursor`
   (follow `next` until null). Collect each `asset_code` and `total_quantity`.
3. **Mark to market** — for the held assets, `getBestBidAsk`
   (`GET /api/v1/crypto/marketdata/best_bid_ask/?symbol=<ASSET>-USD`, repeat `symbol` for each) and multiply
   quantity by price to value the position.
4. **Open orders** — `getOrders` (`GET /api/v1/crypto/trading/orders/?state=open`) to list working orders;
   page via `next`.

## Rules

- All four operations are read-only (`connected`/`read`) — no confirmation needed.
- Always exhaust `next` cursors before reporting totals so paginated results are complete.
