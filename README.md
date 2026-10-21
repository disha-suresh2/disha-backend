# PotencyGuard Backend

A robust decision-support and telemetry-ingestion backend for temperature-sensitive medicine supply chains (such as vaccines and biologics). Built for SAP Hackfest 2026, Bengaluru Region.

> **Honest Labelling Notice (Rule 9)**:
> All SAP integrations in this codebase are implemented as local mock adapters (`MockSapAdapter`), storing simulated business events in SQLite (`sap_outbox`). No live connection to SAP S/4HANA or SAP BTP is made. All telemetry data, temperature readings, sensor signatures, and shipment histories are strictly synthetic and simulated for demonstration and evaluation purposes.

---

## 1. Requirements
- Python 3.11 or newer (tested on Python 3.14)
- SQLite 3 with WAL mode enabled
- Virtual environment with dependencies from `requirements.txt`

## 2. Installation
To install the backend into a local virtual environment:

```bash
# From the backend directory
python -m venv .venv

# On Windows:
.\.venv\Scripts\python.exe -m pip install -r requirements.txt

# On Linux / macOS:
./.venv/bin/python -m pip install -r requirements.txt
```

## 3. Running the Server
Start the FastAPI server on `127.0.0.1:8000`:

```bash
# Windows
.\.venv\Scripts\python.exe scripts/start_server.py

# Linux / macOS
./.venv/bin/python scripts/start_server.py
```

Verify service health:
```bash
curl http://127.0.0.1:8000/api/v1/health
```

## 4. Running the Tests
Run the entire automated test suite:

```bash
# Windows
.\.venv\Scripts\python.exe -m pytest

# Linux / macOS
./.venv/bin/python -m pytest
```

## 5. Demo Scenarios & Simulation
PotencyGuard includes four deterministic demo scenarios to demonstrate real-time telemetry ingestion, potency tracking, tail-risk CVaR disposition, and cryptographic tamper detection:

- **S1 (Normal Journey)**: 60 records, constant 5°C, final potency ~0.9993, auto-approved `continue`.
- **S2 (Cooling Drop & Recovery)**: 70 records, excursion to 11.3°C, recovers safely, auto-approved `continue`.
- **S3 (Heat Wave Cooling Failure - The Core Demo)**: 92 records, temperature peaks at 35.9°C. Potency drops to ~0.928. While naive expected loss recommends `continue` (low average loss), tail-risk CVaR reveals an catastrophic 6.4% loss risk ($50,000 loss). The planner recommends `reroute_to_depot`, escalating to human QA approval.
- **S4 (Adversarial Tampering & Attacks)**: 35 records, injecting physical jump flags, forged signatures (`BAD_SIGNATURE`), sequence conflicts (`SEQ_CONFLICT`), and database record tampering (`demo/tamper`), automatically breaking verification.

To run a scenario:
```powershell
.\.venv\Scripts\python.exe scripts/run_scenario.py s1 --reset
.\.venv\Scripts\python.exe scripts/run_scenario.py s3 --reset
```

## 6. Folder Map
- `app/`: Core backend application
  - `api/`: REST endpoints and WebSocket live stream
  - `core/`: Pure business logic (potency kinetics, thermal simulation, CVaR planner, governance, canonical serialization)
  - `db/`: SQLite connection, table schema, and data repositories
  - `schemas/`: Pydantic v2 data models and validation rules
  - `services/`: Ingestion pipeline, chain verification, audit logging, alerting, and mock SAP adapter
  - `demo/`: Synthetic edge producer and demo scenarios S1–S4
- `scripts/`: Operational utilities (server runner, demo runners, evidence generator, example edge client)
- `tests/`: Automated pytest suites, test vectors, and fixtures
- `docs/`: Technical specifications, architecture, evidence studies, and phase reports
- `data/`: Local SQLite database storage (git-ignored)
