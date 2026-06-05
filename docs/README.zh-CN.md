# RapidX AI 量化交易大赛 — 参赛包

> 在 LTP RapidX 仿真环境中构建、测试并提交您的 AI 交易智能体。

---

## 您将获得

| | |
|---|---|
| **仿真账户** | UAT 模拟撮合引擎 — 虚拟资金、真实行情、不涉及任何真实资产 |
| **MCP 工具 + CLI** | 账户 / 仓位 / 订单 / 算法单 / 行情 — 全部对智能体开放 |
| **三种接入方式** | MCP Server · 协议化 CLI（任意 exec-only agent）· 原始 REST/WebSocket |
| **默认安全模式** | 所有写操作强制预览；交易需显式启用并确认参数 |

CLI 与 MCP Server 通过 **npm** 发布为 `@liquiditytech/rapidx-cli`。  
Skills 从 `https://github.com/LiquidityTech/ltp-rapidx-skill` 分发。

---

## 前置要求

- Node.js **20** 或更高版本
- npm
- 大赛 Portfolio Key（`LTP_ACCESS_KEY` + `LTP_SECRET_KEY`）
- `LTP_API_HOST` — 主办方提供的 API 地址（**必填，无默认值**）
- （可选）支持 MCP 的智能体宿主：Claude Desktop、Cursor、Codex 等

---

## 第一步 — 安装 Skills（如宿主支持）

Skills 指导智能体完成 CLI 安装、凭证配置、MCP 配置、自检与交易工作流。

| 智能体宿主 | 安装命令 |
|-----------|---------|
| **Codex** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a codex -y` |
| **Claude Code** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a claude-code -y` |
| **Cursor** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a cursor -y` |
| **OpenCode** | `npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a opencode -y` |
| **OpenClaw** | `openclaw skills install ltp-rapidx-config && openclaw skills install ltp-rapidx-trading` |
| **Hermes** | `hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config && hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading` |

安装后，请您的智能体执行 `ltp-rapidx-config`，后续安装步骤将自动完成。

如果宿主**不支持 Skills**，请跳至第二步手动配置。

---

## 第二步 — 安装 CLI

```bash
npm install -g @liquiditytech/rapidx-cli
```

验证：

```bash
rapidx --version
rapidx schema --json
```

---

## 第三步 — 配置凭证

```bash
export LTP_ACCESS_KEY="your_access_key"
export LTP_SECRET_KEY="your_secret_key"
export LTP_API_HOST="<主办方提供的API地址>"
```

> `LTP_API_HOST` 无默认值，缺少时返回 `RCORE01003`。  
> 切勿将密钥粘贴到公开聊天、仓库、截图或日志中。

---

## 第四步 — 选择接入路径

### 路径 A — 支持 MCP 的智能体（推荐）

将以下内容加入智能体 MCP 配置：

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "your_access_key",
        "LTP_SECRET_KEY": "your_secret_key",
        "LTP_API_HOST": "<主办方提供的API地址>"
      }
    }
  }
}
```

重启宿主后，MCP 工具以 `rapidx/` 前缀对外可见，如 `rapidx/market/get-ticker`。

### 路径 B — CLI-only 智能体 / 脚本

```bash
# 只读自检
rapidx self-check --read-only --json

# 行情查询
rapidx market get-ticker --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json

# 账户快照
rapidx account overview --json
rapidx account balance --input '{"mode":"portfolio"}' --json

# 订单预览 → 再提交
rapidx order place-preview --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"60000","quantity":"0.001","maxNotional":"100","clientOrderId":"agent-001","postOnly":true}' --json
```

每次响应均使用稳定的 JSON 信封：

```json
{
  "ok": true,
  "status": "PASS",
  "data": {},
  "evidence": [{ "source": "real_tool_call", "toolOrCommandEvidence": "rapidx ..." }]
}
```

CLI-only 模式使用 `--input '<json>'` 或 `--input @/path/to/file.json`，拒绝复杂 shell 组合（`cd ... && node ...`、`sh -c`、命令替换）。

### 路径 C — 原始 REST / WebSocket

不使用 CLI 的直接 HTTP 接入方式，见 [`rapidx-api.zh-CN.md`](./rapidx-api.zh-CN.md)。

---

## 第五步 — 运行自检

```bash
rapidx auth check
rapidx self-check --read-only --json
```

每个检查项报告 **PASS** / **EXPECTED_ERROR** / **FAIL** / **NOT_VERIFIED**。

---

## 写操作 — 强制预览

所有写操作（订单、仓位、杠杆、算法单）均需先执行**预览步骤**再提交。

```bash
# 1. 预览 → 获取 previewId + continueConsentId
rapidx order place-preview --input '{...}' --json

# 2. 提交（带原始参数 + previewId + continueConsentId）
rapidx order place --input '{...,"previewId":"rpv_xxx","continueConsentId":"confirm_rpv_xxx"}' --json

# 3. 确认状态
rapidx order list --input '{"symbol":"BINANCE_PERP_BTC_USDT"}' --json
```

预览与提交**必须使用同一通道**（同为 CLI 或同为 MCP，不得混用）。

---

## 工具目录

| 分类 | CLI | MCP 工具 |
|------|-----|---------|
| **诊断** | `rapidx self-check --read-only` · `rapidx schema` · `rapidx doctor` | `rapidx/self-check` · `rapidx/tools` |
| **行情（7）** | `rapidx market get-ticker` · `get-orderbook` · `get-klines` · `get-funding-rate` · `get-mark-price` · `get-symbol-info` · `get-open-interest` | `rapidx/market/*` |
| **账户（3）** | `rapidx account overview` · `balance` · `set-position-mode` | `rapidx/account/*` |
| **订单（7+）** | `rapidx order place-preview` · `place` · `amend-preview` · `amend` · `cancel-preview` · `cancel` · `get` · `list` · `history` | `rapidx/order/*` |
| **仓位（4）** | `rapidx position list` · `history` · `close` · `set-leverage` | `rapidx/position/*` |
| **算法单（4）** | `rapidx algo list` · `place` · `amend` · `cancel` | `rapidx/algo/*` |
| **交易工具** | `rapidx trade preview` · `verify-live` | `rapidx/trade/*` |

完整规范见 [`rapidx-api.zh-CN.md`](./rapidx-api.zh-CN.md)。

---

## 安全模型

- **所有写操作强制预览**：订单、仓位、杠杆、算法单写入均需先 preview，预览不提交真实订单。
- **不涉及真实资产**：大赛运行于 UAT 模拟撮合引擎。
- **密钥永不外泄**：所有输出、日志、报告中密钥均经掩码处理。
- **每次写操作后状态确认**：提交后需调用 `rapidx order get` 或 `rapidx position list` 确认，禁止盲目重试。
- **`LTP_API_HOST` 必填**：缺失时返回 `RCORE01003`。

---

## 下一步

| 文档 | 内容 |
|------|------|
| [`resources.zh-CN.md`](./resources.zh-CN.md) | 端点、npm 包、品种、日程、支持 |
| [`rapidx-api.zh-CN.md`](./rapidx-api.zh-CN.md) | 完整 CLI + MCP + REST/WS 参考 |

---

## 支持

- **文档**：<https://github.com/LiquidityTech/ltp-arena-docs>
- **Issue 追踪**：<https://github.com/liquiditytech/rapidx-cli/issues>
- **联系方式**：详见 `resources.zh-CN.md`

---

**English**: [README.md](./README.md)
