# Polymarket Arbitrage Bot · Polymarket Trading Bot · @distict-baguette Fork

**Polymarket arbitrage bot** and **Polymarket copytrading bot** for automated prediction-market trading. This **Polymarket trading bot** that trades Polymarket’s 15-minute Up/Down markets (e.g. BTC, ETH) using the CLOB API, WebSocket orderbook, and an adaptive price predictor.

---


## What this bot does

Automated **Polymarket** trading on **15-minute Up/Down markets**. It uses a price predictor to choose direction, places a first-side limit at best ask, then hedges with a second-side limit at `0.98 − firstSidePrice`. Built with TypeScript and Polymarket’s CLOB API. Suitable as a **Polymarket arbitrage bot** or **Polymarket copytrading bot** for 15m markets.

## About the developer

If have any questions, contact here:  [Telegram](https://t.me/@crewsxdev).

## Proof of work

Bot logs from live runs: [logs](https://github.com/CrewSX/polymarket-trading-bot/tree/main/logs).

### Video of analytics for https://polymarket.com/@distinct-baguette

https://github.com/user-attachments/assets/b534ed0b-4d7f-46aa-851a-31abf609ee8b

---

### Backtest Results

```
markets_count:     177
markets_replayed:  177
total_entries:     108
total_dip_orders:  0
total_risk_sells:  66
skipped_no_pred:   0
skipped_low_conf:  68
skipped_no_entry:  0
skipped_startup:   69
total_pnl:         1938.00
stats:             win_rate=23.73% avg_win=+46.14 avg_loss=+0.00 max_dd=0.00
config:            {'order_size': 100.0, 'min_order_size': 5.0, 'ask_offset': 0.0, 'bid_offset': 0.0, 'max_spread': 0.02, 'immediate_buy_enabled': True, 'entry_buy_price': 0.6, 'delayed_entry_buy': 0.7, 'delay_seconds': 0.0, 'dip_buy_enabled': False, 'dip_buy_price': 0.7, 'risk_sell_enabled': True, 'risk_sell_price': 0.2, 'model_15m_path': 'prediction_15m/models/btc_15min_updown_lgbm.pkl', 'model_table': 'price_cache', 'model_limit_1s': 30000, 'min_signal_probability': 0.52, 'table': 'price_cache'}
daily_stats:
  date         markets  up  down   pnl       win_rate  avg_cost/market  total_cost
  2026-03-18     93   81    12  +1070.00    24.73%            41.94      3900.00
  2026-03-19     84   58    26   +868.00    22.62%            30.71      2580.00
overall_cost_stats:
  avg_cost_per_market: 36.61
  total_cost:          6480.00
  overall_win_rate:    23.73%
  up_trades:           139
  down_trades:         38
  pnl_per_market:      +10.95

*** TOTAL PNL: +1938.00 ***
```

---

## Overview

- **Strategy**: Predict Up/Down from live orderbook via an adaptive price predictor; buy the predicted side at best ask (GTC), then place the opposite side at `0.98 − firstSidePrice` (GTC).
- **Markets**: Configurable list (e.g. `btc`, `eth`); slugs are resolved as `{market}-updown-15m-{startOf15mUnix}` via Polymarket Gamma API.
- **Stack**: TypeScript, Node (or Bun), `@polymarket/clob-client`, WebSocket orderbook, Ethers.js for allowances/redemption.

## Requirements

- Node.js 18+ (or Bun)
- Polygon wallet with USDC
- RPC URL for Polygon (e.g. Alchemy) for allowances and redemption

## Install

```bash
git clone https://github.com/CrewSX/polymarket-trading-bot.git
cd polymarket-trading-bot
npm install
```

## Configuration

Copy the example env and set at least `PRIVATE_KEY` and `COPYTRADE_MARKETS`:

```bash
cp .env.temp .env
```

| Variable | Description | Default |
|----------|-------------|---------|
| `PRIVATE_KEY` | Wallet private key | **required** |
| `COPYTRADE_MARKETS` | Comma-separated markets (e.g. `btc`) | `btc` |
| `COPYTRADE_SHARES` | Shares per side per trade | `5` |
| `COPYTRADE_TICK_SIZE` | Price precision | `0.01` |
| `COPYTRADE_PRICE_BUFFER` | Price buffer for execution | `0` |
| `COPYTRADE_WAIT_FOR_NEXT_MARKET_START` | Wait for next 15m boundary before starting | `false` |
| `COPYTRADE_MAX_BUY_COUNTS_PER_SIDE` | Max buys per side per market (0 = no cap) | `0` |
| `CHAIN_ID` | Chain ID (Polygon) | `137` |
| `CLOB_API_URL` | Polymarket CLOB API base URL | `https://clob.polymarket.com` |
| `RPC_URL` / `RPC_TOKEN` | RPC for allowances/redemption | — |
| `BOT_MIN_USDC_BALANCE` | Min USDC to start | `1` |
| `LOG_DIR` / `LOG_FILE_PREFIX` | Log directory and file prefix | `logs` / `bot` |

API credentials are created on first run and stored in `src/data/credential.json`.

## Usage

**Run the Polymarket trading bot**

```bash
npm start
# or: bun src/index.ts
```

**Redemption**

```bash
# Auto-redeem resolved markets (holdings file)
npm run redeem:holdings
# or: bun src/auto-redeem.ts [--dry-run] [--clear-holdings] [--api] [--max N]

# Redeem by condition ID
npm run redeem
# or: bun src/redeem.ts [conditionId] [indexSets...]
bun src/redeem.ts --check <conditionId>
```

**Development**

```bash
npx tsc --noEmit
bun --watch src/index.ts
```

## Project structure

| Path | Role |
|------|------|
| `src/index.ts` | Entry: credentials, CLOB, allowances, min balance, start `CopytradeArbBot`. |
| `src/config/index.ts` | Loads `.env` and exposes config (chain, CLOB, copytrade, logging). |
| `src/order-builder/copytrade.ts` | **CopytradeArbBot**: 15m slug resolution, WebSocket orderbook, predictor → first-side buy + second-side hedge; state in `src/data/copytrade-state.json`. |
| `src/providers/clobclient.ts` | Polymarket CLOB client singleton (credentials + `PRIVATE_KEY`). |
| `src/providers/websocketOrderbook.ts` | WebSocket to Polymarket CLOB market channel; best bid/ask by token ID. |
| `src/utils/pricePredictor.ts` | **AdaptivePricePredictor**: direction, confidence, signal (BUY_UP / BUY_DOWN / HOLD). |
| `src/utils/redeem.ts` | CTF redemption, resolution checks, auto-redeem from holdings or API. |
| `src/security/allowance.ts` | USDC and CTF approvals. |
| `src/data/token-holding.json` | Token holdings for redemption (generated). |
| `src/data/copytrade-state.json` | Per-slug state (prices, timestamps, buy counts). |

## Risk and disclaimer

Trading prediction markets involves significant risk. This software is provided as-is. Use at your own discretion and only with funds you can afford to lose.

## License

ISC
