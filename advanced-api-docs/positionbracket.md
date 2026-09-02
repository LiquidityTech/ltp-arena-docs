# Get Position Tier


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/positionbracket

**Rate limit**: 1 request per 20 seconds

## Endpoint

```
GET /api/v1/trading/positionBracket
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `sym` | string | No | Trading pair unique identifier ,   example: BINANCE_PERP_BTC_USDT,OKX_PERP_BTC_USDT, |

## Response (HTTP 200)

```json
{
    "BINANCE_PERP_ETH_USDT": [
      {
        "sym": "BINANCE_PERP_ETH_USDT",
        "minNotionalValue": "0",
        "maxNotionalValue": "15000000",
        "maxLeverage": "20",
        "mmRate": "0.0200",
        "riskLevel": "1"
      },
      {
        "sym": "BINANCE_PERP_ETH_USDT",
        "minNotionalValue": "15000000",
        "maxNotionalValue": "30000000",
        "maxLeverage": "10",
        "mmRate": "0.0500",
        "riskLevel": "2"
      }
    ],
    "OKX_PERP_ETH_USDT": [
      {
        "sym": "OKX_PERP_ETH_USDT",
        "minNotionalValue": "0",
        "maxNotionalValue": "15000000",
        "maxLeverage": "20",
        "mmRate": "0.0200",
        "riskLevel": "1"
      },
      {
        "sym": "OKX_PERP_ETH_USDT",
        "minNotionalValue": "15000000",
        "maxNotionalValue": "30000000",
        "maxLeverage": "10",
        "mmRate": "0.0500",
        "riskLevel": "2"
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

curl -s "${BASE}/api/v1/trading/positionBracket?sym=BINANCE_PERP_BTC_USDT" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
