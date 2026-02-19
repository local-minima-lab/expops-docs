# Environment Isolation

ExpOps automatically manages virtual environments for each project.

## Environment Managers

ExpOps supports multiple environment managers:

### venv

Python's built-in virtual environment manager:
- Default for most projects
- Lightweight and fast
- Python 3.8+ required

### conda

Conda environment manager:
- Useful for complex dependencies
- Supports non-Python dependencies

## Automatic Management

Environments are automatically:
- Created when needed
- Updated when dependencies change
- Activated during execution
- Isolated per project

## Configuration

Environment settings are configured in `configs/project_config.yaml`. Each environment has a name and one or more manager entries (`venv` or `conda`) whose value is a list of requirement files:

```yaml
environment:
  my-project-env:
    venv: ["my-project/requirements.txt"]
  my-project-env-reporting:
    venv: ["my-project/charts/requirements.txt"]
  my-project-conda:
    conda: ["my-project/environment.yml"]
```

Inline requirements or conda dependencies in the config are not supported; use files.

## Benefits

Environment isolation provides:
- **Dependency isolation**: No conflicts between projects
- **Reproducibility**: Consistent environments
- **Clean separation**: Training vs. reporting dependencies
- **Easy cleanup**: Remove environments when done


