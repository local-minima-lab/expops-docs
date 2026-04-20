# Model Code

Model code in ExpOps projects defines the ML pipeline using decorators and functions.

## File Location

**Location**: `models/<model_name>.py`

## Required Imports

**Always import from `expops.core`**:

```python
from expops.core import (
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

1. Declare explicit parameters for the upstream output keys and hyperparameter keys it needs
2. Return a dictionary (required - non-dict returns will raise an error)
3. Return only serializable data (dictionaries, lists, primitives - not complex objects)
4. Ensure output keys are unique across upstream dependencies (collisions raise errors)

```python
@process()
def define_my_process(cleaned_df, learning_rate: float = 0.001):
    # cleaned_df is injected from an upstream process return key
    result = perform_work(cleaned_df, learning_rate)
    
    # MUST return a dictionary with serializable values
    return {
        'scores': result  # Must be serializable (dict, list, primitive types)
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
def train(prep_data: SerializableData):
    # Training logic
    X = np.array(prep_data['processed_df'])
    model = train_model(X)
    return {'model': model}
```

**Step Notes**:

- Steps are defined **inside** process functions
- Steps can accept explicit parameters passed from the process
- Steps execute sequentially within their parent process


### Metrics Logging

Use `log_metric()` for experiment tracking:

```python
from expops.core import log_metric

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

Processes receive data from all upstream processes by **matching parameter names** to upstream return keys and hyperparameter keys:

```python
@process()
def define_downstream_process(model, X_test):
    # model and X_test are injected from upstream outputs
    predictions = model.predict(X_test)
    return {'predictions': predictions.tolist()}
```

**Important**: Inputs are injected by name. If two upstream processes return the same key (or a key collides with a hyperparameter), the runtime raises an error.

## Example

See template projects for complete examples:

- `sklearn-basic`: Simple sklearn pipeline
- `premier-league`: Complex pipeline with multiple steps