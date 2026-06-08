# RapidX CLI 与 MCP 参考

> 34 个 MCP 工具与 41 个 CLI 能力的完整参考。

CLI 与 MCP Server 共用同一套 core executor。`rapidx mcp serve` 暴露的能力与 CLI 子命令完全对应，不存在并行实现。

---

## 响应信封

每个 CLI 命令和 MCP 工具均返回：

```json
{
  "ok": true,
  "status": "PASS",
  "data": {},
  "auditId": "optional-audit-id",
  "evidence": [
    {
      "source": "real_tool_call",
      "toolOrCommandEvidence": "rapidx market get-ticker",
      "timestamp": "2026-06-08T00:00:00.000Z"
    }
  ]
}
```

错误响应可能包含 `details` 对象：`upstreamStatus`、`upstreamCode`、`upstreamMessage`、`hint`、`attempts`。

| status | 含义 |
|--------|------|
| `PASS` | 成功完成 |
| `FAIL` | 失败（本地、上游或未知） |
| `BLOCKED` | 本地校验拦截（如缺少预览 token） |
| `NOT_VERIFIED` | 无法确认真实 API 调用 — 视为未完成 |
| `EXPECTED_ERROR` | 自检探测以预期方式失败 |
| `INVALID_INPUT` | 本地入参校验失败（未调用 RapidX） |
| `BUSINESS_ERROR` | RapidX 以业务错误拒绝请求 |
| `NOT_FOUND` | 查询后未找到目标资源 |
| `PERMISSION_SCOPE_ERROR` | 凭证缺少所需账户或 portfolio 权限 |

---

## 预览、确认与自动化

所有写操作均需**先预览再提交**。预览与提交必须使用同一通道（同为 CLI 或同为 MCP）。

### 手动确认模式（默认）

智能体创建预览，向用户展示结果，确认后再提交。

预览响应包含：

```json
{
  "previewId": "rpv_xxx",
  "businessParams": { "symbol": "...", "side": "BUY", ... },
  "riskNotes": [],
  "expiresAt": "2026-06-08T00:05:00.000Z",
  "confirmation": {
    "submitToken": "confirm_rpv_xxx",
    "requiredFields": ["previewId", "continueConsentId"]
  }
}
```

预览记录约 5 分钟后失效。过期或提交参数与预览不一致时返回 `BLOCKED`，需重新创建预览。

### 自动化模式

自动化模式允许智能体在不需要每次对话确认的情况下提交预览。**仅在用户在对话中明确启用后方可使用。**

在**预览请求**（而非提交请求）中启用：

```json
{
  "automationMode": true,
  "automationConsentText": "I enable RapidX automation mode for BINANCE_PERP_BTC_USDT with maxNotional 100 and accept automated preview-submit execution.",
  "automationScope": "single-preview"
}
```

规则：
- `automationConsentText` 必须传递用户的原始授权文字，不得改写或扩大范围。
- 必须包含确切的 symbol 和 `maxNotional`。
- 文字中必须明确表达自动化意图（`automation`、`automated` 或 `自动`）。
- 提交仍需 `previewId` 和 `continueConsentId`；自动化改变的是确认模式，不改变提交接口形态。

接受后，预览响应包含：

```json
{
  "automation": {
    "enabled": true,
    "confirmationMode": "automation-preview",
    "scope": "single-preview",
    "maxNotional": "100"
  }
}
```

---

## 工具参考

### 诊断与发现

| CLI | MCP 工具 | 说明 |
|-----|---------|------|
| `rapidx schema --json` | `rapidx/tools` | 列出全部能力、schema 和预览要求 |
| `rapidx self-check --read-only --json` | `rapidx/self-check` | 验证只读连通性 |
| `rapidx update check --json` | `rapidx/update/check` | 检查版本发布状态与升级建议 |
| `rapidx doctor --json` | 仅 CLI | 本地诊断：版本、凭证、调用方式 |
| `rapidx auth check` | 仅 CLI | 凭证解析检查（不打印完整密钥） |
| `rapidx invocation check` | 仅 CLI | 校验智能体调用方式是否合规 |
| `rapidx config list/get/set/unset` | 仅 CLI | 本地配置辅助（不替代运行时环境变量） |

### 行情数据

Ticker、orderbook、klines 直接调用 Binance/OKX 公开 API；资金费率、标记价格、品种信息走 RapidX API。

| CLI | MCP 工具 | 数据来源 |
|-----|---------|---------|
| `rapidx market get-ticker` | `rapidx/market/get-ticker` | Binance / OKX 公开 |
| `rapidx market get-orderbook` | `rapidx/market/get-orderbook` | Binance / OKX 公开 |
| `rapidx market get-klines` | `rapidx/market/get-klines` | Binance / OKX 公开 |
| `rapidx market get-open-interest` | `rapidx/market/get-open-interest` | Binance / OKX 公开 |
| `rapidx market get-funding-rate` | `rapidx/market/get-funding-rate` | RapidX API |
| `rapidx market get-mark-price` | `rapidx/market/get-mark-price` | RapidX API |
| `rapidx market get-symbol-info` | `rapidx/market/get-symbol-info` | RapidX API |

