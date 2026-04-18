# Predictdog API Reference

Base URL: `https://api.predictdog.xyz`
Auth header: `x-api-key: $PREDICTDOG_API_KEY`

---

## Auth / User

### GET /api/auth/me
Get current user info and wallet balance.

**Response:**
```json
{
  "ok": true,
  "user": {
    "id": 1,
    "wallet": {
      "proxyWalletAddress": "0x...",
      "proxyBalanceUsd": 42.50,
      "setupStatus": "COMPLETED"
    }
  }
}
```
Use `proxyWalletAddress` for analytics endpoints. Use `proxyBalanceUsd` for balance display.

If `wallet` is null or `setupStatus` ≠ `COMPLETED`, direct user to predictdog.xyz to complete onboarding.

---

## Trading

### POST /api/trade/readiness
Check if user is ready to trade before placing orders. Must include the trade details.

**Request body:**
```json
{
  "venue": "POLYMARKET",
  "trade": {
    "tokenId": "string",
    "side": "BUY" | "SELL",
    "orderType": "MARKET" | "LIMIT",
    "amount": 10.00,
    "limitPrice": 0.65,
    "strategyContext": {
      "definitionId": "polymarket-btc-5m-directional",
      "executionScopeKey": "btc-updown-5m-1710000000",
      "eventSlug": "btc-updown-5m-1710000000",
      "marketId": "market-id",
      "outcomeLabel": "Up",
      "riskConfig": {
        "takeProfitPrice": 0.65,
        "stopLossPrice": 0.35
      },
      "metadata": {
        "surface": "crypto-recurring",
        "asset": "BTC",
        "interval": "5m",
        "roundStartSec": 1710000000
      }
    }
  }
}
```
`limitPrice` is required only for LIMIT orders.
`strategyContext` is optional. Use it for recurring crypto BUY orders when the trade should be tracked as a recurring crypto strategy entry.

`strategyContext.riskConfig` supports:
```json
{
  "takeProfitPrice": 0.65,
  "stopLossPrice": 0.35
}
```

Constraints:
- TP/SL is supported for recurring crypto BUY orders only.
- `takeProfitPrice` and `stopLossPrice` must each be in `(0,1)`.
- If both are present, `takeProfitPrice` must be greater than `stopLossPrice`.

**Success:** `{ "ok": true }`

**Failure:** `{ "ok": false, "kind": "...", "error": "..." }`

| kind | Action |
|------|--------|
| `WALLET_NOT_INITIALIZED` | → predictdog.xyz to complete setup |
| `INSUFFICIENT_BALANCE` | → predictdog.xyz → Wallet → Deposit |
| `APPROVALS_PENDING` | → predictdog.xyz to approve contracts |

### POST /api/trade/order
Place a trade order.

**Request body:**
```json
{
  "tokenId": "string",     // CLOB token ID for the outcome
  "side": "BUY" | "SELL",
  "orderType": "MARKET" | "LIMIT",
  "amount": 10.00,         // BUY: USDC amount, SELL: number of shares
  "limitPrice": 0.65,      // Required for LIMIT orders (0-1 range)
  "strategyContext": {
    "definitionId": "polymarket-btc-5m-directional",
    "executionScopeKey": "btc-updown-5m-1710000000",
    "eventSlug": "btc-updown-5m-1710000000",
    "marketId": "market-id",
    "outcomeLabel": "Up",
    "riskConfig": {
      "takeProfitPrice": 0.65,
      "stopLossPrice": 0.35
    },
    "metadata": {
      "surface": "crypto-recurring",
      "asset": "BTC",
      "interval": "5m",
      "roundStartSec": 1710000000
    }
  }
}
```

Field notes:
- `amount` keeps the standard semantics: BUY = USDC amount, SELL = shares.
- `strategyContext` is optional and intended for recurring crypto BUY orders.
- For ordinary Polymarket events, omit `strategyContext`.
- For user-facing Polymarket summaries, format share prices in cents (`¢`) instead of dollars.

**Success response:**
```json
{ "ok": true, "orderType": "MARKET", "result": { ... } }
```

**Error response:**
```json
{
  "ok": false,
  "kind": "INSUFFICIENT_BALANCE" | "NO_LIQUIDITY" | "ORDERBOOK_MISSING" | "CLOB_RATE_LIMITED",
  "error": "Human readable message",
  "retryAfterMs": 5000
}
```

### POST /api/trade/order/cancel
Cancel an open order.

**Request body:**
```json
{ "orderId": "string" }
```

**Success response:**
```json
{ "ok": true, "makerAddress": "0x...", "orderId": "string" }
```

### GET /api/trade/open-positions
Get current portfolio with PnL.

