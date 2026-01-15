# Project Structure Overview

Each ExpOps project follows a standardized directory structure that organizes configuration, code, data, and artifacts.

## Directory Layout

```
my-project/
├── configs/
│   ├── project_config.yaml      # Main project configuration
│   └── cluster_config.yaml      # Optional cluster configuration
├── models/
│   └── <model_name>.py          # Model implementation
├── charts/
│   ├── plot_metrics.py          # Static chart generation
│   ├── plot_metrics.js          # Dynamic chart generation
│   └── requirements.txt         # Chart dependencies
├── data/                         # Input datasets
├── requirements.txt              # Main project dependencies
├── logs/                         # Execution logs
├── keys/                         # Credentials (e.g., firestore.json)
└── artifacts/                    # Generated artifacts
    ├── charts/
```

## Key Components

### Configuration Files

The `configs/` directory contains all project configuration:
- **project_config.yaml**: Main configuration (required)
- **cluster_config.yaml**: Cluster execution settings (optional)

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

See [Reporting](reporting.md) for details.

### Dependencies

- **requirements.txt**: Main dependencies for training/inference
- **charts/requirements.txt**: Reporting-specific dependencies

See [Dependencies](dependencies.md) for details.
