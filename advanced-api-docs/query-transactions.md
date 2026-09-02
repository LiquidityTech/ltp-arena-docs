# Query Transactions


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-transactions

**Rate limit**: 1 request per 10 seconds

## Endpoint

```
GET /api/v1/trading/executions
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `orderId` | string | No |  |
| `sym` | string | No | Filter by symbol (optional) |
| `businessType` | string | No |  |
| `exchange` | string | No |  |
| `begin` | string | No |  |
| `end` | string | No |  |
| `limit` | string | No |  |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": [
    {
      "transactionId": "exec_123456",
      "portfolioId": "1234567891234567",
      "orderId": "order_456",
      "exchangeType": "BINANCE",
      "businessType": "PERP",
      "sym": "BINANCE_PERP_BTC_USDT",
      "side": "BUY",
      "quantity": "0.01",
      "price": "62000",
      "tradingFee": "0.62",
      "tradingFeeCoin": "USDT",
      "fee": "0.62",
      "feeCoin": "USDT",
      "rebate": "0",
      "rebateCoin": "",
      "rpnl": "100.50",
      "clientOrderId": "client_order_789",
      "algoOrderId": "0",
      "createAt": "1769068022485",
      "execType": "TAKER",
      "tradeSource": ""
    }
  ]
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

curl -s "${BASE}/api/v1/trading/executions?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
