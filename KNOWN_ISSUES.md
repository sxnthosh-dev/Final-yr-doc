# KNOWN ISSUES AND DEBUGGING HISTORY

> **Evidence limitation:** No historical terminal logs, stack traces, screenshots, source-code diffs or successful troubleshooting transcripts were present in the accessible project material. This document therefore does not invent error messages or claim fixes. It records known uncertainties, design risks and the debugging procedure for the next development session.

## 1. Recovered historical errors

**No specific error message or failed command was recoverable from the accessible sources.**

| Error message | Component | Established root cause | Troubleshooting/fix evidence | Current status |
|---|---|---|---|---|
| None supplied in accessible source material | — | — | No terminal transcript available | UNKNOWN – Requires user confirmation |

Do not fabricate an error history. If the user provides logs or the repository, append exact errors and distinguish attempted fixes from successful fixes.

## 2. Known technical risks and unresolved issues

### 2.1 Implementation state is unverified
- **Affected:** All components.
- **Evidence:** Only project brief/design discussions are available; no source repository was supplied.
- **Root cause:** Repository and complete raw conversation history are outside this transfer evidence.
- **Next step:** Locate repository, inspect files and Git history, run tests and update `IMPLEMENTATION_STATUS.md`.

### 2.2 MQTT duplicate delivery
- **Affected:** Edge–Fog ingestion and database persistence.
- **Risk:** QoS 1 may deliver a message more than once.
- **Established fix:** None implemented/evidenced.
- **Recommended approach:** Include stable `message_id`; enforce idempotent processing or unique constraint; test duplicate delivery.

### 2.3 Edge–Fog connectivity loss
- **Affected:** Edge agent and telemetry transport.
- **Risk:** Telemetry loss during disconnection.
- **Design response:** SQLite local buffer was proposed.
- **Implementation status:** UNKNOWN.
- **Next step:** Test offline persistence, bounded retention, replay order, duplicate handling and behavior when buffer is full.

### 2.4 Schema drift and train/serve skew
- **Affected:** MQTT payload, preprocessing and AI inference.
- **Risk:** Training and live inference may use different feature order, units, normalization or missing-value behavior.
- **Fix evidence:** None.
- **Next step:** Version the telemetry/feature schema and package preprocessing parameters with each model version; add contract tests.

### 2.5 Bi-LSTM future-data leakage
- **Affected:** Failure-prediction model.
- **Risk:** A bidirectional sequence model may accidentally consume observations beyond the prediction timestamp if windows/labels are defined incorrectly.
- **Fix evidence:** No model implementation available.
- **Next step:** Define lookback window, prediction horizon and point-in-time label; split chronologically/by node and assert no future features leak into input.

### 2.6 Model performance targets not validated
- **Affected:** AI/ML and review claims.
- **Risk:** >=90% performance and sub-second detection are stated as targets, not measured results.
- **Next step:** Establish baselines and report actual precision/recall/F1/FPR, latency and hardware configuration.

### 2.7 RCA may overstate causality
- **Affected:** Context/RCA module.
- **Risk:** Correlation and dependency evidence cannot alone prove causation.
- **Design safeguard:** Report ranked probable root causes with supporting evidence and uncertainty.
- **Next step:** Build labeled fault-injection scenarios and evaluate top-k RCA accuracy where ground truth is known.

### 2.8 Security scope may be overstated
- **Affected:** Security analyzer and project claims.
- **Risk:** Generic infrastructure metrics do not establish malware detection capability.
- **Next step:** Define exact telemetry sources, threat/event classes, datasets and evaluation. Describe the module as security-event/suspicious-behavior analysis unless validated malware detection exists.

### 2.9 Autonomous remediation safety
- **Affected:** Policy engine and executor.
- **Risk:** Incorrect or unauthorized actions can cause outage or data loss.
- **Required safeguards:** Allow-list, RBAC, approval, confidence/severity threshold, cooldown, audit, health check, escalation, kill switch; rollback only if safe and tested.
- **Next step:** Execute only against disposable test services; add denial and failure-path tests before any real host integration.

### 2.10 Local resource constraints
- **Affected:** VM and container deployment.
- **Known constraint:** Windows host with 16 GB RAM.
- **Prior recommendation:** Ubuntu 24.04 guest with approximately 8 GB RAM, 4 vCPUs and 60 GB dynamic disk; multiple Edge nodes should initially be containers, not full VMs.
- **Risk:** Running multiple deep-learning training jobs and all containers may exhaust memory/CPU.
- **Next step:** Monitor host/guest resource use; train models separately/offline; limit container resources and avoid allocating all host RAM to VM.

### 2.11 Technology choices not fully locked
- **Affected:** AI training and infrastructure.
- **Known ambiguity:** Project brief mentions PyTorch/TensorFlow; later setup guidance prefers PyTorch. TimescaleDB is optional. Kubernetes/Terraform/Prometheus/Grafana/Loki were not selected.
- **Next step:** Confirm actual package manifests and make choices based on existing code and scope; do not replace working dependencies casually.

### 2.12 Academic references need verification
- **Affected:** Literature review and report.
- **Risk:** Some titles/author/year metadata in the brief may be incomplete or need verification.
- **Next step:** Verify each citation against original PDF or publisher metadata; do not invent DOI, issue, volume or page numbers.

## 3. Debugging record template

Append one entry per issue:

```text
Issue ID:
Date/time and environment:
Component/version:
Exact command or user action:
Exact error output:
Expected behavior:
Actual behavior:
Reproduction steps:
Root cause: (confirmed / suspected / unknown)
Troubleshooting attempted:
Attempt result: (failed / partial / successful)
Successful fix and commit:
Regression test:
Remaining risk:
Next action:
```

## 4. Safe debugging procedure

1. Reproduce before editing.
2. Capture full logs and versions; redact secrets.
3. Inspect related code/configuration and recent Git changes.
4. State whether root cause is confirmed or only a hypothesis.
5. Make the smallest change that tests the hypothesis.
6. Run the narrow test, then relevant integration tests.
7. Record exact result and commit reference.
8. Do not report a fix as successful until the failing reproduction passes.
9. Do not delete volumes, databases, branches or user files without explicit approval and backup.

## 5. Useful diagnostic commands (suggested, not recovered historical commands)

```bash
# Repository and environment
git status --short
git log --oneline -10
python3 --version
docker --version
docker compose version

# Compose state and logs (only after inspecting the compose file)
docker compose ps
docker compose logs --tail=200

# Container resource usage
docker stats --no-stream

# MQTT test (requires broker and mosquitto-clients installed)
mosquitto_sub -h localhost -t 'infrastructure/edge/+/metrics' -v
mosquitto_pub -h localhost -t 'infrastructure/edge/edge-01/metrics' -m '{"node_id":"edge-01","status":"test"}'
```

Do not publish test payloads to a production broker. The example payload is only a connectivity probe and does not conform to a finalized schema.
