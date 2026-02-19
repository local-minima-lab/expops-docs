# Project Structure Overview

Each ExpOps project follows a standardized directory structure that organizes configuration, code, data, and artifacts.

## Directory Layout

```
my-project/
├── configs/
│   ├── project_config.yaml      # Main project configuration
│   └── compute_config.yaml      # Optional cluster configuration
├── models/
│   └── <model_name>.py          # Model implementation
├── charts/
│   ├── plot_metrics.py          # Static chart generation
│   ├── plot_metrics.js          # Dynamic chart generation
│   └── requirements.txt         # Chart dependencies
├── data/                         # Input datasets
├── requirements.txt              # Main project dependencies
├── logs/                         # Execution logs
└── artifacts/                    # Generated artifacts
    ├── charts/
```

For GCP-backed backends (e.g. Firestore), place credentials in the project root (e.g. `firestore.json`) and set `credentials_json: firestore.json` in your config.

## Key Components

### Configuration Files

The `configs/` directory contains all project configuration:
- **project_config.yaml**: Main configuration (required)
- **compute_config.yaml**: Cluster execution settings (optional)

See [Configuration Files](configuration.md) for details.

### Model Code

The `models/` directory contains your ML pipeline implementation:
- Process definitions with `@process()` decorator
- Step functions with `@step()` decorator
- Pipeline logic and data transformations

See [Model Code](model-code.md) for details.

### Chart Generation

The `charts/` directory contains visualization code:
- **plot_metrics.py**: Static PNG chart generation
- **plot_metrics.js**: Dynamic interactive charts

See [Chart Generation](charts.md) for details.

### Dependencies

- **requirements.txt**: Main dependencies for training/inference
- **charts/requirements.txt**: Reporting-specific dependencies