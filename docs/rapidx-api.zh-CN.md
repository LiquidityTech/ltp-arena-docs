# RapidX API 参考 — 大赛版

> 完整 API 接口：CLI 命令 + MCP 工具 + REST + WebSocket。

三种通道共用同一套认证、错误码与安全策略。

| 通道 | 适用场景 |
|------|---------|
| **MCP 工具**（`rapidx mcp serve`） | 支持 MCP 的宿主：Claude / Cursor / Codex / Continue |
| **CLI 命令**（`rapidx ...`） | CLI-only 宿主、shell 脚本、CI |
| **REST + WebSocket** | 自定义代码、底层集成 |

---

## 1. 认证

### 必填环境变量

| 变量 | 是否必填 | 说明 |
|------|---------|------|
| `LTP_ACCESS_KEY` | 是 | Access Key |
| `LTP_SECRET_KEY` | 是 | Secret Key |
| `LTP_API_HOST` | **是 — 无默认值** | 主办方提供的 API 地址，缺失返回 `RCORE01003` |

### REST 签名算法

```
1. 将请求参数按 key 字母序排列并拼接：
   sorted_payload = "key1=val1&key2=val2&..."

2. 末尾追加 "&" + Unix 时间戳（秒）：
   payload = sorted_payload + "&" + timestamp
   （无参数时：payload = "&" + timestamp）

3. 用 Secret Key 做 HMAC-SHA256，取十六进制：
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

HTTP Header 要求：

| Header | 值 |
|--------|---|
| `X-MBX-APIKEY` | Access Key |
| `nonce` | Unix 时间戳（字符串） |
| `signature` | HMAC-SHA256 签名 |
| `Content-Type` | `application/json` |

### WebSocket 认证签名

与 REST 不同：

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

---

## 2. 响应信封

### CLI / MCP 信封

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
      "timestamp": "2026-06-02T00:00:00.000Z"
    }
  ]
}
```

| status | 含义 |
|--------|------|
| `PASS` | 成功完成 |
| `FAIL` | 失败（本地、上游或未知错误） |
| `BLOCKED` | 本地校验拦截（如缺少预览 token） |
| `NOT_VERIFIED` | 无法确认真实 API 调用 |
| `EXPECTED_ERROR` | 自检探测以预期方式失败 |
| `INVALID_INPUT` | 本地入参校验失败（未调用 RapidX） |
| `BUSINESS_ERROR` | RapidX 以业务错误拒绝请求 |
| `NOT_FOUND` | 查询后未找到目标资源 |
| `PERMISSION_SCOPE_ERROR` | 凭证缺少所需账户或 portfolio 权限 |

错误响应可能包含 `details` 对象：`upstreamStatus`、`upstreamCode`、`upstreamMessage`、`hint`、`attempts`。

### REST 信封

```json
{
  "code": 200000,
  "message": "Success",
  "data": {}
}
```

---

## 3. 预览-提交模式

所有写操作均需**先预览再提交**。

> 预览与提交**必须使用同一通道**（同为 CLI 或同为 MCP，不得混用）。

### 订单写操作流程

```bash
# 第一步：预览 — 返回 previewId + submitToken
rapidx order place-preview --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true
}' --json
```

预览响应：

```json
{
  "ok": true,
  "status": "PASS",
  "data": {
    "previewId": "rpv_xxx",
    "confirmation": {
      "submitToken": "confirm_rpv_xxx"
    }
  }
}
```

```bash
# 第二步：提交（带原始参数 + previewId + continueConsentId）
rapidx order place --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "orderType": "LIMIT",
  "price": "65000",
  "quantity": "0.001",
  "maxNotional": "100",
  "clientOrderId": "agent-001",
  "postOnly": true,
  "previewId": "rpv_xxx",
  "continueConsentId": "confirm_rpv_xxx"
}' --json

# 第三步：确认状态
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx position list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### 非订单写操作（杠杆、持仓模式、平仓）

通过 `rapidx trade preview` 传入 `targetCapabilityId`：

```bash
# 设置杠杆
rapidx trade preview --input '{
  "targetCapabilityId": "position.set-leverage",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "leverage": 5
}' --json

# 切换持仓模式
rapidx trade preview --input '{
  "targetCapabilityId": "account.set-position-mode",
  "exchange": "BINANCE",
  "mode": "NET"
}' --json