**Response:**
```json
{
  "ok": true,
  "makerAddress": "0x...",
  "positions": [
    {
      "venue": "polymarket",
      "title": "Will BTC exceed $100k by end of 2025?",
      "outcome": "Yes",
      "size": 50.0,
      "avgPrice": 0.55,
      "currentPrice": 0.72,
      "cashPnl": 8.50,
      "percentPnl": 30.9,
      "initialValue": 27.50,
      "currentValue": 36.00,
      "canSell": true
    }
  ],
  "summary": {
    "totalPositions": 3,
    "totalPnl": 12.40,
    "winningPositions": 2,
    "losingPositions": 1
  }
}
```

### GET /api/trade/open-orders
Get unfilled limit orders.

**Response:**
```json
{
  "ok": true,
  "orders": [
    {
      "venue": "polymarket",
      "orderId": "string",
      "side": "BUY",
      "price": 0.60,
      "originalSize": 20.0,
      "remainingSize": 20.0,
      "createdAt": "2025-01-01T00:00:00Z",
      "canCancel": true
    }
  ],
  "warnings": []
}
```

### GET /api/trade/claimable-positions
Get positions eligible for payout redemption (resolved markets).

### POST /api/trade/claim
Claim payout for a resolved position.

### POST /api/trade/claim-batch
Batch claim all claimable positions.

---

## Markets

### GET /api/markets/search?q=<query>
Search prediction markets by keyword. Supports any language — input is resolved to English keywords via LLM before searching.

**Query params:**
- `q` (required): free-text search query (English or other languages)

**Response:**
```json
{
  "ok": true,
  "query": "Bitcoin",
  "rawQuery": "比特币",
  "results": [
    {
      "venue": "polymarket",
      "id": "event-id",
      "slug": "will-btc-exceed-100k",
      "title": "Will BTC exceed $100k in 2025?",
      "volume": 2100000,
      "markets": [
        {
          "id": "market-id",
          "question": "Will BTC exceed $100k in 2025?",
          "outcomes": ["Yes", "No"],
          "prices": [0.72, 0.28],
          "clobTokenIds": ["token-id-yes", "token-id-no"]
        }
      ]
    },
    {
      "venue": "predict",
      "id": "predict-market-id",
      "slug": "btc-above-100k",
      "title": "BTC Above $100k",
      "question": "Will Bitcoin be above $100k?",
      "outcomes": ["Yes", "No"]
    }
  ]
}
```
**Trading from search results:**
- `outcomes[i]` maps to `clobTokenIds[i]` — use `clobTokenIds[i]` as the `tokenId` when placing an order for outcome `i`.
- Example: to buy "Yes" (index 0), use `markets[0].clobTokenIds[0]` as `tokenId` with `side: "BUY"`.

**Recurring crypto trading from search results:**
- Queries like `BTC 5m up`, `ETH 15m down`, or `SOL daily up` may resolve to recurring crypto rounds.
- If the matched event is a recurring crypto round and the user is placing a BUY order, attach a recurring `strategyContext`.
- If the user also specified TP/SL, place those values inside `strategyContext.riskConfig`.

### GET /api/events/aggregated
Browse top events across Polymarket and Kalshi.

**Response:**
```json
[
  {
    "id": "agg_...",
    "title": "2024 US Presidential Election",
    "volume": 1500000,
    "polymarket": { "price": 0.54, "url": "https://polymarket.com/event/...", "volume": 1000000 },
    "kalshi": { "price": 0.52, "url": "https://kalshi.com/markets/...", "volume": 500000 },
    "spread": 0.02,
    "roi": 0.037
  }
]
```

---

## Analytics / PnL

### GET /api/analytics/user/:address/portfolio-analytics
Historical PnL and trading statistics. Requires auth (`x-api-key` header).

**Path param:** `address` = user's proxy wallet address (get from `/auth/me`)

**Response:**
```json
{
  "ok": true,
  "walletAddress": "0x...",
  "analytics": {
    "currentPnl": 12.40,
    "realizedPnlAllTime": 85.20,
    "totalBoughtAllTime": 350.00,
    "cumulativeExecutedVolumeAllTimeUsd": 700.00,
    "historicalWinRate": 0.62,
    "wins": 18,
    "losses": 11,
    "closedPositionsCount": 29,
    "marketsTradedAllTime": 22,
    "computedAt": "2025-01-15T10:00:00Z"
  }
}
```

---

## Wallet / Balance

Balance is available via `/auth/me` → `user.wallet.proxyBalanceUsd` (USDC.e on Polygon).

### GET /api/funder/pol-balance
Get POL (gas token) balance.

---

## Restricted Endpoints (do not call)

The following are off-limits for this skill:
- `POST /api/wallet/deposit-plans` — deposits
- `GET /api/wallet/deposit-plans/*` — deposit status
- `POST /api/wallet/withdraw-plans` — withdrawals
- `POST /api/turnkey/withdraw/*` — Turnkey withdrawals
