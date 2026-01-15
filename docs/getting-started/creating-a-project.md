# Creating a Project

You can create a new ExpOps project either from scratch or using a template.

## Create from Template

Templates provide a pre-configured project structure with example code:

```bash
expops create my-project --template sklearn-basic
```

Available templates:

- `sklearn-basic`: Runnable project skeleton with a tiny sklearn model
- `premier-league`: Comprehensive ML project with cluster config and dynamic charts

## Create from Scratch

To create a new project without a template:

```bash
expops create my-project
```

This creates a minimal project structure that you can customize.

## Project Structure

After creation, your project will have the following structure:

```
my-project/
├── configs/
│   └── project_config.yaml
├── models/
├── charts/
├── data/
├── requirements.txt
└── ...
```

## Configuration Notes

### Caching and Web UI

By default, projects use an in-memory KV backend which **does not support persistent caching or web UI**. For local development with these features:

1. Configure a persistent KV backend (Firestore) in `configs/project_config.yaml`
2. For Firestore: Add credentials to `keys/firestore.json`

See the [Backends](../advanced/backends.md) documentation for detailed setup instructions.

