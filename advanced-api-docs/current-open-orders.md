# Current Open Orders


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/current-open-orders

**Rate limit**: 240 requests per 60 seconds

## Endpoint

```
GET /api/v1/trading/orders
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Filter by symbol (optional) |
| `exchange` | string | No |  |
| `businessType` | string | No |  |
| `begin` | string | No |  |
| `end` | string | No |  |
| `page` | string | No |  |
| `pageSize` | string | No |  |

## Response (HTTP 200)

```json
{
    "code": 200000,
    "message": "Success",
    "data": {
      "page": 1,
      "pageSize": 20,
      "pageNum": 1,
      "totalSize": 1,
      "list": [
        {
          "portfolioId": "1704269773524000",
          "portfolioName": null,
          "orderId": "2171125266142788",
          "clientOrderId": "2171125266142788",
          "orderState": "OPEN",
          "sym": "BINANCE_PERP_BTC_USDT",
          "side": "SELL",
          "exchangeOrderType": "LIMIT",
          "exchangeType": "BINANCE",
          "businessType": "PERP",
          "orderQty": "0.002",
          "quoteOrderQty": "0",
          "limitPrice": "73330",
          "timeInForce": "GTC",
          "executedQty": "0",
          "executedAmount": "0",
          "executedAvgPrice": "0",
          "reason": "",
          "action": "",
          "actionMsg": "",
          "cancelType": "",
          "amendType": "",
          "createAt": "1779940601086",
          "updateAt": "1779940601116",
          "fee": "0",
          "feeCoin": "",
          "orderType": "DMA",
          "reduceOnly": true,
          "leverage": 5,
          "lastExecutedQty": "0",
          "lastExecutedPrice": "0",
          "lastExecutedAmount": "0",
          "borrowAmount": "0",
          "borrowAsset": "",
          "positionSide": "NONE",
          "algoOrderId": null,
          "rebate": "0",
          "rebateCoin": "",
          "tpTriggerPrice": "",
          "tpTriggerType": "",
          "tpPrice": "",
          "slTriggerPrice": "",
          "slTriggerType": "",
          "slPrice": ""
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
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/orders" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
