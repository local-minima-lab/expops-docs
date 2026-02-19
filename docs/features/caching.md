# Caching & Reproducibility

ExpOps provides intelligent multi-level caching and reproducibility guarantees. The system supports both step-level and process-level caching for maximum flexibility.

## Artifact layout

When caching is enabled, the platform separates **lightweight manifests** from **heavy artifacts**:

- **Step/process manifests (`.pkl` files)**:
    - Stored under a cache prefix (for example `cache/steps/steps/` in GCS).
    - Contain small Python dictionaries describing the result of a step or process.
    - Heavy artifacts (models, large arrays) are represented as *references* rather than inlined objects.
- **Models (`models/` prefix)**:
    - Any recognized model object (for example, most `sklearn` and `xgboost` estimators) is always serialized separately.
    - Stored as `.joblib` blobs under a `models/<run_id>/<process_name>/...` path.
- **Payloads (`payloads/` prefix)**:
    - Large array-like payloads (NumPy arrays, lists/tuples coercible to arrays) above a small size threshold are stored as compressed `.npz` blobs.

## Caching levels

### Step-level caching

Each pipeline step can be cached independently:

- **Cache key**: Based on step inputs, configuration hash, and function code hash.
- **Cache lookup**: Automatic before step execution.
- **Granularity**: Individual steps within a process.
- **Use case**: Skip specific steps that haven't changed, even if other steps in the process have.

**Example**: If a data preprocessing step hasn't changed, it can be skipped even if the training step needs to run.

### Process-level caching

Entire processes (containing multiple steps) can be cached as a single unit:

- **Cache key**: Based on process inputs, configuration hash, and process function code hash.
- **Cache lookup**: Automatic before process execution.
- **Granularity**: Entire process as a single unit.
- **Use case**: Skip an entire process if all its inputs and configuration are unchanged.

**Example**: If a complete training pipeline hasn't changed, the entire process can be skipped, avoiding execution of all its constituent steps.

### How they work together

- Step-level caching is checked first when executing individual steps.
- Process-level caching is checked when starting a process execution.
- If a process is cached, all its steps are skipped.
- If a process isn't cached but some steps are, only the uncached steps execute.
- Both levels use the same cache backends and KV stores.

## Cache backends

### Local (SQLite)

Persistent local KV store for runs/metrics/charts:

- Default for local runs.
- No external services required.
- Shared by pipeline runs and the UI.

### Google Cloud Storage (GCS)

Remote backend for shared caching:

- Cross-machine sharing.
- Persistent storage.
- Requires GCP credentials.

When an object store is configured, manifests are written as `.pkl` blobs and heavy artifacts are written under the `models/` and `payloads/` prefixes in the same bucket. When no object store is configured, all of these artifacts are stored on the local filesystem under the project cache directory.

## Configuration

Cache settings in `configs/project_config.yaml`:

```yaml
experiment:
  parameters:
    cache:
      backend:
        # Persistent local KV store (default for local runs)
        type: local
```

Data path configuration (optional):

```yaml
data:
  path: "my-project/data/train.csv"
```

## Cache invalidation

Caches are invalidated when:

- **Step-level**: Step code changes (detected via function hash), step inputs change, or step configuration changes.
- **Process-level**: Process code changes (detected via function hash), process inputs change, or process configuration changes.
- **Data hash**: CSV content changes at `data.path`; for data-parallel pipelines, only the affected partitions re-run.