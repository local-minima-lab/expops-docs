# Data Parallelism

Data parallelism duplicates the downstream process graph and feeds each duplicate a different data partition.
Use it when a process returns row-splittable data (pandas DataFrame, numpy array, list of rows, etc.) and you want to fan out work.

## Config

### Split from upstream (no `code`)

```yaml
processes:
  - name: "nn_data_parallel"
    description: "Split upstream data into 3 partitions"
    data_parallelism:
      size: 3
      data_name: df
```

- `data_name` tells the system which upstream output key to split.
- The split process has no `code`; it will merge its upstream outputs and split the `data_name` key.
- The script path for such helper split nodes is optional.

### User-defined splitter (`code`)

```yaml
processes:
  - name: "nn_data_parallel"
    description: "Custom splitter"
    code: "define_nn_data_parallel"
    data_parallelism:
      size: [50, 20, 20]
```

```python
from expops.core import process

@process()
def define_nn_data_parallel():
    # Return a list of rows to split
    return [r1, r2, r3, ...]
```

- Return a **list of rows**; the framework will split it based on `size`.
- If `data_name` is omitted and the function returns a list, the data key defaults to `data`.
- `script_path` is required when using an explicit `code` function.

### Size formats

- `size: 3` → split into 3 equal partitions
- `size: [50, 20, 20]` → explicit row counts per partition (must sum to total rows)

## Aggregation

Add a dedicated aggregation process to collapse the **latest** data-parallel layer:

```yaml
processes:
  - name: "aggregate_results"
    data_aggregation: true
    code: "define_aggregate_results"
```

Aggregation processes **must** define a `code` function reference.

### Input shape at aggregation

For any upstream output key that appears across the duplicated branches, the aggregation process receives a dictionary keyed by partition:

```python
@process()
def define_aggregate_results(df):
    # df looks like: {"data1": <partition1>, "data2": <partition2>, ...}
    ...
```

## Notes

- Data parallelism duplicates downstream nodes in the graph and is visible in the UI.
- Chart probe paths can use XPath selector syntax over the pipeline tree (including `@partition='p1'` predicates) to automatically expand to data-parallel partitions; see the chart documentation for details.
- Multiple data-parallel layers are supported. Each aggregation collapses only the
  most recent data-parallel layer; outer layers are still represented by separate
  process nodes.
- Cache invalidation uses data hashes: downstream of a data-parallel split, the
  cache key uses the hash of the specific partition. If another data-parallel
  layer is applied, the partition hash is replaced with the new layer's partition
  hash so only the affected partitions re-run.
