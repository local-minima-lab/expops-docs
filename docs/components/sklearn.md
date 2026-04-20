# sklearn Component Library

The `expops-sklearn` package provides pre-built pipeline steps for common scikit-learn workflows: loading CSV data, splitting datasets, and training/evaluating estimators.

## Installation

Install into the same environment as the `expops` CLI:

```bash
pip install expops-sklearn
```

## Available Components

### `sklearn.csv_to_xy`

Loads a CSV file and splits it into feature matrix `X` and label vector `y`.

**Outputs**: `X`, `y`

```yaml
- name: "data_source"
  component: "sklearn.csv_to_xy"
  parameters:
    label_column: "label"   # column name to use as y (required)
```

---

### `sklearn.train_test_split`

Splits `X` and `y` into training and test sets.

**Inputs**: `X`, `y`

**Outputs**: `X_train`, `X_test`, `y_train`, `y_test`

```yaml
- name: "split_data"
  component: "sklearn.train_test_split"
  parameters:
    test_size: 0.2
    random_state: 42
```

---

### `sklearn.<Estimator>.fit_predict`

Trains any sklearn estimator on `X` and `y`, then predicts on the same data.

**Inputs**: `X`, `y` (use `input_transform` to map from upstream output names)

**Outputs**: `model`, `predictions`

Replace `<Estimator>` with any sklearn classifier or regressor class name:

```yaml
- name: "train_model"
  component: "sklearn.LogisticRegression.fit_predict"
  input_transform:
    X: "X_train"
    y: "y_train"
  parameters:
    max_iter: 200
    C: 1.0
```

Other estimator examples:

- `sklearn.RandomForestClassifier.fit_predict`
- `sklearn.MLPClassifier.fit_predict`
- `sklearn.GradientBoostingClassifier.fit_predict`
- `sklearn.SVC.fit_predict`

All constructor keyword arguments can be passed under `parameters`.

---

### `sklearn.<Estimator>.predict`

Runs inference with a previously trained model.

**Inputs**: `model`, `X` (use `input_transform` to map from upstream output names)

**Outputs**: `predictions`

```yaml
- name: "predict_model"
  component: "sklearn.LogisticRegression.predict"
  input_transform:
    X: "X_test"
  output_mapping:
    predictions: "y_pred"
  metrics:
    - name: "accuracy"
      fn: "accuracy_score"
      inputs:
        y_true: "y_test"
        y_pred: "y_pred"
```

---

## Metrics

The `metrics` block logs evaluation scores at the end of a process step. Each entry calls a scoring function from `sklearn.metrics` (or any callable accessible by name):

```yaml
metrics:
  - name: "accuracy"
    fn: "accuracy_score"
    inputs:
      y_true: "y_train"     # key in process outputs
      y_pred: "predictions" # key in process outputs
```

Logged metrics are stored in the KV backend and can be queried via [probe paths](../features/reporting.md#probe-paths) for use in chart components or custom `@chart()` functions.

---

## Iterative Logging

For estimators that train iteratively (e.g. `MLPClassifier`), use `iterative_logging` to record a metric value at each training epoch:

```yaml
- name: "train_mlp"
  component: "sklearn.MLPClassifier.fit_predict"
  input_transform:
    X: "X_train"
    y: "y_train"
  parameters:
    max_iter: 200
  iterative_logging:
    metric_name: "train_loss"   # name to store under in KV
    eval_split: "train"         # which split to evaluate on each iteration
  metrics:
    - name: "accuracy"
      fn: "accuracy_score"
      inputs:
        y_true: "y_train"
        y_pred: "predictions"
```

The logged values are stored step-by-step as `{"1": val1, "2": val2, ...}` and can be plotted as a time series using a [matplotlib Histogram](matplotlib.md) or a [dynamic JS chart](../features/reporting.md#dynamic-charts).

---

## Full Example

The following config trains a LogisticRegression and an MLPClassifier in parallel, then evaluates both on the test set:

```yaml
experiment:
  cache:
    backend:
      type: local
  pipeline:
    process_adjlist: |
      data_source split_data
      split_data train_model
      split_data predict_model
      train_model predict_model
      split_data train_mlp
      split_data predict_mlp
      train_mlp predict_mlp
    processes:
      - name: "data_source"
        component: "sklearn.csv_to_xy"
        parameters:
          label_column: "label"

      - name: "split_data"
        component: "sklearn.train_test_split"
        parameters:
          test_size: 0.2
          random_state: 42

      - name: "train_model"
        component: "sklearn.LogisticRegression.fit_predict"
        input_transform:
          X: "X_train"
          y: "y_train"
        parameters:
          max_iter: 200
        metrics:
          - name: "accuracy"
            fn: "accuracy_score"
            inputs:
              y_true: "y_train"
              y_pred: "predictions"

      - name: "predict_model"
        component: "sklearn.LogisticRegression.predict"
        input_transform:
          X: "X_test"
        output_mapping:
          predictions: "y_pred"
        metrics:
          - name: "accuracy"
            fn: "accuracy_score"
            inputs:
              y_true: "y_test"
              y_pred: "y_pred"

      - name: "train_mlp"
        component: "sklearn.MLPClassifier.fit_predict"
        input_transform:
          X: "X_train"
          y: "y_train"
        parameters:
          max_iter: 200
        iterative_logging:
          metric_name: "train_loss"
          eval_split: "train"
        metrics:
          - name: "accuracy"
            fn: "accuracy_score"
            inputs:
              y_true: "y_train"
              y_pred: "predictions"

      - name: "predict_mlp"
        component: "sklearn.MLPClassifier.predict"
        input_transform:
          X: "X_test"
        metrics:
          - name: "accuracy"
            fn: "accuracy_score"
            inputs:
              y_true: "y_test"
              y_pred: "predictions"
```

See also: [matplotlib components](matplotlib.md) for adding accuracy bar charts and loss histograms to this pipeline.