# 平仓
rapidx trade preview --input '{
  "targetCapabilityId": "position.close",
  "symbol": "BINANCE_PERP_BTC_USDT",
  "reduceOnly": true,
  "maxNotional": "10"
}' --json
```

`position.close` 不接受 `side` 或 `quantity`；部分平仓请使用 reduce-only 订单。无持仓时预览返回 `BLOCKED`（`NO_POSITION_TO_CLOSE`）。

`maxNotional` 是安全上限，非目标订单金额。

### 撤单 — 交易所层可能异步

成功的撤单响应表示撤单已**被接受**，而非已确认。检查 `cancelAccepted`、`terminalStateConfirmed` 和 `recommendedAction`；若终态未确认，轮询 `rapidx order get` 直到订单到达 `CANCELED` 或其他终态。

---

## 4. CLI 命令参考

### 格式

```bash
rapidx <domain> <action> [--input '<json>' | --input @/absolute/path/file.json] --json
```

### 诊断与发现

```bash
rapidx schema --json                         # 全部能力与 schema
rapidx self-check --read-only --json         # 只读连通性检查
rapidx doctor --json                         # 本地诊断
rapidx auth check                            # 凭证检查（不打印密钥）
rapidx invocation check                      # 校验调用方式
rapidx update check --json                   # 检查 CLI/Skills 更新
```

### 行情数据

```bash
rapidx market get-ticker         --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-orderbook      --input '{"symbol":"BINANCE_PERP_BTC_USDT","level":20}' --json
rapidx market get-klines         --input '{"symbol":"BINANCE_PERP_BTC_USDT","interval":"1h","limit":100}' --json
rapidx market get-funding-rate   --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-mark-price     --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-symbol-info    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx market get-open-interest  --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

### 账户

```bash
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json   # portfolio 凭证
rapidx account balance --input '{"mode":"account"}' --json     # 需要 account 级凭证
```

使用 portfolio 凭证调用 `mode:"account"` 返回 `PERMISSION_SCOPE_ERROR`。

### 订单

```bash
# 预览（所有写操作必须先预览）
rapidx order place-preview  --input @order.json --json
rapidx order amend-preview  --input '{"orderId":"1234567890123456","price":"64000"}' --json
rapidx order cancel-preview --input '{"orderId":"1234567890123456"}' --json

# 提交（带 previewId + continueConsentId）
rapidx order place  --input @order_with_consent.json --json
rapidx order amend  --input '{"orderId":"...","price":"64000","previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json
rapidx order cancel --input '{"orderId":"...","previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json

# 读取
rapidx order get     --input '{"orderId":"1234567890123456"}' --json
rapidx order list    --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
rapidx order history --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

`orderId` 必须为 16 位 RapidX 订单 ID。格式错误 → `RCORE00002`；格式正确但不存在 → `NOT_FOUND`。

### 仓位

```bash
rapidx position list    --json
rapidx position history --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# 写操作 — 先通过 rapidx trade preview 获取 previewId
rapidx position close        --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx","symbol":"BINANCE_PERP_BTC_USDT","reduceOnly":true,"maxNotional":"10"}' --json
rapidx position set-leverage --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx","symbol":"BINANCE_PERP_BTC_USDT","leverage":5}' --json
```

### 算法单

```bash
# 预览（通过 rapidx trade preview，targetCapabilityId="algo.place"）
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

# 提交（带 previewId + continueConsentId）
rapidx algo place  --input '{"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx",...}' --json
rapidx algo amend  --input '{"algoOrderId":"...","previewId":"rpv_xxx","continueConsentId":"..."}' --json
rapidx algo cancel --input '{"algoOrderId":"...","previewId":"rpv_xxx","continueConsentId":"..."}' --json
rapidx algo list   --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

TPSL 算法单可能使用 `MARKET` 执行，这是 RapidX 规则的预期行为。

### 实盘验证

需要用户明确授权。提交真实 post-only 限价单，再查询、改单、撤单、验证清理。

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

验证步骤：自检 → 品种规则查询 → 内部预览 → post-only 限价下单 → 查单 → 改单 → 撤单 → 清理检查。

---

## 5. MCP 工具参考

