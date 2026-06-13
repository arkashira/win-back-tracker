# STORIES.md – win‑back‑tracker

## Overview
This backlog captures the product backlog for **win‑back‑tracker**, an automated win‑back campaign platform for retail chains.  
Stories are grouped into **Epics** that map to the core feature set described in the README.  
Stories are ordered to deliver a Minimum Viable Product (MVP) first, then incremental enhancements.

---

## Epics

| Epic | Description | MVP Priority |
|------|-------------|--------------|
| **E1 – Data Ingestion & Segmentation** | Connect to retailer data sources, ingest transaction history, and automatically segment lapsed high‑value customers. | ✅ |
| **E2 – Campaign Engine** | Create, schedule, and personalize win‑back offers based on segmentation output. | ✅ |
| **E3 – Multi‑Channel Delivery** | Dispatch offers through email, SMS, push notifications, and in‑store kiosks. | ✅ |
| **E4 – Compliance & Consent** | Enforce GDPR/CCPA/PCI‑DSS rules, manage consent, and retain audit logs. | ✅ |
| **E5 – Real‑Time Dashboard** | Visualize campaign health, conversion, and revenue recovered. | ✅ |
| **E6 – API & Webhooks** | Expose RESTful endpoints and webhook hooks for external system integration. | ⬆️ |
| **E7 – Plugin System** | Provide a plug‑in architecture for custom channels, analytics, or business rules. | ⬆️ |
| **E8 – Production‑Ready Operations** | CI/CD, monitoring, scaling, and disaster‑recovery tooling. | ⬆️ |

> **✅** = Included in MVP release  
> **⬆️** = Planned for post‑MVP releases

---

## User Stories

### Epic E1 – Data Ingestion & Segmentation

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E1‑01** | **As a Data Engineer, I want to upload a CSV export of POS transactions, so that the system can build a customer purchase history.** | - UI provides a drag‑and‑drop CSV uploader.<br>- System validates schema (customer_id, transaction_date, amount, store_id).<br>- Successful upload shows “X records ingested”. |
| **E1‑02** | **As a Data Engineer, I want to configure an API connector to our ERP, so that new transactions are streamed in near‑real‑time.** | - Connector can be enabled via a config file (endpoint, auth token).<br>- Polling interval is configurable (default 5 min).<br>- New records appear in the “Recent Ingest” view within 2 min of arrival. |
| **E1‑03** | **As a Marketing Analyst, I want the system to automatically segment lapsed customers with a predicted revenue‑recovery score, so that I can target the most valuable prospects.** | - Segmentation runs nightly on the latest data.<br>- Output includes `customer_id`, `score` (0‑100), `last_purchase_date`.<br>- Top‑N segment can be exported as CSV or viewed in UI.<br>- Model version is logged for audit. |
| **E1‑04** | **As a Product Owner, I want to define custom segmentation rules (e.g., “no purchase > 180 days & spend > $500”), so that we can tailor campaigns per business need.** | - UI rule builder supports date, numeric, and categorical filters.<br>- Rules are saved with a name and description.<br>- Applying a rule instantly updates the “Current Segment” list. |

### Epic E2 – Campaign Engine

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E2‑01** | **As a Campaign Manager, I want to create a win‑back offer template with dynamic placeholders, so that each message feels personalized.** | - Template editor supports variables: `{{first_name}}`, `{{discount}}`, `{{expiry_date}}`.<br>- Preview renders with sample data.<br>- Templates can be saved, versioned, and set as “active”. |
| **E2‑02** | **As a Campaign Manager, I want to schedule a campaign to run for a specific date range, so that offers are sent at the optimal time.** | - Calendar UI to pick start/end dates and time‑zone.<br>- System prevents overlapping campaigns for the same segment unless explicitly allowed.<br>- Scheduled campaigns appear in a “Upcoming” list. |
| **E2‑03** | **As a Data Scientist, I want the engine to assign a discount amount based on the predicted recovery score, so that higher‑value prospects receive larger incentives.** | - Business rule: `discount = base + (score/100)*max_extra`.<br>- Discount is injected into the template at send time.<br>- Rule can be edited without redeploying code. |
| **E2‑04** | **As a QA Engineer, I want a “dry‑run” mode that simulates sending messages without actually delivering them, so that we can validate logic safely.** | - Toggle “Dry Run” on campaign creation.<br>- Dry‑run generates a log file with rendered messages and target channels.<br>- No external API calls are made in dry‑run mode. |

