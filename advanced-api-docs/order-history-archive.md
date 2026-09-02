# Order History Archive


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/order-history-archive

**Rate limit**: 6 requests per 10 seconds

## Endpoint

```
GET /api/v1/trading/archive/history/orders
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Filter by symbol (optional) |
| `exchange` | string | No |  |
| `businessType` | string | No |  |
| `begin` | string | No |  |
| `end` | string | No |  |
| `filterExecuted` | boolean | No |  |
| `page` | string | No | Page number (default 1) |
| `pageSize` | string | No | Items per page (default 100, max 1000) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "page": 1,
    "pageSize": 10,
    "pageNum": 1,
    "totalSize": 1,
    "list": [
      {
        "portfolioId": "1732260294043000",
        "portfolioName": "Main Portfolio",
        "orderId": "2106591138643265",
        "clientOrderId": "myorder001",
        "orderState": "FILLED",
        "sym": "BINANCE_SPOT_ETH_USDT",
        "side": "BUY",
        "exchangeOrderType": "LIMIT",
        "exchangeType": "BINANCE",
        "orderType": "DMA",
        "businessType": "SPOT",
        "orderQty": "0.5",
        "quoteOrderQty": "",
        "limitPrice": "3100",
        "timeInForce": "GTC",
        "reason": "",
        "executedQty": "0.5",
        "executedAvgPrice": "3100",
        "executedAmount": "1550",
        "reduceOnly": false,
        "lastExecutedQty": "0.5",
        "lastExecutedPrice": "3100",
        "lastExecutedAmount": "1550",
        "leverage": 1,
        "fee": "1.55",
        "feeCoin": "USDT",
        "rebate": "0",
        "rebateCoin": "",
        "borrowAmount": "0",
        "borrowAsset": "",
        "positionSide": "NONE",
        "tpTriggerPrice": "",
        "tpTriggerType": "LAST_PRICE",
        "tpPrice": "0",
        "slTriggerPrice": "",
        "slTriggerType": "LAST_PRICE",
        "slPrice": "0",
        "createAt": "1741788600000",
        "updateAt": "1741788615000",
        "cancelType": "",
        "amendType": "",
        "algoOrderId": ""
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

curl -s "${BASE}/api/v1/trading/archive/history/orders?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
