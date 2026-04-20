# Distributed Computing

ExpOps supports distributed execution on clusters and local multi-worker parallelism.

## Local Execution

Run pipelines locally with multi-worker parallelism:

```bash
expops run my-project --local
```

This uses local workers for parallel step execution.

## Cluster Execution

Execute pipelines on distributed clusters using Dask as the underlying distributed computing framework.

**Note**: Dask is the execution engine used by all cluster providers, not a provider itself. Providers determine how the Dask cluster is created or connected.

### SLURM Provider

Run on SLURM clusters by creating a Dask cluster via SLURM job submission:

1. Configure cluster in `configs/compute_config.yaml`:
```yaml
provider: slurm
num_workers: 4
options:
  worker_cores: 2
  worker_memory: 4GB
```

2. Run without `--local` flag:
```bash
expops run my-project
```

The SLURM provider uses `dask-jobqueue` to automatically submit Dask scheduler and worker jobs to the SLURM cluster.

## Configuration

Cluster settings are defined in `configs/compute_config.yaml`:

- **Provider**: `slurm`
- **Workers**: Number of worker nodes
- **Resources**: Cores and memory per worker
- **Queue settings**: Job queue and walltime

## Resource Management

ExpOps automatically:

- Allocates resources to workers
- Manages job submission
- Handles worker failures
- Distributes steps across workers

