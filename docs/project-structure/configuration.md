# Configuration Files

ExpOps projects use YAML configuration files to define project settings, pipeline structure, and execution parameters.

## Configuration Files Overview

ExpOps projects use two main configuration files:

1. **`configs/project_config.yaml`** (required) - Main project configuration
2. **`configs/cluster_config.yaml`** (optional) - Cluster execution settings

## Project Configuration (`project_config.yaml`)

The main configuration file contains these top-level sections:

```yaml
metadata:          # Project name, description, version
environment:       # Virtual environment and dependencies
reproducibility:   # Random seed configuration
model:             # Model framework, paths, hyperparameters, pipeline, cache
reporting:         # Chart entrypoints and probe paths
```

### Key Sections

- **`metadata`**: Project identification (name, description, version)
- **`environment`**: Virtual environment settings with separate requirements for training and reporting
- **`reproducibility`**: Random seed configuration
- **`model.parameters.pipeline`**: Pipeline DAG structure and process definitions
  - See [Pipeline Execution](../features/pipelines.md) for details
- **`model.parameters.cache`**: Cache backend and KV backend configuration
  - See [Caching & Reproducibility](../features/caching.md) and [Backends](../advanced/backends.md) for details
- **`reporting`**: Chart entrypoints and chart definitions
  - See [Reporting Features](../features/reporting.md) for details

## Cluster Configuration (`cluster_config.yaml`)

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

