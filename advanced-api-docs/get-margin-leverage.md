# Get Margin Leverage


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-margin-leverage

**Rate limit**: 20 requests per 10 seconds

## Endpoint

```
GET /api/v1/trading/margin/leverage
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `exchangeType` | string | No | Exchange, e.g. `BINANCE` (optional) |
| `coin` | string | No | Coin symbol, e.g. `BTC` (optional) |
| `page` | string | No |  |
| `pageSize` | string | No |  |

## Response (HTTP 200)

```json
{
    "code": 200000,
    "message": "Success",
    "data": {
        "page": 1,
        "pageSize": 1000,
        "pageNum": 1,
        "totalSize": 3,
        "list": [
            {
                "coin": "ETH",
                "exchangeType": "BINANCE",
                "leverage": "1"
            },
            {
                "coin": "USDC",
                "exchangeType": "BINANCE",
                "leverage": "1"
            },
            {
                "coin": "USDT",
                "exchangeType": "BINANCE",
                "leverage": "2"
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
PARAMS="exchangeType=BINANCE&coin=BTC"
MSG="${{PARAMS:+${{PARAMS}}&}}${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/margin/leverage?exchangeType=BINANCE&coin=BTC" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
