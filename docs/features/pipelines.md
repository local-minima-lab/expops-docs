# DAG Pipeline Execution

ExpOps uses NetworkX to represent and execute ML pipelines as directed acyclic graphs (DAGs) at the process level.

## Pipeline Definition

Pipelines are defined in `configs/project_config.yaml` under `model.parameters.pipeline` using two main components:

### 1. Process Adjacency List (`process_adjlist`)

Defines the DAG structure as a multi-line string (NetworkX adjacency list format):

```yaml
model:
  parameters:
    pipeline:
      process_adjlist: |
        feature_engineering preprocess
        preprocess train_model
        train_model evaluate_model
        evaluate_model plot_metrics
```

Each line defines edges: the first token is the source process, the second token is the target process that depends on it.

**Example**: The above creates a DAG where:

- `feature_engineering` runs first
- `preprocess` depends on `feature_engineering`
- `train_model` depends on `preprocess`
- `evaluate_model` depends on `train_model`
- `plot_metrics` depends on `evaluate_model`

**Parallel execution**: Processes with no dependencies on each other run in parallel:
```yaml
process_adjlist: |
  feature_engineering preprocess_a
  feature_engineering preprocess_b 
```

### 2. Process Definitions (`processes`)

Each process must be explicitly defined with its configuration:

```yaml
processes:
  - name: "feature_engineering"
    description: "Load and prepare data"
    code_function: "define_feature_engineering_process"
  
  - name: "train_model"
    description: "Train the model"
    code_function: "define_training_process"
    hyperparameters:
      learning_rate: 0.001
      epochs: 50
  
  - name: "plot_metrics"
    type: chart
    description: "Generate visualization"
```

**Process attributes**:

- `name`: Unique process identifier (must match names in `process_adjlist`)
- `description`: Human-readable description
- `code_function`: Name of the Python function that defines the process (see below)
- `hyperparameters`: Optional hyperparameters passed to the process
- `type`: Optional type (e.g., `"chart"` for chart generation processes)

## Process Functions

Processes are implemented in Python using the `@process()` decorator. The function name must match the `code_function` in the config:

```python
from mlops.core import process, step, 

@process()
def define_feature_engineering_process(data, hyperparameters):
    """Process that loads and prepares data."""
    
    @step()
    def load_data():
        # Load data from file
        df = pd.read_csv("data.csv")
        return {'df': df.to_dict(orient='list')}
    
    @step()
    def clean_data(raw):
        # Clean the data
        df = pd.DataFrame(raw['df'])
        # ... cleaning logic ...
        return {'cleaned_df': df.to_dict(orient='list')}
    
    # Execute steps in order
    raw = load_data()
    cleaned = clean_data(raw=raw)
    return cleaned
```

**Key points**:

- Process functions receive `data` (results from upstream processes) and `hyperparameters`
- Steps are defined inside the process function using `@step()` decorator
- Steps execute sequentially within a process
- Process returns a dictionary that becomes available to downstream processes

## Data Flow Between Processes

Processes access data from upstream processes via the `data` parameter:

```python
@process()
def define_training_process(data, hyperparameters):
    # Access data from upstream process
    fe_data = data.get('feature_engineering', {})
    df = pd.DataFrame(fe_data.get('cleaned_df', {}))
    
    # Use the data for training
    model = train_model(df)
    return {'model': model}
```

The `data` dictionary contains results from all upstream processes, keyed by process name.

## Execution Order

ExpOps automatically determines execution order based on:

- **DAG structure**: Defined by `process_adjlist`
- **Process dependencies**: Inferred from the adjacency list
- **Available resources**: Distributed across workers when using cluster execution

Processes execute in topological order (respecting dependencies), and independent processes run in parallel.
