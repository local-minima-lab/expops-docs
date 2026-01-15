# Environment Isolation

ExpOps automatically manages virtual environments for each project.

## Environment Types

### Training Environment

Used for model training and inference:
- Dependencies from `requirements.txt`
- ML frameworks and data processing libraries
- Model-specific dependencies

### Reporting Environment

Used for chart generation:
- Dependencies from `charts/requirements.txt`
- Visualization libraries (matplotlib, seaborn)
- Minimal dependencies to reduce overhead

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

Environment settings are configured in `configs/project_config.yaml`:

```yaml
environment:
  type: venv  # or conda
```

## Benefits

Environment isolation provides:

- **Dependency isolation**: No conflicts between projects
- **Reproducibility**: Consistent environments
- **Clean separation**: Training vs. reporting dependencies
- **Easy cleanup**: Remove environments when done


