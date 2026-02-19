# Seed Parallelism

Seed parallelism duplicates the downstream process graph and runs each duplicate under a different random seed. It uses deterministic task-level seeding so that each duplicated branch is reproducible.

## Config

### Predefined seed split (no `code_function`)

```yaml
processes:
  - name: "seed_parallel"
    description: "Fan out by seed"
    seed_parallelism:
      seeds: [41, 42, 43]
```

- The split process has no `code_function`; it will merge upstream outputs and pass them through.
- The engine duplicates all downstream nodes until a `seed_aggregation` node is reached.
- Duplicated nodes use a `_seed<value>` suffix (for example, `train_model_seed41`).
- `script_path` is optional for split nodes with no `code_function`.

### User-defined seed split (`code_function`)

```yaml
processes:
  - name: "seed_parallel"
    description: "Custom seed hook"
    code_function: "define_seed_parallel_process"
    seed_parallelism:
      seeds: [41, 42, 43]
```

```python
from expops.core import process

@process()
def define_seed_parallel_process(seeds, **inputs):
    return dict(inputs)
```

- The `seeds` list is passed to the user-defined function as a kwarg.
- `script_path` is required when using `code_function`.

## Aggregation

Add a dedicated aggregation process to collapse the **latest** seed-parallel layer:

```yaml
processes:
  - name: "aggregate_seeds"
    seed_aggregation: true
    code_function: "define_aggregate_seeds"
```

### Input shape at aggregation

When duplicate keys appear across seed branches, the aggregation process receives a dictionary keyed by seed:

```python
@process()
def define_aggregate_seeds(metrics):
    # metrics looks like: {"seed41": <val>, "seed42": <val>, ...}
    ...
```

When a node is **both** `data_aggregation` and `seed_aggregation`, the nested map is keyed by data first, then seed:

```python
@process()
def define_aggregate_data_and_seed(metrics):
    # metrics looks like: {"data1": {"seed41": <val>, ...}, "data2": {...}}
    ...
```

## Notes

- Seed parallelism duplicates downstream nodes and is visible in the UI.
- Task-level RNG seeding uses the most recent (innermost) seed layer when multiple
  seed-parallel layers are nested.
- Each aggregation collapses only the most recent seed-parallel layer; outer layers
  remain as separate process nodes.
