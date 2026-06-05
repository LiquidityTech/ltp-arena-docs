# LTP RapidX API 参考手册

> **AI 量化交易大赛专用版 · UAT 仿真环境**
> 
> 本文档列出大赛期间可用的全部接口，均已在 UAT 环境验证通过。

---

## 目录

1. [环境地址](#1-环境地址)
2. [认证方式](#2-认证方式)
3. [通用响应格式](#3-通用响应格式)
4. [REST API](#4-rest-api)
   - 4.1 账户服务
   - 4.2 资产服务
   - 4.3 订单服务
   - 4.4 仓位服务
   - 4.5 成交服务
   - 4.6 账单服务
   - 4.7 行情与规则服务
5. [WebSocket API](#5-websocket-api)
   - 5.1 私有 WebSocket（交易）
   - 5.2 行情 WebSocket
6. [Symbol 命名规范](#6-symbol-命名规范)
7. [注意事项](#7-注意事项)

---

## 1. 环境地址

| 类型 | 地址 |
|------|------|
| REST API | `https://uat-api.liquiditytech.com` |
| 私有 WebSocket（交易） | `wss://wss-uat.liquiditytech.com/v1/private` |
| 行情 WebSocket（OPEN API） | `wss://mds-uat.liquiditytech.com/marketdata/v2/public` |

> **重要提示**：UAT 仿真环境的订单会路由到真实交易所账户执行，请务必使用大赛分配的账户 API Key，不得使用自有生产账户。

### 行情 WebSocket 连接限制

| 模式 | 最大连接数 | 最大订阅币对数 |
|------|-----------|---------------|
| 未登录 | 5 | 5 |
| 已登录（携带 API Key） | 40 | 50 |

> 量化策略通常需要订阅多个品种，**强烈建议使用 API Key 登录行情 WebSocket** 以获得更高限额。行情 WebSocket 的登录方式与私有 WebSocket 相同（见第 5.1 节）。

---

## 2. 认证方式

所有 REST 请求均需在 HTTP Header 中携带以下字段：

| Header | 说明 |
|--------|------|
| `X-MBX-APIKEY` | 您的 API Key |
| `nonce` | 当前 Unix 时间戳（秒），`string` 类型 |
| `signature` | HMAC-SHA256 签名（见下方说明） |
| `Content-Type` | `application/json` |

### 签名算法

```
1. 将请求参数按 key 字母序排列，拼接为 query string：
   sorted_payload = "key1=val1&key2=val2&..."

2. 末尾追加 & + timestamp：
   payload = sorted_payload + "&" + timestamp
   （若无参数，payload = "&" + timestamp）

3. 用 Secret Key 对 payload 做 HMAC-SHA256，取十六进制结果：
   signature = HMAC-SHA256(secret_key, payload).hexdigest()
```

**Python 示例：**

```python
import hmac, hashlib, time

def sign(params: dict, secret_key: str) -> tuple[str, str]:
    timestamp = str(int(time.time()))
    sorted_payload = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    payload = (sorted_payload + "&" if sorted_payload else "&") + timestamp
    signature = hmac.new(secret_key.encode(), payload.encode(), hashlib.sha256).hexdigest()
    return signature, timestamp
```

---

## 3. 通用响应格式

```json
{
  "code": 200000,
  "message": "Success",
  "data": { ... }
}
```

| 字段 | 说明 |
|------|------|
| `code` | `200000` 表示成功，其他值为错误码 |
| `message` | 状态描述 |
| `data` | 业务数据，结构因接口而异 |

---

## 4. REST API

### 4.1 账户服务

#### 查询账户信息

```
GET /api/v1/trading/account
```

无需参数。返回各交易所子账户的保证金、风险、仓位等汇总信息。

**响应示例（data 为数组，每个元素对应一个交易所账户）：**

```json
[
  {
    "portfolioId": "1000000000000001",
    "exchangeType": "BINANCE",
    "equity": "102.23",
    "availableMargin": "93.79",
    "maintainMargin": "0.29",
    "riskRatio": "0.0029",
    "accountStatus": "NORMAL",
    "positionMode": "BOTH",
    "upnl": "0"
  }
]
```

| 字段 | 说明 |
|------|------|
| `exchangeType` | 交易所类型：`BINANCE` / `OKX` / `EDX` |
| `equity` | 账户权益（USDT） |
| `availableMargin` | 可用保证金（USDT） |
| `riskRatio` | 风险比率 |
| `accountStatus` | 账户状态：`NORMAL` / `MARGIN_CALL` / `LIQUIDATION` |
| `positionMode` | 持仓模式：`BOTH`（双向）/ `NET`（单向） |

---

### 4.2 资产服务

#### 查询投资组合资产

```
GET /api/v1/trading/portfolio/assets
```

无需参数。返回各交易所各币种的持仓明细。

**响应示例（data.list 为数组）：**

```json
{
  "page": 1,
  "pageSize": 1000,
  "totalSize": 5,
  "list": [
    {
      "portfolioId": "1000000000000001",
      "coin": "USDT",
      "exchangeType": "BINANCE",
      "balance": "107.98",
      "available": "101.98",
      "frozen": "6",
      "equity": "107.98",
      "maxTransferable": "96.19"
    }
  ]
}
```

---

### 4.3 订单服务

#### 下单

```
POST /api/v1/trading/order
```

**请求参数（JSON Body）：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sym` | string | ✓ | 交易对，如 `BINANCE_PERP_BTC_USDT` |
| `side` | string | ✓ | `BUY` / `SELL` |
| `positionSide` | string | ✓ | `LONG` / `SHORT`（双向持仓模式必填） |
| `orderType` | string | ✓ | `LIMIT` / `MARKET` |
| `timeInForce` | string | 限价必填 | `GTC` / `IOC` / `FOK` |
| `orderQty` | string | ✓ | 委托数量（基础币，如 BTC） |
| `limitPrice` | string | 限价必填 | 委托价格 |
| `clientOrderId` | string | 推荐 | 自定义订单 ID，最长 40 字符 |

**响应 data：**

```json
{
  "orderId": "1000000000000002",
  "clientOrderId": "my_order_001"
}
```

---

#### 改单（修改价格/数量）

```
PUT /api/v1/trading/order
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 系统订单 ID（与 clientOrderId 二选一） |
| `clientOrderId` | string | 自定义订单 ID |
| `replacePrice` | string | 新价格（可选） |
| `replaceQty` | string | 新数量（可选） |

**响应 data：**

```json
{
  "orderId": "1000000000000002",
  "orderState": "OPEN"
}
```

---

#### 撤单

```
DELETE /api/v1/trading/order
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 系统订单 ID（与 clientOrderId 二选一） |
| `clientOrderId` | string | 自定义订单 ID |

**响应 data：**

```json
{
  "orderId": "1000000000000002",
  "clientOrderId": "my_order_001",
  "orderState": "OPEN",
  "action": "CANCEL_PENDING"
}
```

---

#### 批量撤单

```
DELETE /api/v1/trading/cancelAll
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 指定交易对撤单（与 exchangeType 二选一） |
| `exchangeType` | string | 指定交易所全部撤单 |

---

#### 查询单笔订单

```
GET /api/v1/trading/order
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 系统订单 ID（与 clientOrderId 二选一） |
| `clientOrderId` | string | 自定义订单 ID |

**响应 data（订单详情）：**

| 字段 | 说明 |
|------|------|
| `orderState` | `NEW` / `OPEN` / `PARTIALLY_FILLED` / `FILLED` / `CANCELLED` / `REJECTED` |
| `executedQty` | 已成交数量 |
| `executedAvgPrice` | 成交均价 |
| `limitPrice` | 委托价格 |
| `orderQty` | 委托数量 |
| `fee` | 手续费 |
| `leverage` | 杠杆倍数 |

---

#### 查询当前挂单

```
GET /api/v1/trading/orders
```

无需参数，返回所有未完结订单列表（`data.list`）。

---

#### 查询历史订单

```
GET /api/v1/trading/history/orders
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

#### 查询历史订单（归档）

```
GET /api/v1/trading/archive/history/orders
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

### 4.4 仓位服务

#### 查询持仓

```
GET /api/v1/trading/position
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，指定交易对 |
| `exchange` | string | 可选，指定交易所 |

---

#### 查询历史持仓

```
GET /api/v1/trading/history/position
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，指定交易对 |

---

#### 查询 ADL 排名

```
GET /api/v1/adl/rank
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，指定交易对 |

---

#### 查询杠杆

```
GET /api/v1/trading/perp/leverage
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，指定交易对 |
| `exchange` | string | 可选，指定交易所 |

**响应 data（数组）：**

```json
[
  { "sym": "BINANCE_PERP_BTC_USDT", "leverage": "5" }
]
```

---

#### 设置杠杆

```
POST /api/v1/trading/position/leverage
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sym` | string | ✓ | 交易对 |
| `leverage` | string | ✓ | 杠杆倍数，如 `"5"` |

> 注意：下单前请确保账户可用保证金足够覆盖订单名义价值 / 杠杆。建议在下单前调用此接口设置合适杠杆。

---

#### 平仓（单个方向）

```
DELETE /api/v1/trading/position
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sym` | string | ✓ | 交易对 |
| `positionSide` | string | ✓ | `LONG` / `SHORT` |

---

#### 批量平仓

```
DELETE /api/v1/trading/positions
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `exchangeType` | string | 指定交易所 |
| `closeAllPos` | string | `"true"` 表示全部平仓 |

---

### 4.5 成交服务

#### 查询成交记录

```
GET /api/v1/trading/executions
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

#### 查询成交记录（分页）

```
GET /api/v1/trading/executions/pageable
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

#### 查询成交记录（归档分页）

```
GET /api/v1/trading/archive/executions/pageable
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

### 4.6 账单服务

#### 查询交易账单

```
GET /api/v1/trading/statement
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，过滤交易对 |

---

#### 查询交易统计

```
GET /api/v1/trading/user/tradingStats
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `begin` | string | ✓ | 开始时间戳（秒） |
| `end` | string | ✓ | 结束时间戳（秒） |

---

### 4.7 行情与规则服务

#### 查询交易对信息

```
GET /api/v1/trading/sym/info
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，不传返回全部 |

返回最小下单量、价格精度、数量精度、最大杠杆等规则。

---

#### 查询资金费率

```
GET /api/v1/market/fundingRate
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `sym` | string | ✓ | 交易对 |

---

#### 查询标记价格

```
GET /api/v1/market/markPrice
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，不传返回全部 |

---

#### 查询用户费率

```
GET /api/v1/trading/userFeeRate
```

无需参数，返回 Maker/Taker 费率。

---

#### 查询仓位梯度（Position Tier）

```
GET /api/v1/trading/positionBracket
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `sym` | string | 可选，不传返回全部 |

---

#### 查询借贷梯度

```
GET /api/v1/trading/loan/info
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `exchangeType` | string | 可选，如 `BINANCE` |
| `coin` | string | 可选，如 `BTC` |

---

#### 查询折扣率

```
GET /api/v1/trading/coin/discount
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `coin` | string | 可选，如 `BTC` |

---

#### 查询保证金杠杆

```
GET /api/v1/trading/margin/leverage
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `exchangeType` | string | 可选，如 `BINANCE` |
| `coin` | string | 可选，如 `BTC` |

---

## 5. WebSocket API

### 5.1 私有 WebSocket（交易）

**连接地址：** `wss://wss-uat.liquiditytech.com/v1/private`

> 使用 `websockets` 库连接时，需强制使用 HTTP/1.1 ALPN 协议（AWS ALB 不支持通过 HTTP/2 升级 WebSocket）。

#### 登录认证

连接成功后必须先发送 login 消息，否则后续操作均会失败。

**签名算法（WebSocket 与 REST 不同）：**

```
message = timestamp + "GET" + "/users/self/verify"
sign = HMAC-SHA256(secret_key, message).hexdigest()
```

**发送：**

```json
{
  "action": "login",
  "args": {
    "apiKey": "YOUR_API_KEY",
    "timestamp": "1778140847",
    "sign": "e54702b1..."
  }
}
```

**接收（成功）：**

```json
{
  "event": "login",
  "code": 0,
  "msg": ""
}
```

---

#### 下单

```json
{
  "id": "place-1",
  "action": "place_order",
  "args": {
    "clientOrderId": "my_order_001",
    "sym": "BINANCE_PERP_BTC_USDT",
    "side": "BUY",
    "positionSide": "LONG",
    "orderType": "LIMIT",
    "timeInForce": "GTC",
    "orderQty": "0.003",
    "limitPrice": "80000"
  }
}
```

**响应：**

```json
{
  "id": "place-1",
  "event": "place_order",
  "code": 200000,
  "msg": "Success",
  "data": {
    "orderId": "1000000000000003",
    "clientOrderId": "my_order_001"
  }
}
```

---

#### 改单

```json
{
  "id": "replace-1",
  "action": "replace_order",
  "args": {
    "orderId": "1000000000000003",
    "replacePrice": "79000"
  }
}
```

**响应：**

```json
{
  "id": "replace-1",
  "event": "replace_order",
  "code": 200000,
  "data": {
    "orderId": "1000000000000003",
    "orderState": "OPEN"
  }
}
```

---

#### 撤单

```json
{
  "id": "cancel-1",
  "action": "cancel_order",
  "args": {
    "orderId": "1000000000000003"
  }
}
```

---

#### 订单状态推送（Orders 频道）

登录后，系统会自动推送订单状态变更。

```json
{
  "channel": "Orders",
  "data": {
    "orderId": "1000000000000003",
    "clientOrderId": "my_order_001",
    "orderState": "FILLED",
    "action": "AMEND_COMPLETED",
    "executedQty": "0.003",
    "executedAvgPrice": "80000"
  }
}
```

| `orderState` 值 | 说明 |
|-----------------|------|
| `NEW` | 新建，等待交易所确认 |
| `OPEN` | 已挂单 |
| `PARTIALLY_FILLED` | 部分成交 |
| `FILLED` | 完全成交 |
| `CANCELLED` | 已撤销 |
| `REJECTED` | 被拒绝 |

#### Keepalive

每 15 秒发送一次 `"ping"`，服务端回复 `"pong"`，防止连接超时断开。

---

### 5.2 行情 WebSocket

**连接地址：** `wss://mds-uat.liquiditytech.com/marketdata/v2/public`

无需认证，连接后直接订阅频道。

#### 订阅最优报价（BBO）

```json
{
  "event": "subscribe",
  "arg": [
    { "channel": "BBO", "sym": "BINANCE_PERP_BTC_USDT" }
  ]
}
```

**推送数据：**

```json
{
  "channel": "BBO",
  "data": {
    "sym": "BINANCE_PERP_BTC_USDT",
    "bid": "81422.3",
    "ask": "81422.4"
  }
}
```

---

## 6. Symbol 命名规范

格式：`{交易所}_{产品类型}_{基础币}_{计价币}`

| 示例 | 说明 |
|------|------|
| `BINANCE_PERP_BTC_USDT` | Binance BTC 永续合约 |
| `BINANCE_PERP_ETH_USDT` | Binance ETH 永续合约 |
| `BINANCE_PERP_XRP_USDT` | Binance XRP 永续合约 |
| `BINANCE_PERP_BNB_USDT` | Binance BNB 永续合约 |
| `BINANCE_PERP_ADA_USDT` | Binance ADA 永续合约 |
| `OKX_PERP_BTC_USDT` | OKX BTC 永续合约 |
| `OKX_PERP_ETH_USDT` | OKX ETH 永续合约 |
| `OKX_PERP_ADA_USDT` | OKX ADA 永续合约 |
| `EDX_PERP_ETH_USDT` | EDX ETH 永续合约 |

可通过 `GET /api/v1/trading/sym/info` 获取全量可交易品种列表。

---

## 7. 注意事项

### 环境说明

- UAT 仿真环境与生产环境**共用交易所账户**，订单会真实路由到交易所执行。
- 大赛账户已预置初始资金，请勿尝试大额下单或恶意刷单。

### 下单前置检查

1. **杠杆设置**：下单前通过 `POST /api/v1/trading/position/leverage` 确认杠杆设置，确保 `可用保证金 × 杠杆 ≥ 订单名义价值 × 1.1`（10% 安全缓冲）。
2. **双向持仓模式**：账户默认为 `BOTH`（双向持仓）模式，下单时 `positionSide` 字段**必须填写** `LONG` 或 `SHORT`，否则订单会被拒绝。
3. **最小下单量**：各品种最小下单量不同，可通过 `GET /api/v1/trading/sym/info` 查询 `minQty` 字段。

### 错误处理建议

- WebSocket 的 push 推送可能延迟或丢失，建议在超时（如 30s）后使用 REST 接口轮询订单状态作为降级方案。
- 撤单后订单状态会短暂停留在 `OPEN`（`action: CANCEL_PENDING`），最终变为 `CANCELLED`。

---

## 8. 接入验证

安装 CLI 后，运行内置自检：

```bash
rapidx auth check
rapidx self-check --read-only --json
```

每个检查项报告 **PASS** / **EXPECTED_ERROR** / **FAIL** / **NOT_VERIFIED**，无需额外安装依赖。


## 接口可用性汇总

| 接口 | 方法 | 路径 | UAT 状态 |
|------|------|------|----------|
| 查询账户 | GET | `/api/v1/trading/account` | ✅ 可用 |
| 查询资产 | GET | `/api/v1/trading/portfolio/assets` | ✅ 可用 |
| 下单 | POST | `/api/v1/trading/order` | ✅ 可用 |
| 改单 | PUT | `/api/v1/trading/order` | ✅ 可用 |
| 撤单 | DELETE | `/api/v1/trading/order` | ✅ 可用 |
| 批量撤单 | DELETE | `/api/v1/trading/cancelAll` | ✅ 可用 |
| 查询单笔订单 | GET | `/api/v1/trading/order` | ✅ 可用 |
| 查询挂单 | GET | `/api/v1/trading/orders` | ✅ 可用 |
| 查询历史订单 | GET | `/api/v1/trading/history/orders` | ✅ 可用 |
| 查询历史订单（归档） | GET | `/api/v1/trading/archive/history/orders` | ✅ 可用 |
| 查询持仓 | GET | `/api/v1/trading/position` | ✅ 可用 |
| 查询历史持仓 | GET | `/api/v1/trading/history/position` | ✅ 可用 |
| 平仓 | DELETE | `/api/v1/trading/position` | ✅ 可用 |
| 批量平仓 | DELETE | `/api/v1/trading/positions` | ✅ 可用 |
| 查询杠杆 | GET | `/api/v1/trading/perp/leverage` | ✅ 可用 |
| 设置杠杆 | POST | `/api/v1/trading/position/leverage` | ✅ 可用 |
| 查询 ADL 排名 | GET | `/api/v1/adl/rank` | ✅ 可用 |
| 查询成交 | GET | `/api/v1/trading/executions` | ✅ 可用 |
| 查询成交（分页） | GET | `/api/v1/trading/executions/pageable` | ✅ 可用 |
| 查询成交（归档） | GET | `/api/v1/trading/archive/executions/pageable` | ✅ 可用 |
| 查询账单 | GET | `/api/v1/trading/statement` | ✅ 可用 |
| 查询交易统计 | GET | `/api/v1/trading/user/tradingStats` | ✅ 可用 |
| 查询交易对信息 | GET | `/api/v1/trading/sym/info` | ✅ 可用 |
| 查询资金费率 | GET | `/api/v1/market/fundingRate` | ✅ 可用 |
| 查询标记价格 | GET | `/api/v1/market/markPrice` | ✅ 可用 |
| 查询用户费率 | GET | `/api/v1/trading/userFeeRate` | ✅ 可用 |
| 查询仓位梯度 | GET | `/api/v1/trading/positionBracket` | ✅ 可用 |
| 查询借贷梯度 | GET | `/api/v1/trading/loan/info` | ✅ 可用 |
| 查询折扣率 | GET | `/api/v1/trading/coin/discount` | ✅ 可用 |
| 查询保证金杠杆 | GET | `/api/v1/trading/margin/leverage` | ✅ 可用 |
| WS 下单 | WS | `place_order` | ✅ 可用 |
| WS 改单 | WS | `replace_order` | ✅ 可用 |
| WS 撤单 | WS | `cancel_order` | ✅ 可用 |
| WS 订单推送 | WS | `Orders` channel | ✅ 可用 |
| WS BBO 行情 | WS | `BBO` channel | ✅ 可用 |