### 配置

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<your-access-key>",
        "LTP_SECRET_KEY": "<your-secret-key>",
        "LTP_API_HOST": "<provided-api-host>"
      }
    }
  }
}
```

连接时服务端返回：

```json
{
  "protocolVersion": "2025-03-26",
  "serverInfo": { "name": "rapidx", "version": "<current>" },
  "capabilities": { "tools": {} }
}
```

### 工具名称

MCP `tools/call` 响应内容在 `content[0].text` 中包含 RapidX JSON 信封。

| 分类 | MCP 工具 |
|------|---------|
| 发现与诊断 | `rapidx/tools` · `rapidx/self-check` · `rapidx/update/check` |
| 行情 | `rapidx/market/get-ticker` · `rapidx/market/get-orderbook` · `rapidx/market/get-klines` · `rapidx/market/get-funding-rate` · `rapidx/market/get-mark-price` · `rapidx/market/get-symbol-info` · `rapidx/market/get-open-interest` |
| 账户 | `rapidx/account/overview` · `rapidx/account/balance` · `rapidx/account/set-position-mode` |
| 订单 | `rapidx/order/place-preview` · `rapidx/order/place` · `rapidx/order/amend-preview` · `rapidx/order/amend` · `rapidx/order/cancel-preview` · `rapidx/order/cancel` · `rapidx/order/get` · `rapidx/order/list` · `rapidx/order/history` |
| 交易工具 | `rapidx/trade/preview` · `rapidx/trade/verify-live` · `rapidx/order/preview`（别名） |
| 仓位 | `rapidx/position/list` · `rapidx/position/history` · `rapidx/position/close` · `rapidx/position/set-leverage` |
| 算法单 | `rapidx/algo/list` · `rapidx/algo/place` · `rapidx/algo/amend` · `rapidx/algo/cancel` |

`rapidx/trading-verification` 作为 `rapidx/trade/verify-live` 的兼容别名保留。

---

## 6. 关键字段参考

### 下单 / 预览字段

| 字段 | 必填 | 说明 |
|------|------|------|
| `symbol` | 是 | LTP 标的，如 `BINANCE_PERP_BTC_USDT` |
| `side` | 是 | `BUY` / `SELL` |
| `orderType` | 是 | `LIMIT` / `MARKET` |
| `price` | 限价必填 | 委托价格 |
| `quantity` | 条件必填 | 基础币或合约张数。现货市价买入按报价金额用 `amount`，勿同时传两者 |
| `maxNotional` | 是 | 安全金额上限（报价货币）|
| `clientOrderId` | 推荐 | 幂等追踪键，用于状态确认与故障排查 |
| `postOnly` | 否 | `true` 强制 GTX（post-only），若立即成交则被拒绝 |
| `previewId` | 仅提交 | 来自预览响应 `data.previewId` |
| `continueConsentId` | 仅提交 | 来自预览响应 `data.confirmation.submitToken` |

> 下单前务必通过 `rapidx market get-symbol-info` 查询品种规则：`minNotional`、`lotSize`、`tickSize`、`contractSize` 因品种而异。

### 订单状态值

| `orderState` | 说明 |
|-------------|------|
| `NEW` | 已确认，等待交易所确认 |
| `OPEN` | 挂单中 |
| `PARTIALLY_FILLED` | 部分成交 |
| `FILLED` | 全部成交 |
| `CANCELED` | 已撤销 |
| `REJECTED` | 被交易所拒绝 |

### 账户余额

```bash
# Portfolio 凭证（大赛默认）
rapidx account balance --input '{"mode":"portfolio"}' --json

# 需要 account 级凭证：
rapidx account balance --input '{"mode":"account"}' --json
```

---

## 7. 原始 REST API 端点

| 服务 | 方法 | 路径 |
|------|------|------|
| 账户 | GET | `/api/v1/trading/account` |
| 资产 | GET | `/api/v1/trading/portfolio/assets` |
| 下单 | POST | `/api/v1/trading/order` |
| 改单 | PUT | `/api/v1/trading/order` |
| 撤单 | DELETE | `/api/v1/trading/order` |
| 批量撤单 | DELETE | `/api/v1/trading/cancelAll` |
| 查询订单 | GET | `/api/v1/trading/order` |
| 当前挂单 | GET | `/api/v1/trading/orders` |
| 历史订单 | GET | `/api/v1/trading/history/orders` |
| 持仓 | GET | `/api/v1/trading/position` |
| 历史持仓 | GET | `/api/v1/trading/history/position` |
| 平仓 | DELETE | `/api/v1/trading/position` |
| 查询杠杆 | GET | `/api/v1/trading/perp/leverage` |
| 设置杠杆 | POST | `/api/v1/trading/position/leverage` |
| 成交记录 | GET | `/api/v1/trading/executions` |
| 账单 | GET | `/api/v1/trading/statement` |
| 交易对信息 | GET | `/api/v1/trading/sym/info` |
| 资金费率 | GET | `/api/v1/market/fundingRate` |
| 标记价格 | GET | `/api/v1/market/markPrice` |
| 用户费率 | GET | `/api/v1/trading/userFeeRate` |

完整 REST 规范见 [`api-reference.zh-CN.md`](api-reference.zh-CN.md)。

---

## 8. WebSocket

### 私有 WebSocket

**URL**：`wss://wss-uat.liquiditytech.com/v1/private`

