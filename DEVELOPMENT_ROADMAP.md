# DEVELOPMENT ROADMAP

> Roadmap derived from the documented project scope and a previously discussed 16-week plan. The actual repository status is unknown, so Phase 1 is mandatory. Week numbers are planning estimates, not commitments or evidence of progress.

## Guiding strategy

Build a small, testable end-to-end vertical slice before adding sophisticated AI, dynamic knowledge graphs or infrastructure orchestration. Keep the local academic deployment achievable on a Windows host with 16 GB RAM using one Ubuntu VM and Docker containers.

## Phase 1 — Existing implementation assessment

**Priority:** P0 / immediate.

**Objectives:** Establish the real project state and avoid duplicate or destructive work.

**Tasks:**
- Locate repository, confirm branch/remotes and inspect Git status/history.
- Inventory source files, package manifests, Docker files, environment templates, SQL migrations, tests and documentation.
- Run current tests/build/startup commands without modifying files.
- Record OS, Python/Node/Docker/Compose/PostgreSQL versions and VM resource allocation.
- Map each actual file to features in `IMPLEMENTATION_STATUS.md`.

**Dependencies:** Access to Ubuntu development environment and repository.

**Deliverables:** Repository inventory, baseline test report, updated implementation-status table and list of blockers.

**Acceptance criteria:** Every claimed completed feature points to source files and a successful test/manual verification; unverified features remain UNKNOWN.

## Phase 2 — Fix incomplete and broken components

**Priority:** P0.

**Objectives:** Stabilize existing code before expanding scope.

**Tasks:**
- Reproduce errors from logs/tests.
- Fix configuration, dependency and startup problems incrementally.
- Preserve current working behavior; add regression tests.
- Remove secrets from tracked files if any are found; rotate exposed credentials rather than merely deleting them.
- Align package versions and document environment setup.

**Dependencies:** Phase 1 inventory.

**Deliverables:** Clean baseline, reproducible setup and passing existing tests.

**Acceptance criteria:** Clean checkout can install/build/start according to README; test failures are documented and not hidden.

## Phase 3 — Edge and Fog telemetry vertical slice

**Priority:** P0.

**Objectives:** Deliver reliable telemetry from one Edge node to Fog.

**Tasks:**
- Implement/verify psutil collector.
- Define versioned JSON schema with node ID, UTC timestamp, message ID, sequence number and units.
- Configure Mosquitto and private MQTT topics.
- Implement Fog subscriber, validation, duplicate handling and structured logs.
- Add SQLite buffering and reconnect replay.
- Add three simulated Edge agents after one-node flow works.

**Dependencies:** Phase 1–2; Ubuntu/Docker environment.

**Deliverables:** Working Edge→MQTT→Fog pipeline, schema documentation and integration tests.

**Acceptance criteria:** Valid message is ingested; invalid message is rejected; duplicate message is idempotent; disconnect/reconnect preserves buffered telemetry.

## Phase 4 — Database integration and backend APIs

**Priority:** P0.

**Objectives:** Persist telemetry and expose stable contracts.

**Tasks:**
- Confirm PostgreSQL as baseline.
- Define migrations, PK/FK constraints, indexes and retention policy.
- Implement node and telemetry persistence.
- Add FastAPI endpoints for nodes and telemetry.
- Add request validation, error handling, pagination/time filters and health endpoint.
- Add JWT/RBAC before exposing sensitive operations.

**Dependencies:** Phase 3 schema and message flow.

**Deliverables:** Database migrations, API schemas, OpenAPI documentation and tests.

**Acceptance criteria:** Integration tests verify data round-trip, invalid input handling, authorization and database health.

## Phase 5 — Frontend dashboard

**Priority:** P1.

**Objectives:** Make the operational state visible.

**Tasks:**
- Inspect existing React app before choosing structure.
- Build node list/status, latest metrics, history charts, anomalies, predictions, incidents and remediation-history views.
- Integrate with FastAPI using documented contracts.
- Add loading, empty, error and stale-data states.
- Use Recharts if present/approved.

**Dependencies:** Phase 4 API contracts.

**Deliverables:** Working dashboard with API integration.

**Acceptance criteria:** Dashboard accurately displays seeded and live persisted data; handles API failure without crashing.

## Phase 6 — AI/ML baseline and prediction

**Priority:** P1.

**Objectives:** Introduce measurable intelligence without compromising data correctness.

**Tasks:**
- Define dataset sources, licensing, labels and reproducible synthetic fault scenarios.
- Define feature schema and training/serving preprocessing.
- Train Isolation Forest baseline.
- Add Autoencoder only after baseline and data validation.
- Define failure label, prediction horizon and sequence window before Bi-LSTM.
- Split by time/node to prevent leakage; calibrate thresholds on validation data.
- Version model artifacts, preprocessing and thresholds.
- Export to ONNX only if compatible and beneficial; benchmark Fog inference.

**Dependencies:** Reliable telemetry and data storage from Phases 3–4.

**Deliverables:** Training scripts, dataset documentation, model artifacts, evaluation report and inference API/service.

**Acceptance criteria:** Reproducible training/evaluation; metrics include precision, recall, F1 and false-positive rate; no future-data leakage; inference latency and resource use recorded.

