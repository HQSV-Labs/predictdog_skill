# Predictdog Skill — Example Interactions

---

## New User Onboarding

**User:** "I want to trade on Predictdog" (no API key configured)

```
To get started with Predictdog:

1. Visit predictdog.xyz to create an account
2. Deposit USDC into your wallet (Wallet → Deposit)
3. Go to Settings → API Keys to generate your API key
4. Set PREDICTDOG_API_KEY and PREDICTDOG_API_URL in your environment

Once set up, you can search markets, view your portfolio, and trade — all from here.
```

---

**User:** has API key, but wallet not set up or balance is $0

When `POST /api/trade/readiness` returns an error:
```
Your wallet isn't ready for trading yet.

Please visit predictdog.xyz to:
- Complete wallet setup (if first time)
- Deposit USDC (Wallet → Deposit)

Once you have funds, come back and I'll execute the trade for you.
```

---

## Searching Markets

**User:** "search for bitcoin markets"

1. `GET /api/markets/search?q=bitcoin`
2. Display results:
   ```
   Found 5 markets for "bitcoin":

   1. Will BTC exceed $100k in 2025?
      Yes: 0.72 | No: 0.28 | Volume: $2.1M

   2. BTC above $80k by June?
      Yes: 0.45 | No: 0.55 | Volume: $850K
   ```
3. Ask: "Would you like to trade any of these?"

---

## Viewing Portfolio

**User:** "show my portfolio" / "what positions do I have?"

1. `GET /api/trade/open-positions`
2. `GET /auth/me` (for balance)
3. Display:
   ```
   Portfolio Summary
   Balance: $42.50 USDC
   Total PnL: +$12.40

   Positions (3):
   ✅ BTC > $100k (Yes) — 50 shares @ $0.72 | +$8.50 (+30.9%)
   ✅ Trump wins 2024 (Yes) — 30 shares @ $0.85 | +$4.20 (+16.5%)
   ❌ ETH > $5k (Yes) — 20 shares @ $0.30 | -$0.30 (-5.0%)
   ```

---

## Checking PnL

**User:** "what's my PnL?" / "how am I doing?"

1. `GET /api/trade/open-positions` → `summary.totalPnl`
2. `GET /auth/me` → `user.wallet.proxyWalletAddress`
3. `GET /api/analytics/user/:address/portfolio-analytics`
4. Display:
   ```
   PnL Report
   Open positions: +$12.40
   All-time realized: +$85.20
   Win rate: 62% (18W / 11L)
   Total volume traded: $700
   Markets traded: 22
   ```

---

## Placing a Trade

**User:** "buy $20 Yes on BTC above 100k"

1. Search: `GET /api/markets/search?q=BTC above 100k`
2. From results: `markets[0].clobTokenIds[0]` = tokenId for "Yes", `clobTokenIds[1]` = tokenId for "No"
3. Check readiness: `POST /api/trade/readiness`
4. Check balance: `GET /auth/me` → `proxyBalanceUsd`
5. **Show confirmation before submitting:**
   ```
   Trade Confirmation
   Market: Will BTC exceed $100k in 2025?
   Side: BUY Yes
   Amount: $20.00 USDC
   Current price: $0.72
   Est. shares: ~27.8
   Balance after: $22.50

   Confirm? (yes/no)
   ```
6. On confirmation: `POST /api/trade/order`
   ```json
   { "tokenId": "...", "side": "BUY", "orderType": "MARKET", "amount": 20.00 }
   ```

---

## Managing Open Orders

**User:** "show my open orders" / "cancel my limit orders"

1. `GET /api/trade/open-orders`
2. Display:
   ```
   Open Orders (2):
   1. BUY Yes @ $0.60 — 20 shares remaining (BTC > $100k)
   2. SELL No @ $0.40 — 15 shares remaining (Trump wins)
   ```
3. If cancel requested, confirm: "Cancel order #1 for BTC > $100k? (yes/no)"
4. `POST /api/trade/order/cancel` with `{ "orderId": "..." }`

---

## Claiming Resolved Markets

**User:** "any markets I can claim?" / "redeem my winnings"

1. `GET /api/trade/claimable-positions`
2. If found:
   ```
   Claimable Positions:
   - Trump wins 2024 (Yes): 30 shares → ~$30.00

   Claim all? (yes/no)
   ```
3. `POST /api/trade/claim-batch`

---

## Error Handling

| Error kind | Message to user |
|-----------|----------------|
| `INSUFFICIENT_BALANCE` | "Not enough USDC. Your balance is $X." |
| `NO_LIQUIDITY` | "No liquidity at that price. Try a market order instead." |
| `ORDERBOOK_MISSING` | "Market not found or unavailable." |
| `CLOB_RATE_LIMITED` | "Too many requests. Please wait a moment." |
