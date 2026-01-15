# Chart Generation

This guide covers how to write chart code for ExpOps. For configuration details, see [Reporting Features](../features/reporting.md).

## Static Charts

Static charts generate PNG image files that are saved to disk.

**Configuration**: Chart entrypoints are configured in `project_config.yaml` under `reporting.static_entrypoint`. See [Reporting Features](../features/reporting.md) for configuration details.

### Chart Functions

Functions decorated with `@chart()` generate visualizations. **Chart functions have strict requirements**:

#### Required Function Signature

**Every static chart function MUST:**

1. Accept `metrics` as the first parameter (Dict[str, Any])
2. Accept `ctx` as the second parameter (ChartContext)
3. Use `ctx.savefig()` to save figures

```python
from mlops.reporting import chart, ChartContext
from typing import Dict, Any
import matplotlib.pyplot as plt

@chart()
def plot_metrics(metrics: Dict[str, Any], ctx: ChartContext) -> None:
    """
    Chart function signature requirements:
    - metrics: Dict containing metrics from probe_paths (REQUIRED)
    - ctx: ChartContext for saving figures (REQUIRED)
    - Returns: None (void function)
    """
    # Access metrics directly from the metrics dict
    # Metrics are keyed by probe_path keys from project_config.yaml
    train_metrics = metrics.get('train', {})
    eval_metrics = metrics.get('eval', {})
    
    # Extract specific metric values
    train_acc = train_metrics.get('accuracy', {})
    eval_acc = eval_metrics.get('accuracy', {})
    
    # Generate matplotlib plot
    fig, ax = plt.subplots(figsize=(10, 6))
    # ... plotting code ...
    
    # MUST use ctx.savefig() to save the figure
    ctx.savefig('plot_metrics.png', fig=fig, dpi=150)
    plt.close(fig)
```

**Note**: The function name should match the chart `name` defined in `project_config.yaml`, or be registered via the `@chart()` decorator.

### Metrics Access

Metrics are passed directly to chart functions via the `metrics` parameter:

- **Metrics structure**: The `metrics` dict is keyed by the probe_path keys from `project_config.yaml`
- **Access pattern**: `metrics.get('probe_key', {})` returns metrics from that probe path
- **Metric values**: Each probe path contains metrics logged from that process/step
- **Step-based metrics**: Metrics logged with `step=` parameter are stored as dicts like `{"1": value1, "2": value2, ...}`

**Example**:

Given this config:
```yaml
reporting:
  charts:
    - name: "my_chart"
      probe_paths:
        train: "train_model"
        eval: "evaluate_model"
```

The chart function receives:
```python
@chart()
def my_chart(metrics: Dict[str, Any], ctx: ChartContext) -> None:
    # metrics['train'] contains metrics from train_model process
    train_data = metrics.get('train', {})
    
    # metrics['eval'] contains metrics from evaluate_model process
    eval_data = metrics.get('eval', {})
    
    # Access specific metrics (may be dicts if logged with step=)
    train_acc = train_data.get('accuracy', {})
    eval_acc = eval_data.get('accuracy', {})
```

### Output

Static charts produce image files (PNG) saved to:
```
artifacts/charts/<run-id>/
```

## Dynamic Charts

Dynamic charts provide real-time, interactive visualizations.

**Configuration**: Chart entrypoints are configured in `project_config.yaml` under `reporting.dynamic_entrypoint`. See [Reporting Features](../features/reporting.md) for configuration details.

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
  venv:
    reporting:
      name: "my-project-env-reporting"
      requirements_file: "projects/my-project/charts/requirements.txt"
```

The `requirements_file` path is relative to the workspace root. This allows you to:

- Keep visualization libraries separate from training dependencies
- Use minimal dependencies for chart generation
- Include libraries like matplotlib, seaborn, plotly, etc.