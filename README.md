# Predictdog Skill

Trade on Polymarket, view your portfolio, and check PnL — all from your AI agent.

This skill connects to the [Predictdog](https://predictdog.xyz) API and works with **Claude Code** and **OpenClaw**.

## What You Can Do

- **Search markets** — "find BTC markets", "search Trump election"
- **View portfolio** — positions, PnL, open orders
- **Place trades** — "buy $20 Yes on BTC above 100k"
- **Trade recurring crypto rounds** — "buy BTC 5m up", optionally with `tp/sl`
- **Cancel orders** — list and cancel unfilled limit orders
- **Claim payouts** — redeem resolved market winnings

## Prerequisites

You need a Predictdog API key:

1. Sign up at [predictdog.xyz](https://predictdog.xyz)
2. Deposit funds into your wallet (Wallet → Deposit)
3. Go to **Settings → API Keys** and generate a key

Set it as an environment variable:

```bash
export PREDICTDOG_API_KEY=pd_pat_your_key_here
```

Or just tell your agent the key when it asks.

## Installation

### Claude Code

```bash
# Download and install
curl -L https://github.com/HQSV-Labs/predictdog_skill/releases/latest/download/predictdog-skill.skill \
  -o predictdog-skill.skill

# Unzip into Claude skills directory
unzip predictdog-skill.skill -d ~/.claude/skills/
```

Or clone and install manually:

```bash
git clone git@github.com:HQSV-Labs/predictdog_skill.git ~/.claude/skills/predictdog-skill
```

### OpenClaw

```bash
git clone git@github.com:HQSV-Labs/predictdog_skill.git ~/.openclaw/skills/predictdog-skill
```

Or via the OpenClaw CLI (if available):

```bash
openclaw skills install github:HQSV-Labs/predictdog_skill
```

## Usage

Once installed, just talk to your agent naturally:

```
"search for NBA finals markets"
"what's my portfolio?"
"buy $10 Yes on Spurs winning the NBA Finals"
"buy BTC 5m up"
"buy BTC 5m up tp 0.65 sl 0.35"
"show my open orders"
"what's my PnL?"
```

The agent will call the Predictdog API and handle everything — including confirming trades before executing them.

For recurring crypto BUY orders on Polymarket, the agent may attach a recurring `strategyContext` with optional TP/SL risk config so the trade is tracked as a recurring crypto strategy entry.

## API

This skill uses the Predictdog REST API at `https://api.predictdog.xyz`. See [`references/api.md`](references/api.md) for the full endpoint reference.

Deposits and withdrawals are intentionally disabled in this skill — use [predictdog.xyz](https://predictdog.xyz) for fund management.

## License

MIT
