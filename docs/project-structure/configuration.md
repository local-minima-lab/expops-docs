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

- **Scripts section**: The top-level `scripts` map still defines script keys to file paths. The **first key** is treated as the default script for processes that do not explicitly specify a script key in `code`.
- **Code field**: Each process may define a single `code` field instead of separate `script` and `code_function` fields:
  - `code: "script_key.function_name"` → use the script registered under `script_key` and call `function_name` from that module.
  - `code: "function_name"` → use the first script key in `scripts` and call `function_name`.
- **Omitted code**:
  - For ordinary processes, if `code` is omitted the system assumes a function with the same name as the process, loaded from the default script.
  - For data/seed split helper nodes (processes that only define `data_parallelism` or `seed_parallelism` and no `code`), the system treats them as function-less split nodes.

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
- **Reporting/Charts**: [Reporting Features](../features/reporting.md)
- **Cluster Execution**: [Cluster Configuration](../advanced/cluster-config.md) and [Distributed Computing](../features/distributed.md)

## Example Configurations

See template projects for complete examples:
- **`sklearn-basic`**: Basic local execution setup
- **`premier-league`**: Comprehensive setup with cluster configuration and dynamic charts