```bash
rapidx market get-ticker        --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-open-interest --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-orderbook     --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json
rapidx market get-klines        --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json
rapidx market get-funding-rate  --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-mark-price    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-symbol-info   --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### 账户

| CLI | MCP 工具 |
|-----|---------|
| `rapidx account overview` | `rapidx/account/overview` |
| `rapidx account balance` | `rapidx/account/balance` |
| `rapidx account set-position-mode` | `rapidx/account/set-position-mode` |

```bash
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # portfolio 凭证
rapidx account balance --input '{"mode":"account"}' --json     # 需要 account 级凭证
```

使用 portfolio 凭证调用 `mode:"account"` 返回 `PERMISSION_SCOPE_ERROR`。

### 订单

| CLI | MCP 工具 |
|-----|---------|
| `rapidx order preview` | `rapidx/order/preview` |
| `rapidx order place-preview` | `rapidx/order/place-preview` |
| `rapidx order place` | `rapidx/order/place` |
| `rapidx order amend-preview` | `rapidx/order/amend-preview` |
| `rapidx order amend` | `rapidx/order/amend` |
| `rapidx order cancel-preview` | `rapidx/order/cancel-preview` |
| `rapidx order cancel` | `rapidx/order/cancel` |
| `rapidx order get` | `rapidx/order/get` |
| `rapidx order list` | `rapidx/order/list` |
| `rapidx order history` | `rapidx/order/history` |

```bash
# 预览
rapidx order place-preview --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "positionSide": "LONG",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true
}' --json

# 提交 — continueConsentId = 预览响应中的 submitToken 值
rapidx order place --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "positionSide": "LONG",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true,
  "previewId": "rpv_xxx",
  "continueConsentId": "confirm_rpv_xxx"
}' --json
```

使用 `quantity` 填基础资产或合约张数；现货市价买入按金额使用 `amount`，不得同时传两者。

撤单在交易所层可能异步，需轮询 `rapidx order get` 确认终态。

### 仓位

| CLI | MCP 工具 |
|-----|---------|
| `rapidx position list` | `rapidx/position/list` |
| `rapidx position history` | `rapidx/position/history` |
| `rapidx position close` | `rapidx/position/close` |
| `rapidx position set-leverage` | `rapidx/position/set-leverage` |

```bash
# 设置杠杆
rapidx trade preview --input '{"targetCapabilityId":"position.set-leverage","symbol":"BINANCE_PERP_BTC_USDT","leverage":5}' --json

# 平仓（不接受 side/quantity；部分平仓使用 reduce-only 订单）
rapidx trade preview --input '{"targetCapabilityId":"position.close","symbol":"BINANCE_PERP_BTC_USDT","reduceOnly":true,"maxNotional":"100"}' --json
```

无持仓时返回 `BLOCKED`（`NO_POSITION_TO_CLOSE`）。`maxNotional` 是安全上限，非目标平仓金额。

### 算法单

| CLI | MCP 工具 |
|-----|---------|
| `rapidx algo list` | `rapidx/algo/list` |
| `rapidx algo place` | `rapidx/algo/place` |
| `rapidx algo amend` | `rapidx/algo/amend` |
| `rapidx algo cancel` | `rapidx/algo/cancel` |

TPSL 示例：

```bash
rapidx trade preview --input '{
  "targetCapabilityId": "algo.place",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "SELL",
  "orderType": "MARKET",
  "maxNotional": "100",
  "clientOrderId": "tpsl-001",
  "algoType": "TPSL",
  "conditionType": "ENTIRE_CLOSE_POSITION",
  "stopLossPrice": "62000",
  "takeProfitPrice": "70000"
}' --json
```

### 实盘验证

| CLI | MCP 工具 |
|-----|---------|
| `rapidx trade verify-live` | `rapidx/trade/verify-live` |
| `rapidx self-check trade-verify` | — | *（CLI 兼容别名，等同于 verify-live）* |

```bash
rapidx trade verify-live --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "maxNotional": "10",
  "clientOrderId": "verify-live-001",
  "explicitUserConsent": true,
  "acceptedRiskText": "I authorize a real verification order for BINANCE_PERP_BTC_USDT BUY maxNotional 10 with cancel cleanup."
}' --json
```

`rapidx/trading-verification` 作为兼容别名保留。

---

## 故障排查

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| `RCORE01001` | 凭证缺失 | 配置 `LTP_ACCESS_KEY` 和 `LTP_SECRET_KEY` |
| `RCORE01003` | API 地址缺失 | 配置 `LTP_API_HOST` |
| `RCORE00002` | 订单 ID 格式错误 | 使用 16 位 RapidX `orderId` |
| `RCORE01004` | 凭证 scope 不匹配 | 使用具备所需权限的凭证 |
| `RCORE22004` | 资源不存在 | 检查 `orderId`、`clientOrderId` 或 symbol |
| `RCLI20002` | 缺少预览/同意 token | 先运行匹配的预览命令 |
| `RCLI22001` | RapidX 上游业务错误 | 读取上游消息，调整请求 |
| `RCORE23002` | 网络或超时 | 检查连通性后重试 |
| `RCORE23003` | 速率限制 | 稍后重试，避免轮询循环 |

---

## 升级

```bash
rapidx update check --json
npm install -g @liquiditytech/rapidx-cli@latest
rapidx schema --json
rapidx self-check --read-only --json
```

---

**English**: [02-mcp-reference.md](./02-mcp-reference.md)
