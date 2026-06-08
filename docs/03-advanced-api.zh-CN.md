# 进阶：直接调用 REST 与 WebSocket API

> 适用于不经过 CLI 或 MCP 层、直接调用 RapidX 的自定义代码和底层集成场景。

**大多数参赛者应使用 CLI 或 MCP** — 见 [`01-quickstart.zh-CN.md`](./01-quickstart.zh-CN.md)。本文档适用于需要直接 HTTP 或 WebSocket 访问的场景：对性能要求极高的交易机器人、非 Node.js 语言环境、或自定义工具链。

---

## 认证

### REST 必填 Header

| Header | 值 |
|--------|---|
| `X-MBX-APIKEY` | Access Key |
| `nonce` | 当前 Unix 时间戳（秒，字符串） |
| `signature` | HMAC-SHA256 签名（见下） |
| `Content-Type` | `application/json` |

### REST 签名算法

```
1. 将请求参数按 key 字母序排列并拼接：
   sorted_payload = "key1=val1&key2=val2&..."

2. 末尾追加 "&" + 时间戳：
   payload = sorted_payload + "&" + timestamp
   （无参数时：payload = "&" + timestamp）

3. 用 Secret Key 做 HMAC-SHA256，取十六进制：
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

**Python 示例：**

```python
import hmac, hashlib, time

def sign(params: dict, secret_key: str):
    timestamp = str(int(time.time()))
    sorted_payload = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    payload = (sorted_payload + "&" if sorted_payload else "&") + timestamp
    sig = hmac.new(secret_key.encode(), payload.encode(), hashlib.sha256).hexdigest()
    return sig, timestamp
```

### WebSocket 认证签名

与 REST 不同，用于私有 WebSocket 登录：

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

---

## REST API 端点

Base URL：主办方提供的 `LTP_API_HOST` 值。

### 账户与资产

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/trading/account` | 各交易所账户概览 |
| GET | `/api/v1/trading/portfolio/assets` | 投资组合资产明细 |
| GET | `/api/v1/trading/user/tradingStats` | 交易统计（需 `begin`、`end` 参数） |
| GET | `/api/v1/trading/userFeeRate` | Maker/Taker 费率 |

### 订单

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/v1/trading/order` | 下单 |
| PUT | `/api/v1/trading/order` | 改单 |
| DELETE | `/api/v1/trading/order` | 撤单 |
| DELETE | `/api/v1/trading/cancelAll` | 批量撤单（`sym` 或 `exchangeType`） |
| GET | `/api/v1/trading/order` | 查询单笔订单 |
| GET | `/api/v1/trading/orders` | 当前挂单列表 |
| GET | `/api/v1/trading/history/orders` | 历史订单（`sym` 可选） |
| GET | `/api/v1/trading/archive/history/orders` | 归档历史订单 |
| GET | `/api/v1/trading/executions` | 成交记录 |
| GET | `/api/v1/trading/executions/pageable` | 分页成交记录 |
| GET | `/api/v1/trading/statement` | 交易账单 |

**下单字段：**

| 字段 | 必填 | 说明 |
|------|------|------|
| `sym` | 是 | 如 `BINANCE_PERP_BTC_USDT` |
| `side` | 是 | `BUY` / `SELL` |
| `positionSide` | 双向持仓必填 | `LONG` / `SHORT` |
| `orderType` | 是 | `LIMIT` / `MARKET` |
| `orderQty` | 是 | 基础资产或合约张数 |
| `limitPrice` | 限价必填 | 委托价格 |
| `timeInForce` | 否 | `GTC`（默认）/ `IOC` / `FOK` / `GTX` |
| `clientOrderId` | 推荐 | 最长 40 字符 |

**订单状态：** `NEW` → `OPEN` → `PARTIALLY_FILLED` → `FILLED` / `CANCELLED` / `REJECTED`

### 仓位

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/trading/position` | 当前持仓（`sym`、`exchange` 可选） |
| GET | `/api/v1/trading/history/position` | 历史持仓 |
| DELETE | `/api/v1/trading/position` | 平仓（`sym`、`positionSide`） |
| DELETE | `/api/v1/trading/positions` | 批量平仓（`exchangeType`、`closeAllPos:"true"`） |
| GET | `/api/v1/trading/perp/leverage` | 查询杠杆 |
| POST | `/api/v1/trading/position/leverage` | 设置杠杆（`sym`、`leverage`） |
| GET | `/api/v1/adl/rank` | ADL 排名 |

