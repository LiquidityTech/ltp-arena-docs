# RapidX AI Quantitative Trading Competition — Participant Kit

> Build, test, and submit your AI trading agent against the LTP RapidX simulation environment.

---

## What You Get

| | |
|---|---|
| **Simulated portfolio** | UAT mock matching engine — virtual capital, real market data, no real assets at risk |
| **MCP tools + CLI** | Account / position / order / algo-order / market — all available to your agent |
| **Three integration paths** | MCP server · Protocolized CLI (any exec-only agent) · Raw REST/WebSocket |
| **Safety by default** | All writes require preview; trading must be explicitly enabled and parameters confirmed |

The CLI and MCP server are distributed through **npm** as `@liquiditytech/rapidx-cli`.  
Skills are distributed from `https://github.com/LiquidityTech/ltp-rapidx-skill`.

---

## Prerequisites

- Node.js **20** or later
- npm
- A competition Portfolio Key (`LTP_ACCESS_KEY` + `LTP_SECRET_KEY`)
- `LTP_API_HOST` — the API host provided by the competition organizer (**required, no default**)
- (Optional) An MCP-capable agent host: Claude Desktop, Cursor, Codex, etc.

---

## Step 1 — Install Skills (if your agent supports them)

Skills guide the agent through CLI install, credential setup, MCP config, self-check, and trading workflows.

| Agent host | Install command |
|------------|-----------------|
| **Codex** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a codex -y` |
| **Claude Code** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a claude-code -y` |
| **Cursor** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a cursor -y` |
| **OpenCode** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a opencode -y` |
| **OpenClaw** | `openclaw skills install ltp-rapidx-config && openclaw skills install ltp-rapidx-trading` |
| **Hermes** | `hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config && hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading` |

After installing, ask your agent to follow `ltp-rapidx-config`. It will complete the rest of setup automatically.

If your agent does **not** support skills, continue to Step 2 and configure manually.

---

## Step 2 — Install the CLI

```bash
npm install -g @liquiditytech/rapidx-cli
```

Verify:

```bash
rapidx --version
rapidx schema --json
```

---

## Step 3 — Configure Credentials

```bash
export LTP_ACCESS_KEY="your_access_key"
export LTP_SECRET_KEY="your_secret_key"
export LTP_API_HOST="<host-provided-by-organizer>"
```

> `LTP_API_HOST` has no default. RapidX returns `RCORE01003` if it is missing.  
> Never paste keys into public chats, repositories, screenshots, or logs.

---

## Step 4 — Choose Your Integration Path

### Path A — MCP-Capable Agent (Recommended)

Add to your agent's MCP config:

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "your_access_key",
        "LTP_SECRET_KEY": "your_secret_key",
        "LTP_API_HOST": "<host-provided-by-organizer>"
      }
    }
  }
}
```

Restart the agent host. MCP tools become available with the `rapidx/` prefix, e.g. `rapidx/market/get-ticker`.

### Path B — CLI-Only Agent / Scripts

```bash
# Read-only self-check
rapidx self-check --read-only --json

# Market data
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# Account snapshot
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json

# Order preview → then submit
rapidx order place-preview --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"60000","quantity":"0.001","maxNotional":"100","clientOrderId":"agent-001","postOnly":true}' --json
```

Every response uses the stable JSON envelope:

```json
{
  "ok": true,
  "status": "PASS",
  "data": {},
  "evidence": [{ "source": "real_tool_call", "toolOrCommandEvidence": "rapidx ..." }]
}
```

CLI-only mode uses `--input '<json>'` or `--input @/path/to/file.json`. The CLI rejects complex shell composition (`cd ... && node ...`, `sh -c`, command substitution).

### Path C — Raw REST / WebSocket

Direct HTTP and WebSocket access without the CLI. See [`rapidx-api.md`](./rapidx-api.md).

---

## Step 5 — Run Self-Check

```bash
rapidx auth check
rapidx self-check --read-only --json
```

Each check item reports **PASS** / **EXPECTED_ERROR** / **FAIL** / **NOT_VERIFIED**.

---

## Write Operations — Preview Required

All write operations (orders, position changes, leverage, algo orders) require a **preview step** before submission.

```
# 1. Preview → get previewId + continueConsentId
rapidx order place-preview --input '{...}' --json

# 2. Submit with same params + previewId + continueConsentId
rapidx order place --input '{...,"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json

# 3. Confirm state
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

Preview and submit **must use the same runtime** (both CLI or both MCP — do not mix).

---

## Tool Catalogue

| Category | CLI | MCP tool |
|----------|-----|----------|
| **Diagnostics** | `rapidx self-check --read-only` · `rapidx schema` · `rapidx doctor` | `rapidx/self-check` · `rapidx/tools` |
| **Market (7)** | `rapidx market get-ticker` · `get-orderbook` · `get-klines` · `get-funding-rate` · `get-mark-price` · `get-symbol-info` · `get-open-interest` | `rapidx/market/*` |
| **Account (3)** | `rapidx account overview` · `balance` · `set-position-mode` | `rapidx/account/*` |
| **Order (7+)** | `rapidx order place-preview` · `place` · `amend-preview` · `amend` · `cancel-preview` · `cancel` · `get` · `list` · `history` | `rapidx/order/*` |
| **Position (4)** | `rapidx position list` · `history` · `close` · `set-leverage` | `rapidx/position/*` |
| **Algo Order (4)** | `rapidx algo list` · `place` · `amend` · `cancel` | `rapidx/algo/*` |
| **Trade utils** | `rapidx trade preview` · `verify-live` | `rapidx/trade/*` |

Full specs in [`rapidx-api.md`](./rapidx-api.md).

---

## Safety Model

- **Preview before every write**: all order, position, leverage, and algo writes require a preview step. The preview never submits a real order.
- **No real assets at risk**: competition runs on the UAT mock matching engine.
- **Secrets never leak**: keys are masked in all output, logs, and reports.
- **Status confirmation after every write**: check `rapidx order get` or `rapidx position list` after submits — no blind retries.
- **`LTP_API_HOST` is required**: missing host returns `RCORE01003`.

---

## What's Next

| Document | What's in it |
|----------|--------------|
| [`resources.md`](./resources.md) | Endpoints, npm packages, symbols, schedule, support |
| [`rapidx-api.md`](./rapidx-api.md) | Complete CLI + MCP + REST/WS reference |

---

## Support

- **Documentation**: <https://github.com/LiquidityTech/ltp-arena-docs>
- **Issue tracker**: <https://github.com/liquiditytech/rapidx-cli/issues>
- **Contact**: see `resources.md`

---

**中文版**: [README.zh-CN.md](./README.zh-CN.md)
