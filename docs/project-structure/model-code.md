# Model Code

Model code in ExpOps projects defines the ML pipeline using decorators and functions.

## File Location

**Location**: `models/<model_name>.py`

## Required Imports

**Always import from `mlops.core`**:

```python
from mlops.core import (
    step, 
    process, 
    log_metric
)
```

## Key Components

### Process Definitions

Functions decorated with `@process()` define pipeline processes. **Process functions have strict requirements**:

#### Required Function Signature

**Every process function MUST:**

1. Accept `data` as the first parameter (required)
2. Accept `hyperparameters` as the second parameter (optional)
3. Return a dictionary (required - non-dict returns will raise an error)
4. Return only serializable data (dictionaries, lists, primitives - not complex objects)

```python
@process()
def define_my_process(data, hyperparameters):
    # Access upstream process data
    upstream_data = data.get('upstream_process_name', {})
    
    # Use hyperparameters if needed
    learning_rate = (hyperparameters or {}).get('learning_rate', 0.001)
    
    # Process logic here
    result = perform_work(upstream_data, learning_rate)
    
    # MUST return a dictionary with serializable values
    return {
        'result': result  # Must be serializable (dict, list, primitive types)
    }
```

### Step Functions

Functions decorated with `@step()` perform specific operations within a process:

```python
@step()
def load_data():
    # Data loading logic
    df = pd.read_csv("data.csv")
    return {'df': df.to_dict(orient='list')}

@step()
def preprocess(raw: SerializableData):
    """
    Steps can accept SerializableData type hint for type checking.
    SerializableData is a type alias for Dict[str, Any].
    """
    df = pd.DataFrame(raw['df'])
    # Preprocessing logic
    processed = clean_data(df)
    return {'processed_df': processed.to_dict(orient='list')}

@step()
def train(prep_data: SerializableData, hyperparameters: Dict[str, Any] | None = None):
    # Training logic
    X = np.array(prep_data['processed_df'])
    model = train_model(X)
    return {'model': model}
```

**Step Notes**:
- Steps are defined **inside** process functions
- Steps can access `hyperparameters` if passed from the process
- Steps execute sequentially within their parent process


### Metrics Logging

Use `log_metric()` for experiment tracking:

```python
from mlops.core import log_metric

# Log scalar metrics (step omitted - auto-increments)
log_metric("accuracy", 0.95)
log_metric("loss", 0.05)
```

#### Step Parameter

The `step` parameter is **optional** and controls how metrics are tracked over time:

**When `step` is omitted (default behavior)**:
- The system automatically increments from the largest existing step for that metric
- If no previous metrics exist for that metric name, it starts at step `1`
- Each subsequent call without `step` increments by 1

**When `step` is explicitly provided**:
- You control the step number yourself
- Useful for training loops, epochs, iterations, or any custom progression
- Steps can be any positive integer

```python
# Training loop with explicit step numbers
for epoch in range(100):
    loss = train_one_epoch()
    # Use epoch number as step
    log_metric("train_loss", loss, step=epoch + 1)
```

### Data Flow Between Processes

Processes receive data from all upstream processes via the `data` dictionary:

```python
@process()
def define_downstream_process(data, hyperparameters):
    """
    'data' contains results from ALL upstream processes, keyed by process name.
    """
    # Access specific upstream process
    training_result = data.get('training_process', {})
    fe_result = data.get('feature_engineering_process', {})
    
    # Use the data
    model = training_result.get('model')
    X_test = fe_result.get('X_test', [])
    
    # Process and return
    predictions = model.predict(X_test)
    return {'predictions': predictions.tolist()}
```

**Important**: The `data` parameter is automatically populated by the framework with results from upstream processes based on the pipeline DAG defined in `project_config.yaml`.

## Example

See template projects for complete examples:
- `sklearn-basic`: Simple sklearn pipeline
- `premier-league`: Complex pipeline with multiple steps