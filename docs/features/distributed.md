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

1. Install SLURM dependencies:
```bash
pip install expops[slurm]
```

2. Configure cluster in `configs/cluster_config.yaml`:
```yaml
provider: slurm
workers: 4
cores_per_worker: 2
memory_per_worker: 4GB
```

3. Run without `--local` flag:
```bash
expops run my-project
```

The SLURM provider uses `dask-jobqueue` to automatically submit Dask scheduler and worker jobs to the SLURM cluster.

## Configuration

Cluster settings are defined in `configs/cluster_config.yaml`:

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

