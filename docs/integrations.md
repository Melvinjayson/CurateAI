# CurateAI Integrations

This document describes the public integration contracts for external systems and the expected endpoints.
All URLs below are illustrative and should be updated for the target environment.

## Common Conventions

* **Base API URL**: `https://api.curateai.example/v1`
* **Auth**: OAuth2 client credentials, bearer token in `Authorization` header.
* **Content-Type**: `application/json`
* **Idempotency**: Use `Idempotency-Key` header for POST requests.
* **Correlation**: Use `X-Request-Id` for end-to-end tracing.

### Standard Response Shape
```json
{
  "data": { },
  "meta": {
    "requestId": "uuid",
    "status": "success"
  },
  "errors": [ ]
}
```

### Standard Error Codes
| HTTP Status | Code | Meaning |
| --- | --- | --- |
| 400 | `invalid_request` | Required field missing or invalid. |
| 401 | `unauthorized` | Missing or invalid credentials. |
| 409 | `conflict` | Duplicate submission or idempotency conflict. |
| 422 | `unprocessable_entity` | Semantic validation failed. |
| 500 | `internal_error` | Unexpected server error. |

## Venus Integration (Ingestion Partner)

Venus supplies venture intake data into CurateAI.

### Expected Public Endpoints

* **Submit venture**: `POST https://api.curateai.example/v1/intake/ventures`
  * Payload: venture profile with contact, sector, geography, and traction metrics.
  * Response: `{ "ventureId": "...", "status": "received" }`

* **Submit bulk ventures**: `POST https://api.curateai.example/v1/intake/ventures/bulk`
  * Payload: array of venture payloads.
  * Response: `{ "batchId": "...", "status": "queued" }`

### Minimal Venture Payload (Example)
```json
{
  "externalId": "venus-123",
  "name": "Example Labs",
  "sector": "climate",
  "stage": "seed",
  "geography": "EU",
  "traction": {
    "revenueUsd": 250000,
    "users": 1200
  },
  "contacts": [
    { "name": "Alex Founder", "email": "alex@example.com" }
  ]
}
```

### Callback/Webhook Expectations

* **Ingestion status callback**: `POST https://venus.example/webhooks/curateai/ingestion-status`
  * CurateAI notifies Venus of processing outcomes per venture ID.

## Odysseia Integration (Activation Partner)

Odysseia consumes curated ventures and match results for downstream activation workflows.

### Expected Public Endpoints

* **Fetch curated venture**: `GET https://api.curateai.example/v1/curation/ventures/{ventureId}`
* **Fetch matches**: `GET https://api.curateai.example/v1/curation/ventures/{ventureId}/matches`

### Scheduled Export Contract

* **Export curated batch**: `POST https://api.curateai.example/v1/activation/exports`
  * Payload: `{ "partner": "odysseia", "format": "json", "filters": { ... } }`
  * Response: `{ "exportId": "...", "status": "started" }`

### Match Payload (Example)
```json
{
  "ventureId": "cur-789",
  "programId": "prog-456",
  "score": 0.82,
  "rationale": [
    "Sector alignment: climate",
    "Geography: EU"
  ]
}
```

### Callback/Webhook Expectations

* **Export completion**: `POST https://odysseia.example/webhooks/curateai/export-status`
  * CurateAI notifies Odysseia when an export is ready.

## Aura Integration (Reporting Partner)

Aura consumes reporting metrics and summary analytics.

### Expected Public Endpoints

* **Reporting summary**: `GET https://api.curateai.example/v1/activation/reports/summary`
* **Program KPIs**: `GET https://api.curateai.example/v1/activation/reports/programs/{programId}`

### Scheduled Reports

* **Generate report**: `POST https://api.curateai.example/v1/activation/reports`
  * Payload: `{ "reportType": "weekly", "audience": "aura" }`
  * Response: `{ "reportId": "...", "status": "queued" }`

### Reporting Payload (Example)
```json
{
  "reportId": "rep-123",
  "period": "2025-01-01/2025-01-07",
  "metrics": {
    "intakeCount": 120,
    "scoredCount": 110,
    "matchedCount": 85,
    "activatedCount": 45
  }
}
```

### Callback/Webhook Expectations

* **Report ready**: `POST https://aura.example/webhooks/curateai/report-ready`
  * CurateAI notifies Aura when a report is available.
