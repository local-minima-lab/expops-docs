# Backends

ExpOps supports multiple backends for caching and storage.

## Cache Backends

### Local Filesystem

Default backend for local development:

```yaml
experiment:
  parameters:
    cache:
      backend: local
```

**Features**:
- Fast access
- No external dependencies
- Limited to single machine
- No web UI metrics support

The default cache/KV backend is local (SQLite), which is persistent and works across process restarts.

### Google Cloud Storage (GCS)

Remote backend for shared caching:

```yaml
experiment:
  parameters:
    cache:
      backend: gcs
      bucket: my-bucket
```

**Features**:
- Cross-machine sharing
- Persistent storage
- Web UI support
- Requires GCP credentials

**Setup**:
1. Create GCS bucket
2. Set up credentials (e.g. `firestore.json` in project root)
3. Configure bucket name in config

### Custom Backends

Implement custom backends for other storage systems.

## KV Backends

Key-value backends for metrics, metadata, and cache indexing:

The KV backend stores cache metadata (indexes that track where cached results are located). This is separate from the cache backend which stores the actual cached data files.

**Default**: If not specified, the system uses a local (SQLite) KV store, which is persistent. For remote or shared setups, configure Firestore.

### Firestore

Google Cloud Firestore:

```yaml
experiment:
  parameters:
    cache:
      backend: gcs
      kv_backend: firestore
```

**Features**:
- Persistent cache metadata
- Enables caching across runs
- Web UI metrics support
- Requires GCP credentials

**Setup**:
1. Create Firestore database
2. Add credentials to `firestore.json` in the project root
3. Configure in project config

## Configuration

Backend settings in `configs/project_config.yaml`:

```yaml
experiment:
  parameters:
    cache:
      backend: gcs  # or local, custom
      bucket: my-bucket  # for GCS
      kv_backend: firestore  # optional: firestore  (default is local SQLite)
```

**Note**: The `kv_backend` setting controls where cache metadata is stored. The default is local (SQLite). Use `firestore` for remote or shared setups.

## Web UI Requirements

For web UI metrics and charts:
- Use remote backend (GCS, etc.)
- Configure KV backend for metrics
- Ensure credentials are set up

