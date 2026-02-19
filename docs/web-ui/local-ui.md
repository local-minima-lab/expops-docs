# Local Web UI

ExpOps includes a local web UI for browsing projects, runs, and viewing charts.

## Starting the Server

Start the web UI server:

```bash
python -m expops.web.server
```

The server starts on `http://127.0.0.1:8000` by default.

## Accessing the UI

Open your browser and navigate to:

```
http://127.0.0.1:8000
```

## Features

### Browse Projects and Runs

- Select a project from the dropdown
- Choose a Run ID (from the configured KV backend)
- View run details and metadata

### View Static Charts

- Browse generated PNG charts
- View chart images
- Download charts

### Interact with Dynamic Charts

- Real-time metric visualization
- Live updates during execution
- Multiple chart types

## Requirements

The web UI requires:
- A configured KV backend in `configs/project_config.yaml`
- Metrics and charts from pipeline runs
- Static charts: PNG files in artifacts
- Dynamic charts: JavaScript chart definitions
