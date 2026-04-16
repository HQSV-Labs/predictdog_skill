---
name: predictdog
description: "Trade on Polymarket, view portfolio, check PnL, and search prediction markets via the Predictdog API. Use when the user wants to: search markets (e.g. 'find BTC markets'), place trades (e.g. 'buy $10 Yes on Trump wins'), view their portfolio or positions, check PnL/analytics, manage open orders, claim resolved positions, or interact with predictdog.xyz. Requires PREDICTDOG_API_KEY configured in environment or provided by user."
---

# Predictdog Skill

Trade prediction markets and manage your portfolio via the Predictdog API.

## Setup

One value required before any API call:
- `PREDICTDOG_API_KEY` — user's API key

Check for this as an environment variable first. If not found, ask the user:
> "Please provide your Predictdog API key. You can generate one at predictdog.xyz → Settings → API Keys."

**Base URL is fixed:** `https://api.predictdog.xyz` — do not ask the user for this.

## Authentication

All requests require the header:
```
x-api-key: <PREDICTDOG_API_KEY>
```

## New User / Not Set Up

If the user doesn't have an account or API key yet, or if any API call returns a wallet setup error (`setupStatus` ≠ `COMPLETED`), direct them to the website:

> "To get started, visit **predictdog.xyz** to create an account and deposit funds. Once set up, go to Settings → API Keys to generate your API key."

If the user has an API key but `POST /api/trade/readiness` returns an error:

| Error kind | Meaning | Response to user |
|-----------|---------|-----------------|
| `WALLET_NOT_INITIALIZED` | Wallet not set up | "Your wallet isn't set up yet. Visit predictdog.xyz to complete setup." |
| `INSUFFICIENT_BALANCE` | No USDC to trade | "Your balance is $0. Deposit funds at predictdog.xyz → Wallet → Deposit." |
| `APPROVALS_PENDING` | Contract approvals needed | "Please complete wallet setup at predictdog.xyz before trading." |

## Common Workflows

### 1. Search Markets
```
GET /api/markets/search?q=<query>
```
Returns matching events with `markets[].clobTokenIds` needed for trading.
Show title, outcomes, prices, and volume.

### 2. View Portfolio
```
GET /api/trade/open-positions
GET /api/auth/me
```
Positions include `cashPnl`, `percentPnl`, `currentPrice`, `avgPrice`, `size`.
`/api/auth/me` gives wallet address and USDC balance (`proxyBalanceUsd`).

### 3. Check PnL
Quick summary: use `summary.totalPnl` from `GET /api/trade/open-positions`.
Historical analytics:
1. `GET /api/auth/me` → extract `user.wallet.proxyWalletAddress`
2. `GET /api/analytics/user/:address/portfolio-analytics`

### 4. Place a Trade

To find the `tokenId` for trading:
1. Search with `GET /api/markets/search?q=<query>`
2. From results: `markets[outcomeIndex].clobTokenIds[outcomeIndex]` is the `tokenId`
   - `outcomes[0]` corresponds to `clobTokenIds[0]`, etc.

Before submitting:
- Check readiness: `POST /api/trade/readiness`
- Handle readiness errors (see New User / Not Set Up above)
- Show confirmation to user and wait for explicit approval
- **Never trade without explicit user confirmation**

```
POST /api/trade/order
Body: { tokenId, side, orderType, amount, limitPrice? }
```

### 5. Cancel an Order
```
GET /api/trade/open-orders   ← list first
POST /api/trade/order/cancel
Body: { orderId }
```
Confirm with user before cancelling.

### 6. View Open Orders
```
GET /api/trade/open-orders
```

### 7. Claim Resolved Positions
```
GET /api/trade/claimable-positions
POST /api/trade/claim-batch
```

## Restrictions

- **No transfers**: Never call deposit or withdraw endpoints
  - `/api/wallet/deposit-plans`
  - `/api/wallet/withdraw-plans`
  - `/api/turnkey/withdraw/*`
- Always confirm before placing or cancelling any order
- For deposits and withdrawals, always redirect to predictdog.xyz

## API Reference

See [references/api.md](references/api.md) for complete endpoint details and response types.
See [references/examples.md](references/examples.md) for sample interactions including new user onboarding.
