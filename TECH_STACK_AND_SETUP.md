# TECH STACK AND LOCAL SETUP

> This file records the proposed stack and previously recommended local environment. It does not assert that software is installed or that a deployment has succeeded. Inspect the actual repository and machine before executing setup commands.

## 1. Hardware and operating-system constraints

- **Host OS:** Windows (version/build UNKNOWN).
- **Host RAM:** 16 GB.
- **Virtualization recommendation:** VirtualBox.
- **Guest OS recommendation:** Ubuntu 24.04 LTS.
- **Suggested initial VM allocation:** 8 GB RAM, 4 vCPUs, 60 GB dynamically allocated virtual disk. Adjust based on CPU and host load; do not allocate all host RAM.
- **Deployment style:** Docker containers inside Ubuntu VM.
- **Edge simulation:** Start with multiple agent containers rather than multiple full VMs.

The exact installed versions and current VM state are UNKNOWN.

## 2. Technology inventory

| Layer | Technology | Purpose | Selection status |
|---|---|---|---|
| Edge | Python 3.12 | Agent and collectors | Recommended |
| Edge metrics | psutil | Host CPU/RAM/disk/network metrics | Proposed |
| Edge buffer | SQLite | Offline telemetry buffering | Proposed |
| Messaging | MQTT | Edge–Fog messaging | Proposed |
| Broker | Eclipse Mosquitto 2 | MQTT broker | Proposed |
| Fog/API | Python + FastAPI | Ingestion services/API | Proposed |
| ML baseline | scikit-learn | Isolation Forest | Proposed |
| Deep learning | PyTorch | Autoencoder and Bi-LSTM training | Preferred in later guidance; confirm |
| Alternative DL | TensorFlow | Alternative training framework mentioned in brief | Not locked |
| Model runtime | ONNX Runtime | Fog inference for compatible exported models | Proposed |
| RCA | NetworkX | Initial dependency graph | Proposed |
| Database | PostgreSQL 16 | Persistent telemetry/incidents | Proposed |
| Time-series extension | TimescaleDB | Optional time-series capabilities | Optional, not selected as required |
| Frontend | React + Vite | Web dashboard | Proposed |
| Charts | Recharts | Metric visualizations | Proposed |
| Auth | JWT + RBAC | User and action authorization | Proposed |
| Host/guest | Windows + Ubuntu 24.04 | Local development | Recommended |
| VM | VirtualBox | Linux guest | Recommended |
| Containers | Docker + Compose | Local multi-service deployment | Recommended |
| VCS | Git/GitHub | Collaboration and history | Recommended |

Kubernetes, Terraform, Prometheus, Grafana, Loki, Alertmanager, cloud vendor, LLM integration and distributed tracing are not established selections in available evidence.

## 3. Local architecture

```text
Windows host (16 GB RAM)
└── VirtualBox
    └── Ubuntu 24.04 LTS guest
        └── Docker Engine + Docker Compose
            ├── edge-agent-1
            ├── edge-agent-2
            ├── edge-agent-3
            ├── mosquitto
            ├── fog-service(s)
            ├── postgres
            ├── cloud-api (FastAPI)
            └── dashboard (React build)
```

This is a suggested development topology. A final container list should be based on actual services and resource usage.

## 4. Installation sequence (proposed)

No exact historical installation transcript or successful command output was available. Use current official installation instructions for VirtualBox, Ubuntu and Docker; package repositories and commands can change.

### 4.1 Prepare Ubuntu guest

