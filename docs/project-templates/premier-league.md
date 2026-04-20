# premier-league Template

The `premier-league` template is a comprehensive ML project that demonstrates advanced ExpOps features.

## Create Project

```bash
expops create my-project --template premier-league
```

## What's Included

- **Configuration**: Full `project_config.yaml` with cluster support
- **Model**: Complex pipeline for football match prediction
- **Charts**: Both static and dynamic charts
- **Compute Config**: SLURM cluster configuration
- **Requirements**: Complete dependency setup

## Project Structure

```
my-project/
├── .my-project/
│   ├── envs/
│   ├── logs/
│   ├── cache/<version_hash>/<encoded_probe_path>/
│   ├── artifacts/<version_hash>/<encoded_probe_path>/
│   └── metrics.sqlite
├── configs/
│   ├── project_config.yaml
│   └── compute_config.yaml
├── src/
│   ├── models/
│   │   └── premier_league_model.py
│   └── charts/
│       ├── plot_metrics.py
│       └── plot_metrics.js
├── requirements.txt
├── requirements-charts.txt
└── data/
    └── England CSV.csv
```

When you run the template, its virtual environments are created under the
project-local hidden `.my-project/envs/` directory (for example
`.my-project/envs/premier-league-env`). The older workspace-level `.venvs/`
directory is kept only for legacy, non-project usage and is no longer used
for project runs.

## Features Demonstrated

### Distributed Execution

Includes `compute_config.yaml` for SLURM cluster execution:

- Worker configuration
- Resource allocation
- Queue settings

### Dynamic Charts

JavaScript-based real-time visualizations:

- Live metric updates
- Interactive exploration
- Web UI integration

### Complex Pipeline

Multiple pipeline steps:

- Data loading
- Feature engineering
- Model training
- Evaluation
- Ensemble methods

## Running

### Local Execution

```bash
expops run my-project --local
```

### Cluster Execution

```bash
expops run my-project
```

## Configuration

The template demonstrates:

- **Remote cache backend**: GCS or other backends
- **Cluster execution**: SLURM integration
- **Dynamic reporting**: Real-time charts
- **Complex DAG**: Multiple dependent steps