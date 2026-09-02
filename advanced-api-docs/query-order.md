# Query Order Detail


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/query-order

**Rate limit**: 240 requests per 60 seconds

## Endpoint

```
GET /api/v1/trading/order
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `orderId` | string | No | Platform-generated order ID |
| `clientOrderId` | string | No | Client-assigned order ID (alternative to `orderId`) |

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": {
    "portfolioId": "",
    "portfolioName": null,
    "orderId": "",
    "clientOrderId": "",
    "orderState": "REJECT",
    "sym": "BINANCE_PERP_FXS_USDT",
    "side": "BUY",
    "exchangeOrderType": "LIMIT",
    "exchangeType": "BINANCE",
    "businessType": "PERP",
    "orderQty": "50",
    "quoteOrderQty": "0",
    "limitPrice": "0.1",
    "timeInForce": "GTC",
    "executedQty": "0",
    "executedAmount": "0",
    "executedAvgPrice": "0",
    "reason": "",
    "action": "",
    "actionMsg": "",
    "cancelType": "SYSTEM",
    "amendType": "",
    "createAt": "1769068022485",
    "updateAt": "1769068022495",
    "fee": "0",
    "feeCoin": "",
    "orderType": "DMA",
    "reduceOnly": false,
    "leverage": 3,
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
}
```

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

NONCE=$(date +%s)
PARAMS="orderId=1234567890123456"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/order?orderId=1234567890123456" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
