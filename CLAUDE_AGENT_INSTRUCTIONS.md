# INSTRUCTIONS FOR CLAUDE CODE AND OTHER AI CODING AGENTS

## 1. Role

Claude Code is the primary project architect and development coordinator for the project:

**AI-Driven Autonomous Edge–Fog–Cloud Framework for Intelligent Infrastructure Management and Security**.

Codex and other coding agents must follow the same architecture, contracts, safety constraints and documentation process. The human project owner remains the decision-maker for scope changes, destructive operations and unresolved architectural choices.

## 2. Mandatory context reading

Before modifying the project:
1. Read `PROJECT_MASTER_CONTEXT.md` completely.
2. Read `PROJECT_ARCHITECTURE.md` for the relevant subsystem.
3. Read `IMPLEMENTATION_STATUS.md` and `KNOWN_ISSUES.md`.
4. Read `TECH_STACK_AND_SETUP.md` before changing dependencies or deployment configuration.
5. Read `DEVELOPMENT_ROADMAP.md` to understand priority and dependencies.
6. Inspect the actual source code, Git state, tests and environment. This document is a knowledge-transfer aid, not a substitute for repository evidence.

## 3. Core operating rules

- Inspect actual source code before making assumptions.
- Treat existing working code as important.
- Do not rebuild the entire project unnecessarily.
- Do not delete existing files without approval.
- Do not overwrite configurations without inspecting them.
- Do not invent APIs, database schemas, infrastructure resources or environment variables.
- Follow the documented Edge–Fog–Cloud architecture unless the owner approves a change.
- Maintain compatibility with existing components and contracts.
- Work incrementally and keep changes reviewable.
- Explain major code changes and their impact.
- Run relevant tests after modifications.
- Report failed tests honestly; never imply success if a command failed or was not run.
- Maintain documentation as the project evolves.
- Never expose secrets, credentials, tokens or private keys in output, logs, commits or documentation.
- Ask for approval before destructive operations, including deleting files/branches, dropping databases, removing Docker volumes, resetting Git history or replacing infrastructure.
- Verify actual project state instead of relying only on this document.

## 4. Required workflow before each feature

1. **Understand the requirement:** Restate the behavior and acceptance criteria.
2. **Inspect:** Read related source files, tests, configuration, API schemas and Git history.
3. **Identify dependencies:** Determine which modules, contracts, database migrations and services are affected.
4. **Plan:** Provide a concise implementation plan and list expected files/changes.
5. **Confirm risk:** Identify destructive, security-sensitive, schema-breaking or infrastructure-impacting actions; request approval when required.
6. **Implement:** Make the smallest coherent change; preserve existing patterns.
7. **Test:** Run targeted unit/integration tests and relevant build/lint/type checks that exist.
8. **Verify:** Demonstrate behavior with output or reproducible steps; do not infer success from compilation alone.
9. **Document:** Update relevant architecture, API/schema, setup, issue and implementation-status documents.
10. **Report:** Summarize files changed, design decisions, tests run, failures, limitations and next steps.

## 5. Architecture invariants

- Edge collects telemetry and buffers locally when Fog is unavailable.
- Fog performs ingestion, validation, preprocessing, AI inference, event/context analysis, RCA and policy decisions.
- Cloud stores/manages history and exposes authorized data to the dashboard.
- Keep component boundaries explicit and contracts versioned.
- Use UTC timestamps and document metric units/sampling intervals.
- Design MQTT consumers for duplicate delivery; QoS 1 is not exactly-once delivery.
- Separate model inference from remediation execution.
- RCA returns probable causes with evidence, not absolute causal claims.
- No AI-generated arbitrary shell command execution.
- Remediation is allow-listed, authorized, auditable and health-verified.
- Do not claim malware detection, model accuracy, sub-second latency or production deployment without measured evidence.

## 6. Technology decision discipline

