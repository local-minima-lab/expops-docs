# matplotlib Component Library

The `expops-matplotlib` package provides pre-built chart components that generate plots directly from pipeline metrics — without writing any Python chart code. Charts are defined as pipeline processes and read metrics via [probe paths](../features/reporting.md#probe-paths).

## Installation

Install into the same environment as the `expops` CLI:

```bash
pip install expops-matplotlib
```

**Note**: The matplotlib runner executes in-process with `expops`, not in a separate reporting virtualenv. Both `expops` and `expops-matplotlib` must be installed in the same environment where you invoke `expops run`.

## Available Components

### `matplotlib.BarChart.render`

Generates a bar chart comparing a single metric across multiple probe paths (e.g. comparing train vs. eval accuracy for several models).

**Required config keys**:

- `probe_paths` — XPath selectors mapping display labels to pipeline processes (see [Probe paths](../features/reporting.md#probe-paths))
- `parameters.probe_bar_metric` — the metric name to read from each probe path

**Optional parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | string | Chart title |
| `grid` | bool | Show grid lines (default: `false`) |
| `ylim` | `[min, max]` | Y-axis limits |
| `probe_bar_metric` | string | **Required.** Metric name to plot per probe path |

```yaml
- name: "plot_metrics"
  component: "matplotlib.BarChart.render"
  probe_paths:
    train: "//*[@name='train_model']"
    eval: "//*[@name='predict_model']"
    mlp_train: "//*[@name='train_mlp']"
    mlp_eval: "//*[@name='predict_mlp']"
  parameters:
    title: "Accuracy (LR vs MLP)"
    grid: true
    ylim: [0, 1]
    probe_bar_metric: accuracy
```

---

### `matplotlib.Histogram.render`

Generates a histogram from a metric logged across multiple steps (e.g. per-epoch training loss from `iterative_logging`).

**Required config keys**:

- `probe_paths` — XPath selectors for the process(es) to read metrics from
- `parameters.probe_metric` — the metric name whose values form the histogram

**Optional parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | string | Chart title |
| `grid` | bool | Show grid lines (default: `false`) |
| `bins` | int | Number of histogram bins (default: 10) |
| `probe_metric` | string | **Required.** Metric name to read from probe paths |

```yaml
- name: "mlp_loss_histogram"
  component: "matplotlib.Histogram.render"
  probe_paths:
    mlp_train: "//*[@name='train_mlp']"
  parameters:
    title: "MLP Train Loss Histogram"
    grid: true
    bins: 20
    probe_metric: train_loss
```

This is most useful in combination with [`iterative_logging`](sklearn.md#iterative-logging) on sklearn components that log a metric at each training step.

---

## Probe Paths

Matplotlib components use the same XPath-based probe path system as custom `@chart()` functions. Each key in `probe_paths` becomes a display label; the XPath value selects which pipeline process to read metrics from.

Common patterns:

```yaml
probe_paths:
  train: "//*[@name='train_model']"          # by process name
  eval: "//*[@name='predict_model']"
  p1_train: "//*[@partition='p1']/*[@name='train_model']"  # by partition
```

See [Probe paths](../features/reporting.md#probe-paths) for the full XPath syntax.

---

## Placing Chart Components in the Pipeline

Chart components are declared as regular processes in `experiment.pipeline.processes`. Add them to the end of your `process_adjlist` so they run after the processes they depend on:

```yaml
experiment:
  pipeline:
    process_adjlist: |
      data_source split_data
      split_data train_model
      split_data predict_model
      train_model predict_model
      predict_model plot_metrics    # chart runs after evaluation
    processes:
      - name: "train_model"
        component: "sklearn.LogisticRegression.fit_predict"
        # ...

      - name: "predict_model"
        component: "sklearn.LogisticRegression.predict"
        # ...

      - name: "plot_metrics"
        component: "matplotlib.BarChart.render"
        probe_paths:
          train: "//*[@name='train_model']"
          eval: "//*[@name='predict_model']"
        parameters:
          title: "Train vs Eval Accuracy"
          grid: true
          ylim: [0, 1]
          probe_bar_metric: accuracy
```

---

## matplotlib vs. Custom `@chart()` Functions

| | `expops-matplotlib` component | Custom `@chart()` function |
|---|---|---|
| Setup | `pip install expops-matplotlib` | Write Python in `src/charts/` |
| Config | `component: "matplotlib.BarChart.render"` | `reporting.static_entrypoint` |
| Customisation | Parameters only | Full matplotlib API |
| Best for | Standard bar charts, histograms | Custom layouts, multi-panel figures |

For full control over chart appearance, see [Reporting — Static Charts](../features/reporting.md#static-charts).
