# CurateAI Architecture

## Overview
CurateAI orchestrates a three-stage pipeline that turns raw venture data into actionable activations:

1. **Intake** — ingest venture data from sources (portals, feeds, partner systems).
2. **Curation** — normalize, enrich, score, and match ventures to internal programs.
3. **Activation** — deliver curated insights to downstream systems, dashboards, and reports.

## System Components

### Intake Layer
* **Source Connectors** — adapters that pull or receive venture data from external systems.
* **Ingestion API** — public endpoints for partner submissions and bulk upload.
* **Raw Data Store** — immutable landing zone for received payloads.
* **Ingestion Queue** — decouples intake from downstream processing.

### Curation Layer
* **Normalization Service** — schema harmonization, data cleaning, entity resolution.
* **Scoring Service** — model-driven scoring and risk/fit assessment.
* **Matching Service** — maps ventures to internal programs, cohorts, or partners.
* **Curation Store** — structured datastore holding curated venture profiles.
* **Feature Store** — derived attributes and historical signals for scoring.

### Activation Layer
* **Activation API** — delivers curated entities to consumer systems.
* **Reporting Service** — aggregates metrics for dashboards and exports.
* **Notification/Export Jobs** — scheduled pushes to partners and internal tools.
* **Delivery Queue** — manages export retries and callback delivery.

## Data Flow (Intake → Curation → Activation)

1. **Intake**
   * Source Connectors or partners POST venture payloads to the Ingestion API.
   * Raw payloads are written to the Raw Data Store and queued for processing.

2. **Curation**
   * Normalization Service standardizes schema and enriches metadata.
   * Scoring Service assigns fit, impact, and risk scores.
   * Matching Service links ventures to internal programs or partner criteria.
   * Curated profiles are persisted in the Curation Store.

3. **Activation**
   * Activation API exposes curated ventures to downstream tools.
   * Reporting Service generates dashboards and scheduled exports.
   * Notification/Export Jobs push updates to integrations.

## Core Use Cases

### Venture Ingestion
* Accept partner submissions via the Ingestion API (single or bulk).
* Validate payloads, record provenance, and enqueue for processing.
* Persist raw payloads for auditability and replay.

### Venture Scoring
* Enrich ventures with third-party or internal datasets.
* Compute fit, impact, risk, and momentum scores.
* Store score history for longitudinal analysis.

### Venture Matching
* Apply program rules (sector, stage, geography, capital needs).
* Return ranked matches with rationale and scoring context.

### Reporting
* Aggregate funnel metrics (intake volume, acceptance rate, activation rate).
* Provide cohort-level and program-level KPIs.

## Implementation Structures

### Canonical Entities
* **Venture** — normalized profile of a company or initiative.
* **Score** — computed metrics with model version and feature references.
* **Match** — relationship between a venture and a program/partner.
* **Activation** — export or notification event to downstream systems.

### Event Contracts (Suggested Topics)
* `venture.received`
* `venture.normalized`
* `venture.scored`
* `venture.matched`
* `activation.requested`
* `activation.completed`

### Data Stores (Suggested Responsibilities)
* **Raw Data Store** — append-only storage of original payloads.
* **Curation Store** — current-state normalized ventures and scores.
* **Analytics Warehouse** — reporting aggregates and historical slices.

## Key Service Endpoints (Illustrative)

* **Ingestion API**: `https://api.curateai.example/v1/intake`
  * `POST /ventures` — submit a new venture record.
  * `POST /ventures/bulk` — batch ingestion.

* **Curation API**: `https://api.curateai.example/v1/curation`
  * `GET /ventures/{ventureId}` — retrieve curated venture profile.
  * `GET /ventures/{ventureId}/scores` — view scoring breakdown.
  * `GET /ventures/{ventureId}/matches` — list program matches.

* **Activation API**: `https://api.curateai.example/v1/activation`
  * `POST /exports` — trigger export to downstream partner.
  * `GET /reports/summary` — access reporting summaries.

## Non-Functional Considerations

* **Security** — OAuth2 bearer tokens and signed webhooks for partner callbacks.
* **Observability** — centralized logging, distributed tracing for each pipeline stage.
* **Resilience** — retry queues and dead-letter handling on ingest failures.
