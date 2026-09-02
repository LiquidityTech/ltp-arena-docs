# Error Codes

**Base URL:** Track A Phase II / Track B (Mainnet): `https://api.liquiditytech.com` · Phase I (Sandbox): `https://api.ltp-contest.com`
> **Official API docs:** https://apidocliquidity.readme.io/docs/error-codes

All REST responses use `code: 200000` for success. Any other value indicates an error.

## Common Error Codes

| Code | Category | Meaning |
|------|----------|---------|
| `200000` | Success | Request completed successfully |
| `2000` | Auth | Signature mismatch or missing headers |
| `2001` | Auth | IP address not whitelisted |
| `2002` | Auth | API authorization invalid |
| `2003` | Auth | No access permission |
| `7000` | Auth | `nonce` timestamp too far from server time (±60 s allowed) |
| `100018` | Auth | API key not found |
| `100041` | Auth | API key frozen |
| `400001` | Input | Invalid request parameters |
| `400009` | Input | Invalid symbol format |
| `401018` | Order | Order not found |
| `401023` | Order | Insufficient margin |
| `401058` | Order | Order already in terminal state |
| `401097` | Position | Position quantity is 0 (nothing to close) |
| `401117` | Order | Order already completed — cannot cancel |
| `406010` | Order | Duplicate `clientOrderId` |
| `406012` | Order | Position mode switch blocked (open positions exist) |
| `40000` | System | System error |
| `50001` | System | Service temporarily unavailable |

## Feeds API Error Codes (V1 signature endpoints)

| Code | Meaning |
|------|---------|
| `200` | Success |
| `1003` | API key not found |
| `1004` | Signature error |
| `1007` | IP not on allowlist |
| `1010` | KYC level insufficient |
| `2003` | No access permission |

## Response format on error

```json
{
  "code": 401018,
  "message": "The order was not found",
  "data": {}
}
```

See also: [authentication.md](authentication.md) for auth-specific error codes and troubleshooting.

