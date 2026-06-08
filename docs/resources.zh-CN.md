# 资源清单 — AI 量化交易大赛

> 参赛者将获得或可访问的全部资源汇总。

---

## 1. 大赛账户

每位参赛者获得 UAT 模拟撮合环境上的独立仿真账户。

| 项目 | 数值 |
|------|------|
| Portfolio ID | *（由主办方单独颁发）* |
| Access Key（`LTP_ACCESS_KEY`） | *（单独颁发，通过安全渠道分发）* |
| Secret Key（`LTP_SECRET_KEY`） | *（单独颁发，通过安全渠道分发）* |
| API 地址（`LTP_API_HOST`） | *（由主办方提供 — **必填，无默认值**）* |
| 初始虚拟资金 | 10,000 USDT（以主办方公告为准） |
| 持仓模式 | 默认 `BOTH`（双向持仓） |
| 真实资产风险 | **无**。所有订单路由至模拟撮合引擎。 |

> 凭证通过主办方公告的注册渠道下发。请按生产密钥级别保护，绝不提交版本控制、打印日志或分享。

---

## 2. 服务端点

| 服务 | 地址 |
|------|------|
| REST API | *（使用主办方提供的 `LTP_API_HOST` 值）* |
| 私有 WebSocket（交易与账户事件） | `wss://wss-uat.liquiditytech.com/v1/private` |
| 行情 WebSocket（OPEN API） | `wss://mds-uat.liquiditytech.com/marketdata/v2/public` |

### 行情 WebSocket 限额

| 模式 | 最大连接数 / IP | 最大订阅币对 / 连接 |
|------|----------------|---------------------|
| 未认证 | 5 | 5 |
| 已认证 | 40 | 50 |

> 建议使用 API Key 认证行情 WebSocket 以获得更高额度。

---

## 3. npm 包

| 包名 | 说明 | 安装命令 |
|------|------|---------|
| `@liquiditytech/rapidx-cli` | RapidX CLI（`rapidx` 命令）+ `rapidx mcp serve` | `npm install -g @liquiditytech/rapidx-cli` |

安装完成后运行 `rapidx --version` 确认版本。

若 npm 将 `@liquiditytech` 映射到自定义 registry，请先指向官方源：

```bash
npm config set @liquiditytech:registry https://registry.npmjs.org/
npm install -g @liquiditytech/rapidx-cli@latest
```

---

## 4. Skills

| Skill | 仓库 | 用途 |
|-------|------|------|
| `ltp-rapidx-config` | `https://github.com/LiquidityTech/ltp-rapidx-skill` | 安装 CLI、收集凭证、配置 MCP、运行自检 |
| `ltp-rapidx-trading` | 同上 | 发现工具、读取行情账户、写操作预览与提交、实盘验证 |

各宿主安装命令见 `README.zh-CN.md` 第一步。

---

## 5. CLI 与 MCP 工具目录

所有 CLI 命令格式为 `rapidx <domain> <action> --input '<json>' --json`。  
所有 MCP 工具以 `rapidx/` 前缀命名。

### 诊断与发现

| CLI | MCP 工具 | 说明 |
|-----|---------|------|
| `rapidx schema --json` | `rapidx/tools` | 列出全部能力与 schema |
| `rapidx self-check --read-only --json` | `rapidx/self-check` | 验证只读连通性 |
| `rapidx doctor --json` | 仅 CLI | 本地诊断（版本、凭证来源、调用方式） |
| `rapidx auth check` | 仅 CLI | 凭证检查（不打印完整密钥） |
| `rapidx invocation check` | 仅 CLI | 校验调用方式是否合规 |
| `rapidx update check --json` | `rapidx/update/check` | 检查 CLI/Skills 更新 |

### 行情数据

| CLI | MCP 工具 |
|-----|---------|
| `rapidx market get-ticker` | `rapidx/market/get-ticker` |
| `rapidx market get-orderbook` | `rapidx/market/get-orderbook` |
| `rapidx market get-klines` | `rapidx/market/get-klines` |
| `rapidx market get-funding-rate` | `rapidx/market/get-funding-rate` |
| `rapidx market get-mark-price` | `rapidx/market/get-mark-price` |
| `rapidx market get-symbol-info` | `rapidx/market/get-symbol-info` |
| `rapidx market get-open-interest` | `rapidx/market/get-open-interest` |

### 账户

| CLI | MCP 工具 |
|-----|---------|
| `rapidx account overview` | `rapidx/account/overview` |
| `rapidx account balance` | `rapidx/account/balance` |
| `rapidx account set-position-mode` | `rapidx/account/set-position-mode` |

### 订单（写操作强制预览）

| CLI | MCP 工具 |
|-----|---------|
| `rapidx order place-preview` | `rapidx/order/place-preview` |
| `rapidx order place` | `rapidx/order/place` |
| `rapidx order amend-preview` | `rapidx/order/amend-preview` |
| `rapidx order amend` | `rapidx/order/amend` |
| `rapidx order cancel-preview` | `rapidx/order/cancel-preview` |
| `rapidx order cancel` | `rapidx/order/cancel` |
| `rapidx order get` | `rapidx/order/get` |
| `rapidx order list` | `rapidx/order/list` |
| `rapidx order history` | `rapidx/order/history` |

