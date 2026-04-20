# Creating a Project

You can create a new ExpOps project either from scratch or using a template.

## Create from Template

Templates provide a pre-configured project structure with example code:

```bash
expops create my-project --template sklearn-basic
```

You can also create *in the current directory* (project name inferred from the folder name):

```bash
mkdir my-project && cd my-project
expops create --template sklearn-basic
```

Available templates:
- `sklearn-basic`: Runnable project skeleton with a tiny sklearn model
- `premier-league`: Comprehensive ML project with cluster config and dynamic charts

See the [Templates](../project-templates/available-templates.md) section for more details.

## Create from Scratch

To create a new project without a template:

```bash
expops create my-project
```

This creates a minimal project structure that you can customize.

You can also create *in the current directory* (project name inferred from the folder name):

```bash
mkdir my-project && cd my-project
expops create
```

## Project Structure

After creation, your project will have the following structure:

```
my-project/
├── .gitignore
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
│   └── charts/
├── data/
├── requirements.txt
└── ...
```

The `.my-project/envs/` directory holds the Python virtual environments for
this project (for example `.my-project/envs/my-project-env`). Older versions
of the platform used a workspace-level `.venvs/` folder; that location is now
deprecated for project runs, and new environments are created only under each
project’s hidden `.my-project/envs/` directory.

## Configuration Notes

### Caching and Web App

By default, projects use a local SQLite KV store, which supports persistent caching. For remote or shared setups (e.g. web app across machines):

1. Configure a KV backend (Firestore) in `configs/project_config.yaml`
2. For Firestore: Put credentials in any file (e.g. `firestore.json`) and set its path in the project config (e.g. `credentials_json` under the cache section)

See the [Backends](../advanced/backends.md) documentation for detailed setup instructions.

