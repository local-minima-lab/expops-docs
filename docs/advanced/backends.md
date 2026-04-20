# Backends

ExpOps supports multiple backends for caching, metrics storage, and artifact storage. Backend settings live under `experiment.cache` in `configs/project_config.yaml`.

## KV Backends

The KV (key-value) backend stores run metadata, step status, and logged metrics. It is the primary backend for the web UI and caching.

**Default**: If `backend` is not specified, ExpOps uses a local SQLite store under the project's runtime directory (`.my-project/metrics.sqlite`). This is persistent across process restarts and works without any external dependencies.

### Local (SQLite)

```yaml
experiment:
  cache:
    backend:
      type: local
```

**Features**:

- No external dependencies
- Persistent across restarts
- Supports the web app via SSE streaming (requires local run server)
- Limited to a single machine

### GCP (Firestore)

```yaml
experiment:
  cache:
    backend:
      type: gcp
      gcp_project: my-gcp-project-id
      credentials_json: firestore.json
```

**Features**:

- Persistent metrics accessible across machines
- Enables real-time Firestore listeners in the web app
- Requires a GCP project with Firestore enabled

**Setup**:

1. Create a Firestore database in your GCP project (Native mode)
2. Create a service account with the **Cloud Datastore User** role and download its credentials JSON
3. Place the credentials file in the project root (e.g. `firestore.json`)
4. Set `gcp_project` and `credentials_json` in the backend config

```yaml
experiment:
  cache:
    backend:
      type: gcp
      gcp_project: my-gcp-project-id
      credentials_json: firestore.json   # path relative to project root
```

---

## Object Store Backends

The object store holds cache artifacts and chart outputs. Configure it under `experiment.cache.object_store`.

**Default**: Artifacts are stored on the local filesystem under `.my-project/artifacts/`. No configuration needed for local development.

### GCS (Google Cloud Storage)

```yaml
experiment:
  cache:
    backend:
      type: gcp
      gcp_project: my-gcp-project-id
      credentials_json: firestore.json
    object_store:
      type: gcs
      bucket: my-gcs-bucket-name
```

**Features**:

- Artifacts stored at `gs://<bucket>/<project_id>/artifacts/<version_hash>/...`
- Cache manifests stored at `gs://<bucket>/<project_id>/cache/steps/`
- Shared across machines running the same project

**Setup**:

1. Create a GCS bucket in the same GCP project
2. Grant the service account **Storage Object Admin** on the bucket
3. Add the `object_store` block to the cache config

---

## Full GCP Configuration Example

```yaml
experiment:
  cache:
    backend:
      type: gcp
      gcp_project: my-gcp-project-id
      credentials_json: firestore.json
    object_store:
      type: gcs
      bucket: my-gcs-bucket-name
```

---

## Web App Backend Requirements

The ExpOps Web App reads the backend config from `project_config.yaml` to decide which listener to use:

- **`type: local`** → connects to a local run server that streams updates via SSE
- **`type: gcp`** → connects directly to Firestore for real-time updates

For Firestore-based live updates, the web app also needs a Firebase API key. Provide it via one of:

- `firebase_api_key` inline in the backend config
- `firebase_api_key_file` — path to a file containing the key (relative to project root)
- `FIREBASE_API_KEY` or `NEXT_PUBLIC_FIREBASE_API_KEY` environment variable

See [ExpOps Web App](../web-ui/local-ui.md) for more details.