### 仓位（写操作强制预览）

| CLI | MCP 工具 |
|-----|---------|
| `rapidx position list` | `rapidx/position/list` |
| `rapidx position history` | `rapidx/position/history` |
| `rapidx position close` | `rapidx/position/close` |
| `rapidx position set-leverage` | `rapidx/position/set-leverage` |

### 算法单（写操作强制预览）

| CLI | MCP 工具 |
|-----|---------|
| `rapidx algo list` | `rapidx/algo/list` |
| `rapidx algo place` | `rapidx/algo/place` |
| `rapidx algo amend` | `rapidx/algo/amend` |
| `rapidx algo cancel` | `rapidx/algo/cancel` |

### 交易工具

| CLI | MCP 工具 |
|-----|---------|
| `rapidx trade preview` | `rapidx/trade/preview` |
| `rapidx trade verify-live` | `rapidx/trade/verify-live` |

完整入参/出参规范见 [`rapidx-api.zh-CN.md`](./rapidx-api.zh-CN.md)。

---

## 6. 交易品种

格式：`{交易所}_{产品类型}_{基础币}_{计价币}`

支持的交易所：`BINANCE` · `OKX` · `EDX`

### 常用品种

| Symbol | 说明 |
|--------|------|
| `BINANCE_PERP_BTC_USDT` | Binance BTC 永续合约 |
| `BINANCE_PERP_ETH_USDT` | Binance ETH 永续合约 |
| `BINANCE_PERP_BNB_USDT` | Binance BNB 永续合约 |
| `BINANCE_PERP_XRP_USDT` | Binance XRP 永续合约 |
| `BINANCE_PERP_ADA_USDT` | Binance ADA 永续合约 |
| `BINANCE_PERP_DOGE_USDT` | Binance DOGE 永续合约 |
| `OKX_PERP_BTC_USDT` | OKX BTC 永续合约 |
| `OKX_PERP_ETH_USDT` | OKX ETH 永续合约 |
| `EDX_PERP_ETH_USDT` | EDX ETH 永续合约 |

完整品种列表：

```bash
rapidx market get-symbol-info --json
```

> **下单前务必查询品种规则**：最小名义价值、数量步长、价格步长因品种而异，请先运行 `rapidx market get-symbol-info --input '{"symbol":"..."}' --json`。

---

## 7. 文档

| 文档 | 位置 |
|------|------|
| 快速上手 | [`01-quickstart.zh-CN.md`](./01-quickstart.zh-CN.md) · [`01-quickstart.md`](./01-quickstart.md) |
| CLI 与 MCP 参考 | [`02-mcp-reference.zh-CN.md`](./02-mcp-reference.zh-CN.md) · [`02-mcp-reference.md`](./02-mcp-reference.md) |
| 进阶 REST 与 WebSocket | [`03-advanced-api.zh-CN.md`](./03-advanced-api.zh-CN.md) · [`03-advanced-api.md`](./03-advanced-api.md) |
| 资源清单 | 本文档 |
| GitHub 仓库 | <https://github.com/LiquidityTech/ltp-arena-docs> |

---

## 8. 接入验证

使用内置 CLI 自检，无需额外安装依赖：

```bash
rapidx auth check
rapidx self-check --read-only --json
```

每个检查项报告 **PASS** / **EXPECTED_ERROR** / **FAIL** / **NOT_VERIFIED**。

如需端到端实盘验证（提交真实 post-only 订单后自动撤单）：

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

---

## 9. 大赛日程

| 阶段 | 活动 |
|------|------|
| 报名 | 领取凭证、安装 CLI、运行自检 |
| 开发 | 在 UAT 实时数据上构建并测试智能体 |
| 提交截止 | 冻结智能体代码与模型权重 |
| 评估窗口 | 冻结后的智能体在前向真实数据上自主运行 |
| 评分与颁奖 | 多维度评分（收益、Sharpe、最大回撤、胜率） |

具体日期通过主办方官方渠道公告。

---

## 10. 支持

| 渠道 | 联系方式 |
|------|---------|
| 技术支持邮箱 | `support@liquiditytech.com` |
| Issue 追踪 | <https://github.com/liquiditytech/rapidx-cli/issues> |
| Discord / 社区 | *（报名时由主办方提供链接）* |
| 状态页 | <https://status.liquiditytech.com> |

---

## 11. 安全与合规须知

1. **无真实资产风险** — 大赛运行于 UAT 模拟撮合引擎。
2. **所有写操作强制预览** — 预览过程不提交真实订单。
3. **`LTP_API_HOST` 必填** — 缺失时返回 `RCORE01003`。
4. **每次写操作后状态确认** — 调用 `rapidx order get` 或 `rapidx position list`，禁止盲目重试。
5. **密钥永不外泄** — 所有输出、日志、报告中密钥均经掩码处理。
6. **单赛道竞技** — 所有参赛者在同一条件下竞技。

---

**English**: [resources.md](./resources.md)
