# Local Web UI

ExpOps includes a FastAPI web server for browsing projects, monitoring runs, and viewing charts. Two server variants are available: a read-only monitoring server and a full run server that can also trigger pipeline runs.

## Starting the Server

### Monitoring server (read-only)

Serves the web UI and exposes REST APIs for browsing projects, runs, and charts:

```bash
python -m expops.web.server
```

Starts on `http://127.0.0.1:8000` by default.

### Run server

Includes all monitoring APIs plus the ability to trigger runs and stream live SSE data for the local SQLite backend. Use this when running locally without a Firestore backend:

```bash
python -m expops.web.run_server
```

Starts on `http://127.0.0.1:8765` by default.

Both servers respect `HOST` and `PORT` environment variables:

```bash
PORT=9000 python -m expops.web.run_server
```

## Accessing the UI

Open your browser and navigate to the server root. The static UI is served at:

```
http://127.0.0.1:8765
```

## REST API Reference

All API routes are prefixed with `/api`.

### Projects

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/projects` | List all project IDs in the workspace |
| `GET` | `/api/projects/{project_id}/graph` | Pipeline DAG structure for a project |
| `GET` | `/api/projects/{project_id}/runs` | List run IDs for a project |
| `GET` | `/api/projects/{project_id}/chart-config` | Chart metadata and probe paths from `project_config.yaml` |
| `GET` | `/api/projects/{project_id}/backend-config` | Effective KV backend type and config (run server only) |
| `GET` | `/api/projects/{project_id}/version-hash` | Cache version hash for the project (run server only) |

### Runs

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/projects/{project_id}/runs/{run_id}/status` | Step and process status snapshot |
| `GET` | `/api/projects/{project_id}/runs/{run_id}/charts` | List charts registered for a run |
| `GET` | `/api/projects/{project_id}/runs/{run_id}/charts/fetch` | Fetch a chart image (local path or GCS URI) |
| `GET` | `/api/projects/{project_id}/runs/{run_id}/metrics/{probe_path}` | Metrics for a probe path |

### Triggering Runs (run server only)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/run/{project_id}` | Start a background pipeline run |
| `GET` | `/api/run/{project_id}/active` | Check if a run subprocess is currently active |

### Workspace

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/workspace/info` | Workspace root path and git remote info |

## Backend-Aware Listeners

The web UI selects its real-time data mechanism based on the project's KV backend:

- **Local SQLite** (`backend.type: local`): the run server streams updates via Server-Sent Events (SSE) at `/api/live-data/{project_id}`.
- **GCP Firestore** (`backend.type: gcp`): the frontend subscribes directly to Firestore using the Firebase JS SDK.

The effective backend type is exposed via `GET /api/projects/{project_id}/backend-config`, which the UI reads on startup.

## Requirements

- A `configs/project_config.yaml` must exist in the project directory
- Charts and metrics require at least one completed pipeline run
- For Firestore-based live updates: a valid Firebase API key must be available via `firebase_api_key` in the backend config, a `firebase_api_key_file` path, or the `FIREBASE_API_KEY` environment variable

## Building Custom Dashboards

For custom real-time dashboards, use the [Listener SDK](listener-sdk.md) (`@expops/listener-sdk`). It provides React hooks that connect to either the local SSE endpoint or Firestore, matching whatever backend the project uses.
