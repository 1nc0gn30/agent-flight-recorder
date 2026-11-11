# Agent Flight Recorder

![Agent Flight Recorder Hero Banner](assets/hero-banner.svg)

**Agent Flight Recorder** (`agent-flight-recorder`) is a zero-dependency Python 3.10+ black-box telemetry recorder, time-scrubbing forensics server (`127.0.0.1:8104`), and standalone HTML incident-bundle exporter for AI agents. It captures command execution streams, tool events, policy violations, and cross-daemon telemetry into SQLite (`flight_recorder.db`) with millisecond timeline scrubbing.

- **Zoth Studio v2 Dossier**: [https://zoth.nullai.tech/tools/agent-flight-recorder](https://zoth.nullai.tech/tools/agent-flight-recorder)
- **Repository**: [https://github.com/1nc0gn30/agent-flight-recorder](https://github.com/1nc0gn30/agent-flight-recorder)
- **Default Loopback Endpoint**: `http://127.0.0.1:8104`

---

## Overview

- **Process & Stream Recording (`SessionRecorder`)**: Wraps arbitrary agent commands (`agent-flight record -- <cmd>`), capturing timestamped `stdout`/`stderr` lines, exit status, wall duration, and structured flight events.
- **Cross-Mesh Event Ingestion (`/api/ingest`)**: Receives structured security and operational events from `agent-capsule-jail` (`:8105`), `agent-policy-auditor` (`:8106`), `agent-prompt-firewall` (`:8098`), and `agent-voice-call` (`:8114`).
- **Millisecond Time-Scrubbing (`/api/scrub/<session_id>?offset_ms=...`)**: Reconstructs cumulative session forensics up to any millisecond offset along the flight timeline.
- **Single-File Incident Bundles (`export_bundle`)**: Exports any recorded flight session into a self-contained HTML artifact (`incident-<session_id>.html`) for offline post-mortem review.

---

## Architecture

```text
Agent Process / Sibling Security Daemons
        │
        ├── CLI Wrapper (SessionRecorder) ──┐
        └── POST /api/ingest ───────────────┼──► FlightStorage (SQLite)
                                            │         │
                                            ▼         ▼
                                 FlightDeckServer (:8104) + HTML Bundle Exporter
```

---

## Quickstart

```bash
git clone https://github.com/1nc0gn30/agent-flight-recorder.git
cd agent-flight-recorder
python3 -m pip install -e .

# Start the Time-Scrubbing Flight Deck on 127.0.0.1:8104
agent-flight serve --host 127.0.0.1 --port 8104
```

Or launch directly from the checkout:

```bash
python3 server.py --host 127.0.0.1 --port 8104
```

Open `http://127.0.0.1:8104` in your browser.

---

## CLI & ZothOS v3.4.0 Integration

`agent-flight` (`./bin/agent-flight` and `./bin/agent-flight-recorder-tui`) records agent sessions, queries storage telemetry, and exports offline incident bundles:

```bash
# Inspect daemon status and SQLite flight storage metrics
agent-flight --status
agent-flight --json
agent-flight status --port 8104 --json

# Record an agent execution session into flight_recorder.db
agent-flight record --title "Repo Audit Run" -- python3 my_agent.py

# Export the latest (or specified) session as a self-contained HTML replay bundle
agent-flight export --session <session_id> --output incident-report.html
```

---

## REST API Reference

Mutating `POST` routes enforce local-origin CSRF validation (`127.0.0.1`, `localhost`, `[::1]`, or non-browser CLI/daemon callers).

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/` | Web time-scrubbing flight deck (`public/index.html`) |
| `GET` | `/health`, `/api/health` | Service health (`status: "online"`, `port: 8104`) |
| `GET` | `/api/stats` | Total recorded sessions, forensic events, flight duration, and DB size |
| `GET` | `/api/sessions` | Recent recorded flight sessions (up to 50) |
| `GET` | `/api/session/<id>` | Metadata for a single flight session |
| `GET` | `/api/timeline/<id>` | Ordered forensic event list for `<id>` |
| `GET` | `/api/scrub/<id>?offset_ms=1500` | Reconstructed session state up to `offset_ms` |
| `GET` | `/api/briefing/<id>` | Spoken/executive incident briefing summary for `<id>` |
| `GET` | `/api/stream`, `/api/events/stream` | Real-time Server-Sent Events (SSE) flight telemetry stream |
| `POST` | `/api/ingest` | Ingest a forensic event (`{"session_id", "event_type", "severity", "payload"}`) |
| `POST` | `/api/export/bundle` | Generate a standalone HTML replay bundle for a session |

---

## Testing & Verification

```bash
PYTHONPATH=src python3 -m pytest -q
./bin/agent-flight --help
./bin/agent-flight --json
```

---

## Support & License

100% free and open-source software under the MIT License.

- Optional support: [Buy Us a Coffee via Stripe](https://buy.stripe.com/3cIfZg27PdKo2D64Ocfw40X)
- Engineered by **1nc0gn30** (`@1nc0gn30`, [https://1nc0gn30.si](https://1nc0gn30.si)) • **NullAI** ([https://nullai.tech](https://nullai.tech))
