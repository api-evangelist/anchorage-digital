---
name: Get a quote and execute a trade
description: Request a quote and place a trade on Anchorage Digital Prime, handling quote expiry and credit limits.
api: openapi/anchorage-digital-openapi-original.json
operations: [requestQuote, acceptQuote, newOrderSingle, getOrderStatus]
---

# Execute a trade on Anchorage Digital

Base URL: `https://api.anchorage.com/v2`. Requires `Api-Access-Key` and the **Execute trades** permission.

## Steps
1. `requestQuote` — POST the trading pair and size to get a firm quote. Quotes expire; act promptly.
2. `acceptQuote` — accept the quote to execute at the quoted price. If the quote lapsed you receive `errorType: QuoteExpired` (HTTP 400) — request a fresh quote and retry.
3. Alternatively `newOrderSingle` — place an order directly without the quote/accept handshake.
4. `getOrderStatus` — poll by `orderId` until the order reaches a terminal state.

## Rules
- Check `getCreditLimit` / trading-account balances first if trading on credit.
- Sensitive trading writes may require Ed25519 signing (`Api-Signature` + `Api-Timestamp`).
- Retries: reuse idempotency where supported; back off on `429`/`5xx`; never blindly re-accept an expired quote.
- Read-only checks (`getOrderStatus`, list orders/trades) need **Read trade activity**.