登录后，`Orders` 频道自动推送订单状态变更：

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1000000000000003",
    "orderState": "FILLED",
    "executedQty": "0.001",
    "executedAvgPrice": "65000"
  }
}
```

WebSocket 直接操作：

```json
{ "id": "p1", "action": "place_order",  "args": { ... } }
{ "id": "r1", "action": "replace_order","args": { ... } }
{ "id": "c1", "action": "cancel_order", "args": { ... } }
```

### 行情 WebSocket

**URL**：`wss://mds-uat.liquiditytech.com/marketdata/v2/public`

追加 `?binary=false` 获得纯文本 JSON。

| 频道 | 频率 | 适用范围 |
|------|------|---------|
| `BBO` | 价格变化时 | 全部 |
| `TICKER` | 2000 ms | 全部 |
| `TRADE` | 实时 | 全部 |
| `ORDER_BOOK` | 250 ms | 全部 |
| `KLINE` | 实时 | 全部 |
| `MARK_PRICE` | 实时 | 仅永续 |
| `INDEX_PRICE` | 实时 | 仅永续 |
| `OPEN_INTEREST` | 持仓变化时 | 仅永续 |

订阅：

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

限额按**币对**计，不按频道计。

---

## 9. 错误码

### CLI / MCP 错误码格式

| 前缀 | 范围 |
|------|------|
| `RCORE*` | CLI 与 MCP 共享 |
| `RCLI*` | 仅 CLI |
| `RMCP*` | 仅 MCP |

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| `RCORE01001` | 凭证缺失 | 配置 `LTP_ACCESS_KEY` 和 `LTP_SECRET_KEY` |
| `RCORE01003` | API 地址缺失 | 配置 `LTP_API_HOST` |
| `RCORE00002` | 订单 ID 格式错误 | 使用 16 位 RapidX `orderId` |
| `RCORE01004` | 凭证 scope 不匹配 | 使用具备所需权限的凭证 |
| `RCORE22004` | 资源不存在 | 检查 `orderId`、`clientOrderId` 或标的 |
| `RCLI20002` | 缺少预览/同意 token | 先运行匹配的预览命令 |
| `RCLI22001` | RapidX 上游业务错误 | 读取上游消息，调整请求 |
| `RCORE23002` | 网络或超时错误 | 检查连通性后重试 |
| `RCORE23003` | 速率限制 | 稍后重试，避免轮询循环 |

### REST 错误码

| 错误码 | 含义 |
|--------|------|
| `200000` | 成功 |
| `2002` | API 鉴权失败 |
| `401018` | 订单不存在 |
| `401097` | 仓位数量为 0，无需平仓 |
| `401117` | 订单已完结 |

---

## 10. 故障排查

### npm 找不到包

```bash
npm config get @liquiditytech:registry
# 期望值：https://registry.npmjs.org/

npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

### 写操作返回 BLOCKED

- 是否先调用了匹配的预览命令？`previewId` 是否存在？
- `continueConsentId` 是否等于预览响应的 `data.confirmation.submitToken`？
- 提交请求的业务参数是否与预览一致？
- 检查 `rapidx schema --json` 确认当前字段要求。

### MCP 工具不可见

```json
{ "command": "rapidx", "args": ["mcp", "serve"] }
```

修改 MCP 配置后重载宿主，检查 `rapidx/tools` 列表。

### 升级后仍显示旧版本

```bash
which rapidx
rapidx --version
```

若 MCP 宿主不继承 shell PATH，将 `command` 设置为 `which rapidx` 返回的绝对路径。

### 升级

```bash
rapidx update check --json
npm install -g @liquiditytech/rapidx-cli@latest
rapidx schema --json
rapidx self-check --read-only --json
```

---

**English**: [rapidx-api.md](./rapidx-api.md)
