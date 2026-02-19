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

See the [Templates](../templates/available-templates.md) section for more details.

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
├── .gitignore
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

By default, projects use a local SQLite KV store, which supports persistent caching. For remote or shared setups (e.g. web UI across machines):

1. Configure a KV backend (Firestore) in `configs/project_config.yaml`
2. For Firestore: Put credentials in any file (e.g. `firestore.json`) and set its path in the project config (e.g. `credentials_json` under the cache section)

See the [Backends](../advanced/backends.md) documentation for detailed setup instructions.