## Phase 7 — Context, security analysis and RCA

**Priority:** P1.

**Objectives:** Correlate operational and security evidence into incidents.

**Tasks:**
- Define event schema and supported security telemetry sources.
- Implement controlled synthetic security-event scenarios.
- Build service dependency graph using NetworkX if appropriate.
- Correlate events by time, component, severity and dependency.
- Rank probable causes and return evidence/explanation.
- Avoid claiming comprehensive malware detection or causal certainty.

**Dependencies:** Event persistence, model outputs and service topology data.

**Deliverables:** Security-event analyzer, dependency graph, RCA service and test cases.

**Acceptance criteria:** Test incidents produce explainable ranked candidates; missing context is represented explicitly; event correlation is reproducible.

## Phase 8 — Controlled remediation

**Priority:** P1; safety-critical.

**Objectives:** Demonstrate safe response to a narrow set of test incidents.

**Tasks:**
- Define action allow-list and parameter schemas.
- Define which actions can be automatic and which require human approval.
- Implement policy checks for severity, confidence, authorization, cooldown and system state.
- Implement executor against disposable test services only.
- Add audit log, timeout, health verification and escalation.
- Add rollback only for actions with a safe tested compensating action.
- Add kill switch and tests for rejected/failed actions.

**Dependencies:** Incident model, authentication/RBAC, policy definitions and test environment.

**Deliverables:** Policy engine, approval flow, executor, verifier and audit history.

**Acceptance criteria:** No arbitrary command execution; unauthorized actions are denied; action results are verified; failure is escalated and audited.

## Phase 9 — Cloud/infrastructure integration and observability

**Priority:** P2.

**Objectives:** Improve deployment reproducibility, service health and performance evidence.

**Tasks:**
- Finalize Compose networks, health checks, volumes and restart policies.
- Restrict exposed ports; configure secrets and MQTT authentication/TLS as appropriate.
- Add structured logs, correlation IDs and health endpoints.
- Record latency, resource consumption, traffic and action outcomes.
- Decide whether Prometheus/Grafana/Loki/Alertmanager are justified; they are not currently selected.
- Do not introduce Kubernetes/Terraform unless academic scope and hardware support justify them.

**Dependencies:** Stable integrated services.

**Deliverables:** Secure local deployment profile, operational runbook and performance capture.

**Acceptance criteria:** Clean deployment is repeatable; health checks detect failures; sensitive ports/secrets are protected; measurements can be reproduced.

## Phase 10 — Testing, optimization, final report and demonstration

**Priority:** P0 before submission.

**Tasks:**
- Run CPU, RAM, disk, service failure, network outage, security-event, dependency failure and remediation-failure scenarios.
- Compare Fog-based versus Cloud-only processing using same workload.
- Measure detection latency, traffic, CPU/RAM, model quality and recovery time.
- Repeat trials and report mean/median and variability where useful.
- Document limitations, false positives, failure cases and security boundaries.
- Prepare architecture diagrams, ERD, sequence diagrams, use cases, demo script and viva explanation.
- Verify paper metadata/DOIs from original sources.

**Dependencies:** All implemented modules.

**Deliverables:** Test report, performance comparison, final source, setup guide, report, presentation and reproducible demo.

**Acceptance criteria:** Every reported result traces to a test procedure and raw measurements; no unimplemented feature is presented as complete.

## 11. Suggested 16-week mapping (planning baseline)

| Week | Focus | Milestone |
|---|---|---|
| 1 | Requirements and repository/environment audit | Scope and baseline inventory |
| 2 | Architecture, schemas and literature validation | Design freeze for first vertical slice |
| 3 | MQTT and database skeleton | Broker and DB health checks |
| 4 | Edge collector | One agent emits telemetry |
| 5 | MQTT ingestion and SQLite buffer | Disconnect/replay test |
| 6 | Fog processing | Validated normalized telemetry |
| 7 | Isolation Forest baseline | First anomaly results |
| 8 | Autoencoder / data quality | Comparative anomaly evaluation |
| 9 | Bi-LSTM dataset/labels and training | Prediction experiment, if data supports it |
| 10 | Fog inference integration | Model inference in pipeline |
| 11 | Security events and RCA | Incident correlation demo |
| 12 | Policy and remediation | Controlled test-service recovery |
| 13 | Cloud API/database completion | Stable API contracts |
| 14 | React dashboard | Integrated UI |
| 15 | End-to-end tests and performance comparison | Measured evaluation |
| 16 | Optimization, report and demo | Submission-ready package |

This timeline is a prior planning proposal. If the project is already underway, re-sequence based on the Phase 1 audit rather than assuming Week 1 work remains pending.

## 12. Team coordination

Suggested ownership (confirm assignment to named students):
- Member A: Edge/Fog infrastructure and messaging.
- Member B: AI/ML data, training, evaluation and inference.
- Member C: security/RCA, policy/remediation, Cloud API/database and dashboard.

Ramesh and Yogesh are named teammates; exact mapping to the three roles is UNKNOWN. Agree on message schemas, API contracts and database models before parallel implementation. Use feature branches and integration reviews; avoid separate teams inventing incompatible payloads.
