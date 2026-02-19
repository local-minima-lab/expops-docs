# Configuration Files

ExpOps projects use YAML configuration files to define project settings, pipeline structure, and execution parameters.

## Configuration Files Overview

ExpOps projects use two main configuration files:

1. **`configs/project_config.yaml`** (required) - Main project configuration
2. **`configs/compute_config.yaml`** (optional) - Cluster execution settings

## Project Configuration (`project_config.yaml`)

The main configuration file contains these top-level sections:

```yaml
metadata:          # Project name, description, version
scripts:           # Script keys to file paths; first key is default for processes
environment:       # Named environments (requirements/env files + type)
reproducibility:   # Random seed configuration
data:              # Data sources and data path for hashing
experiment:        # Model framework, paths, parameters, pipeline, cache
reporting:         # Chart entrypoints, probe paths, reporting environment
```

### Key Sections

- **`metadata`**: Project identification (name, description, version)
- **`scripts`**: Maps script keys to file paths (e.g. `main: "project/models/model.py"`). The **first key** is used as the default when a process does not specify `script`. See [Defaults that reduce config](#defaults-that-reduce-config) below.
- **`environment`**: Named environment map (requirements file + type) used by processes and reporting
- **`reproducibility`**: Random seed configuration
- **`data`**: Data source paths and optional data path for hash-based cache invalidation
- **`experiment.parameters.pipeline`**: Pipeline DAG structure and process definitions (including optional fields and defaults)
  - See [Pipeline Execution](../features/pipelines.md) for details
- **`experiment.parameters.cache`**: Cache backend and KV backend configuration
  - See [Caching & Reproducibility](../features/caching.md) and [Backends](../advanced/backends.md) for details
- **`reporting`**: Chart entrypoints and chart definitions
  - See [Reporting Features](../features/reporting.md) for details

### Defaults that reduce config

You can omit some process fields and rely on defaults to keep config minimal:

- **Script**: If a process does not set `script`, the **first key** in the top-level `scripts` section is used. List your main script first so most processes need no `script` field.
- **code_function**: If omitted, it defaults to the process **name**. When your Python function name matches the process name (e.g. process `train_model` and function `train_model`), you can omit `code_function`.

Example minimal process that uses both defaults:

```yaml
processes:
  - name: "train_model"
```

See [Pipeline Execution](../features/pipelines.md) for full process attributes and process definitions.

### Data Hashing

Provide a CSV path so cache invalidation responds to data changes:

```yaml
data:
  path: "my-project/data/train.csv"
```

When the CSV contents change, process and step caches are invalidated using the data hash.

## Cluster Configuration (`compute_config.yaml`)

Optional configuration for distributed execution:

```yaml
provider: slurm
num_workers: 4
options:
  worker_cores: 2
  worker_memory: 4GB
  queue: normal
  walltime: "02:00:00"
```

See [Cluster Configuration](../advanced/cluster-config.md) for detailed setup instructions.

## Quick Reference

For detailed information on each configuration section:

- **Pipeline Definition**: [Pipeline Execution](../features/pipelines.md)
- **Process & Step Code**: [Model Code](model-code.md)
- **Caching**: [Caching & Reproducibility](../features/caching.md)
- **Backends**: [Backends](../advanced/backends.md)
- **Reporting/Charts**: [Reporting Features](../features/reporting.md) and [Chart Generation](charts.md)
- **Cluster Execution**: [Cluster Configuration](../advanced/cluster-config.md) and [Distributed Computing](../features/distributed.md)

## Example Configurations

See template projects for complete examples:
- **`sklearn-basic`**: Basic local execution setup
- **`premier-league`**: Comprehensive setup with cluster configuration and dynamic charts

