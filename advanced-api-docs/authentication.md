# Authentication

**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/docs/authentication

All REST API requests must be authenticated. Two signature schemes are supported — use V2 for trading and account endpoints, V1 for the Feeds REST API.

---

## Overview

| Scheme | Where credentials go | Timestamp format | Used by |
|--------|---------------------|-----------------|---------|
| **V2 (recommended)** | HTTP headers | Unix seconds (`nonce`) | Trading API, Account API, Market API |
| **V1** | Query parameters | UTC string `yyyy-MM-dd'T'HH:mm:ss` | Feeds REST API only |

---

## V2 Signature (Recommended)

### Required Request Headers

| Header | Value |
|--------|-------|
| `X-MBX-APIKEY` | Your Access Key |
| `nonce` | Current Unix timestamp in **seconds** (string) |
| `signature` | HMAC-SHA256 hex digest (see algorithm below) |
| `Content-Type` | `application/json` |

### Signature Algorithm

```
Step 1 — Collect parameters
  GET:              all query parameters
  POST/PUT/DELETE:  all JSON body fields

Step 2 — Sort alphabetically by key
  sorted_payload = "key1=val1&key2=val2&..."
  (empty string if no parameters)

Step 3 — Append nonce
  message = sorted_payload + "&" + nonce
  (no parameters: message = "&" + nonce)

Step 4 — Compute HMAC-SHA256
  signature = HMAC-SHA256(secret_key, message).hexdigest()
```

Server allows **±60 seconds** clock skew on `nonce`.

### Python Example

```python
import hashlib, hmac, time
import requests

API_KEY    = "<your-access-key>"
SECRET_KEY = "<your-secret-key>"
BASE       = "https://api.ltp-contest.com"


def sign_v2(params: dict, nonce: int) -> str:
    """Build HMAC-SHA256 signature for V2."""
    sorted_payload = "&".join(f"{k}={v}" for k, v in sorted(params.items()))
    message = (sorted_payload + "&" if sorted_payload else "&") + str(nonce)
    return hmac.new(SECRET_KEY.encode(), message.encode(), hashlib.sha256).hexdigest()


def api_get(path: str, params: dict = {}) -> dict:
    nonce = int(time.time())
    headers = {
        "X-MBX-APIKEY": API_KEY,
        "nonce": str(nonce),
        "signature": sign_v2(params, nonce),
        "Content-Type": "application/json",
    }
    r = requests.get(BASE + path, params=params, headers=headers, timeout=10)
    return r.json()


def api_post(path: str, body: dict = {}) -> dict:
    nonce = int(time.time())
    headers = {
        "X-MBX-APIKEY": API_KEY,
        "nonce": str(nonce),
        "signature": sign_v2(body, nonce),
        "Content-Type": "application/json",
    }
    r = requests.post(BASE + path, json=body, headers=headers, timeout=10)
    return r.json()


if __name__ == "__main__":
    import json

    # Query account overview (GET, no params)
    result = api_get("/api/v1/trading/account")
    print(json.dumps(result, indent=2))

    # Place a limit order (POST with body)
    result = api_post("/api/v1/trading/order", {
        "sym": "BINANCE_PERP_BTC_USDT",
        "side": "BUY",
        "positionSide": "LONG",
        "orderType": "LIMIT",
        "orderQty": "0.001",
        "limitPrice": "60000",
        "timeInForce": "GTC",
        "clientOrderId": "demo-001",
    })
    print(json.dumps(result, indent=2))
```

### Shell / curl Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"

