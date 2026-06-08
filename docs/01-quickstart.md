# Getting Started with RapidX

> Install the CLI, connect your agent, run your first trade — in under 10 minutes.

RapidX CLI (`@liquiditytech/rapidx-cli` v1.0.31) is the single entry point for market data, account reads, order management, positions, and algo orders. Starting the CLI with `rapidx mcp serve` exposes the same capabilities as structured MCP tools for any MCP-capable agent host.

---

## Before You Begin

| Requirement | Details |
|-------------|---------|
| Node.js | **20 or later** |
| npm | Any recent version |
| Credentials | `LTP_ACCESS_KEY`, `LTP_SECRET_KEY`, `LTP_API_HOST` — provided by the competition organizer |

`LTP_API_HOST` has no default. Missing it returns `RCORE01003`.

---

## Step 1 — Install the CLI

```bash
npm install -g @liquiditytech/rapidx-cli@latest
rapidx --version
```

If npm cannot find the package:

```bash
npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

---

## Step 2 — Configure Credentials

```bash
export LTP_ACCESS_KEY="<your-access-key>"
export LTP_SECRET_KEY="<your-secret-key>"
export LTP_API_HOST="<host-provided-by-organizer>"
```

> Never paste keys into public chats, repositories, screenshots, or logs.

Verify credential resolution (no full secrets printed):

```bash
rapidx auth check
```

---

## Step 3 — Choose Your Path

### Path A — MCP (Recommended)

For agents that natively support MCP: **Claude Code, Codex, Cursor, Continue, OpenCode**.

**Option 1: Install skills (let the agent do the rest)**

Skills guide the agent through the full setup automatically.

```bash
# Claude Code
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config --skill ltp-rapidx-trading -a claude-code -y

# Codex
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config --skill ltp-rapidx-trading -a codex -y

# Cursor
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config --skill ltp-rapidx-trading -a cursor -y

# OpenClaw
openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading

# Hermes
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading
```

After installing, tell your agent: *"Follow ltp-rapidx-config."* The skill configures MCP, sets credentials, and runs self-check.

**Option 2: Configure MCP manually**

Add to your agent's MCP config file (e.g. `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<your-access-key>",
        "LTP_SECRET_KEY": "<your-secret-key>",
        "LTP_API_HOST": "<host-provided-by-organizer>"
      }
    }
  }
}
```

Restart the agent host. MCP tools become available under the `rapidx/` namespace.

### Path B — CLI Only

For exec-only agents and shell scripts. No MCP required.

```bash
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # "portfolio" = competition account type
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

All commands return a stable JSON envelope — see [`02-mcp-reference.md`](./02-mcp-reference.md) §Response envelope.

---

## Step 4 — Run Self-Check

```bash
rapidx self-check --read-only --json
```

Each check item reports `PASS` / `EXPECTED_ERROR` / `FAIL` / `NOT_VERIFIED`. Do not proceed to trading until all critical items pass.

Expected output:

```json
{
  "ok": true,
  "status": "PASS",
  "data": {
    "scope": "quick",
    "status": "PASS",
    "checks": [
      { "name": "discovery",     "status": "PASS" },
      { "name": "public-market", "status": "PASS" }
    ]
  }
}
```

---

## Step 5 — Query Market Data

```bash
# Live ticker
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# Klines (1h, last 100 bars)
rapidx market get-klines --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json

# Order book (20 levels)
rapidx market get-orderbook --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json

# Symbol rules — always check before trading
rapidx market get-symbol-info --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

`get-symbol-info` returns `minNotional`, `lotSize`, `tickSize`, and `contractSize` — read it before sizing any order.

---

## Step 6 — Place Your First Order

All write operations follow **preview → submit**. The preview validates trading rules and returns a token; the submit uses that token.

```bash
# 1. Preview
rapidx order place-preview --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "positionSide": "LONG",
  "orderType": "LIMIT",
  "price": "60000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "my-first-order",
  "postOnly": true
}' --json
```

> `positionSide` is **required** — competition accounts default to `BOTH` (hedge) mode, which requires `LONG` or `SHORT` on every order.
> `postOnly: true` means the order will be rejected if it would immediately fill (protects against accidental market execution).

Preview response — copy `submitToken` for the next step:

```json
{
  "ok": true,
  "status": "PASS",
  "data": {
    "previewId": "rpv_xxx",
    "confirmation": { "submitToken": "confirm_rpv_xxx" }
  }
}
```

```bash
# 2. Submit — same params + previewId + continueConsentId
#    continueConsentId = the submitToken value from the preview response above
rapidx order place --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "positionSide": "LONG",
  "orderType": "LIMIT",
  "price": "60000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "my-first-order",
  "postOnly": true,
  "previewId": "rpv_xxx",
  "continueConsentId": "confirm_rpv_xxx"
}' --json

# 3. Confirm state
rapidx order get --input '{"clientOrderId":"my-first-order"}' --json
```

> Preview and submit must use the **same runtime** (both CLI or both MCP — do not mix).
> Preview records expire after ~5 minutes.

---

## What's Next

| Document | Contents |
|----------|----------|
| [`02-mcp-reference.md`](./02-mcp-reference.md) | Complete CLI + MCP tool reference, automation mode, troubleshooting |
| [`03-advanced-api.md`](./03-advanced-api.md) | Direct REST/WebSocket access for custom integrations |
| [`resources.md`](./resources.md) | Endpoints, npm package, symbol list, schedule, support |

---

**中文版**: [01-quickstart.zh-CN.md](./01-quickstart.zh-CN.md)
