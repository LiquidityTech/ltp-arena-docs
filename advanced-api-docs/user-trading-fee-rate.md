# User Trading Fee Rate


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/user-trading-fee-rate

**Rate limit**: 1 request per 20 seconds

## Endpoint

```
GET /api/v1/trading/userFeeRate
```

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": [
    {
      "exchangeType": "BINANCE",
      "businessType": "SPOT",
      "takerFeeRate": "0.00035",
      "makerFeeRate": "0.00010",
      "level": "2",
      "groupList": [
        {
          "groupName": "A",
          "takerFee": "0.00035",
          "makerFee": "0.00010",
          "symList": [
            "BINANCE_SPOT_ETH_USDC",
            "BINANCE_SPOT_HIGH_USDT",
            "..."
          ]
        },
        {
          "groupName": "B",
          "takerFee": "0.00035",
          "makerFee": "0.00010",
          "symList": []
        },
        {
          "groupName": "C",
          "takerFee": "0.00035",
          "makerFee": "0.00010",
          "symList": [
            "BINANCE_SPOT_XRP_BTC"
          ]
        }
      ]
    },
    {
      "exchangeType": "BINANCE",
      "businessType": "PERP",
      "takerFeeRate": "0.03",
      "makerFeeRate": "0.01",
      "level": "1",
      "groupList": [
        {
          "groupName": "A",
          "takerFee": "0.03",
          "makerFee": "0.01",
          "symList": [
            "BINANCE_PERP_GMX_USDT",
            "BINANCE_PERP_ADA_USDT",
            "..."
          ]
        },
        {
          "groupName": "B",
          "takerFee": "0.029",
          "makerFee": "0.009",
          "symList": []
        }
      ]
    },
    {
      "exchangeType": "OKX",
      "businessType": "SPOT",
      "takerFeeRate": "0.03",
      "makerFeeRate": "0.01",
      "level": "1",
      "groupList": [
        {
          "groupName": "A",
          "takerFee": "0.03",
          "makerFee": "0.01",
          "symList": [
            "OKX_SPOT_BTC_USDT",
            "OKX_SPOT_SOL_USDT",
            "..."
          ]
        }
      ]
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
MSG="&${{NONCE}}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trading/userFeeRate" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