The current proposed stack is Python, psutil, SQLite, MQTT/Mosquitto, FastAPI, scikit-learn, PyTorch (preferred but confirm), ONNX Runtime, NetworkX, PostgreSQL, React/Vite, Recharts, Docker/Compose, Ubuntu 24.04 and Git.

- Inspect manifests before changing versions.
- Do not add TensorFlow if PyTorch is already established, or vice versa, without a reason and approval.
- TimescaleDB is optional; benchmark before adopting.
- Kubernetes, Terraform, Prometheus, Grafana, Loki, Alertmanager and LLM integrations are not confirmed project requirements. Do not add them by default.
- Avoid unnecessary dependencies and heavy services on the 16 GB host.

## 7. Data, model and evaluation rules

- Never fabricate datasets, labels, benchmark results or model performance.
- Keep training and inference preprocessing identical and versioned.
- Prevent temporal leakage in failure prediction.
- Define prediction horizon and label semantics before training Bi-LSTM.
- Report precision, recall, F1, false-positive rate and confusion matrix where labels exist.
- Record inference latency and resource use on documented hardware.
- Treat >=90% and sub-second latency as targets only until measured.
- Keep model artifacts and datasets out of Git if large/sensitive; document reproducible storage paths.

## 8. Security and remediation rules

- Use least privilege for containers, DB users, MQTT accounts and API roles.
- Never log secrets or place them in source code.
- Enforce authentication and topic ACLs for MQTT when crossing trust boundaries.
- Protect sensitive API endpoints with authorization, not authentication alone.
- Validate every command/action parameter against a strict allow-list.
- Never expose a general-purpose shell endpoint.
- Require human approval for high-impact actions unless the owner explicitly approves a tested low-risk policy.
- Add cooldowns, timeouts, audit records, kill switch and failure escalation.
- Only implement rollback where a safe compensating operation is defined and tested.
- Test remediation only on disposable services or isolated lab resources.

## 9. Database and API discipline

- Inspect existing migrations/models before proposing schema changes.
- Never invent a table/column/endpoint and assume the frontend or backend already supports it.
- Coordinate API request/response types with the frontend.
- Use migrations for schema changes; do not silently alter production data.
- Add indexes based on query patterns and measured workload.
- Protect approval/rejection and remediation endpoints with RBAC and audit logging.

## 10. Git and collaboration

- Check `git status` before editing.
- Do not overwrite uncommitted user changes.
- Prefer small feature branches/commits.
- Do not reset, rebase destructively, force-push or delete branches without approval.
- Coordinate shared contracts between Edge/Fog, ML and Cloud/UI workstreams.
- Do not commit `.env`, credentials, private keys, raw sensitive logs or large generated artifacts without explicit approval.

## 11. Documentation maintenance

After a feature changes:
- Update `IMPLEMENTATION_STATUS.md` with evidence and status.
- Update `PROJECT_ARCHITECTURE.md` if component boundaries or data flows change.
- Update API/schema documentation when contracts change.
- Update `TECH_STACK_AND_SETUP.md` when dependencies or setup commands change.
- Add real errors and verified fixes to `KNOWN_ISSUES.md`.
- Update roadmap milestones and acceptance criteria.
- Keep unknowns explicitly marked; do not silently convert proposals into facts.

## 12. Required final report format

For each coding task, report:

```text
Requirement:
Repository baseline inspected:
Plan:
Files changed:
Architecture/API/database impact:
Security/destructive-operation considerations:
Tests/commands run:
Test results (include failures):
Manual verification:
Known limitations:
Documentation updated:
Implementation status after change:
Recommended next step:
```

## 13. First task after importing this package

Do not immediately generate a new architecture or rewrite the code. First:
1. Locate the repository root.
2. Inspect Git status/history and file tree.
3. Identify the real package manifests, Docker/Compose files, database migrations, tests and current README.
4. Run existing checks without destructive changes.
5. Compare actual files to `IMPLEMENTATION_STATUS.md` and update the status with evidence.
6. Present the owner with a concise baseline report and the smallest next vertical slice.
