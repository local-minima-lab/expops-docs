# Chart Generation

This guide covers how to write chart code for ExpOps. For configuration details, see the Probe paths and Chart Dependencies sections below.

## Static Charts

Static charts generate PNG image files that are saved to disk.

**Configuration**: Chart entrypoints are configured in `project_config.yaml` under `reporting.static_entrypoint`. See the Probe paths and Chart Dependencies sections below for details.

### Chart Functions

Functions decorated with `@chart()` generate visualizations. **Chart functions have strict requirements**:

#### Required Function Signature

**Every static chart function MUST:**
1. Accept `metrics` as the first parameter (Dict[str, Any])
2. Accept `ctx` as the second parameter (ChartContext), for metrics and optional context
3. Use `plt.savefig()` to save figures (the process runs with cwd set to the output directory; all files written there are synced to artifacts)

```python
from expops.reporting import chart, ChartContext
from typing import Dict, Any
import matplotlib.pyplot as plt

@chart()
def plot_metrics(metrics: Dict[str, Any], ctx: ChartContext) -> None:
    """
    Chart function signature requirements:
    - metrics: Dict containing metrics from probe_paths (REQUIRED)
    - ctx: ChartContext for metrics and run context (REQUIRED)
    - Returns: None (void function)
    """
    # Access metrics directly from the metrics dict
    # Keys are probe_path keys or resolved paths (see Probe paths section)
    train_metrics = metrics.get('train', {})
    eval_metrics = metrics.get('eval', {})
    
    # Extract specific metric values
    train_acc = train_metrics.get('accuracy', {})
    eval_acc = eval_metrics.get('accuracy', {})
    
    # Generate matplotlib plot
    fig, ax = plt.subplots(figsize=(10, 6))
    # ... plotting code ...
    
    # Use plt.savefig() with a relative path; cwd is the process output dir
    plt.savefig('plot_metrics.png', dpi=150)
    plt.close(fig)
```

**Note**: The function name should match the chart `name` defined in `project_config.yaml`, or be registered via the `@chart()` decorator.

### Metrics Access

Metrics are passed directly to chart functions via the `metrics` parameter:

- **Metrics structure**: The `metrics` dict is keyed by your probe_path keys (when an XPath matches one node) or by resolved probe path strings (when an XPath matches multiple nodes). See [Probe paths](#probe-paths) for how keys are determined.
- **Access pattern**: `metrics.get('key', {})` returns metrics for that key (either a config key or a resolved probe path)
- **Metric values**: Each probe path contains metrics logged from that process/step
- **Step-based metrics**: Metrics logged with `step=` parameter are stored as dicts like `{"1": value1, "2": value2, ...}`

**Example**:

Given this config:
```yaml
reporting:
  charts:
    - name: "my_chart"
      probe_paths:
        train: "//*[@name='train_model']"
        eval: "//*[@name='evaluate_model']"
```

The chart function receives:
```python
@chart()
def my_chart(metrics: Dict[str, Any], ctx: ChartContext) -> None:
    import matplotlib.pyplot as plt
    # metrics['train'] contains metrics from train_model process
    train_data = metrics.get('train', {})
    
    # metrics['eval'] contains metrics from evaluate_model process
    eval_data = metrics.get('eval', {})
    
    # Access specific metrics (will be a dict)
    train_acc = train_data.get('accuracy', {})
    eval_acc = eval_data.get('accuracy', {})
```

### Probe paths

Probe paths must be **XPath selectors**: strings that look like XPath (e.g. start with `//` and contain `@` and `[`). They are evaluated against an internal pipeline tree and may resolve to one or many concrete probe paths.

#### XPath format and pipeline structure

The pipeline is represented as a tree: root `pipeline`, then nested `process` elements with `@name`, optional `@partition` (data split, e.g. `p1`, `p2`) and `@seed` (seed value, e.g. `41`, `42`), and optional child `step` elements with `@name`. XPath is evaluated over this tree (standard lxml). Use `//` for descendant-or-self, `/` for path steps, `*` for any element, and `[@name="..."]` for attribute predicates.

Common patterns:

| Goal | XPath pattern |
|------|----------------|
| Process by name | `//*[@name='process_name']` |
| Process + step | `//*[@name='process_name']/*[@name='step_name']` or `//*[@name='step_name']` if the step name is unique among process names |
| Specific partition/seed | `//*[@partition='p1']/*[@seed='41']/*[@name='process_name']` |
| Any partition/seed | `//*[@partition]/*[@seed]/*[@name='process_name']` |

#### How keys map to chart metrics

- **One XPath match**: The config key is preserved (e.g. `train` → `metrics['train']`).
- **Multiple XPath matches**: Each resolved probe path becomes a key. The key is the canonical XPath-style identifier for that process/step (e.g. `"//*[@partition='p1']/*[@seed='41']/*[@name='nn_training_a']/step[@name='train_and_evaluate_nn_classifier']"`). Chart code can iterate over keys or use prefix/grouping logic to aggregate across partitions or seeds.
- **Literal path**: Single key as in config (e.g. `train: "train_model"` → `metrics['train']`).

### Output

Static charts produce image files (PNG) saved under the unified artifacts layout:
```
.<project_id>/artifacts/<version_hash>/<encoded_probe_path>/<chart_name>.png
```
In GCS: `gs://<bucket>/<project_id>/artifacts/<version_hash>/<encoded_probe_path>/<chart_name>.png`

## Dynamic Charts

Dynamic charts provide real-time, interactive visualizations.

**Configuration**: Dynamic charts are defined as **pipeline processes** in `project_config.yaml` (under `experiment.parameters.pipeline.processes`). Each dynamic chart process must have:

- `code` - a unified code reference that points to your JS chart script and function (e.g. `code: "reporting_js.nn_losses"`), where `reporting_js` is defined under `scripts:` at the top of the config
- `chart_type: "dynamic"`
- `probe_paths` - same XPath semantics as static charts (see [Probe paths](#probe-paths) below)

### Example

**Config** (excerpt from premier-league): register the JS script under `scripts`, then add a process with `chart_type: "dynamic"`:

```yaml
# Under experiment.parameters.pipeline.processes:
        - name: "nn_losses"
          code: "reporting_js.nn_losses"
          environment: "premier-league-env-reporting"
          chart_type: "dynamic"
          probe_paths: ...
```

**Client-side** (subscribe to metrics and update a Chart.js chart):

```javascript
// Subscribe to metrics
subscribeToMetrics((metrics) => {
    // Update Chart.js chart
    chart.data.datasets[0].data = metrics;
    chart.update();
});
```