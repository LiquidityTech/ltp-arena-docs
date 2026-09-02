# Leaderboard API

Query your own team's ranking and scoring snapshot for a competition phase. The `portfolioId` is derived server-side from your API Key — do not pass it.

**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`

**Rate limit**: 10 requests / 60 s

## Endpoint

```
GET /api/v1/tracka/ranking/self
```

## Authentication

V2 signature headers:

| Header | Description |
|--------|-------------|
| `X-MBX-APIKEY` | Your access key |
| `nonce` | Current Unix timestamp (seconds) |
| `signature` | `HMAC-SHA256("<query_string>&<nonce>", secretKey)`, lowercase hex |

> Only business parameters (e.g. `phase`) go into the signed `query_string`. Do **not** include `portfolioId` in the signature computation.

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `phase` | string | Yes | Competition phase identifier, e.g. `PHASE_I`, `PHASE_II` |

## Response (HTTP 200)

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "portfolioId": 2180978100000003,
    "rankNo": 5,
    "teamName": "Alpha Team",
    "compositeScore": "87.32",
    "sharpeRatio": "1.45",
    "sSharpe": "76.20",
    "pnlUsdt": "12345.67",
    "sPnl": "82.10",
    "roi": "0.1234",
    "sRoi": "79.50",
    "mdd": "0.0523",
    "mddTierScore": "90.00",
    "turnoverRate": "0.3210",
    "totalTrades": 1280,
    "quoteVolume": "9876543.21",
    "equity": "112345.67",
    "nav": "1.1234",
    "calcTime": 1753264800000,
    "phase": "PHASE_I",
    "dt": 1753261200000,
    "status": "ACTIVE",
    "isEliminated": false,
    "aiCost": {
      "dt": 1753261200000,
      "cumulativeAiCostUsdt": "12.34",
      "source": "openai",
      "sourceUpdatedAt": 1753261200000
    },
    "aiTokens": {
      "usageDate": 1753228800000,
      "promptTokens": 50000,
      "completionTokens": 10000,
      "totalTokens": 60000,
      "source": "openai",
      "sourceUpdatedAt": 1753261200000
    }
  }
}
```

### Main fields

| Field | Type | Description |
|-------|------|-------------|
| `portfolioId` | long | Portfolio identifier |
| `rankNo` | integer | Current rank position |
| `teamName` | string | Team display name |
| `compositeScore` | string | Composite score |
| `sharpeRatio` | string | Sharpe ratio |
| `sSharpe` | string | Sharpe percentile score |
| `pnlUsdt` | string | Phase PnL (USDT) |
| `sPnl` | string | PnL percentile score |
| `roi` | string | Cumulative return (decimal, `0.1234` = 12.34%) |
| `sRoi` | string | ROI percentile score |
| `mdd` | string | Max drawdown (decimal) |
| `mddTierScore` | string | MDD tier score |
| `turnoverRate` | string | Turnover rate (decimal) |
| `totalTrades` | long | Total executed trades |
| `quoteVolume` | string | Total traded notional (USDT) |
| `equity` | string | Latest equity (USDT) |
| `nav` | string | Latest net asset value (NAV) |
| `calcTime` | long | Calculation timestamp (ms) |
| `phase` | string | Phase identifier |
| `dt` | long | Snapshot hour timestamp (ms) |
| `status` | string | Participant status: `ACTIVE` / `PENDING` / `DISQUALIFIED` |
| `isEliminated` | boolean | Whether the team has been eliminated |
| `aiCost` | object \| null | Cumulative AI cost snapshot; `null` when no data |
| `aiTokens` | object \| null | Daily AI token usage; `null` when no data |

### `aiCost` fields

| Field | Type | Description |
|-------|------|-------------|
| `dt` | long | Snapshot timestamp (ms) |
| `cumulativeAiCostUsdt` | string | Cumulative AI cost (USDT) |
| `source` | string \| null | Data source identifier |
| `sourceUpdatedAt` | long \| null | Source update time (ms) |

### `aiTokens` fields

| Field | Type | Description |
|-------|------|-------------|
| `usageDate` | long | Usage date (ms, day precision) |
| `promptTokens` | long | Input tokens |
| `completionTokens` | long | Output tokens |
| `totalTokens` | long | Total tokens |
| `source` | string \| null | Data source identifier |
| `sourceUpdatedAt` | long \| null | Source update time (ms) |

## Error codes

| `code` | Description |
|--------|-------------|
| `2000` | Signature verification failed (apiKey missing or signature invalid) |
| `30015` | Invalid phase identifier (outside supported range) |
| `30016` | No ranking data for this API Key's portfolio in the given phase |

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"
PHASE="PHASE_I"

NONCE=$(date +%s)
QUERY="phase=${PHASE}"
MSG="${QUERY}&${NONCE}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/tracka/ranking/self?${QUERY}" \
  -X GET \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" | jq .
```

## Notes

- The response is a snapshot computed at `calcTime` / `dt`; it is not real-time. Poll at a modest cadence to track rank changes — the `10 req / 60 s` limit is more than enough.
- `equity` and `nav` here are the authoritative values for the [elimination threshold](../docs/resources.md#11-safety--compliance-reminders): elimination triggers when `equity < 800` USDT or `nav < 0.8`.
- `aiCost.cumulativeAiCostUsdt` reflects progress against the daily $10 USD AI budget — see [AI API Usage](../docs/resources.md#ai-api-usage).