After installing Ubuntu 24.04 in VirtualBox, update packages:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y git curl ca-certificates gnupg build-essential python3 python3-venv python3-pip
```

### 4.2 Install Docker Engine and Compose plugin

Use the current official Docker Engine for Ubuntu instructions for the supported repository and signing-key setup. Do not copy an old repository command blindly. After installation, verify:

```bash
docker --version
docker compose version
sudo docker run --rm hello-world
```

If using Docker without `sudo`, follow the security implications of Docker group membership; membership in the `docker` group effectively grants high privileges. Do not add users to the group without understanding this.

### 4.3 Verify development tools

```bash
git --version
python3 --version
python3 -m venv --help
```

Node.js version and installation method should be selected based on the actual dashboard `package.json`; do not guess the version.

## 5. Repository setup

After locating the actual repository:

```bash
cd /path/to/ai-edge-fog-cloud
pwd
git status --short
find . -maxdepth 3 -type f -not -path './.git/*' | sort
```

If creating a new repository from scratch is explicitly approved, a proposed structure is documented in `PROJECT_MASTER_CONTEXT.md`. Do not create a parallel project tree if an existing repository already contains working code.

## 6. Docker Compose design (not an existing verified file)

Previously discussed starter services:

| Service | Image/build | Role | Proposed port exposure |
|---|---|---|---|
| `mqtt` | `eclipse-mosquitto:2` | Broker | 1883 only for local development/private access |
| `database` | `postgres:16` | PostgreSQL | Prefer no host exposure; private Compose network |
| `fog` | Build from `./fog` | Ingestion, processing, AI/RCA services | Only required API/health ports |
| `cloud-api` | Build from `./cloud` | FastAPI | 8000 for local development |
| `dashboard` | Build from `./dashboard` | React web app | 3000 host → 80 container in prior example |

Suggested persistent volumes: MQTT data and PostgreSQL data. A valid Compose file also needs Dockerfiles, broker configuration, environment templates, health checks, startup dependencies and a private network. `depends_on` alone does not guarantee an application is ready.

### 6.1 Environment variable names known from prior discussion

- `DB_PASSWORD`
- `DATABASE_URL`
- `MQTT_HOST`
- `MQTT_PORT`

The full environment contract is UNKNOWN. Inspect code/config before adding variables. Use placeholder values in `.env.example`; actual `.env` must be ignored by Git.

## 7. Proposed startup and diagnostic commands

Only run after inspecting the repository's Compose and environment configuration:

```bash
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
```

Stop without deleting persistent data:

```bash
docker compose down
```

**Destructive warning:** `docker compose down -v` removes named volumes and can delete database/broker data. Never run without explicit approval and a backup.

## 8. Network guidance

- Use a private Docker network for internal MQTT, Fog and database communication.
- Publish only the dashboard/API ports needed for local access.
- Do not expose PostgreSQL to the public Internet.
- Do not expose unauthenticated MQTT to untrusted networks.
- Exact VirtualBox network mode (NAT, bridged or host-only) is UNKNOWN; choose based on whether host-only access or LAN access is required.
- Add TLS, MQTT authentication and topic ACLs before non-local use.

## 9. Security and configuration

- Never commit `.env`, private keys, tokens, passwords or database dumps containing sensitive information.
- Provide `.env.example` with names and placeholders only.
- Use least-privilege DB users and service accounts.
- Avoid privileged containers and unnecessary host mounts.
- Use non-root container users where feasible.
- Pin image/dependency versions after compatibility testing.
- Rotate any credential found exposed in Git history; deleting the file in a later commit does not revoke the secret.

## 10. Build, test and health checks

Commands depend on actual manifests. First discover:

```bash
find . -maxdepth 4 \( -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'package.json' -o -name 'Dockerfile*' -o -name 'compose.y*' -o -name '*.sql' \) -print
```

Potential checks after implementation:
- Python: project-defined `pytest` command.
- Frontend: package scripts in `package.json` (for example build/test scripts only if present).
- Compose: `docker compose config`, `docker compose ps`, health endpoints.
- MQTT: subscribe/publish test with local broker.
- DB: verify connection and migration status.

No test command is verified as currently available.

## 11. Deployment, rollback and backups

Current intended target is a local academic demo. No cloud provider, CI/CD pipeline, Terraform state or Kubernetes deployment is confirmed.

Before upgrades:
1. Back up PostgreSQL and relevant volumes.
2. Record current Git commit and image tags.
3. Apply migration in a test environment.
4. Verify health and core data paths.
5. Roll back to previous image/commit if health checks fail; database schema rollback requires an explicit migration plan.

Never delete volumes as a generic troubleshooting step. A database restore procedure must be tested before final demo.