### Epic E3 – Multi‑Channel Delivery

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E3‑01** | **As a Marketing Ops user, I want to configure an SMTP provider, so that email win‑back messages can be sent.** | - UI fields: host, port, TLS, username, password.<br>- Test button sends a test email to a configurable address.<br>- Successful test stores credentials encrypted. |
| **E3‑02** | **As a Marketing Ops user, I want to integrate with Twilio (or similar) for SMS delivery, so that we can reach customers on mobile.** | - API key entry and sender number configuration.<br>- Test SMS button verifies delivery.<br>- Failure reasons are logged with error codes. |
| **E3‑03** | **As a Campaign Manager, I want to enable push notifications via Firebase Cloud Messaging, so that app users receive win‑back offers.** | - Upload of Firebase service account JSON.<br>- UI to map `customer_id` → `device_token`.<br>- Push test sends a notification to a test device. |
| **E3‑04** | **As a Store Manager, I want to push offers to in‑store kiosks via a simple webhook, so that customers can redeem offers on‑site.** | - Kiosk webhook URL configurable per store.<br>- Payload format documented (JSON with `offer_code`, `expiry`).<br>- Delivery status (200/4xx) recorded in audit log. |

### Epic E4 – Compliance & Consent

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E4‑01** | **As a Compliance Officer, I need to record consent status for each customer, so that we only contact opted‑in users.** | - Consent flag (`opt_in`) stored per `customer_id`.<br>- UI to bulk import consent CSV and to manually toggle consent.<br>- Campaign engine automatically filters out non‑consented customers. |
| **E4‑02** | **As a Data Engineer, I want automatic data‑retention pruning (e.g., delete records > 24 months), so that we stay within GDPR limits.** | - Configurable retention period (default 24 months).<br>- Scheduler runs nightly and logs number of rows deleted.<br>- Deleted rows are not recoverable; audit log records deletion timestamps. |
| **E4‑03** | **As an Auditor, I need an immutable audit log of all campaign actions (create, edit, send, delete), so that we can prove compliance.** | - Log entries include user, timestamp, action, affected entity IDs.<br>- Logs are write‑once (append‑only) and tamper‑evident (hash chain).<br>- UI view with filter by date/user/action. |

### Epic E5 – Real‑Time Dashboard

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E5‑01** | **As a Marketing Executive, I want a dashboard showing total revenue recovered per campaign, so that I can measure ROI.** | - KPI cards: `Revenue Recovered`, `Customers Reactivated`, `Cost per Reactivation`.<br>- Data updates within 5 min of campaign activity.<br>- Export to CSV button. |
| **E5‑02** | **As a Campaign Manager, I want to see conversion funnel charts (sent → opened → clicked → redeemed), so that I can optimise messaging.** | - Funnel visual with percentages and absolute numbers.<br>- Ability to filter by channel and date range.<br>- Hover tooltip shows raw counts. |
| **E5‑03** | **As a Store Manager, I want a per‑store performance view, so that I can compare locations.** | - Table/list of stores with metrics: `Revenue`, `Redemptions`, `Avg Discount`.<br>- Clickable store row drills down to store‑level details. |
| **E5‑04** | **As a DevOps Engineer, I want health‑status widgets (queue lag, API error rate), so that I can monitor system stability.** | - Real‑time gauges for ingestion lag, send‑rate, error‑rate.<br>- Alerts trigger when thresholds exceeded (configurable). |