### 行情数据（RapidX）

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/trading/sym/info` | 品种规则（`sym` 可选） |
| GET | `/api/v1/market/fundingRate` | 资金费率（`sym` 必填） |
| GET | `/api/v1/market/markPrice` | 标记价格（`sym` 可选） |
| GET | `/api/v1/trading/positionBracket` | 仓位梯度 |
| GET | `/api/v1/trading/loan/info` | 借贷信息 |
| GET | `/api/v1/trading/coin/discount` | 币种折扣率 |
| GET | `/api/v1/trading/margin/leverage` | 保证金杠杆 |

### REST 响应格式

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

`code: 200000` 表示成功，其他值为错误码。

---

## WebSocket

### 私有 WebSocket（交易与账户事件）

**URL：** `wss://wss-uat.liquiditytech.com/v1/private`

**登录：**

```json
{
  "action": "login",
  "args": {
    "apiKey": "YOUR_ACCESS_KEY",
    "timestamp": "1778140847",
    "sign": "..."
  }
}
```

**响应：**

```json
{ "event": "login", "code": 0, "msg": "" }
```

**订单操作：**

```json
{ "id": "p1", "action": "place_order",  "args": { ... } }
{ "id": "a1", "action": "replace_order","args": { "orderId": "...", "replacePrice": "64000" } }
{ "id": "c1", "action": "cancel_order", "args": { "orderId": "..." } }
```

**Orders 推送频道**（登录后自动订阅）：

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1234567890123456",
    "clientOrderId": "agent-001",
    "orderState": "FILLED",
    "executedQty": "0.001",
    "executedAvgPrice": "65000"
  }
}
```

心跳：每 15 秒发送 `"ping"`，服务端回复 `"pong"`。

### 行情 WebSocket

**URL：** `wss://mds-uat.liquiditytech.com/marketdata/v2/public`

追加 `?binary=false` 改为纯文本 JSON（默认 GZIP 二进制帧）。

**限额：**

| 模式 | 最大连接数 / IP | 最大订阅币对 / 连接 |
|------|----------------|---------------------|
| 未认证 | 5 | 5 |
| 已认证 | 40 | 50 |

限额按**币对**计（不按频道）— 同一标的订阅 BBO + TICKER + TRADE 算 1 个币对。

**订阅：**

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO",   "sym": "BINANCE_PERP_BTC_USDT" },
    { "channel": "TRADE", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

**可用频道：**

| 频道 | 频率 | 适用范围 |
|------|------|---------|
| `BBO` | 价格变化时 | 全部 |
| `TICKER` | 2000 ms | 全部 |
| `TRADE` | 实时 | 全部 |
| `ORDER_BOOK` | 250 ms | 全部 |
| `KLINE` | 实时 | 全部 |
| `MARK_PRICE` | 实时 | 仅永续 |
| `INDEX_PRICE` | 实时 | 仅永续 |
| `MARK_PRICE_KLINE` | 实时 | 仅永续 |
| `INDEX_KLINE` | 实时 | 仅永续 |
| `OPEN_INTEREST` | 持仓变化时 | 仅永续 |

心跳：每 20 秒发送 `{ "ping": <timestamp_ms> }`，服务端回复 `{ "pong": <timestamp_ms> }`。

---

## Symbol 格式

```
{交易所}_{产品类型}_{基础币}_{计价币}
```

| 交易所 | 类型 | 示例 |
|--------|------|------|
| `BINANCE` | `PERP` | `BINANCE_PERP_BTC_USDT`、`BINANCE_PERP_ETH_USDT` |
| `OKX` | `PERP` | `OKX_PERP_BTC_USDT`、`OKX_PERP_ETH_USDT` |
| `EDX` | `PERP` | `EDX_PERP_ETH_USDT` |

标记为「仅永续」的频道使用 `SPOT` 标的时返回错误 `11100`。

---

## 常见错误码

### REST

| 错误码 | 含义 |
|--------|------|
| `200000` | 成功 |
| `2002` | API 鉴权失败 |
| `401018` | 订单不存在 |
| `401097` | 仓位数量为 0 |
| `401117` | 订单已完结 |

### 行情 WebSocket

| 错误码 | 含义 |
|--------|------|
| `0` | 成功 |
| `11100` | 标的非法 |
| `11210` | 未认证订阅币对超限（5 个） |
| `11220` | 已认证订阅币对超限（50 个） |
| `11250` | 当前不支持该标的 |
| `11260` | 请求速率超限 |

---

**English**: [03-advanced-api.md](./03-advanced-api.md)
