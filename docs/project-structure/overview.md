# Project Structure Overview

Each ExpOps project follows a standardized directory structure that organizes configuration, code, data, and artifacts.

## Directory Layout

```
my-project/
├── configs/
│   ├── project_config.yaml      # Main project configuration
│   └── compute_config.yaml      # Optional cluster configuration
├── src/
│   ├── <model_name>.py          # Model implementation (main script)
│   ├── plot_metrics.py           # Static chart generation
│   ├── plot_metrics.js          # Optional dynamic chart generation
│   └── ...                      # Other project modules
├── data/                         # Input datasets
├── requirements.txt              # Main project dependencies
├── requirements-charts.txt       # Chart/reporting dependencies
└── .<project_id>/                # Runtime directory (hidden)
    ├── envs/                     # Project virtual environments
    ├── logs/                     # Execution logs
    ├── cache/                    # Step/process cache
    │   └── <version_hash>/<encoded_probe_path>/
    └── artifacts/                # Charts and other artifacts
        └── <version_hash>/<encoded_probe_path>/
```

For GCP-backed backends (e.g. Firestore), set the path to your credentials file in your config (e.g. `credentials_json: src/firestore.json` or `credentials_json: firestore.json`).

## Key Components

### Configuration Files

The `configs/` directory contains all project configuration:
- **project_config.yaml**: Main configuration (required)
- **compute_config.yaml**: Cluster execution settings (optional)

See [Configuration Files](configuration.md) for details.

### Model Code

The `src/` directory contains your ML pipeline and reporting scripts. The main model script (e.g. `<model_name>.py`) is referenced in `project_config.yaml` under `scripts.main`:
- Process definitions with `@process()` decorator
- Step functions with `@step()` decorator
- Pipeline logic and data transformations

See [Model Code](model-code.md) for details.

### Chart Generation

Chart scripts live under `src/` and are referenced in config (`scripts.reporting`, `scripts.reporting_js`):
- **plot_metrics.py**: Static PNG chart generation
- **plot_metrics.js**: Optional dynamic interactive charts

### Dependencies

- **requirements.txt**: Main dependencies for training/inference
- **requirements-charts.txt**: Reporting/chart dependencies