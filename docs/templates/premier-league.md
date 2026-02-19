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
├── configs/
│   ├── project_config.yaml
│   └── compute_config.yaml
├── models/
│   └── premier_league_model.py
├── charts/
│   ├── plot_metrics.py
│   ├── plot_metrics.js
│   └── requirements.txt
├── requirements.txt
└── data/
```

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