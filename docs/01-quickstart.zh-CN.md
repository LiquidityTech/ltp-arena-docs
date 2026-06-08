# RapidX 快速上手

> 安装 CLI、接入智能体、完成首单 — 10 分钟内跑通。

RapidX CLI（`@liquiditytech/rapidx-cli` v1.0.31）是行情查询、账户读取、订单管理、仓位和算法单的统一入口。通过 `rapidx mcp serve` 启动后，同一套能力以 MCP 工具的形式对支持 MCP 的智能体宿主开放。

---

## 开始前的准备

| 要求 | 说明 |
|------|------|
| Node.js | **20 或更高版本** |
| npm | 任意较新版本 |
| 凭证 | `LTP_ACCESS_KEY`、`LTP_SECRET_KEY`、`LTP_API_HOST` — 由主办方提供 |

`LTP_API_HOST` 无默认值，缺失时返回 `RCORE01003`。

---

## 第一步 — 安装 CLI

```bash
npm install -g @liquiditytech/rapidx-cli@latest
rapidx --version
```

若 npm 找不到包：

```bash
npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

---

## 第二步 — 配置凭证

```bash
export LTP_ACCESS_KEY="<your-access-key>"
export LTP_SECRET_KEY="<your-secret-key>"
export LTP_API_HOST="<主办方提供的API地址>"
```

> 切勿将密钥粘贴到公开聊天、仓库、截图或日志中。

验证凭证解析（不打印完整密钥）：

```bash
rapidx auth check
```

---

## 第三步 — 选择接入方式

### 方式 A — MCP（推荐）

适用于原生支持 MCP 的智能体宿主：**Claude Code、Codex、Cursor、Continue、OpenCode**。

**选项 1：安装 Skills（让智能体完成后续配置）**

Skills 会引导智能体完成全套安装配置。

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

安装完成后，告诉智能体：*「请按照 ltp-rapidx-config 操作。」* Skill 会自动完成 MCP 配置、凭证设置和自检。

**选项 2：手动配置 MCP**

在智能体的 MCP 配置文件（如 `claude_desktop_config.json`）中添加：

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<your-access-key>",
        "LTP_SECRET_KEY": "<your-secret-key>",
        "LTP_API_HOST": "<主办方提供的API地址>"
      }
    }
  }
}
```

重启宿主后，MCP 工具以 `rapidx/` 前缀对外可见。

### 方式 B — 仅 CLI

适用于 exec-only 智能体和 shell 脚本，无需 MCP。

```bash
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # "portfolio" = 大赛账户类型
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

所有命令返回稳定的 JSON 信封 — 详见 [`02-mcp-reference.zh-CN.md`](./02-mcp-reference.zh-CN.md)「响应信封」章节。

---

## 第四步 — 运行自检

```bash
rapidx self-check --read-only --json
```

每个检查项报告 `PASS` / `EXPECTED_ERROR` / `FAIL` / `NOT_VERIFIED`。关键项全部通过后再进行交易。

预期输出：

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

## 第五步 — 查询行情

```bash
# 实时 Ticker
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# K 线（1h，最近 100 根）
rapidx market get-klines --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json

# 订单簿（20 档）
rapidx market get-orderbook --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json

# 品种规则 — 下单前必查
rapidx market get-symbol-info --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

`get-symbol-info` 返回 `minNotional`（最小名义价值）、`lotSize`（数量步长）、`tickSize`（价格步长）和 `contractSize`（合约规模），下单前务必查询。

---

## 第六步 — 提交首单

所有写操作遵循**预览 → 提交**流程：预览校验交易规则并返回 token，提交时携带该 token。

```bash
# 1. 预览
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

> `positionSide` **必填** — 大赛账户默认为 `BOTH`（双向持仓）模式，每笔订单必须指定 `LONG` 或 `SHORT`。
> `postOnly: true` 表示若订单会立即成交则被拒绝（防止意外以市价成交）。

预览响应 — 记录 `submitToken`，下一步使用：

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
# 2. 提交 — 带原始参数 + previewId + continueConsentId
#    continueConsentId = 上一步预览响应中的 submitToken 值
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

# 3. 确认状态
rapidx order get --input '{"clientOrderId":"my-first-order"}' --json
```

> 预览与提交必须使用**同一通道**（同为 CLI 或同为 MCP，不得混用）。
> 预览记录约 5 分钟后失效。

---

## 下一步

| 文档 | 内容 |
|------|------|
| [`02-mcp-reference.zh-CN.md`](./02-mcp-reference.zh-CN.md) | 完整 CLI + MCP 工具参考、自动化模式、故障排查 |
| [`03-advanced-api.zh-CN.md`](./03-advanced-api.zh-CN.md) | 直接调用 REST/WebSocket（进阶集成） |
| [`resources.zh-CN.md`](./resources.zh-CN.md) | 端点、npm 包、品种列表、日程、支持 |

---

**English**: [01-quickstart.md](./01-quickstart.md)