### Epic E6 – API & Webhooks

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E6‑01** | **As an Integrations Engineer, I want a REST endpoint to fetch the current segment list, so that downstream systems can use it.** | - `GET /api/v1/segments/{segment_id}` returns JSON array of `customer_id` and `score`.<br>- Supports pagination and `If-Modified-Since` caching.<br>- Auth via API key. |
| **E6‑02** | **As a Partner, I want webhook callbacks on “offer redeemed” events, so that we can update loyalty balances in real time.** | - Configurable webhook URL per partner.<br>- Payload includes `customer_id`, `offer_id`, `redeemed_at`, `value`.<br>- Retries with exponential back‑off up to 5 attempts; failures logged. |
| **E6‑03** | **As a Security Engineer, I need rate‑limiting on all public APIs, so that we protect against abuse.** | - 100 requests/second per API key limit.<br>- HTTP 429 response with `Retry-After` header.<br>- Metrics exported to Prometheus. |

### Epic E7 – Plugin System

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E7‑01** | **As a Developer, I want a plugin SDK that lets me add a new delivery channel (e.g., WhatsApp), so that we can expand reach.** | - SDK provides `ChannelPlugin` interface with `send(customer, payload)` method.<br>- Plugins discovered via `plugins/` directory and loaded at startup.<br>- Sample plugin included with documentation. |
| **E7‑02** | **As a Data Scientist, I want to plug in a custom scoring model (e.g., XGBoost), so that we can improve segmentation accuracy.** | - Model packaged as a Python module adhering to `Scorer` interface (`fit`, `predict`).<br>- Configurable via `scoring_model` setting.<br>- System validates model signature before activation. |
| **E7‑03** | **As a QA Engineer, I want the ability to disable/enable plugins at runtime, so that we can test core functionality without external dependencies.** | - Admin UI toggle per plugin.<br>- Disabled plugins are skipped in the processing pipeline and logged. |

### Epic E8 – Production‑Ready Operations

| # | Story | Acceptance Criteria |
|---|-------|----------------------|
| **E8‑01** | **As a DevOps Engineer, I need CI pipelines that run unit, integration, and security tests on every PR, so that code quality is enforced.** | - GitHub Actions workflow triggers on PR and push to `main`.<br>- Stages: lint → unit → integration → OWASP Dependency‑Check.<br>- Build badge reflects pass/fail status. |
| **E8‑02** | **As a Site Reliability Engineer, I want automated scaling policies for the ingestion and sending workers, so that the system handles peak loads.** | - Kubernetes HPA rules based on CPU and queue depth.<br>- Load test (simulated 5 M transactions/day) shows no >2 s latency.<br>- Scaling events logged to observability platform. |
| **E8‑03** | **As a Backup Engineer, I need daily encrypted backups of the customer and campaign databases, stored in a separate region, so that we can recover from disasters.** | - Backup job runs at 02:00 UTC, creates encrypted snapshot.<br>- Retention: 30 days.<br>- Restore test documented and verified quarterly. |
| **E8‑04** | **As a Support Engineer, I want a searchable log aggregation UI (e.g., Loki/Grafana), so that I can troubleshoot issues quickly.** | - All service logs shipped to central Loki cluster.<br>- Pre‑built Grafana dashboards for “Ingestion Errors”, “Send Failures”, and “API Latency”. |

---

## Prioritisation for MVP (Release 1.0)

| Priority | Epic | Stories Included |
|----------|------|-------------------|
| **P1 – Core Flow** | E1, E2, E3, E4, E5 | E1‑01, E1‑03, E2‑01, E2‑02, E3‑01, E3‑02, E4‑01, E4‑03, E5‑01, E5‑02 |
| **P2 – Reliability & Ops** | E6, E8 | E6‑01, E8‑01, E8‑02 |
| **P3 – Extensibility** | E7 | E7‑01 (basic plugin skeleton) |
| **P4 – Advanced Features** | Remaining stories in all epics (custom rules, multi‑store view, webhook callbacks, retention pruning, etc.) |

The MVP will deliver a **fully automated win‑back pipeline** from data ingestion to real‑time performance insights while satisfying compliance requirements and operational robustness. Subsequent releases will expand integration options, customisation, and scalability features.
