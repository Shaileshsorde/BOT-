# Binance Futures Testnet Trading Bot

A lightweight Python CLI application that places **MARKET**, **LIMIT**, and **STOP\_MARKET** orders on the Binance USDT-M Futures Testnet, with structured logging, robust error handling, and a clean layered architecture.

---

## Project Structure

```
trading_bot/
├── bot/
│   ├── __init__.py
│   ├── client.py          # Binance REST client (signing, HTTP, error handling)
│   ├── orders.py          # Order placement logic + OrderResult model
│   ├── validators.py      # Input validation (symbol, side, type, qty, price)
│   └── logging_config.py  # Rotating file + console logging
├── cli.py                 # CLI entry point (argparse, sub-commands: ping / account / order)
├── logs/
│   └── trading_bot.log    # Auto-created; sample included
├── requirements.txt
└── README.md
```

---

## Setup

### 1. Python version

Python **3.8 or newer** is required.

### 2. Get Testnet API credentials

1. Go to [https://testnet.binancefuture.com](https://testnet.binancefuture.com)
2. Sign in with GitHub
3. Navigate to **API Key** and generate a key pair
4. Copy your **API Key** and **Secret Key**

> The testnet resets periodically. If you see error `-2015` (Invalid API key), regenerate your credentials.

### 3. Install the only dependency

```bash
pip install requests
```

### 4. Set credentials as environment variables

**Windows (Command Prompt)**
```bat
set BINANCE_API_KEY=your_api_key_here
set BINANCE_API_SECRET=your_api_secret_here
```

**Windows (PowerShell)**
```powershell
$env:BINANCE_API_KEY = "your_api_key_here"
$env:BINANCE_API_SECRET = "your_api_secret_here"
```

**macOS / Linux**
```bash
export BINANCE_API_KEY="your_api_key_here"
export BINANCE_API_SECRET="your_api_secret_here"
```

You can also pass credentials directly on the command line via `--api-key` / `--api-secret`, but environment variables are preferred.

---

## How to Run

All commands are run from inside the `trading_bot/` folder:

```bash
cd trading_bot
```

### Check connectivity

```bash
python cli.py ping
```
```
────────────────────────────────────────────────────────────
  Server Connectivity Check
────────────────────────────────────────────────────────────
  Server time (ms) : 1752484322104

  ✔  Connection to Binance Futures Testnet is healthy.
```

---

### Check account balance

```bash
python cli.py account
```
```
────────────────────────────────────────────────────────────
  Account Information
────────────────────────────────────────────────────────────
  Total Wallet Balance   : 10000.00 USDT
  Available Balance      : 9942.65 USDT
  Total Unrealised PnL   : 0.00 USDT
  Can Trade              : True
```

---

### Place a MARKET order

```bash
# Market BUY — 0.001 BTC
python cli.py order --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001

# Market SELL — 0.01 ETH
python cli.py order --symbol ETHUSDT --side SELL --type MARKET --quantity 0.01
```

---

### Place a LIMIT order

```bash
# Limit BUY at $58,000 — GTC (default)
python cli.py order --symbol BTCUSDT --side BUY --type LIMIT --quantity 0.001 --price 58000

# Limit SELL at $70,000 — IOC
python cli.py order --symbol BTCUSDT --side SELL --type LIMIT --quantity 0.001 --price 70000 --tif IOC
```

Sample output:
```
────────────────────────────────────────────────────────────
  Order Request Summary
────────────────────────────────────────────────────────────
  Symbol         : BTCUSDT
  Side           : SELL
  Type           : LIMIT
  Quantity       : 0.001
  Limit Price    : 70000
  Time-In-Force  : GTC (default)

────────────────────────────────────────────────────────────
  Placing Order …
────────────────────────────────────────────────────────────

────────────────────────────────────────────────────────────
  Order Response
────────────────────────────────────────────────────────────
  Order ID       : 4022715892
  Symbol         : BTCUSDT
  Side           : SELL
  Type           : LIMIT
  Status         : NEW
  Orig Qty       : 0.001
  Executed Qty   : 0
  Limit Price    : 70000
  Time-In-Force  : GTC

  ✔  Order placed!  orderId=4022715892  status=NEW
```

---

### Place a STOP\_MARKET order (bonus)

```bash
# Stop-Market BUY — triggers when ETHUSDT reaches $3,200
python cli.py order --symbol ETHUSDT --side BUY --type STOP_MARKET --quantity 0.01 --stop-price 3200

# Stop-Market SELL (stop-loss) — triggers when BTCUSDT drops to $55,000
python cli.py order --symbol BTCUSDT --side SELL --type STOP_MARKET --quantity 0.001 --stop-price 55000
```

---

### Reduce-only order

```bash
python cli.py order --symbol BTCUSDT --side SELL --type MARKET --quantity 0.001 --reduce-only
```

---

## CLI Reference

```
usage: cli [-h] [--api-key KEY] [--api-secret SECRET] COMMAND ...

Commands:
  ping      Test connectivity to Binance Testnet
  account   Display account balance information
  order     Place a new futures order

Order flags:
  --symbol / -s     Trading pair (e.g. BTCUSDT)              [required]
  --side            BUY or SELL                              [required]
  --type / -t       MARKET | LIMIT | STOP_MARKET             [required]
  --quantity / -q   Order quantity in base asset             [required]
  --price / -p      Limit price (required for LIMIT)
  --stop-price      Stop trigger price (required for STOP_MARKET)
  --tif             Time-in-force: GTC | IOC | FOK  (default: GTC)
  --reduce-only     Mark as reduce-only order
```

---

## Logging

All activity is written to **`logs/trading_bot.log`** (auto-created on first run).

| Level   | Destination      | Content                                        |
|---------|------------------|------------------------------------------------|
| DEBUG   | File only        | Full request params, raw JSON API responses    |
| INFO    | Console + file   | Order summaries, status, connectivity events   |
| WARNING | Console + file   | Input validation failures                      |
| ERROR   | Console + file   | API errors, network failures                   |

Rotation: 5 MB per file, 3 backups kept.

---

## Error Handling

| Scenario                             | Behaviour                                          |
|--------------------------------------|----------------------------------------------------|
| Missing `BINANCE_API_KEY` / `SECRET` | Clear startup message, exit code 1                 |
| Bad symbol / side / type             | Validation error before API call, exit code 2      |
| Missing `--price` on LIMIT order     | Validation error, no API call made                 |
| Binance rejects the order            | Error code + message printed and logged            |
| Network unreachable                  | Human-readable message, logged as ERROR            |
| Request timeout (>10 s)              | TimeoutError surfaced, logged as ERROR             |

---

## Architecture

```
cli.py                    ← argparse, coloured output, exit codes
└── bot/
    ├── orders.py         ← validate → call client → return OrderResult
    ├── client.py         ← sign, HTTP, raise BinanceAPIError
    ├── validators.py     ← pure validation, no side effects
    └── logging_config.py ← one-time setup, shared across all layers
```

The CLI layer never touches the API directly — it only calls `place_order()`.  
The client layer knows nothing about CLI args or validation.  
This separation makes each layer independently testable.

---

## Assumptions

- **One-way position mode** (`positionSide=BOTH`) is assumed. If your account uses hedge mode, you'll need `LONG` / `SHORT` — extend `client.place_order` accordingly.
- Quantity precision must respect the symbol's lot-size filter. For BTCUSDT testnet, `0.001` is the standard minimum.
- Only `requests` is required — no Binance SDK is used. All signing is done with Python's stdlib `hmac` / `hashlib`.
- Log files in `logs/` are from live testnet sessions (Sessions 1–3) plus intentional error scenarios (Sessions 4–5).
