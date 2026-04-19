# Component Libraries

Component libraries let you use pre-built pipeline steps without writing custom `@step` code. Instead of implementing training, splitting, or charting logic yourself, you declare a `component:` key in your process config and the platform loads the implementation from an installed plugin package.

## How It Works

Component libraries register themselves under the `expops.component_runners` Python entry-point group. When ExpOps executes a pipeline, it inspects all `component:` references in the config, then loads only the matching plugin packages before running the pipeline.

For example, a process with `component: "sklearn.LogisticRegression.fit_predict"` causes the platform to load the `sklearn` entry point, which registers the LogisticRegression fit+predict runner under that name.

## Installing Component Packages

Component packages are installed into the same environment as the `expops` CLI:

```bash
pip install expops-sklearn      # sklearn estimators, data loading, splitting
pip install expops-matplotlib   # bar charts, histograms
```

The packages must be installed in the environment where you invoke `expops run`, not inside the project's isolated virtual environment.

## Using Components in Config

Add a `component:` key to any process in your `project_config.yaml`:

```yaml
experiment:
  pipeline:
    processes:
      - name: "split_data"
        component: "sklearn.train_test_split"
        parameters:
          test_size: 0.2
```

The full component string format is `<library>.<ComponentClass>.<operation>` (or `<library>.<operation>` for stateless steps like `sklearn.csv_to_xy`).

### `input_transform`

Maps canonical component input names to the output names produced by upstream processes:

```yaml
- name: "train_model"
  component: "sklearn.LogisticRegression.fit_predict"
  input_transform:
    X: "X_train"   # canonical input "X" ← upstream output "X_train"
    y: "y_train"   # canonical input "y" ← upstream output "y_train"
```

Without `input_transform`, the component expects its canonical input names to match upstream output names exactly.

### `output_mapping`

Renames component outputs before they are stored for downstream processes:

```yaml
- name: "predict_model"
  component: "sklearn.LogisticRegression.predict"
  input_transform:
    X: "X_test"
  output_mapping:
    predictions: "y_pred"   # canonical output "predictions" → stored as "y_pred"
```

### `parameters`

Passes keyword arguments to the component (hyperparameters, chart settings, etc.):

```yaml
- name: "train_model"
  component: "sklearn.LogisticRegression.fit_predict"
  parameters:
    max_iter: 200
    C: 0.5
```

### `metrics`

Logs evaluation metrics at the end of a process step. Each metric calls a sklearn-compatible scoring function:

```yaml
metrics:
  - name: "accuracy"
    fn: "accuracy_score"
    inputs:
      y_true: "y_train"
      y_pred: "predictions"
```

Logged metrics are stored in the KV backend and can be read back via [probe paths](../features/reporting.md#probe-paths).

## Components vs. Custom Code

| | Component | Custom `@step` code |
|---|---|---|
| Setup | `pip install expops-sklearn` | Write Python in `src/` |
| Config | `component: "sklearn.LogisticRegression.fit_predict"` | `code: "model.train"` |
| Flexibility | Fixed interface, configurable via `parameters` | Full control |
| Best for | Standard ML steps, quick prototyping | Custom models, novel preprocessing |

Both approaches can coexist in the same pipeline.

## Available Libraries

- [sklearn](sklearn.md) — Data loading, splitting, and sklearn estimators
- [matplotlib](matplotlib.md) — Bar charts and histograms from probe metrics
