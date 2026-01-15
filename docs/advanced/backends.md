# Backends

ExpOps supports multiple backends for caching and storage.

## Cache Backends

### Local Filesystem

Default backend for local development:

```yaml
model:
  parameters:
    cache:
      backend: local
```

**Features**:

- Fast access
- No external dependencies
- Limited to single machine
- No web UI metrics support

**Caching Limitation**: When using the default in-memory KV backend, caching only works within a single process execution. The cache metadata (which tracks where cached results are stored) is lost when the process restarts, so cached results cannot be retrieved across runs even if the cache files exist on disk. For persistent caching across runs, you must configure a persistent KV backend (see [KV Backends](#kv-backends) below).

### Google Cloud Storage (GCS)

Remote backend for shared caching:

```yaml
model:
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
2. Set up credentials in `keys/`
3. Configure bucket name in config

### Custom Backends

Implement custom backends for other storage systems.

## KV Backends

Key-value backends for metrics, metadata, and cache indexing:

The KV backend stores cache metadata (indexes that track where cached results are located). This is separate from the cache backend which stores the actual cached data files.

**Default**: If not specified, the system uses an in-memory KV store which is **not persistent**. This means:

- Cache metadata is lost when the process restarts
- Caching only works within a single process execution
- Cached results cannot be retrieved across runs

For persistent caching across runs, configure a persistent KV backend (Firestore).

### Firestore

Google Cloud Firestore:

```yaml
model:
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
2. Add credentials to `keys/firestore.json`
3. Configure in project config

## Configuration

Backend settings in `configs/project_config.yaml`:

```yaml
model:
  parameters:
    cache:
      backend: gcs  # or local, custom
      bucket: my-bucket  # for GCS
      kv_backend: firestore  # optional: firestore, redis, or memory (default, not persistent)
```

**Note**: The `kv_backend` setting controls where cache metadata is stored. For persistent caching across runs, use `firestore`. The default `memory` backend only works within a single process execution.

## Web UI Requirements

For web UI metrics and charts:
- Use remote backend (GCS, etc.)
- Configure KV backend for metrics
- Ensure credentials are set up