# ── helper: sign a query string ────────────────────────────────────────────
# Usage: sign <sorted_query_params_string>
sign() {
  local payload="$1"
  local nonce
  nonce=$(date +%s)
  local message="${payload:+${payload}&}${nonce}"
  local sig
  sig=$(echo -n "$message" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')
  echo "${nonce} ${sig}"
}

# ── GET /api/v1/trading/account (no query params) ──────────────────────────
read -r NONCE SIG <<< "$(sign "")"

curl -s "${BASE}/api/v1/trading/account" \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" \
  -H "Content-Type: application/json" | jq .

# ── GET with query params: GET /api/v1/trading/perp/leverage?sym=BINANCE_PERP_BTC_USDT
# Sorted params: sym=BINANCE_PERP_BTC_USDT
read -r NONCE SIG <<< "$(sign "sym=BINANCE_PERP_BTC_USDT")"

curl -s "${BASE}/api/v1/trading/perp/leverage?sym=BINANCE_PERP_BTC_USDT" \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" \
  -H "Content-Type: application/json" | jq .
```

---

## V1 Signature (Feeds API Only)

V1 passes all credentials as query parameters — no headers are needed. This scheme is accepted only by the Feeds REST endpoints (`/api/v1/feeds/*`).

### Query Parameters

| Parameter | Description |
|-----------|-------------|
| `apiKey` | Your Access Key |
| `timestamp` | UTC datetime string, format `yyyy-MM-dd'T'HH:mm:ss`, e.g. `2026-07-02T03:30:00` |
| `sign` | HMAC-SHA256 hex digest (see algorithm below) |

Server allows **±5 minutes** clock skew on `timestamp`.

### Signature Algorithm

```
Step 1 — Collect all parameters including apiKey and timestamp,
          exclude sign and any parameters with empty values.

Step 2 — Sort by key alphabetically.

Step 3 — Join as "key1=value1&key2=value2&..."

Step 4 — HMAC-SHA256(secret_key, joined_string).hexdigest()
```

### Python Example

```python
import hashlib, hmac
from datetime import datetime, timezone
import requests

API_KEY    = "<your-access-key>"
SECRET_KEY = "<your-secret-key>"
BASE       = "https://api.ltp-contest.com"


def sign_v1(params: dict) -> str:
    """Build HMAC-SHA256 signature for V1 (Feeds API)."""
    filtered = {k: v for k, v in params.items()
                if k != "sign" and v is not None and v != ""}
    message = "&".join(f"{k}={v}" for k, v in sorted(filtered.items()))
    return hmac.new(SECRET_KEY.encode(), message.encode(), hashlib.sha256).hexdigest()


def feeds_get(path: str, params: dict = {}) -> dict:
    ts = datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%S")
    all_params = {"apiKey": API_KEY, "timestamp": ts, **params}
    all_params["sign"] = sign_v1(all_params)
    r = requests.get(BASE + path, params=all_params, timeout=10)
    return r.json()


if __name__ == "__main__":
    import json, time

    # Query latest news
    result = feeds_get("/api/v1/feeds/queryNews", {"page": "1", "pageSize": "10"})
    print(json.dumps(result, indent=2))
```

---

## Common Authentication Errors

| Code | Meaning | Fix |
|------|---------|-----|
| `2000` | Signature mismatch or missing headers | Verify HMAC inputs — check sort order and string concatenation |
| `2001` | IP not whitelisted | Contact the organizer to whitelist your IP |
| `2002` | Invalid API authorization | Check `X-MBX-APIKEY` value |
| `7000` | `nonce` too far from server time | Sync your system clock; server allows ±60 s |
| `100018` | API key not found or missing | Verify `X-MBX-APIKEY` header is present and correct |
| `100041` | API key frozen | Contact the organizer |
| `1004` (V1) | Signature error (Feeds API) | Re-check V1 algorithm — note `timestamp` format differs from V2 |

### Troubleshooting checklist

1. **Headers present**: `X-MBX-APIKEY`, `nonce`, `signature` are all set.
2. **Correct secret**: the HMAC uses `SECRET_KEY`, not `API_KEY`.
3. **Sort order**: parameters are sorted by key in ASCII/alphabetical order.
4. **Nonce is seconds**: `nonce` is a Unix timestamp in **seconds**, not milliseconds.
5. **No extra parameters**: do not include parameters you are not sending in the signature.
6. **GET vs POST**: for GET requests sign the query parameters; for POST sign the JSON body fields.
7. **V1 timestamp format**: must be UTC string `yyyy-MM-dd'T'HH:mm:ss` (not Unix ms).

