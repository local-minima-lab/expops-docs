# DAG Pipeline Execution

ExpOps uses NetworkX to represent and execute ML pipelines as directed acyclic graphs (DAGs) at the process level.

## Pipeline Definition

Pipelines are defined in `configs/project_config.yaml` under `experiment.parameters.pipeline` using two main components:

### 1. Process Adjacency List (`process_adjlist`)

Defines the DAG structure as a multi-line string (NetworkX adjacency list format):

```yaml
experiment:
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
    environment: "my-project-env"
  
  - name: "train_model"
    description: "Train the model"
    code_function: "define_training_process"
    environment: "my-project-env"
    parameters:
      learning_rate: 0.001
      epochs: 50
  
  - name: "plot_metrics"
    type: chart
    description: "Generate visualization"
    environment: "my-project-env-reporting"
```

**Process attributes**:
- `name`: Unique process identifier (must match names in `process_adjlist`)
- `description`: Human-readable description
- `script` (optional): Key from the top-level `scripts` section to use for this process. Defaults to the first key in `scripts` if omitted. See [Configuration](../project-structure/configuration.md) for the `scripts` section and defaults.
- `code_function`: Name of the Python function that defines the process (see below). Defaults to the process name if omitted, so you can omit it when the function name matches the process name.
- `environment`: Environment name to use (defaults to the first environment if omitted)
- `parameters`: Optional parameters injected by name into the process function
- `type`: Optional type (e.g., `"chart"` for chart generation processes)
- `data_parallelism`: Optional split configuration (see [data-parallelism](../features/data-parallelism.md))
- `data_aggregation`: Optional flag to merge the latest data-parallel layer (see [data-parallelism](../features/data-parallelism.md))

## Process Functions

Processes are implemented in Python using the `@process()` decorator. The function name must match the `code_function` in the config:

```python
from expops.core import process, step, 

@process()
def define_feature_engineering_process(test_size: float = 0.2):
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
- Process functions declare explicit parameters for upstream output keys and process parameter keys
- Steps are defined inside the process function using `@step()` decorator
- Steps execute sequentially within a process
- Process returns a dictionary that becomes available to downstream processes

## Data Flow Between Processes

Processes access data from upstream processes via matching parameter names:

```python
@process()
def define_training_process(cleaned_df):
    df = pd.DataFrame(cleaned_df)
    model = train_model(df)
    return {'model': model}
```

Inputs are injected by name. If multiple upstream processes return the same key (or a key collides with a parameter), the runtime raises an error.

For `data_aggregation` processes, duplicate upstream keys are provided as a dictionary keyed by partition (`data1`, `data2`, ...).

## Execution Order

ExpOps automatically determines execution order based on:
- **DAG structure**: Defined by `process_adjlist`
- **Process dependencies**: Inferred from the adjacency list
- **Available resources**: Distributed across workers when using cluster execution

Processes execute in topological order (respecting dependencies), and independent processes run in parallel.
