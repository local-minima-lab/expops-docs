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
├── configs/
│   └── project_config.yaml
├── models/
│   └── sklearn_model.py
├── charts/
│   ├── plot_metrics.py
│   └── requirements.txt
├── requirements.txt
└── data/
```

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
- Experiment parameters in `project_config.yaml`
- Pipeline steps in `models/sklearn_model.py`
- Chart visualizations in `charts/plot_metrics.py`
