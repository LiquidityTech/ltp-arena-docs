# Get Portfolio Overview


**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/reference/get-portfolio-overview

**Rate limit**: 1 request per 3 seconds

## Endpoint

```
GET /api/v1/trading/account
```

## Response (HTTP 200)

```json
{
  "code": 200000,
  "message": "Success",
  "data": [
    {
      "portfolioId": "17xxxxxxxxxxxxxx",
      "exchangeType": "BINANCE",
      "equity": "5685978.07787916",
      "maintainMargin": "164182.02640801",
      "positionValue": "3465349.06217066",
      "uniMMR": "28.4139281239",
      "riskRatio": "0.0352",
      "accountStatus": "NORMAL",
      "marginValue": "4666269.16976568",
      "frozenMargin": "482754.35903806",
      "perpMargin": "482754.35903806",
      "debtMargin": "0",
      "openLossMargin": "1212.87217175",
      "validMargin": "4665056.29759393",
      "availableMargin": "4182301.93855587",
      "upnl": "4654259.07370681",
      "positionMode": "NET"
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

curl -s "${BASE}/api/v1/trading/account" \
  -X GET \
  -H "X-MBX-APIKEY: ${{API_KEY}}" \
  -H "nonce: ${{NONCE}}" \
  -H "signature: ${{SIG}}" \
  -H "Content-Type: application/json" | jq .
```
