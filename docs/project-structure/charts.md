# Chart Generation

ExpOps supports both static and dynamic chart generation for visualizing experiment results.

## Static Charts

Static charts generate PNG image files that are saved to disk.

### Configuration

Static chart scripts are configured in `configs/project_config.yaml`:

```yaml
reporting:
  static_entrypoint: "my-project/charts/plot_metrics.py"
  charts:
    - name: "my_chart"
      probe_paths:
        data_source: "//*[@name=\"process_name\"]/step[@name=\"step_name\"]"
```

The `static_entrypoint` path is relative to the workspace root. You can place your chart scripts anywhere and configure the path accordingly.

### Chart Functions

Functions decorated with `@chart()` generate visualizations:

```python
from expops.reporting.registry import chart
from expops.reporting.context import ChartContext

@chart()
def plot_metrics(context: ChartContext):
    # Access metrics from previous steps
    metrics = context.get_metrics()
    
    # Generate matplotlib plot
    import matplotlib.pyplot as plt
    plt.figure()
    # ... plotting code ...
    plt.savefig("output.png")
```

**Note**: The function name should match the chart `name` defined in `project_config.yaml`, or be registered via the `@chart()` decorator.

### Metrics Access

Charts can read metrics from previous pipeline steps via `ChartContext`:
- Access logged metrics
- Read step outputs
- Combine data from multiple steps

### Output

Static charts produce image files (PNG) saved to:
```
artifacts/charts/<run-id>/
```

## Dynamic Charts

Dynamic charts provide real-time, interactive visualizations.

### Configuration

Dynamic chart scripts are configured in `configs/project_config.yaml`:

```yaml
reporting:
  dynamic_entrypoint: "my-project/charts/plot_metrics.js"
  charts:
    - name: "my_dynamic_chart"
      type: dynamic
      probe_paths:
        data_source: "//*[@name=\"process_name\"]/step[@name=\"step_name\"]"
```

The `dynamic_entrypoint` path is relative to the workspace root. If not specified, the system will attempt to derive it from `static_entrypoint` by changing the `.py` extension to `.js`.

### Features

- **Real-time updates**: Charts update as metrics are logged during execution
- **Chart.js integration**: Uses Chart.js library for interactive visualizations
- **Live metrics**: Subscribes to metric streams from multiple pipeline steps
- **Web UI integration**: Rendered in the web UI for interactive exploration

## Probe Path Selectors (XPath)

Charts use **XPath** selectors to target pipeline steps (or processes). The system converts the pipeline DAG into an XML-like tree and evaluates your XPath against it; matches are resolved to concrete probe paths.

**Pipeline tree structure**:
- Root: `<pipeline>`
- Data-parallel process: `<process name="..." partition="p1" />` (one per partition; `partition` is `p1`, `p2`, …)
- Seed-parallel process: `<process name="..." seed="41" />` (one per seed value)
- Leaf process: `<process name="nn_training_a" />` (no `partition`/`seed`)
- Step: `<step name="train_and_evaluate_nn_classifier" />` as child of the process that contains it

**Format**: Only XPath 1.0 expressions are accepted. Paths that do not look like XPath are not valid and are skipped (with a warning).

**Examples**:
```yaml
probe_paths:
  # Target a specific partition and seed
  nn_a_p1_seed41: "//*[@partition=\"p1\"]/*[@seed=\"41\"]/*[@name=\"nn_training_a\"]/step[@name=\"train_and_evaluate_nn_classifier\"]"
  # All partitions, specific seed
  nn_a_all_seed41: "//*[@partition]/*[@seed=\"41\"]/*[@name=\"nn_training_a\"]/*[@name=\"train_and_evaluate_nn_classifier\"]"
  # Process-level (no step)
  ensemble: "//*[@name=\"ensemble_inference\"]"
  # Single process/step (no parallelism)
  feat: "//*[@name=\"feature_engineering_generic\"]/step[@name=\"feature_analysis\"]"
```

Selectors expand to concrete probe paths internally; chart code receives metrics keyed by the **resolved probe path** (e.g. `nn_training_a__p1_seed41/train_and_evaluate_nn_classifier`).

### Example

```javascript
// Subscribe to metrics
subscribeToMetrics((metrics) => {
    // Update Chart.js chart
    chart.data.datasets[0].data = metrics;
    chart.update();
});
```

## Chart Dependencies

Chart dependencies are configured separately from the main project dependencies to reduce training environment overhead.

### Configuration

Chart dependencies are specified in `project_config.yaml`:

```yaml
environment:
  my-project-env-reporting: ["my-project/charts/requirements.txt", "venv"]

reporting:
  environment: "my-project-env-reporting"
```

The requirements file path (first list entry) is relative to the workspace root. This allows you to:
- Keep visualization libraries separate from training dependencies
- Use minimal dependencies for chart generation
- Include libraries like matplotlib, seaborn, plotly, etc.

## Viewing Charts

### Static Charts

Static charts are saved as PNG files and can be:
- Viewed in the file system
- Displayed in the web UI

### Dynamic Charts

Dynamic charts are available in the web UI:
- Real-time updates during execution
- Interactive exploration
- Multiple chart types