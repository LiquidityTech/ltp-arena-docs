# Query Transactions (Pageable)


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-transactions-pageable

**Rate limit**: 1 request per 10 seconds

## Endpoint

```
GET /api/v1/trading/executions/pageable
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `orderId` | string | No |  |
| `sym` | string | No | Filter by symbol (optional) |
| `exchange` | string | No |  |
| `businessType` | string | No |  |
| `begin` | string | No |  |
| `end` | string | No |  |
| `page` | string | No | Page number |
| `pageSize` | string | No | Items per page |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "page": 1,
    "pageSize": 1000,
    "pageNum": 1,
    "totalSize": 1,
    "list": [
      {
        "transactionId": "exec_123456",
        "portfolioId": "1234567891234567",
        "orderId": "order_456",
        "exchangeType": "BINANCE",
        "businessType": "MARGIN",
        "sym": "BINANCE_MARGIN_BTC_USDT",
        "side": "SELL",
        "quantity": "0.001",
        "price": "81619.34",
        "tradingFee": "0.01877245",
        "tradingFeeCoin": "USDT",
        "fee": "0.01877245",
        "feeCoin": "USDT",
        "rebate": "0",
        "rebateCoin": "",
        "rpnl": "0",
        "clientOrderId": "2104172237333826",
        "algoOrderId": "0",
        "createAt": "1763977805203",
        "execType": "TAKER",
        "tradeSource": ""
      }
    ]
  }
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
PARAMS="sym=BINANCE_PERP_BTC_USDT"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/executions/pageable?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
