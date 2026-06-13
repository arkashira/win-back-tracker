# REQUIREMENTS.md  

## 1. Overview  
**win‑back‑tracker** is an automated win‑back campaign platform for retail chains. It ingests POS/CRM data, identifies high‑value lapsed customers, generates personalized offers, and delivers them across multiple channels while providing real‑time performance dashboards and full compliance support.

The purpose of this document is to define **concrete, testable requirements** that guide implementation, testing, and delivery. All requirements are scoped to the current repository (`axentx/win-back-tracker`) and must be achievable without duplicating existing Axentx products.

---

## 2. Functional Requirements  

| ID | Requirement | Acceptance Criteria |
|----|-------------|----------------------|
| **FR‑1** | **Data Ingestion** – The system must import customer transaction data from either (a) CSV files or (b) a RESTful API provided by the retailer’s POS/CRM. | • Supports CSV files up to 5 GB (streamed processing). <br>• API connector authenticates via OAuth 2.0 client‑credentials flow. <br>• Ingestion logs success/failure per file or API batch. |
| **FR‑2** | **Lapsed‑Customer Segmentation** – Identify customers who have not transacted for a configurable “grace period” (default 90 days) and rank them by projected revenue loss. | • Uses a deterministic scoring model (recency × frequency × monetary). <br>• Returns a ranked list (top N configurable) within 5 seconds for ≤ 1 M customers. |
| **FR‑3** | **Personalized Offer Generation** – For each segmented customer, generate a personalized offer (discount %, bundle, loyalty points) based on business rules and historical purchase patterns. | • Offers are stored as JSON objects with fields: `customer_id`, `offer_type`, `value`, `expiry`. <br>• Business‑rule engine is configurable via a YAML file without code changes. |
| **FR‑4** | **Multi‑Channel Delivery** – Dispatch offers via at least three channels: email, SMS, and push notification. | • Integrates with SendGrid (email), Twilio (SMS), and Firebase Cloud Messaging (push). <br>• Guarantees delivery attempt within 2 minutes of offer creation. |
| **FR‑5** | **Compliance Toolkit** – Enforce GDPR/CCPA consent, data‑retention, and audit logging. | • Stores consent flag per customer; offers are not sent if consent is false. <br>• Auto‑purges personal data after a configurable retention period (default 24 months). <br>• Immutable audit log records every data access and outbound message. |
| **FR‑6** | **Real‑Time Dashboard** – Provide a web UI that displays key metrics: revenue recovered, conversion rate, messages sent per channel, and campaign health. | • Dashboard refreshes automatically every 30 seconds. <br>• Supports filtering by store location, date range, and offer type. |
| **FR‑7** | **API & Webhooks** – Expose REST endpoints for (a) uploading data, (b) retrieving segment lists, (c) triggering manual campaigns, and (d) receiving webhook callbacks from external systems. | • All endpoints are versioned (`/api/v1/...`). <br>• Responses conform to OpenAPI 3.0 schema. |
| **FR‑8** | **Plugin System** – Allow third‑party modules to add new channels or custom analytics without modifying core code. | • Plugins are discovered via a `plugins/` directory and loaded at runtime. <br>• Plugin API includes `init()`, `process_offer(offer)`, and `shutdown()`. |
| **FR‑9** | **Scalability** – System must handle **≥ 10 M transactions per month** and **≥ 1 M concurrent customers** with horizontal scaling. | • Stateless services containerized (Docker) and orchestrated via Kubernetes. <br>• Autoscaling triggers when CPU > 70 % or queue length > 10 k. |
| **FR‑10** | **Testing & CI** – Provide unit, integration, and end‑to‑end tests covering ≥ 80 % of codebase, executed on every push. | • GitHub Actions workflow (`ci.yml`) runs lint, test, and security scans. <br>• Test suite must complete within 15 minutes on default runner. |

---

## 3. Non‑Functional Requirements  

| ID | Category | Requirement | Metric / Target |
|----|----------|-------------|-----------------|
| **NFR‑1** | **Performance** | End‑to‑end latency (data ingest → offer dispatch) must be ≤ 30 seconds for a batch of 10 k customers. | ≤ 30 s |
| **NFR‑2** | **Throughput** | System must process at least 5 k offers per second sustained. | ≥ 5 k offers/s |
| **NFR‑3** | **Reliability** | Achieve 99.9 % uptime (excluding planned maintenance). | ≤ 43 min downtime/month |
| **NFR‑4** | **Data Integrity** | No loss of transactional records; checksum verification after each ingest batch. | 0 % loss |
| **NFR‑5** | **Security** | All external communication encrypted (TLS 1.2+). Secrets stored in HashiCorp Vault or K8s Secrets. | Full encryption, secret management |
| **NFR‑6** | **Compliance** | Must pass internal GDPR/CCPA audit checklist; support Data Subject Access Requests (DSAR) within 30 days. | Audit pass, DSAR ≤ 30 d |
| **NFR‑7** | **Observability** | Emit structured logs, metrics (Prometheus), and traces (OpenTelemetry) for all services. | Exported to Grafana/Prometheus |
| **NFR‑8** | **Maintainability** | Code coverage ≥ 80 %; linting with `ruff` (Python) or `eslint` (JS) enforced. | Coverage ≥ 80 % |
| **NFR‑9** | **Portability** | Deployable on any cloud provider (AWS, GCP, Azure) using Helm charts. | Helm‑installable |
| **NFR‑10** | **Extensibility** | Plugin API must be versioned; backward‑compatible for at least two major releases. | Semantic versioning |

---

## 4. Constraints  

1. **Technology Stack** – Must align with the decisions documented in `decisions/tech-stack.md` (e.g., Python 3.11, FastAPI, PostgreSQL, Redis, Docker, Kubernetes).  
2. **Open‑Source Licensing** – All third‑party libraries must be compatible with the project’s MIT license.  
3. **Data Residency** – For EU retailers, all personal data must reside in EU‑based data centers.  
4. **Resource Limits** – Initial production deployment limited to a 4‑vCPU, 16 GB RAM node for cost‑validation; must scale out thereafter.  
5. **No Duplication** – Features overlapping with existing Axentx products (e.g., `iceoryx2` IPC library) are out of scope.

---

## 5. Assumptions  

| ID | Assumption |
|----|------------|
| **A‑1** | Retailers can provide a daily CSV dump or an API endpoint with a stable schema (customer_id, transaction_date, amount, store_id). |
| **A‑2** | Email, SMS, and push providers (SendGrid, Twilio, FCM) are provisioned and credentials are supplied via secret management. |
| **A‑3** | Retailers have a consent flag stored in their source system; win‑back‑tracker will not need to collect consent itself. |
| **A‑4** | The scoring model for segmentation will be a simple RFM algorithm initially; more sophisticated ML models are future enhancements. |
| **A‑5** | The dashboard will be accessed by internal marketing teams behind corporate VPN or SSO; no public internet exposure required. |
| **A‑6** | The system will run on Kubernetes 1.28+; the target environment supports Helm 3. |

---

## 6. Acceptance Criteria Summary  

- All functional requirements FR‑1 – FR‑10 are implemented, demonstrated, and pass automated acceptance tests.  
- Non‑functional targets NFR‑1 – NFR‑10 are measured in a staging environment and meet or exceed the listed metrics.  
- Documentation (README, API spec, deployment Helm chart) is complete and version‑controlled.  
- Security and compliance scans (Snyk, Trivy, GDPR checklist) return no critical findings.  

---  

*Document version: 1.0 – 2026‑06‑13*  
*Prepared by: Senior Product/Engineering Lead, Axentx*
