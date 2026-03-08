# sklearn-basic Template

The `sklearn-basic` template provides a minimal, runnable project skeleton for learning ExpOps.

## Create Project

```bash
expops create my-project --template sklearn-basic
```

## What's Included

- **Configuration**: Basic `project_config.yaml` with local execution
- **Model**: Simple sklearn model training pipeline
- **Charts**: Basic static chart generation
- **Requirements**: Minimal dependencies
- **Structure**: Complete project layout

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
│   └── project_config.yaml
├── src/
│   ├── models/
│   │   └── model.py
│   └── charts/
│       └── plot_metrics.py
├── requirements.txt
├── requirements-charts.txt
└── data/
    └── train.csv
```

Project-specific Python virtual environments are created under the hidden
runtime directory `.my-project/envs/` (for example
`.my-project/envs/sklearn-basic-env`). Earlier versions of the platform used
a workspace-level `.venvs/` directory; that location is now deprecated for
project runs.

## Running

Run locally:

```bash
expops run my-project --local
```

## Configuration

The template uses:
- **Local cache backend**: Fast local development
- **venv environment**: Python virtual environment
- **Basic pipeline**: Load → Preprocess → Train → Evaluate

## Customization

You can customize:
- Experiment parameters in `configs/project_config.yaml`
- Pipeline steps in `src/models/model.py`
- Chart visualizations in `src/charts/plot_metrics.py`
