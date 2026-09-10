# Track B Ranking — Self Query

Query your own fund's latest ranking and scoring snapshot for a Track B competition phase. The caller passes `participantId` (the participation identifier); the server resolves it to the bound fund and returns the latest ranking snapshot.

**Base URL:** `https://api.ltp-contest.com`

## Endpoint

```
GET /api/v1/trackb/ranking/self
```

## Authentication

V2 signature headers:

| Header | Description |
|--------|-------------|
| `X-MBX-APIKEY` | Your access key |
| `nonce` | Current Unix timestamp (seconds) |
| `signature` | `HMAC-SHA256("<query_string>&<nonce>", secretKey)`, lowercase hex |

> Only the business parameters submitted in this request (i.e. `participantId`) go into the signed `query_string`. The `paramMap` used for signing must match the URL query parameter set exactly.

**Signature example:**

```
query_string = participantId=<participantId>
sign_content = participantId=<participantId>&1753264849
signature    = HMAC-SHA256(sign_content, SECRET_KEY)
```

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `participantId` | string | Yes | Participation identifier; the server resolves it to the fund identifier (`fund/xxxx`) |

## Response (HTTP 200)

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "fundName": "fund/abc123",
    "rankNo": 5,
    "teamName": "Alpha Team",
    "compositeScore": "87.32",
    "sharpeRatio": "1.45",
    "sSharpe": "76.20",
    "roi": "0.1234",
    "sRoi": "79.50",
    "mdd": "0.0523",
    "mddTierScore": "90",
    "avgNetAssets": "105000.00",
    "scalingScore": "95",
    "accumNav": "1.1234",
    "netAssets": "112345.67",
    "calcTime": 1753264800000,
    "phase": "PHASE_I",
    "dt": 1753261200000,
    "status": "ACTIVE",
    "isEliminated": false
  }
}
```

> The self-query response does **not** include an equity history curve (`equityCurve`); it returns only the latest snapshot metrics.

### Main fields

| Field | Type | Description |
|-------|------|-------------|
| `fundName` | string | Fund identifier (resolved from `participantId`, format `fund/xxxx`) |
| `rankNo` | integer | Current rank position |
| `teamName` | string | Team display name |
| `compositeScore` | string | Composite score |
| `sharpeRatio` | string | Sharpe ratio |
| `sSharpe` | string | Sharpe percentile score |
| `roi` | string | Cumulative return (decimal, `0.1234` = 12.34%) |
| `sRoi` | string | ROI percentile score |
| `mdd` | string | Max drawdown (decimal) |
| `mddTierScore` | string | MDD tier score |
| `avgNetAssets` | string | Average net assets (Scaling basis) |
| `scalingScore` | string | Scaling capacity score |
| `accumNav` | string | Accumulated NAV |
| `netAssets` | string | Current net assets |
| `calcTime` | long | Calculation timestamp for this record (ms, UTC) |
| `phase` | string | Competition phase identifier |
| `dt` | long | Snapshot hour timestamp (ms, UTC) |
| `status` | string | Participant status: `ACTIVE` / `PENDING` / `DISQUALIFIED` |
| `isEliminated` | boolean | Whether the fund has been eliminated (Track B has no hard elimination rule; elimination is an operational/manual decision) |

### Data conventions

- Numeric metrics: `string`, `scale=20`, `HALF_UP`, trailing zeros stripped.
- Timestamps: `long`, UTC epoch-millisecond.

## Error codes

| `code` | Description |
|--------|-------------|
| `2000` | Signature verification failed (invalid signature, invalid/expired `nonce`) |
| `100018` | API key does not exist or has been disabled (`USER_API_NOT_EXIST`) |
| `1100004` | No ranking data yet for the fund bound to this `participantId` (`FEEDS_RANKING_NOT_FOUND`) |
| `1100005` | The `participantId` is not bound to any competition fund (`FEEDS_FUND_NOT_BOUND`, Track B specific) |

## Example

```bash
API_KEY="<your-access-key>"
SECRET_KEY="<your-secret-key>"
BASE="https://api.ltp-contest.com"
PARTICIPANT_ID="<your-participant-id>"

NONCE=$(date +%s)
QUERY="participantId=${PARTICIPANT_ID}"
MSG="${QUERY}&${NONCE}"
SIG=$(echo -n "$MSG" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -s "${BASE}/api/v1/trackb/ranking/self?${QUERY}" \
  -X GET \
  -H "X-MBX-APIKEY: ${API_KEY}" \
  -H "nonce: ${NONCE}" \
  -H "signature: ${SIG}" | jq .
```

## Notes

- The response is a snapshot computed at `calcTime` / `dt`; it is not real-time. Poll at a modest cadence to track rank changes.
- Unlike Track A, Track B has no hard elimination trigger — `isEliminated` reflects an operational/manual decision, not an automatic threshold.
- For the Track A equivalent, see [Leaderboard API (Track A)](leaderboard.md).
