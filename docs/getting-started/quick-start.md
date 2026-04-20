# Quick Start

This guide will help you get started with ExpOps using the built-in template.

## Step 1: Create a Workspace

First, create a workspace directory for your ExpOps projects:

```bash
mkdir -p ~/expops-workspace && cd ~/expops-workspace
```

## Step 2: Create a Project from Template

Use the `sklearn-basic` template to create your first project:

```bash
expops create sklearn-basic --template sklearn-basic
```

This creates a new project at `sklearn-basic/` with:
- Configuration files
- Model code
- Chart generation scripts
- Requirements files
- Example data structure

## Step 3: Run the Project

Run the project locally:

```bash
expops run sklearn-basic --local
```

This will:

1. Set up the virtual environment
2. Install dependencies
3. Execute the pipeline
4. Generate artifacts and charts

## Step 4: View Results

The project configuration is located at:
```
sklearn-basic/configs/project_config.yaml
```

By default, the template uses a **local (SQLite) cache backend**, which is persistent. To enable cross-process live metrics (web app) or remote backends, update `experiment.parameters.cache` in the project config.

### Important: Caching and Web App Requirements

The default `cache.backend.type: local` supports persistent caching. For web app metrics and charts across runs, or for remote/shared setups, set `cache.backend.type` to `gcp` (Firestore) and optionally configure `cache.object_store` for GCS. Example in `project_config.yaml`:

```yaml
experiment:
  parameters:
    cache:
      backend:
        type: local   # default; use gcp for Firestore (web app / shared runs)
        # For GCP (Firestore):
        # type: gcp
        # gcp_project: <your-gcp-project-id>
        # credentials_json: firestore.json
      # Optional object store for cache artifacts (e.g. GCS):
      # object_store:
      #   type: gcs
      #   bucket: <your-gcs-bucket-name>
      #   # Manifests will be stored under gs://<bucket>/<project_id>/cache/steps/ automatically.
```

**Setup steps for GCP (Firestore)**:
1. Create a Firestore database in Google Cloud
2. Add credentials to `sklearn-basic/firestore.json`
3. Set `cache.backend.type` to `gcp` and set `gcp_project` and `credentials_json` under `cache.backend`

See the [Backends](../advanced/backends.md) documentation for more details and setup instructions.

## Running on a Cluster

To run on a distributed cluster (e.g., SLURM):

1. Add a `compute_config.yaml` under `configs/`
2. Remove the `--local` flag when running:

```bash
expops run sklearn-basic
```
