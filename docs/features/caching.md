# Caching & Reproducibility

ExpOps provides intelligent multi-level caching and reproducibility guarantees. The system supports both step-level and process-level caching for maximum flexibility.

## Caching Levels

### Step-Level Caching

Each pipeline step can be cached independently:

- **Cache key**: Based on step inputs, configuration hash, and function code hash
- **Cache lookup**: Automatic before step execution
- **Granularity**: Individual steps within a process
- **Use case**: When you want to skip specific steps that haven't changed, even if other steps in the process have

**Example**: If a data preprocessing step hasn't changed, it can be skipped even if the training step needs to run.

### Process-Level Caching

Entire processes (containing multiple steps) can be cached as a single unit:

- **Cache key**: Based on process inputs, configuration hash, and process function code hash
- **Cache lookup**: Automatic before process execution
- **Granularity**: Entire process as a single unit
- **Use case**: When you want to skip an entire process if all its inputs and configuration are unchanged

**Example**: If a complete training pipeline hasn't changed, the entire process can be skipped, avoiding execution of all its constituent steps.

### How They Work Together

- **Step-level caching** is checked first when executing individual steps
- **Process-level caching** is checked when starting a process execution
- If a process is cached, all its steps are skipped
- If a process isn't cached but some steps are, only the uncached steps execute
- Both levels use the same cache backends and KV stores

## Cache Backends

### Google Cloud Storage (GCS)

Remote backend for shared caching:
- Cross-machine sharing
- Persistent storage
- Requires GCP credentials


## Configuration

Cache settings in `configs/project_config.yaml`:

```yaml
model:
  parameters:
    cache:
      backend: local  # or gcs
```

## Reproducibility

### Random Seed Management

ExpOps manages random seeds for:
- NumPy random number generation
- Python's random module
- ML framework random states (sklearn, PyTorch, TensorFlow)

### Configuration

Seed settings in `configs/project_config.yaml`:

```yaml
reproducibility:
  seed: 42
```

## Cache Invalidation

Caches are invalidated when:

- **Step-level**: Step code changes (detected via function hash), step inputs change, or step configuration changes
- **Process-level**: Process code changes (detected via function hash), process inputs change, or process configuration changes

## Benefits

Multi-level caching and reproducibility provide:

- **Faster iterations**: Skip unchanged steps or entire processes
- **Flexible granularity**: Choose step-level for fine-grained control or process-level for coarse-grained optimization
- **Reproducible results**: Same inputs produce same outputs
- **Cost savings**: Avoid redundant computation at both step and process levels

